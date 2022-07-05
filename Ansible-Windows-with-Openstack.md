Create your VMs using a command like so:
`openstack server create --flavor xlarge --image WinSrv2019-17763-2022 --boot-from-volume 250 --user-data openstack-cloudint.yml --nic net-id=8051145a-2de0-4a29-a5ab-9f1e13a68ca9,v4-fixed-ip=192.168.56.10  --key-name deployment DC01`

Details of this command:
* --boot-from-volume is the size in GB of the root disk
* --userdata is this file in the current path: [prepare-windows-for-ansible](https://github.com/RIT-GCI-CyberRange/Openstack-Guides/blob/main/ansible-guides/prepare-for-ansible-windows)
* --nic net-id is the UUID of the network you created, it can be found under the network information v4-fixed-ip= is the fixed IP within the network subnet
* --key-name is the key-pair you have in openstack that you use in linux

Create a jump box into the network and use it to deploy to your local windows machines. The username and password the script created is called `ansible`