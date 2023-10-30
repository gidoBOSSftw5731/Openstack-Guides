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

Select the Rule "Custom TCP Rule", Then put a Description, Select the Direction of "Ingress", Select Open Port "Port", Type Port "22", Remote should be "CIDR" and CIDR should be "129.21.0.0/16" (To allow the only RIT IPs to access it, You can also put your IP/32 in there as well to lock it down.)

![](https://raw.githubusercontent.com/RIT-GCI-CyberRange/Openstack-Guides/main/guide-images/secgroups-2.png)

Once completed. Your rules should look like this:

![](https://raw.githubusercontent.com/RIT-GCI-CyberRange/Openstack-Guides/main/guide-images/secgroups-3.png)


### Floating IPs / Public IPs

To get a Public IP for your VM you should have the quota for a Floating IP. These do 1:1 NAT to the VM you have select. They require a Router to work and only work on ONE hop (One Router in between the public internet and the VM) For the default MAIN-NAT this is the case.

You can change what VM your IP goes to at any time.

First go to Compute, Instances, then the dropdown arrow and select "Floating IP Associations"

![](https://raw.githubusercontent.com/RIT-GCI-CyberRange/Openstack-Guides/main/guide-images/floatingip-1.png)

Then you will want to create a floating IP if you don't have one already. To do this. Click the PLUS Icon next to IP Address.

![](https://raw.githubusercontent.com/RIT-GCI-CyberRange/Openstack-Guides/main/guide-images/floatingip-2.png)

You will be greeted with selecting the Pool, Set it to "external249" and give it a friendly description. Once completed. Click Allocate IP.

![](https://raw.githubusercontent.com/RIT-GCI-CyberRange/Openstack-Guides/main/guide-images/floatingip-3.png)

You should now have your new IP selected under IP Address. Go ahead and click "Associate".

You will now note that the public IP is listed in the instances list now.

![](https://raw.githubusercontent.com/RIT-GCI-CyberRange/Openstack-Guides/main/guide-images/floatingip-4.png)


### SSH to your new instance. 

Taking that PEM file provided before as your SSH private key. you should be able to log into your ubuntu VM now!

give it a try. you will need a command like `ssh -i example-key.pem ubuntu@<FLOATINGIP>`

It may complain that your SSH key is insecure. to fix this do `chmod 700 example-key.pem`

Here is a example of logging in using Windows and the downloaded private key.

![](https://raw.githubusercontent.com/RIT-GCI-CyberRange/Openstack-Guides/main/guide-images/ssh-1.png)

### Getting the Openstack CLI

_**All examples below are using Ubuntu 22.04**_

First, we need to download and copy over our _**openrc**_ file. This file contains all the details for the openstack cli to connect to our instance.

Go to "API Access" and select Download OpenStack RC File, Then select "OpenStack RC File". This will then download a file like `<projectname>-openrc.sh`. 

![](https://raw.githubusercontent.com/RIT-GCI-CyberRange/Openstack-Guides/main/guide-images/openrc-1.png)

Upload this file to your Ubuntu VM you created before. You can use SCP, or even just copy and paste it into a new file on the VM over SSH. A example SCP command `scp -i .\Downloads\example-key.pem .\Downloads\cyberadmin-openrc.sh ubuntu@129.21.249.113:/home/ubuntu/`

Next we will install the CLI clients. on most systems you can use "pip" for this.

run the following commands on the Ubuntu box to get everything started.


`sudo apt update`

`sudo apt upgrade`

`sudo apt install python3-pip`

`sudo pip3 install python-openstackclient`




Once installed every time you log in to SSH you need to run `source ~/<projectname>-openrc.sh` It should ask for your password and return you to the command line. This will keep you logged in until you close the SSH connection.


For details on how to use the CLI, Please see the Docs: [python-openstackclient](https://docs.openstack.org/python-openstackclient/latest/cli/index.html)

For basic commands try `openstack server list` or `openstack server create --help`























