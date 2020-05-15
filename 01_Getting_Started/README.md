# LAB SETUP
My lab is fully virtualized using VMware Fusion Pro 11.5.3 on my powerful iMAC (MAC OS X Catalina w/ 32 GB RAM).

2 VMs are set up:
- VM EVE-NG PRO that hosts EVE-NG Professional version;
- VM MUSE that hosts my automation tools (Ansible, Git, etc.).

A specific VMnet (vmnet10, subnet 10.0.0.0/24) is also reserved for specific communications between these VMs.

## EVE-NG PRO VM
This VMs is a standard EVE-NG VM with 2 NICS:
- NIC 1 is shared with my iMAC,
- NIC 2 is tied to vmnet10.

### EVE-NG lab
| Node | Subtype | Management IPs |
| --- | --- | --- |
| lab-nat | Cisco CSR 1000V (XE 16.x) |
| lab-dupl | Cisco IOS 7206VXR (Dynamips) |
| lab_sw_1 | Cisco vIOS Switch | (N/A) |
| lab_sw_2 | Cisco vIOS Switch | (N/A) |

SSH is enabled and the username admin (password cisco} is defined on routers.


#### LAB-NAT 
| Interface | IP Address | Comment |
| --- | --- | --- |
| Gi1 | 1.0.0.1/24 | connected to lab_sw_1
| Gi2 | 22.0.0.1/24 | connected to lab_sw_2
| Gi3 | 10.0.0.2/24 | connected to cloud interface (vmnet10)

#### LAB-DUPL
| Interface | IP Address | Comment |
| --- | --- | --- |
| Fa0/0 | 22.0.0.22/24 | connected to lab_sw_2 |
| Fa2/0 | 10.0.0.3/24 | connected to cloud interface (vmnet10) |


## Automation Host VM
For this VM, I simply used the IaC (Infrastructure as Code) Development Workstation based on Ubuntu 18.0.4 Desktop.
All the details are given in the following website: https://github.com/carlbuchmann/iac-dev 

| Package | Version |
| --- | --- |
| ansible | 2.9.9 |
| git | 2.17.1 |
| python | 2.7.17 |

The VM has 2 NICs: the first is shared with my iMAC and the second is connected to vmnet10 for router management access.

# VERIFICATIONS

## SSH access
```console
muse@ubuntu:~$ ssh admin@10.0.0.2
Password: 



LAB-NAT#exit
Connection to 10.0.0.2 closed.
muse@ubuntu:~$ ssh admin@10.0.0.3
Password: 

LAB-DUP#exit
Connection to 10.0.0.3 closed by remote host.
Connection to 10.0.0.3 closed.
muse@ubuntu:~$ 
```


## Ansible raw access

### Inventory file
```yml
[lab]
LAB-NAT ansible_host=10.0.0.2
LAB-DUP ansible_host=10.0.0.3

[lab:vars]
ansible_connection=network_cli
ansible_network_os=ios
ansible_user=admin
ansible_password=cisco
```

### Ansible Access: Final Test
```console
muse@ubuntu:~/BNAS/01_Getting_Started$ ansible all -i inventory -c network_cli -m cli_command -a "command='show version | inc up'"
LAB-NAT | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false, 
    "stdout": "Technical Support: http://www.cisco.com/techsupport\nLAB-NAT uptime is 3 hours, 45 minutes", 
    "stdout_lines": [
        "Technical Support: http://www.cisco.com/techsupport", 
        "LAB-NAT uptime is 3 hours, 45 minutes"
    ]
}
LAB-DUP | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false, 
    "stdout": "Technical Support: http://www.cisco.com/techsupport\nLAB-DUP uptime is 2 hours, 42 minutes\nThis configuration is within the PCI bus capacity and is supported. \nThis configuration is within the PCI bus capacity and is supported.", 
    "stdout_lines": [
        "Technical Support: http://www.cisco.com/techsupport", 
        "LAB-DUP uptime is 2 hours, 42 minutes", 
        "This configuration is within the PCI bus capacity and is supported. ", 
        "This configuration is within the PCI bus capacity and is supported."
    ]
}
muse@ubuntu:~/BNAS/01_Getting_Started$ 
```