First. Lets get a basic Ubuntu VM online so you can use the openstack CLI clients and related tools. These can be installed on Windows or Mac but work best in Linux.

After you are logged in and got your password changed. Lets launch a instance and log into it.

First you will need to create a SSH key and provide the public key for it to openstack. You can do this directly in openstack as well to generate your key. We will cover both.

### **Creating a key**

Select _Compute_, Select _Key Pairs_, Click "_Create Key Pair_"

![](https://raw.githubusercontent.com/RIT-GCI-CyberRange/Openstack-Guides/main/guide-images/create-key-1.png)

Then enter a friendly key name. Then select _SSH Key_.

![](https://raw.githubusercontent.com/RIT-GCI-CyberRange/Openstack-Guides/main/guide-images/create-key-2.png)

Once completed and submitted, it will have you copy or download the PEM file for you. 

Keep this file safe, you will need it later!

### Creating the Instance 

Select _Compute_,  Select _Instances_, and click " Launch Instance "

![](https://raw.githubusercontent.com/RIT-GCI-CyberRange/Openstack-Guides/main/guide-images/create-vm-1.png)

We will be creating a Ubuntu 22.04 VM. Select _Source_, Select the UP Arrow under _Available_ related to the UbuntuJammy2204 Image. 

Set the Volume Size to 80(GB) and Select "Delete Volume on Instance Delete" 

![](https://raw.githubusercontent.com/RIT-GCI-CyberRange/Openstack-Guides/main/guide-images/create-vm-2.png)

Next is "Flavor". This is the "Size" of the VM you are creating. The vCPU, RAM are set in stone. the rest are just minimums (like Disk sizes)

Select the "Medium" flavor using the UP array next to it. This will create a VM with 4 Cores and 2GB of ram.

![](https://raw.githubusercontent.com/RIT-GCI-CyberRange/Openstack-Guides/main/guide-images/create-vm-3.png)

Next is the Networks. You should have access to MAIN-NAT. MAIN-NAT is a very large internal network with internet access. Getting a public IP for your instance will come later. 

Select MAIN-NAT using the UP Arrow.

![](https://raw.githubusercontent.com/RIT-GCI-CyberRange/Openstack-Guides/main/guide-images/create-vm-4.png)

All other settings are at their defaults. Feel free to explore the options. You can always create a new VM!

Once Completed the _Launch Instance_ button should be able to be clicked and your instance will start to be created!

You will see it listed on the instances page with its details. Click on the Instance Name to see more details about the instance.


### VM Firewall and Security Groups

To be able to SSH into your VM, we will need to open the firewall up in openstack

First, Select _Network_, Then _Security Groups_. Then _Manage Rules_ for the "Default" Security Group.

![](https://raw.githubusercontent.com/RIT-GCI-CyberRange/Openstack-Guides/main/guide-images/secgroups-1.png)

This will change the main firewall rule that all future VMs will be spawned under unless changed at the time of the instance creation.

We are going to allow all SSH Traffic inbound to the VM. To do this Click "_Add Rule_" to get started.

Select the Rule "Custom TCP Rule", Then put a Description, Select the Direction of "Ingress", Select Open Port "Port", Type Port "22", Remote should be "CIDR" and CIDR should be "0.0.0.0/0" (To allow the whole internet to access it, You can also put your IP/32 in there as well to lock it down.

![](https://raw.githubusercontent.com/RIT-GCI-CyberRange/Openstack-Guides/main/guide-images/secgroups-2.png)

Once completed. Your rules should look like this:

![](https://raw.githubusercontent.com/RIT-GCI-CyberRange/Openstack-Guides/main/guide-images/secgroups-3.png)















