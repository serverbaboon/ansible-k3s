**Ansible Playbook to install k3s on to a Raspberry Pi4 cluster**

This is a work in progress project to create and upgrade k3s clusters.

At the moment this is limited (or at least only tested on) to a cluster of Raspberry Pi4s with Ubuntu 19.10.

This is only of interest if you want to use Ansible, if you are just interested in k3s then do yourself a favour and look at 

https://github.com/alexellis/k3sup

This is learning Ansible exercise for me and I have been 'influenced' by 

https://github.com/geerlingguy/raspberry-pi-dramble

https://github.com/mrlesmithjr/ansible-rpi-k8s-cluster

**Notes**

Using Ubuntu 19.10 as this was the only arm64 distro that fixed the Raspberry Pi4 4GB usb issue when I started. I wanted to use arm64 as I felt this would have better and wider support. It does but sometimes barely.

Arm64 while having wider support still has issues with available images for Kubernetes cnmponents, csi providers being a case in point.
 
I am cheating here and expecting a the hostnames and ip addresses of the raspberry pi's are already setup.


There are two major playbooks main.yml and upgrade.yml.

**Main.yml** - Is the main playbook to download and configure a k3s cluster.
At the moment its just a simple Single Master cluster. It closely replicates the the official Rancher scripts so creates the systemd services and utility scripts that the Rancher scripts do.

Set the version you want to install in the vars file, please use the exact version number as it appears in the Rancher github repo. The playbook will checksum the download to make sure it's not corrupt.

`k3s_version:  v1.0.1`

The below options control adding and removing named packages also the general upgrading of the os

`k3s_apt_upgrades: false`

`k3s_prereqs:  true`

The file var/packages.yml has two lists specifying the packages you want to install and those you want to remove. 

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

**Upgrade.yml** - This will download the specified version of k3s then step though all the nodes cordoning them and restarting the k3s services. 
It works through the nodes one at a time pausing a set time between each. Just update the vars.yml file with the version you want.
At the moment I am using the `kubectl  cordon` command as I am not sure about the `kubectl drain` command and dealing with the aborts.
This needs more thought and probably needs manual intervention when you have PV/PVCs.


There is also an uninstall playbook that merely calls the uninstall scripts copied over.

A couple of points...

First the phrase I hate.. '**This is not a production ready cluster**'

and

Assume this will destroy data somehow, especially the uninstall playbook. Also to note the destination path for the kubeconfig file, it defaults to ~/.kube/config and while creating a backup before doing anything the file will be overwritten.
 
 **TODO**
 
 Populate Service environment files with options.
 
 Multimaster setup
 
 LowLevel Raspberry PI setup
 
 
 