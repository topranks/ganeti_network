
Simple script to generate /etc/network/interfaces file for a given ganeti host.

Also will ensure switch configuration in Netbox is correctly set up for the ganeti host.

Dependencies: python3-jinja2 python3-pynetbox

Example:
```
*******************************************************************************

cmooney@wikilap:~$ ./ganeti_network.py --host ganeti7001
Netbox API token:

Running Netbox import script to get latest interface names from PuppetDB...
PuppetDB import script made changes: Set asw1-b3-magru et-0/0/9 tagged vlans to [<VLAN: public1-b3-magru (711)>, <VLAN: sandbox1-b3-magru (731)>] matching eno12399np0
PuppetDB import completed.
Host primary physical interface according to PuppetDB is eno12399np0
The /etc/network/interfaces config has been written to ganeti7001.txt
NOTE: Netbox vlan settings were updated, please run sre.network.configure-switch-interfaces cookbook.



cmooney@wikilap:~$ 
cmooney@wikilap:~$ cat ganeti7001.txt 
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).


source /etc/network/interfaces.d/*

# The loopback network interface
auto lo private public sandbox
iface lo inet loopback

# The primary network interface
allow-hotplug eno12399np0
iface private inet static
    address 10.140.0.11/24
    gateway 10.140.0.1
    # dns-* options are implemented by the resolvconf package, if installed
    dns-nameservers 10.3.0.1
    dns-search magru.wmnet
    bridge_ports    eno12399np0
    bridge_stp      off
    bridge_maxwait  0
    bridge_fd       0

    up ip token set ::10:140:0:11 dev private
    up ip addr add 2a02:ec80:700:101:10:140:0:11/64 dev private



iface public inet manual
    pre-up ip link add name 711 link eno12399np0 type vlan id 711
    post-down ip link delete dev 711 type vlan
    bridge_ports   711
    bridge_stp     off
    bridge_maxwait 0
    bridge_fd      0
    up sysctl net.ipv6.conf.public.accept_ra=0


iface sandbox inet manual
    pre-up ip link add name 731 link eno12399np0 type vlan id 731
    post-down ip link delete dev 731 type vlan
    bridge_ports   731
    bridge_stp     off
    bridge_maxwait 0
    bridge_fd      0
    up sysctl net.ipv6.conf.sandbox.accept_ra=0


*******************************************************************************


NOTE: Netbox vlan settings were updated, please run sre.network.configure-switch-interfaces cookbook.
```
