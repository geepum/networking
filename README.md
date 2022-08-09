# Networking

## Commands

### Basic commands
- ip default-g 1.1.10.254 	: ip default-gateway
- vlan 10			: setup vlan
- name VLAN_10			: name vlan
- int range f1/1 , f1/15	: interface with non consecutive port nums
- int range f1/1 - 2		: interface with consecutive port num
- sw m a			: switch mode access	<=> no ..
- sw a v 10			: switch access vlan 10	<=> no ..
- sw t e d			: switch trunk encapsulation dot1q
- sw m tr			: switch mode trunk

### setting up vlan for routers
  1. int f0/0.10	<=> no ...
  2. en dot1q 10	<=> no ...
  3. ip add 1.1.10.254 255.255.255.0	<=> no ...

### Native VLAN
  - SW : sw tr native vlan 10	<=> no ...
  - R  : encap d 10 native 	<=> no ...

### Setting up ip routing
  - ip route (destination network ip) (broadcast) (out interface) (next ip)
  - ip route 100.30.0.0 255.255.255.0 s1/0 204.200.7.2

- ping 1.1.4.4 source 1.1.1.1

### pap
  - R1(config)# username R2 pass cisco => int s1/0 => en ppp => ppp pap sent R1 pass cisco => pp authen pap => clock rate 64000
  - R2(config)# username R1 pass cisco => int s1/0 => en ppp => ppp authen pap => ppp pap sent R2 pass cisco => clock rate 64000

### chap
  - R1(config)# username R2 pass cisco => int s1/0 => en ppp => ppp authen chap => ip add 1.1.10.1 255.255.255.0 => no sh

### L3 SW SVI
  - If set as `no ip routing`, change to `ip routing`
  - vlan 30 => vlan 40 => int f2/3 => sw m a => sw a v 30 => int f2/4 => sw m a => sw a v 40 => int vlan 30 => ip add 1.1.30.254 255.255.255.0 => int vlan 40 => ip add 1.1.40.254 255.255.255.0

#### Frame relay
enable
conf t
no ip domain lookup
int s1/0
  no sh
  enc frame
  no fram inver
  clock rate 64000
  exit
line c 0
  logg sy
  exec-timeout 0
  exit
line vty 0 4
  pass cisco
  exit
hostname 
---------------------------------------------------------------
R1)
conf t
int lo 0
  ip add 1.1.1.1 255.255.255.0
  exit
int s1/0
  ip add 1.1.12.1 255.255.255.0
  fram map ip 1.1.12.2 102 br
  exit
ip route 1.1.2.0 255.255.255.0 s1/0 1.1.12.2
ip route 1.1.3.0 255.255.255.0 s1/0 1.1.12.2
ip route 1.1.4.0 255.255.255.0 s1/0 1.1.12.2
ip route 1.1.23.0 255.255.255.0 s1/0 1.1.12.2
ip route 1.1.34.0 255.255.255.0 s1/0 1.1.12.2
do wr

R2)
conf t
int lo 0
  ip add 1.1.2.2 255.255.255.0
  exit
int s1/0.12 m
  ip add 1.1.12.2 255.255.255.0
  fram map ip 1.1.12.1 201 br
  exit
int s1/0.23 m
  ip add 1.1.23.2 255.255.255.0
  fram map ip 1.1.23.3 203 br
  exit
ip route 1.1.1.0 255.255.255.0 s1/0.12 1.1.12.1
ip route 1.1.3.0 255.255.255.0 s1/0.23 1.1.23.3
ip route 1.1.4.0 255.255.255.0 s1/0.23 1.1.23.3
ip route 1.1.34.0 255.255.255.0 s1/0.23 1.1.23.3
do wr

R3)
conf t
int lo0
  ip add 1.1.3.3 255.255.255.0
  exit
int s1/0.23 m
  ip add 1.1.23.3 255.255.255.0
  fram map ip 1.1.23.2 302 br
  exit
int s1/0.34 p
  ip add 1.1.34.3 255.255.255.0
  fram interface 304
  exit
ip route 1.1.1.0 255.255.255.0 s1/0.23 1.1.23.2
ip route 1.1.2.0 255.255.255.0 s1/0.23 1.1.23.2
ip route 1.1.12.0 255.255.255.0 s1/0.23 1.1.23.2
ip route 1.1.4.0 255.255.255.0 s1/0.34 1.1.34.4
exit
wr

R4)
conf t
int lo0
  ip add 1.1.4.4 255.255.255.0
  exit
int s1/0.34 p
  ip add 1.1.34.4 255.255.255.0
  fram interface-dlci 403
  exit
ip route 1.1.1.0 255.255.255.0 s1/0.34 1.1.34.3
ip route 1.1.2.0 255.255.255.0 s1/0.34 1.1.34.3
ip route 1.1.3.0 255.255.255.0 s1/0.34 1.1.34.3
ip route 1.1.12.0 255.255.255.0 s1/0.34 1.1.34.3
ip route 1.1.23.0 255.255.255.0 s1/0.34 1.1.34.3
exit
wr

------------------------------------------------

R1) ip route 1.1.0.0 255.255.192.0 s1/0 1.1.12.2
R2) ip route 1.1.0.0 255.255.192.0 s1/0.23 1.1.23.3
    ip route 1.1.1.0 255.255.255.0 s1/0.12 1.1.12.1
R3) ip route 1.1.4.0 255.255.255.0 s1/0.34 1.1.34.4
    ip route 1.1.0.0 255.255.224.0 s1/0.23 1.1.23.2
R4) ip route 1.1.0.0 255.255.224.0 s1/0.34 1.1.34.3 OR
(better) ip route 1.1.0.0 255.255.192.0 s1/0.34 1.1.34.3

### NAT_PT
- `no sh`
- `ip add dhcp` => internet facing interface
- general ip routes
- `ip route 0.0.0.0 0.0.0.0 f0/1 10.0.0.1`
- `ip route 0.0.0.0 0.0.0.0 s1/0.12 1.1.12.1`
- `ip route 0.0.0.0 0.0.0.0 s1/0.23 1.1.23.2`
- `ip route 0.0.0.0 0.0.0.0 s1/0.34 1.1.34.3`
- `access-list 10 permit 1.1.1.0 0.0.0.255` => same thing until 1.1.4.0 0.0.0.255
  - numbered cannot be removed, so it need to be re-written from the beginning
  - however, named can be removed
- `ip nat inside source list 10 int f0/1 overload`
- `int s1/0` => `ip nat inside`
- `int f0/1` => `ip nat out`
- `ping 10.0.0.1 source 1.1.4.4`

### Initial setup
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

### Debugging
- sh ip int br
- sh fram map
- sh run int s1/0
- debug arp 		: ARP packet debug on <=> no ...
- sh arp		: show mac address table
- sh fram map
- sh vlan-s		: show vlan-switch
- sh int trunk		: show interface trunk
- sh cdp n		: sh cdp (cisco discovery protocol) neighbor
- sh ip route		: show ip route
- sh controller s1/0	: check sex of interface ports
- (serial interface) cdp run => int s1/0 => cdp en
- (in case of mis config of int) => no... => no ip add => sh => exit => no int s1/0.23 => int s1/0.233
- deb ip pack 		: debugging on 
- sh fram lmi 		: CCITT => q933a
- sh ip nat trans	: show ip nat info
- sh run | begin nat	: show ip nat config
- # debug ppp (options)	: debug on for ppp
