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

### Using the SSHJumpHost to SSH to your instance without a floating IP

As the Cyber Range has a limited number of public IPv4 addresses, it is not practical to assign a public address to every machine in a project. To mitigate this issue, we have created a SSH Jump Box that allows you to SSH to your instances without a public IP address.

In order to use the SSH Jump Box, you must first add your public SSH keys to Openstack as detailed above. After this has been done, visit https://people.rit.edu/~fffics/ and click the "Sync SSHJump Keys" button. This will add your public keys to the SSH Jump Box.

Next, attach your machine to the `SSHJumpNet` network. This network does not have a default gateway or internet. SSHJump lives at `100.65.1.1`.
 It is not a given that your machine will gain an IP address automatically via DHCP on a secondary network, so it may be required to either set `SSHJumpNet` as your default/primary network interface or manually assign an IP address to the interface (e.g. using cloud-init to run `ip a add 100.65.x.x/16`.)

Once your machine has an IP address on `SSHJumpNet`, you can SSH to your instance using the following command:

ssh -J <ritusername>@ssh.cyberrange.rit.edu <sshusername>@100.65.X.X

Do note that  ssh -i privatekey.pem  will only give your private key to the destination, not to SSHJump. Please see this except from the man page of SSH

     -J destination
             Connect to the target host by first making a ssh connection
             to the jump host described by destination and then
             establishing a TCP forwarding to the ultimate destination
             from there.  Multiple jump hops may be specified separated
             by comma characters.  This is a shortcut to specify a
             ProxyJump configuration directive.  Note that configuration
             directives supplied on the command-line generally apply to
             the destination host and not any specified jump hosts.  Use
             ~/.ssh/config to specify configuration for jump hosts.

Here is a example ~/.ssh/config to give a custom private key to ssh to use with SSHJump system

```
Host ssh.cyberrange.rit.edu
    HostName ssh.cyberrange.rit.edu
    User <ritusername>
    Port 22
    IdentityFile ~/customkey.pem
```

Also another trick is to have all of 100.65.0.0/16 go over the proxy, You can add this before the above to have it automatically proxy over SSHJump for you without having to give -J

```
Host 100.65.*
    Port 22
    IdentityFile ~/openstackkey.pem
    ProxyJump ssh.cyberrange.rit.edu
```

One important note is that there is *no* client isolation on `SSHJumpNet`, which means that your machine, by default, will be able to communicate with any other machine on the network. This is not only a security risk (which will make people be disappointed in you) but could easily impact other users of the stack as well (which makes people angry at you.) The method to prevent this is to ensure that the security groups applied ONLY permit traffic from the jump host (`100.65.0.1`) over your SSH Port (TCP 22 by default) via the `SSHJumpNet` network. Any other `SSHJumpNet` traffic *must* be dropped. The rule that accomplishes this task is shown below:

![Allowing SSH traffic via the jump host](https://raw.githubusercontent.com/gidoBOSSftw5731/Openstack-Guides/main/guide-images/sshjumpnet-01.png)

Since Openstack does not have the ability to create DROP rules in security groups, you *must* ensure that you do not have any ingress rules to allow traffic from `0.0.0.0` through. It is much preferred that you only whitelist RIT's network (which is detailed below.)

If you have a need to have a service open to the whole internet, you can set the Port Security Group on the port attached to `SSHJumpNet` separately. It's also desired that, if anything you are running has the ability to negatively affect other machines on the network, you specifically create a security group for that machine that only allows traffic from the jump host, including denying all egress traffic over the `SSHJumpNet` network. As Security Groups are stateful, there's no need to be concerned with Egress rules. 

Common Errors

- `<ritusername>@ssh.cyberrange.rit.edu: Permission denied (publickey)`

    - This issue is causes by your username being wrong on the SSHJump host, or the Private Key was rejected, check all settings and resync your SSHJump keys

- `<sshusername>@100.65.X.X: Permission denied (publickey).`

    - Your VM Rejected your key. Please check your settings, Don't forget about the caveats with `ssh -i`

- `channel 0: open failed: connect failed: No route to host or other connection type errors`

    - Your VM is unable to be reached by SSHJump, Please check your network settings, and make sure your VM has picked up its SSHJumpNet IP. The Openstack dashboard only says if an IP address has been assigned, you must utilize the console if you want to verify if the VM actaully acquired an address.


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