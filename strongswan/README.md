# strongSwan IPSec VPN service

![strongSwan IPSec VPN service](https://www.strongswan.org/images/strongswan.png)

## Introduction

You can set up the strongSwan IPSec VPN service to securely connect your Kubernetes cluster with an on-premises network. The strongSwan IPSec VPN service provides a secure end-to-end communication channel over the internet that is based on the industry-standard Internet Protocol Security (IPsec) protocol suite. To set up a secure connection between your cluster and an on-premises network, you must have an IPsec VPN gateway installed in your on-premises data center.

The strongSwan IPSec VPN service is integrated within your Kubernetes cluster.  There is no need to have an external gateway device.  The IPSec VPN endpoint is enabled as a Kubernetes pod/container running in your cluster.  Routes are automatically configured on all of the worker nodes of the cluster when VPN connectivity is established.  These routes can allow two way connectivity from any pod on any worker node through the VPN tunnel to (or from) the remote system.

The strongSwan IPSec VPN service consists of a Kubernetes Load Balancer service, a VPN pod deployment, a VPN routes daemon set, and multiple config maps. All of these resources and bundled together and delivered as a single Kubernetes Helm chart.

You can setup the strongSwan IPSec VPN Service using a Helm chart that allows you to configure and deploy the strongSwan IPSec VPN service inside of a Kubernetes pod.

Because strongSwan is integrated within your cluster, you don't need an external gateway device. When VPN connectivity is established, routes are automatically configured on all of the worker nodes in the cluster. These routes allow two-way connectivity through the VPN tunnel between pods on any worker node and the remote system. For example, the following diagram shows how an app in IBM Cloud Kubernetes Service can communicate with an on-premises server via a strongSwan VPN connection:

![Using Helm to setup strongSwan IPSec VPN service](https://raw.githubusercontent.com/IBM-Bluemix-Docs/containers/master/images/cs_vpn_strongswan1.png)

For additional information on how you can use the strongSwan IPsec VPN service to securely connect your worker nodes to your on-premises data center, see [Setting up VPN connectivity](https://cloud.ibm.com/docs/containers?topic=containers-vpn#vpn) in the [IBM Cloud Container Service](https://www.ibm.com/cloud/container-service) documentation.

## Prerequisites

To set up a secure connection between your cluster and an on-premises network, you must have an IPsec VPN gateway installed in your on-premises data center. There are many ways to get an on-premises IPsec VPN gateway - it can be a hardware device, a Virtual Machine running strongSwan, a Kubernetes Cluster running the strongSwan helm chart.

For additional information on how you can use the strongSwan IPsec VPN service to securely connect your worker nodes to your on-premises data center, see [Setting up VPN connectivity](https://cloud.ibm.com/docs/containers?topic=containers-vpn#vpn) in the [IBM Cloud Container Service](https://www.ibm.com/cloud/container-service) documentation.

## PodSecurityPolicy Requirements

**__NOTE: This applies to IBM Cloud Private only.__**

This chart requires a PodSecurityPolicy to be bound to the target namespace prior to installation.  The following predefined PodSecurityPolicy is used:

- Predefined PodSecurityPolicy name: [`ibm-privileged-psp`](https://ibm.biz/cpkspec-psp)

## Resources Required

- This chart will work in any size cluster with any size worker present

## Installing the Chart

- Extract the default configuration settings to a local YAML file.

    ```bash
    helm inspect values ibm/strongswan > config.yaml
    ```

- Open the `config.yaml` file and update the default values according to the VPN configuration you want.  If a property has specific values that you can choose from, those values are listed in comments above each property in the file.  See the `Configuration` section for a complete list of all of the configuration values that can be modified

    **Important:** If you do not need to change a property, comment that property out by placing a `#` in front of it.

- Install the Helm Chart to your cluster with the updated config.yaml file

    ```bash
    helm install -f config.yaml --name=vpn ibm/strongswan
    ```

- Check the chart deployment status

    ```bash
    helm status vpn
    ```

## Chart Details

Specify each parameter that you want to specify in a yaml file and use the  `-f` option with the yaml file name to `helm install`. For example,

```bash
helm install -f config.yaml --name=vpn ibm/strongswan
```

The file config.yaml above should specify the various parameters you want to set for your VPN configuration. The cluster where the helm install is done using the above command will have strongSwan setup with the specified configuration.

## Verifying the Chart

- If the VPN on the on-premises gateway is not active, start it

- Check the status of the VPN. A status of `ESTABLISHED` means that the VPN connection was successful.

    ```bash
    export STRONGSWAN_POD=$(kubectl get pod -l app=strongswan,release=vpn -o jsonpath='{ .items[0].metadata.name }')
    kubectl exec $STRONGSWAN_POD -- ipsec status
    ```

    Example output:

    ```bash
    Security Associations (1 up, 0 connecting):
        k8s-conn[1]: ESTABLISHED 17 minutes ago, 172.30.244.42[ibm-cloud]...192.168.253.253[on-prem]
        k8s-conn{2}: INSTALLED, TUNNEL, reqid 12, ESP in UDP SPIs: c78cb6b1_i c5d0d1c3_o
        k8s-conn{2}: 172.21.0.0/16 172.30.0.0/16 === 10.91.152.128/26
    ```

## Validation Tests

Five helm test programs are included with the strongSwan chart.  These test programs can be used to verify the VPN connectivity once the chart has been installed.  Some of these tests  require the `remote.privateIPtoPing` configuration setting to be specified.  Other tests require specific `local.subnet` to be exposed in the VPN configuration or other configuration settings. If those subnets are not exposed, the validation test program will fail.  Depending on the network connectivity requirements, it might be perfectly acceptable for some of these tests to fail.

Use  `helm test` to run the validation test programs:

```bash
$ helm test vpn
RUNNING: vpn-strongswan-check-config
PASSED: vpn-strongswan-check-config
RUNNING: vpn-strongswan-check-state
PASSED: vpn-strongswan-check-state
RUNNING: vpn-strongswan-ping-remote-gw
PASSED: vpn-strongswan-ping-remote-gw
RUNNING: vpn-strongswan-ping-remote-ip-1
PASSED: vpn-strongswan-ping-remote-ip-1
RUNNING: vpn-strongswan-ping-remote-ip-2
PASSED: vpn-strongswan-ping-remote-ip-2
```

NOTES:

- Helm will run these five test programs in a random order
- Helm does not provide any way to run a subnet or a single test program.  All of the validation test programs are run each time.

The five strongSwan VPN validation test programs are:

1. `vpn-strongswan-check-config` : Performs syntax validation of the ipsec.conf file that is generated from the `config.yaml` file
1. `vpn-strongswan-check-state` : Check to see if the VPN connection is `ESTABLISHED`
1. `vpn-strongswan-ping-remote-gw` : Attempt to ping the `remote.gateway` IP address that was configured in the `config.yaml` file.  It is possible that the VPN could be in `ESTABLISHED` state, but this test case could still fail if ICMP packets are being blocked by a firewall
1. `vpn-strongswan-ping-remote-ip-1` : Attempt to ping the `remote.privateIPtoPing` IP address from a Kubernetes pod.  In order for this test case to pass, the Kubernetes pod subnet, 172.30.0.0/16 needs to be listed in the `local.subnet` property OR `enablePodSNAT` needs to be enabled.
1. `vpn-strongswan-ping-remote-ip-2` : Attempt to ping the `remote.privateIPtoPing` IP address from a worker node.  In order for this test case to pass, the cluster worker node's private subnet needs to be listed in the `local.subnet` property

If any of these test programs fail, the output of the validation test can be viewed by looking at the logs of the test pod:

```bash
kubectl logs  <test program>
```

Prior to running these validation tests again, the current Kubernetes test pods need to be cleaned up:

```bash
kubectl get pods -a -l app=strongswan-test
kubectl delete pods -l app=strongswan-test
```

## Uninstalling the Chart

To uninstall / delete:

```bash
helm delete --purge vpn
```

## Configuration

The helm chart has the following configuration options:

| Value                        | Description                                       | Default                        |
|------------------------------|---------------------------------------------------|--------------------------------|
| `validate`                   | Type of validation to be done on ipsec.conf       | strict                         |
| `overRideIpsecConf`          | Provide alternative ipsec.conf to use             |                                |
| `overRideIpsecSecrets`       | Provide alternative ipsec.secrets to use          |                                |
| `enablePodSNAT`              | Enable SNAT for pod outbound traffic              | auto                           |
| `enableRBAC`                 | Enable creation of RBAC resources                 | true                           |
| `enableServiceSourceIP`      | Enable externalTrafficPolicy=local on service     | false                          |
| `enableSingleSourceIP`       | Force all Kubernetes traffic to be hidden         | false                          |
| `localNonClusterSubnet`      | Local non cluster subnets to expose over the VPN  |                                |
| `localSubnetNAT`             | Allow NAT of the private local IP subnets         |                                |
| `remoteSubnetNAT`            | Allow NAT of the private remote IP subnets        |                                |
| `loadBalancerIP`             | Public IP address to use for Load Balancer        |                                |
| `zoneLoadBalancer`           | List of Load balancer IP addrs for each zone      |                                |
| `connectUsingLoadBalancerIP` | Connect outbound VPN using LB VIP as source       | auto                           |
| `nodeSelector`               | Kubernetes node selector to apply to VPN pod      |                                |
| `zoneSelector`               | Availability zone for VPN pod and LB service      |                                |
| `zoneSpecificRoutes`         | Limit route config to workers in specified zone   | false                          |
| `privilegedVpnPod`           | Run the VPN pod with privileged authority         | false                          |
| `helmTestsToRun`             | List of tests to run during `helm test`           | ALL                            |
| `tolerations`                | Kubernetes tolerations to apply to daemon set     |                                |
| `strongswanLogging`          | What components log and how much                  | (see file)                     |
| `ipsec.keyexchage`           | Protocol to use with the VPN connection           | ikev2                          |
| `ipsec.esp`                  | ESP encryption/authentication algorithms to use   |                                |
| `ipsec.ike`                  | IKE encryption/authentication algorithms to use   |                                |
| `ipsec.auto`                 | Initiate the VPN connection or listen             | add                            |
| `ipsec.closeaction`          | Action to take if remote peer unexpectedly close  | restart                        |
| `ipsec.dpdaction`            | Controls the use of the DPD protocol              | restart                        |
| `ipsec.ikelifetime`          | How long keying channel should last               | 3h                             |
| `ipsec.keyingtries`          | How many attempts to negotiate a connection       | %forever                       |
| `ipsec.lifetime`             | How long should instance of connection last       | 1h                             |
| `ipsec.margintime`           | How long before expiry should re-negotiate begin  | 9m                             |
| `ipsec.additionalOptions`    | Any additional ipsec.conf options desired         |                                |
| `local.subnet`               | Local subnet(s) exposed over the VPN              | 172.30.0.0/16,172.21.0.0/16    |
| `local.id`                   | String identifier for the local side              | ibm-cloud                      |
| `remote.gateway`             | IP address of the on-prem VPN gateway             | %any                           |
| `remote.subnet`              | Remote subnets to access from the cluster         | 192.168.0.0/24                 |
| `remote.id`                  | String identifier for the remote side             | on-prem                        |
| `remote.privateIPtoPing`     | IP address in the remote subnet to use for tests  |                                |
| `preshared.secret`           | Pre-shared secret.  Stored in ipsec.secrets       | "strongswan-preshared-secret"  |
| `monitoring.enable`          | Enable monitoring for the VPN connection          | false                          |
| `monitoring.clusterName`     | Name of Kubernetes cluster                        |                                |
| `monitoring.privateIPs`      | IP(s) for monitoring to ping                      |                                |
| `monitoring.httpEndpoints`   | HTTP endpoint(s) for monitoring to test           |                                |
| `monitoring.timeout`         | Time that a monitoring test must complete in      | 5 (seconds)                    |
| `monitoring.delay`           | Time delay between monitoring test cycles         | 120 (seconds)                  |
| `monitoring.slackWebhook`    | Slack Webhook URL from Incoming Webhooks config   |                                |
| `monitoring.slackChannel`    | Slack channel to post monitoring results          |                                |
| `monitoring.slackUsername`   | Slack username to associate with monitor messages | "IBM strongSwan VPN"           |
| `monitoring.slackIcon`       | Slack icon to associate with monitor messages     | ":swan:"                       |

## Slack Configuration

The strongSwan VPN can be configured to automatically post VPN connectivity messages to a Slack channel.
This is accomplished using the Slack Custom Integrations **Incoming WebHooks** app.

Before strongSwan VPN can be configured to send Slack notifications, the **Incoming WebHooks** app needs to be installed into your Slack workspace and configured:

- install this app into your workspace by going to <https://slack.com/apps/A0F7XDUAZ-incoming-webhooks> and clicking **Request to Install**.  If this app is not listed in your Slack setup, contact your slack administrator.
- once your request to install has been approved, click **Add Configuration** to create a new configuration
- on the **Post to Channel** drop down box, select an existing Slack channel -OR- create a new Slack channel
- once the new configuration has been created, a unique **Webhook URL** will be generated.  The Webhook URL should look something like: <https://hooks.slack.com/services/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>

This unique **Webhook URL** needs to be specified in your strongSwan VPN helm configuration file.
The strongSwan logic will automatically post VPN connectivity message to this **Webhook URL** which will add the messages to the Slack channel that you configured.

In order to verify that strongSwan VPN slack notifications will work, the following commands will send a test message to your **Webhook URL**:

```bash
export WEBHOOK_URL="<PUT YOUR WEBHOOK URL HERE>"
curl -X POST -H 'Content-type: application/json' -d '{"text":"VPN test message"}' $WEBHOOK_URL
```

It is important to verify that the test message was successful.

## Troubleshooting

To help resolve some common VPN configuration issues, a `vpnDebug` tool has been created and is delivered with the strongSwan image.  To run this debug tool:

```bash
export STRONGSWAN_POD=$(kubectl get pod -l app=strongswan,release=vpn -o jsonpath='{ .items[0].metadata.name }')`
kubectl exec $STRONGSWAN_POD -- vpnDebug
```

The tool will dump out several pages of information as it runs various tests trying to determine common networking issues.  Output lines that begin with: `ERROR`, `WARNING`, `VERIFY`, or `CHECK` indicate possible errors with the VPN connectivity.

## Limitations

There are a few scenarios in which strongSwan helm chart may not be the best choice:

- The strongSwan helm chart does not support "route based" IPSec VPNs. If a "route based" IPSec VPN is required, a different VPN solution will need to be used.
- The strongSwan helm chart only supports IPSec VPNs using "preshared keys". If certificates are required, a a different VPN solution will need to be used.
- The strongSwan helm chart is not a general purpose VPN gateway solution. It is not designed to allow multiple clusters and other IaaS resources to share a single VPN connection. If multiple clusters need to share a single VPN connection, a different VPN solution should be considered.
- The strongSwan helm chart runs as a Kubernetes pod inside of the cluster. As a result, the performance of the VPN will be affected by the memory and network usage of Kubernetes and other pods that are running in the cluster. In performance critical environments, a VPN solution running outside of the cluster on dedicated hardware should be considered.
- The strongSwan helm chart does not provide any metrics or monitoring of the network traffic flowing over the VPN connection. Other Kubernetes tools should be used for this purpose.
- The strongSwan helm chart runs a single VPN pod as the IPSec tunnel endpoint. Kubernetes will restart the pod if it fails, but there will be a slight down time while the new pod starts up and the VPN connection is re-established. If faster error recovery or a more elaborate HA solution is required, a different VPN solution should be considered.
