# Networking

## Commands

### ip add
- `ip add 5.5.0.1 255.255.255.0 secondary`

### default gateway IP
- ip default-g 1.1.10.254 	: ip default-gateway

### VLAN
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

### Frame relay
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

### multi-point frame-relay
- R1) `conf t` => `int lo0` => `ip add 10.10.1.1 255.255.255.0` => `exit` => `int s1/0` => `ip add 10.10.123.1 255.255.255.0` => `fram map ip 10.10.123.2 102 b` => `fram map ip 10.10.123.3 102 b`
- R2) `conf t` => `int lo0` => `ip add 10.10.2.2 255.255.255.0` => `exit` => `int s1/0.123 m` => `ip add 10.10.123.2 255.255.255.0` => `fram map ip 10.10.123.1 201 b` => fram map ip 10.10.123.3 203 b`
- R3) `conf t` => `int lo0` => `ip add 10.10.3.3 255.255.255.0` => `exit` => int s1/0.123 m` => `ip add 10.10.123.3 255.255.255.0` => `fram map ip 10.10.123.2 302 b` => `fram map ip 10.10.123.1 302 b` => `int s1/0.34 p` => `ip add 150.1.34.1 255.255.255.0` => `fram inter 304`
- R4) `conf t` => `int lo0` => `ip add 150.1.4.4 255.255.255.0` => `exit` => `int s1/0.34 p` => `ip add 150.1.34.254 255.255.255.0` => `fram inter 403`

### route-map
- basic config
- fram map ip
- next hop ping testing
- ip routing
- ping testing
- router rip
- route-map
  - R3) `route-map STATIC` / automatically adds 10 at the end => `match ip address prefix NET1` => `set metric 3` => `route-m STATIC 20` => `match ip add pre NET2` => `set metric 2` => route-m CONNECTED` => `match interface s1/0.123 lo0` => `set metric 1` => `ip prefix-list NET1 permit 16.16.1.0/24` => `ip pre NET2 permit 16.16.2.0/24`
- redistribute
  - R3) `redistribute static route STATIC` => `redistribute connected route CONNECTED`
- ping testing

### dhcp relay
- VLAN
  - SW1) `vlan 10` => `int range f1/1 - 2` => `no sh` => `sw m a` => `sw a v 10` => `exit`
  - SW1) `int vlan 10` => `ip add 1.1.10.154 255.255.255.0` => `exit`

- L2 WAN - PPP and IP
  - SW1) `int f0/0` => `no sh` => `ip add dhcp`
  - SW1) `username R1 pass cisco` => `int s2/0` => `no sh` => `en ppp` => `ppp authen chap` => `clock rate 64000` => `ip add 1.1.12.5 255.255.255.252` => `exit`
  - R1) `int f0/0` => `no sh` => `ip add 1.1.20.254 255.255.255.0` => `exit`
  - R1) `username SW1 pass cisco` => `int s1/0` => `no sh` => `en ppp` => `ppp authen chap` => `ip add 1.1.12.6 255.255.255.252`

- RIP
  - SW1) `router rip` => `ver 2` => `network 1.0.0.0` => `no auto` => `passive default` => `no passive s2/0` => `int s2/0` => `ip split`
  - SW1) `router rip` => `default-information originate` OR `redistribute static metric 1` but this is not used often and not recommended
  - R1) `router rip` => `ver 2` => `network 1.0.0.0` => `no auto` => `passive f0/0` => `int f0/0` => `ip sp`


- L3 routing protocol
  - SW1) `ip route 0.0.0.0 0.0.0.0 f0/0 10.0.0.1`

- DHCP
  - SW1) `ip dhcp pool NET10` => `network 1.1.10.0 /24` => `default-router 1.1.10.254` => `dns-server 1.1.10.100 168.126.63.1` => `lease 7` => `exit`
  - SW1) `ip dhcp pool NET20` => `network 1.1.20.0 /24` => `default-router 1.1.20.254` => `dns-server 1.1.10.100 168.126.63.1` => `l 7` => `exit`
  - SW1) `ip dhcp pool NET10` => `class 10` => `address range 1.1.10.1 1.1.10.253`
  - SW1) `ip dhcp pool NET20` => `class 20` => `address range 1.1.20.1 1.1.20.253`
  - SW1) `int vlan 10` => `ip dhcp client class-id 10`
  - SW1) `int s2/0` => ip dhcp client class-id 20`
  - SW1) `ip dhcp excluded-address 1.1.10.100`
  - R1) `int f0/0` => `ip helper-address 1.1.12.5`

- NAT
  - SW1) `access-list 1 permit 1.1.10.0 0.0.0.255`
  - SW1) `access 1 permit 1.1.20.0 0.0.0.255`
  - SW1) `access 1 permit 1.1.12.4 0.0.0.3` Only for ping test
  - SW1) `ip nat inside source list 1 int f0/0 overload`
  - SW1) `int vlan 10` => `ip nat inside`
  - SW1) `int s2/0` => `ip nat inside`
  - SW1) `int f0/0` => `ip nat outside`

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

### domain
- if `no ip domain look` => change it to `ip domain look`
  - but if asked with unknown IPs/addresses, it will loop
- `ip name-server 8.8.8.8`
- `ping www.google.com`

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

### username / password
- `username master password admin@$` => `line c 0` => `password cisco` => `login local`
- `username user01 password router@$` => `line vty 0 4` => `login local` => `telnet (ip)`
- `enable password router@$`
- to encrypto all passwords so that it doesn't show on running-config
  - `service password-encryption

### banner / motd
- `banner mord` or `motd banner` => `exit` out all the way to check the message of the day or banner


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
- debug ppp (options)	: debug on for ppp
- debug ip rip		: uses broadcast
- un all		: undebug all
- sh ip protocol	: shows the protocol information
- sh clock
- sh ip access		: shows permits and access list 
- sh ip nat tr		: shows nat translation
- sh ip prefix		
- sh route-map 
