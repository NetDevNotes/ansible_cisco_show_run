# Ansible Perfoming a Show Run on a Cisco Switch :coffee:

* This play performs a `show run` on a Cisco switch and outputs the config to a text file. 
* The inventory file uses groups-of-groups but we are only interested in the switch in this play.
* The inventrory file uses group vars to define key value pairs for passing through the username and password.
* The playbook uses the `ios_command` Ansible module.
* The password var is in clear text for the moment.

# How did I create this play?

1. Created a new repository called *ansible_cisco_show_run* at [GitHub](www.github.com)
2. When creating repository tick the box that automatically creates a README.md
3. In GitHub, click **Clone or Download** and copy the repositories URL
> https://github.com/NetDevNotes/ansible_cisco_show_run.git
4. On my workstation, performed a `git clone https://github.com/NetDevNotes/ansible_cisco_show_run.git`
5. The `git clone` downloaded the folder and README.md to my local workstation, cd into the folder:
```
# cd ansible_cisco_show_run
# pwd
~/scripts/ansible/ansible_cisco_show_run
```
6. I created an inventory file (called `hosts`) which is more complex than it needs to be, but I wanted to learn how to use groups and groups-of-groups.  The bit to understand here though is that I defined host vars `[switch:vars]` for the username and password, amoung some other vars we need to connect to the switch successfully:
```
[ios:children]
# This is a group-of-groups called ios
switch
asa

[switch]
# This is the switch group and a child of the ios group
10.1.1.250

[switch:vars]
# These are group vars for the switch group only
ansible_connection=network_cli
ansible_become=yes
ansible_become_method=enable
ansible_network_os=ios
ansible_user=admin
ansible_ssh_pass=12121212

[asa]
# This is the asa group and also a child of the ios group
10.1.1.253
```
7. I created a simple playbook `showrun.yml` using the ios_command ansible module:
```
---
- name: Backup show run (enable mode commands)
  hosts: switch
  gather_facts: false
  connection: local

  tasks:
    - name: run enable level commands
      ios_command:
 # Use Become
 #       authorize: yes
        commands:
          - show run

      register: print_output

    -  debug: var=print_output.stdout_lines

    - name: save output to a file
      copy: content="{{ print_output.stdout[0] }}" dest="./output/{{ inventory_hostname }}.txt"

# Description:
# Use the ios commands module to get show run information
# Go to enable mode
# Save the output to files
# http://docs.ansible.com/ansible/latest/ios_command_module.html

# Commands to run:
#1) sudo ansible-playbook showrun.yml
```
# Command to Run the Play

`sudo ansible-playbook showrun.yml`

I have to put in the workstations sudo password at the moment, but I'll fix that later..
```
# sudo ansible-playbook showrun.yml
Password:

PLAY [Backup show run (enable mode commands)] ***************************************************************************************

TASK [run enable level commands] ****************************************************************************************************
ok: [10.1.1.250]

TASK [debug] ************************************************************************************************************************
ok: [10.1.1.250] => {
    "print_output.stdout_lines": [
        [
            "Building configuration...",
            "",
            "Current configuration : 3307 bytes",
            "!",
            "version 12.2",
            "no service pad",
            "service timestamps debug datetime msec",
            "service timestamps log datetime msec",
            "service password-encryption",
            "!",
            "hostname SW1-2960",
```
```
TASK [save output to a file] ********************************************************************************************************
changed: [10.1.1.250]

PLAY RECAP **************************************************************************************************************************
10.1.1.250                 : ok=3    changed=1    unreachable=0    failed=0
```
> The output file is within this repository if you want to take a look.  No rocket science there though :smile:

:musical_keyboard:
