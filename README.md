**Ansible Playbook to install k3s on to a Raspberry Pi4 cluster**

This is a work in progress project to create and upgrade k3s clusters, initially on ARM .

At the moment this is limited (or at least only tested on) to a cluster of Raspberry Pi4s with Ubuntu 18.04/19.10 but I have tried to allow for other supported
architectures.

This is only of interest if you want to use Ansible, if you are just interested in k3s then do yourself a favour and look at 

https://github.com/alexellis/k3sup

This is learning Ansible exercise for me and I have been 'influenced' by 

https://github.com/geerlingguy/raspberry-pi-dramble

https://github.com/mrlesmithjr/ansible-rpi-k8s-cluster

My test cluster is based on the work below.

https://github.com/danacr/kubernetes-the-fun-way/tree/master/01-portable-kubernetes-cluster


###### **Notes**

I am using Ubuntu 18.04 and 19.10 as these were the only arm64 distros that fixed the Raspberry Pi4 4GB usb issue when I started. I wanted to use arm64 as I felt this would have better and wider support. It does but sometimes barely.

Arm64 while having wider support still has issues with available images for Kubernetes components, csi providers being a case in point.
 
I am cheating here and expecting a the ip addresses of the raspberry pi's are already setup, and the ssh keys added if you want to use Ansible with keys

I have used the Rancher k3s install script as a reference for what is needed for an install.

**Features**

Create single master cluster using either internal database or an external non tls database (only Postgres or Mysql atm)

Specify the version you want to install.
 
Creates the systemd services.

Creates the utility scripts to uninstall and kill k3s processes

Disable standard k3s components.

Pass arguments to the service startup.

 
There are two major playbooks main.yml and upgrade.yml.

**Main.yml** - Is the main playbook to download and configure a k3s cluster.
At the moment its just a simple Single Master cluster. It closely replicates the the official Rancher scripts so creates the systemd services and utility scripts that the Rancher scripts do.

Set the version you want to install in the vars file, please use the exact version number as it appears in the Rancher github repo. The playbook will checksum the download to make sure it's not corrupt.

`k3s_version:  v1.0.1`

The below options control adding and removing named packages also the general upgrading of the os

`k3s_apt_upgrades: false`

`k3s_prereqs:  true`

The file var/packages.yml has two lists specifying the packages you want to install and those you want to remove. 

There is a section to enable external datbase support, its a WIP so only mysql and postgres and not yet tls, set _k3s_extdb_ to true to  enable.


`k3s_extdb: true`

`k3s_dbhost: db-hostnane`

`k3s_dbusername: k3s`

`k3s_dbpassword: ComplexPassword`

`k3s_dbname: kubernetes`

`k3s_dbtype: postgres`

There is an interesting 'dirty' use case here for an external database, potentially if they were wanted 'test' clusters with different application stacks and configs
could be setup with unique database names. Setup a cluster with an external database, when you are happy destroy the cluster and create another one with a new unique name and config. When you want to return 
to a previous config you can either create a new cluster pointing to the existing database or a copy of it and when the cluster comes up it will redeploy all the 
resources previously deployed in that cluster config. I am guessing this would be completely unsupported but it is an easy way to swap between configs. The immediate gotcha 
and reason not to do do this is storage and stateful sets. That said I have switched between a 'Kubernetes the fun way cluster' with rook-ceph storage and a stateless
cluster and back again successfully with no data loss, the biggest issue was the length of time it took to download all the images over my home broadband.   

 
Things to note:

I am making sure Python3 is installed as I am also pip installing the Openshift k8s python libraries as I am looking at Ansible for k8s cluster management. 
I am removing snap and lxd as I dont need them and I saw reference to removing them in a k3s github issue.

Leave below option set to false, its a work in progress automatic upgrade if the main play book detects that the k3s binary has changed.

`k3s_allow_upgrade: false`
 

The vars file allows you to pass config variables to k3s and these will be added to the service file.
There are separate sections for labels, taints, --no-deploy options. 

There is also a generic options section that in truth overlaps the Labels, Taints and deploy options and can you just use the this section. There is an example file to see the usage.

There is the option to modify the kubeconfig file to allow non sudo access to kubectl, set the value to true.

`k3s_non_root_kubectl: false `

**Upgrade.yml** - This will download the specified version of k3s then step though all the nodes draining them and restarting the k3s services. 
It works through the nodes one at a time pausing a set time between each. Just update the vars.yml file with the version you want.

At the moment I am using the `kubectl  drain --ignore-daemonsets` command.

This needs more thought and probably needs manual intervention when you have PV/PVCs. 
This feature will most likely be removed as Rancher are implementing K3s upgrade controller which will render this feature redundant. 


There is also an uninstall playbook that merely calls the uninstall scripts copied over.

A couple of points...

First the phrase I hate.. '**This is not a production ready cluster**'

and

Assume this will destroy data somehow, especially the uninstall playbook. Also to note the destination path for the kubeconfig file, it defaults to ~/.kube/config.

You can change the file name but it will got to the ~/.kube directory.
 
 **TODO**
 
 Populate Service environment files with options.
 
 Multimaster setup
 
 LowLevel Raspberry PI setup including at least some fw rules. 
 
 
 
