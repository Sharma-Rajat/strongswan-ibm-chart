# strongswan Release Notes

## Breaking Changes

- None

## What's new since v2.4.0

- Updated GO logic to GO 1.12.1
- Updated docker image to use Alpine 3.9

## Fixes

- Fixed nodeSelector logic bug that was introduced in 2.3.5

## Prerequisites

- IBM Kubernetes Service (IKS) cluster with Kubernetes 1.7 or higher  -OR-
- IBM Cloud Private 2.1.0.3 or higher
- Kubernetes Helm 2.8 or higher

## Documentation

See [`Setting up VPN connectivity`](https://console.bluemix.net/docs/containers/cs_vpn.html#vpn).

## Version History

| Version | Date       | Description                                             | Image Tag |
|---------|------------|---------------------------------------------------------|-----------|
| 1.0.0   | 12/06/2017 | Initial prod version of the helm chart                  | 17-12-05  |
| 1.0.8   |  1/10/2018 | Helm test programs, vpnDebug tool, misc fixes           | 18-01-10  |
| 1.1.0   |  2/05/2018 | Allow on-prem to ping cross subnet worker node          | 18-02-05  |
| 1.1.1   |  2/07/2018 | Allow SNAT for pod traffic to on-prem network           | 18-02-06  |
| 1.1.2   |  2/08/2018 | Allow config of tolerations for VPN route daemon        | 18-02-06  |
| 1.1.3   |  2/08/2018 | Don't delete rule if routes still in table              | 18-02-08  |
| 1.1.4   |  2/08/2018 | Add anti-affinity to the VPN pod                        | 18-02-08  |
| 1.1.5   |  2/12/2018 | Add support for local subnet NAT                        | 18-02-11  |
| 1.1.6   |  2/14/2018 | Expose more ipsec.conf properties in helm chart         | 18-02-11  |
| 1.1.7   |  2/17/2018 | Change helm ping tests to fail if remote IP not config  | 18-02-11  |
| 2.0.0   |  2/18/2018 | Prod version - 2.0.0                                    | 18-02-11  |
| 2.0.1   |  2/21/2018 | Do not remove routes if VPN is still active             | 18-02-21  |
| 2.0.2   |  2/25/2018 | Non-buffered logging and ability to config logging      | 18-02-22  |
| 2.0.3   |  2/25/2018 | Always use non-buffered logging                         | 18-02-22  |
| 2.0.4   |  3/02/2018 | Add heritage=tiller label, fix helm test                | 18-02-22  |
| 2.0.5   |  3/06/2018 | Change default for ipsec.closeaction                    | 18-02-22  |
| 2.0.6   |  3/12/2018 | Add arch node affinity                                  | 18-02-22  |
| 2.0.7   |  3/14/2018 | Update chart description, add appVersion                | 18-02-22  |
| 2.0.8   |  3/21/2018 | Add rbac support                                        | 18-03-20  |
| 2.0.9   |  3/28/2018 | Allow outbound VPN connection over LB VIP               | 18-03-28  |
| 2.0.10  |  4/13/2018 | Add enableServiceSourceIP config, IKEv1 validation      | 18-04-13  |
| 2.0.11  |  4/20/2018 | Add support for Calico V3                               | 18-04-20  |
| 2.0.12  |  4/23/2018 | Add support for running in a different namespace        | 18-04-23  |
| 2.0.13  |  4/25/2018 | New privilegedVpnPod config option                      | 18-04-25  |
| 2.1.0   |  4/26/2018 | Prod version - 2.1.0                                    | 18-04-25  |
| 2.1.1   |  5/01/2018 | New enableSingleSourceIP config option                  | 18-05-01  |
| 2.1.2   |  5/22/2018 | Remove env variable passed to VPN pod                   | 18-05-22  |
| 2.1.3   |  5/29/2018 | New remoteSubnetNAT config option                       | 18-05-29  |
| 2.1.4   |  5/31/2018 | New localNonClusterSubnet config option                 | 18-05-31  |
| 2.1.5   |  6/02/2018 | Fix enablePodSNAT:auto when pod subnet not 172.30.0.0   | 18-06-02  |
| 2.1.6   |  6/04/2018 | Fix IPPool create/delete during Kube 1.10 upgrade       | 18-06-04  |
| 2.1.7   |  6/06/2018 | Simplify IPPool delete logic in VPN pod cleanup         | 18-06-06  |
| 2.1.8   |  6/08/2018 | New enableRBAC config option                            | 18-06-06  |
| 2.1.9   |  6/18/2018 | Refactor strongswan GO logic                            | 18-06-18  |
| 2.2.0   |  6/29/2018 | Prod version - 2.2.0                                    | 18-06-18  |
| 2.2.1   |  7/02/2018 | Allow multi-line and spaces in list values.yaml fields  | 18-06-18  |
| 2.2.2   |  7/11/2018 | Add alpine license files to chart                       | 18-06-18  |
| 2.2.3   |  7/26/2018 | Fix validation bug: privateIPtoPing in rightsubnet      | 18-07-26  |
| 2.2.4   |  8/10/2018 | Add flags to enable just the tests you need             | 18-07-26  |
| 2.2.5   |  9/11/2018 | Fix error during helm install if validate="off"         | 18-07-26  |
| 2.2.6   |  9/26/2018 | Prevent LB IP pending when specific LB IP requested     | 18-09-26  |
| 2.2.7   | 10/29/2018 | Adding Monitoring to VPN Tunnel                         | 18-10-29  |
| 2.2.8   | 10/30/2018 | Improve create calico IPPool failure message            | 18-10-30  |
| 2.2.9   | 10/31/2018 | Add resource requests, priorityClassName, apps/v1       | 18-10-30  |
| 2.3.0   | 11/05/2018 | Prod version - 2.3.0                                    | 18-10-30  |
| 2.3.1   | 11/20/2018 | Add support for MZR clusters                            | 18-11-20  |
| 2.3.2   | 12/04/2018 | Add readinessProbe, livenessProbe, and other misc fixes | 18-12-04  |
| 2.3.3   |  1/04/2019 | Update to alpine 3.8 and GO 1.11.4                      | 19-01-04  |
| 2.3.4   |  1/17/2019 | New local.id options: %nodePublicIP and %loadBalancerIP | 19-01-17  |
| 2.3.5   |  1/29/2019 | New zoneLoadBalancer config option                      | 19-01-24  |
| 2.4.0   |  1/31/2019 | Prod version - 2.4.0                                    | 19-01-24  |
| 2.4.1   |  2/05/2019 | Fix nodeSelector logic, require helm 2.8+               | 19-01-24  |
| 2.4.2   |  2/19/2019 | Update to GO 1.11.5                                     | 19-02-19  |
| 2.4.3   |  3/18/2019 | Update to alpine 3.9 and GO 1.12.1                      | 19-03-18  |
| 2.5.0   |  3/26/2019 | Prod version - 2.5.0                                    | 19-03-18  |
