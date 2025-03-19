
Simple script to generate /etc/network/interfaces file for a given ganeti host.

Also will ensure switch configuration in Netbox is correctly set up for the ganeti host.

Dependencies: python3-jinja2 python3-pynetbox

Example:
```
cmooney@wikilap:~$ ./ganeti_network.py --host ganeti1026
Netbox API token: 

Netbox: asw2-a7-eqiad interface xe-7/0/35 changed mode to tagged.
Netbox: asw2-a7-eqiad interface xe-7/0/35 tagged vlans set to [public1-a-eqiad, analytics1-a-eqiad]


ganeti1026 /etc/network/interfaces:

*******************************************************************************
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).


source /etc/network/interfaces.d/*

# The loopback network interface
auto lo private public analytics
iface lo inet loopback

# The primary network interface
allow-hotplug enp175s0f0np0
iface private inet static
    address 10.64.0.84/22
    gateway 10.64.0.1
    # dns-* options are implemented by the resolvconf package, if installed
    dns-nameservers 10.3.0.1
    dns-search eqiad.wmnet
    bridge_ports    enp175s0f0np0
    bridge_stp      off
    bridge_maxwait  0
    bridge_fd       0

    up ip token set ::10:64:0:84 dev private
    up ip addr add 2620:0:861:101:10:64:0:84/64 dev private


iface public inet manual
    pre-up ip link add name 1001 link enp175s0f0np0 type vlan id 1001
    post-down ip link delete dev 1001 type vlan
    bridge_ports   1001
    bridge_stp     off
    bridge_maxwait 0
    bridge_fd      0
    up sysctl net.ipv6.conf.public.accept_ra=0

iface analytics inet manual
    pre-up ip link add name 1030 link enp175s0f0np0 type vlan id 1030
    post-down ip link delete dev 1030 type vlan
    bridge_ports   1030
    bridge_stp     off
    bridge_maxwait 0
    bridge_fd      0
    up sysctl net.ipv6.conf.analytics.accept_ra=0

*******************************************************************************


NOTE: Netbox vlan settings were updated, please run sre.network.configure-switch-interfaces cookbook.
```
