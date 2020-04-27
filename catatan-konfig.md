#Kasih Password
user set namauser password="password"

#limit akses user

user set namauser allowed-address=202.108.5.2

#service list
ip service print
##disable service
ip service disable telnet,ftp,www
###ganti port
ip service set ssh port=10022
###limit user akses service
ip service set winbox address=202.108.5.2
	

#OSPF
routing ospf set instance 0 router-id=1.1.1.1
routing ospf network add area=backbone network=202.108.5.0/29

#BGP
routing bgp instance set 0 as=500 redistribute-connected=yes router-id=1.1.1.1

##Peering BGP
routing bgp peer add remote-address=202.108.5.2 remote-as=600
routing bgp peer add remote-address=202.108.5.3 remote-as=600

#MPLS LDP
mpls ldp set enable=yes lsr-id=202.108.5.1 transport-address=202.108.5.1

##Register MPLS Interface
mpls interface add interface=ether1
mpls interface add interface=ether2
mpls interface add interface=ether3
mpls interface add interface=ether4

#MPLS L2VPN
routing bgp peer add address-families=l2vpn remote-addres=202.108.5.2 remote-as=600 update-source=loopback
interface vpls bgp-vpls add export-route-targets=600:1 import-route-targets=600:1 name=bgp-vpls1 route-distinguisher=600:1 site-id=1
interface bridge add name=100:1
interface bridge port add bridge=100:1 interface=ether2
interface bridge port add bridge=100:1 interface=vpls1

#VRF
ip route vrf add routing-mark=vrf-1 interface=ether2
ip route vrf add routing-mark=vrf-2 interface=ether3

ip route add routing-mark=vrf-1 dst-address=0.0.0.0/0 gateway 202.108.5.1@main
ip route add routing-mark=vrf-2 dst-address=0.0.0.0/0 gateway 202.108.5.1@main

ip firewall mangle add chain=prerouting in-interface=ether2 action=mark-connection new-connection-mark=vrf-1-conn passthrough=yes
ip firewall mangle add chain=prerouting in-interface=ether1 connection-mark=vrf-1-conn action=mark-routing new-routing-mark=vrf-1 passthrough=yes
ip firewall mangle add chain=prerouting in-interface=ether3 action=mark-connection new-connection-mark=vrf-2-conn passthrough=yes
ip firewall mangle add chain=prerouting in-interface=ether1 connection-mark=vrf-2-conn action=mark-routing new-routing-mark=vrf-2 passthrough=yes

#Kirim Logging ke Host
system logging action remote=202.108.5.2:514

#Loop Protect
interface ethernet loop-protect=on

#VLAN
vlan tagged = trunk
vlan untagged = vlan access

interface bridge add name=br-vlan10
interface bridge add name=br-vlan20

interface vlan add name=vlan-10 mtu=1500 arp=enabled vlan-id=10 interface=ether2 use-service-tag=no
interface vlan add name=vlan-20 mtu=1500 arp=enabled vlan-id=20 interface=ether2 use-service-tag=no

interface bridge port add interface=vlan-10 bridge=br-vlan10
interface bridge port add interface=ether2 bridge=br-vlan10

interface bridge port add interface=vlan-20 bridge=br-vlan20
interface bridge port add interface=ether3 bridge=br-vlan20

##Dua VLAN dalam 1 Interface (InterVLAN)
ip address add address=202.108.5.10/24 interface=vlan10 disabled=no
ip address add address=202.108.10.10/24 interface=vlan20 disabled=no

interface vlan add name=vlan10 mtu=1500 arp=enabled vlan-id=10 interface=ether2 use-service-tag=no
interface vlan add name=vlan20 mtu=1500 arp=enabled vlan-id=20 interface=ether2 use-service-tag=no

##Trunking VLAN

interface vlan add name=vlan10 mtu=1500 arp=enabled vlan-id=10 interface=ether2 use-service-tag=no
interface vlan add name=vlan20 mtu=1500 arp=enabled vlan-id=20 interface=ether2 use-service-tag=no

