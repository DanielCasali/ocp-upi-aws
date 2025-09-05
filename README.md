# Simple cloud formation templates for UPI on AWS private subnets 
This cloudformation templates assume you have three private networks on your VPC and you do have a bastion that is within that VPC reachable by all other. 
The bastion should not be in the data path after the installation and it will be used for install and maybe maintenance.

### Pre-requisites
 - VPC created and all networking allowing access to Internet to the documented URL:
    - https://docs.redhat.com/en/documentation/openshift_container_platform/4.18/html/installation_configuration/configuring-firewall
    - This can be via proxy, more information bellow
    - The example uses a public subnet for routing to the internet but for Enterprise is understandable this will be done with Transit Gateways and Direct connections. This just has to be working.
 - Route53 Private hosted zone created
 - All access to run the ocpawscloudformation
 - pull-secret from RedHat from your account

#### This is a yaml file to use without proxy and using the 100.64.0.0/10 for non-routable internal IPs we need to start on 100.68.0.0/14 because 100.64.0.0/16 is used internally by OVN.
```
apiVersion: v1
baseDomain: test.local
compute:
- hyperthreading: Enabled
  name: worker
  replicas: 0
controlPlane:
  hyperthreading: Enabled
  name: master
  replicas: 3
metadata:
  name: ocp
networking:
  clusterNetwork:
  - cidr: 100.68.0.0/14
    hostPrefix: 23
  networkType: OVNKubernetes
  serviceNetwork:
  - 100.72.0.0/16
platform:
  none: {}
pullSecret: ''
sshKey: ''
```
If you need add proxies look fo the official RH documentation.

