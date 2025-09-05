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
</br>
and
</br>
[https://mirror.openshift.com/pub/openshift-v4/amd64/clients/ocp/stable-4.18/openshift-install-linux.tar.gz](https://mirror.openshift.com/pub/openshift-v4/amd64/clients/ocp/stable-4.18/openshift-install-linux.tar.gz)
</br>
Change for the rhel9 if it is your case
</br>
#### After downloading untar your assets and configure the install-config.yaml accordingly:
```
[root@ip-10-0-0-239 ~]# ls
install-config.yaml  openshift-client-linux-amd64-rhel8-4.18.22.tar.gz  README.md
kubectl              openshift-install
oc                   openshift-install-linux-4.18.22.tar.gz
```
#### Copy the clients to you executable preference (/usr/bin or /usr/local/bin)
```
[root@ip-10-0-0-239 ~]# cp oc kubectl /usr/bin/
[root@ip-10-0-0-239 ~]#
```
#### Create the manifest files to serve on the jump/bastion server:
```
[root@ip-10-0-0-239 ~]# mkdir ocp
[root@ip-10-0-0-239 ~]# cp install-config.yaml ocp/
[root@ip-10-0-0-239 ~]# cd ocp/
[root@ip-10-0-0-239 ocp]# ../openshift-install create manifests
INFO Consuming Install Config from target directory
WARNING Making control-plane schedulable by setting MastersSchedulable to true for Scheduler cluster settings
INFO Manifests created in: manifests and openshift
[root@ip-10-0-0-239 ocp]#
```
#### Chaange the masters to be schedulable:
```
[root@ip-10-0-0-239 ocp]# sed -i 's/mastersSchedulable: true/mastersSchedulable: false/' manifests/cluster-scheduler-02-config.yml
[root@ip-10-0-0-239 ocp]#
```
#### Create the ignition files and serve them via http (you can delete this and uninstall httpd after the cluster instalation.
```
[root@ip-10-0-0-239 ocp]# ../openshift-install create ignition-configs
INFO Consuming Master Machines from target directory
INFO Consuming Openshift Manifests from target directory
INFO Consuming Common Manifests from target directory
INFO Consuming OpenShift Install (Manifests) from target directory
INFO Consuming Worker Machines from target directory
INFO Ignition-Configs created in: . and auth
[root@ip-10-0-0-239 ocp]# chmod 644 *.ign
[root@ip-10-0-0-239 ocp]# mkdir /var/www/html/ignition
[root@ip-10-0-0-239 ocp]# cp *.ign /var/www/html/ignition/
[root@ip-10-0-0-239 ocp]#
```
## It is important to make sure you can reach the http server, if you need to test, get a t3.micro instance on the VPC the cluster will be installed.

# Deploy the automation and correctly fullfil the entries.
