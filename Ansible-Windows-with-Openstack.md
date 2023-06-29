Create your VMs using a command like so:
`openstack server create --flavor xlarge --image WinSrv2019-17763-2022 --boot-from-volume 250 --user-data openstack-cloudint.yml --nic net-id=8051145a-2de0-4a29-a5ab-9f1e13a68ca9,v4-fixed-ip=192.168.56.10  --key-name deployment DC01`

Details of this command:
* --boot-from-volume is the size in GB of the root disk
* --userdata is this file in the current path: [prepare-windows-for-ansible](https://github.com/RIT-GCI-CyberRange/Openstack-Guides/blob/main/ansible-guides/prepare-for-ansible-windows)
* --nic net-id is the UUID of the network you created, it can be found under the network information v4-fixed-ip= is the fixed IP within the network subnet
* --key-name is the key-pair you have in openstack that you use in linux

Create a jump box into the network and use it to deploy to your local windows machines. The username and password the script created is called `ansible`


Here is a example script in Ansible to create three servers, make sure to "source openrc.sh" file to login first

```
---
- name: Create servers on OpenStack and setup domain controller
  hosts: localhost
  gather_facts: no
  vars:
    server_image: WinSrv2022-20348-2022
    server_flavor: large
    network_name: corp
    subnet_name: corp_subnet
    subnet_cidr: 10.0.0.0/24
    key_name: demo-key
    security_groups_name: default
    auto_ip: no
    userdata_file: prepare-for-ansible-windows
    existing_server: openstack-toolbox
    router_name: corp_router
    static_ips:
      server1: 10.0.0.10
      server2: 10.0.0.11
      server3: 10.0.0.12

  tasks:
    - name: Load user data
      set_fact:
        user_data: "{{ lookup('file', userdata_file) }}"

    - name: Create network
      os_network:
        name: "{{ network_name }}"
        state: present

    - name: Create subnet
      os_subnet:
        state: present
        network_name: "{{ network_name }}"
        name: "{{ subnet_name }}"
        cidr: "{{ subnet_cidr }}"

    - name: Create router
      os_router:
        state: present
        name: "{{ router_name }}"
        external_gateway_info:
           network: "external249"

    - name: Create servers
      os_server:
        state: present
        name: "{{ item }}"
        image: "{{ server_image }}"
        flavor: "{{ server_flavor }}"
        nics:
          - net-name: "{{ network_name }}"
            v4-fixed-ip: "{{ static_ips[item] }}"
        key_name: "{{ key_name }}"
        security_groups: "{{ security_groups_name }}"
        auto_ip: "{{ auto_ip }}"
        userdata: "{{ user_data }}"
      loop:
        - server1
        - server2
        - server3
```