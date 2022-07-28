# Networking

## Commands

### basic commands
- ip default-g 1.1.10.254 	: ip default-gateway
- vlan 10			: setup vlan
- name VLAN_10			: name vlan
- int range f1/1 , f1/15	: interface with non consecutive port nums
- int range f1/1 - 2		: interface with consecutive port num
- sw m a			: switch mode access	<=> no ..
- sw a v 10			: switch access vlan 10	<=> no ..
- sw t e d			: switch trunk encapsulation dot1q
- sw m tr			: switch mode trunk
- setting up vlan for routers
  1. int f0/0.10	<=> no ...
  2. en dot1q 10	<=> no ...
  3. ip add 1.1.10.254 255.255.255.0	<=> no ...
- Native VLAN
  - SW : sw tr native vlan 10	<=> no ...
  - R  : encap d 10 native 	<=> no ...
- Setting up ip routing
  - ip route (destination network ip) (broadcast) (next interface) (next ip)
  - ip route 100.30.0.0 255.255.255.0 s1/0 204.200.7.2

### initial setup
en
conf t
no ip domain look
line c 0
logg s
exec-t 0
exit
line vty 0 4
pass cisco
login
exit
hostname
no ip routing

### debugging
- sh ip int br
- sh fram map
- sh run int s1/0
- debug arp 		: ARP packet debug on <=> no ...
- sh arp
- sh fram map
- sh vlan-s		: show vlan-switch
- sh int trunk		: show interface trunk
- sh cdp n		: sh cdp (cisco discovery protocol) neighbor
- sh ip route		: show ip route
