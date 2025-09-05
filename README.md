# Simple cloud formation template for UPI on AWS private subnets 
This cloudformation templates assume you have three private networks on your VPC and you do have a bastion that is within that VPC reachable by all other. 
The bastion should not be in the data path after the installation and it will be used for install and maybe maintenance.

### Pre-requisites
 - VPC created and all networking allowing access to Internet to the documented URL:
    - https://docs.redhat.com/en/documentation/openshift_container_platform/4.18/html/installation_configuration/configuring-firewall
    - This can be via proxy, more information bellow
    - The example uses a public subnet for routing to the internet but for Enterprise is understandable this will be done with Transit Gateways and Direct connections. This just has to be working.
 - Route53 Private hosted zone created
 - All access to run the ocpawscloudformation
 - Copy your pull-secret from RedHat from your account to be used in the yaml file.

### About the Openshift template
I am using t3.2xlarge for the bootstrap instance because you need ec2-console to troubleshoot any network issues that can arise on the first install. this instance can be deleted and changed on the cloud formation stack after the successfull install. I am using t2.2xlarge as it is an economical instance to test that the infrastructure and access are adequate for an installation. No additional workload is accounted for in this one. Aditional workers can be added later on and there is no need for bootstrap again.

#### This is a yaml file to use without proxy and using the 100.64.0.0/10 for non-routable internal IPs. In practice, we need to start on 100.68.0.0/14 because 100.64.0.0/16 is used internally by OVN.
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
If you need add proxies look fo the official RH documentation. The no proxy is very important, you should have your internal network that the cluster will connect to, including but not limited to all VPC IPs.

#### If you are using the example infra (the Amazon image I am using is RHEL 8 based) Download:
[https://mirror.openshift.com/pub/openshift-v4/amd64/clients/ocp/stable-4.18/openshift-client-linux-amd64-rhel8.tar.gz](https://mirror.openshift.com/pub/openshift-v4/amd64/clients/ocp/stable-4.18/openshift-client-linux-amd64-rhel8.tar.gz)
and
[https://mirror.openshift.com/pub/openshift-v4/amd64/clients/ocp/stable-4.18/openshift-install-linux.tar.gz](https://mirror.openshift.com/pub/openshift-v4/amd64/clients/ocp/stable-4.18/openshift-install-linux.tar.gz)
</br>
Change for the rhel9 if it is your case
