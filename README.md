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
  - R3) `route-map STATIC` / automatically adds 10 at the end => `match ip address prefix NET1` => `set metric 3` => `route-m STATIC 20` => `match ip add pre NET2` => `set metric 2` => `route-m CONNECTED` => `match interface s1/0.123 lo0` => `set metric 1` => `ip prefix-list NET1 permit 16.16.1.0/24` => `ip pre NET2 permit 16.16.2.0/24`
- redistribute
  - R3) `router rip` => `redistribute static route STATIC` => `redistribute connected route CONNECTED`
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
  - SW1) `ip dhcp pool NET10` => `network 1.1.10.0 /24` => `default-router 1.1.10.254` => `dns-server 1.1.10.100 168.126.63.1` => `lease 7` => `class 10` => `address range 1.1.10.1 1.1.10.253` => `exit` => ip dhcp exclude-address 1.1.10.254`
  - SW1) `ip dhcp pool NET20` => `network 1.1.20.0 /24` => `default-router 1.1.20.254` => `dns-server 1.1.10.100 168.126.63.1` => `l 7`
  - SW1) `ip dhcp pool NET10` => `class 10` => `address range 1.1.10.1 1.1.10.253`
  - SW1) `ip dhcp pool NET20` => `class 20` => `address range 1.1.20.1 1.1.20.253`
  - SW1) `int vlan 10` => `ip dhcp client class-id 10`
  - SW1) `int s2/0` => `ip dhcp client class-id 20`
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
- `ip nat inside source list 10 int f0/1 overload`
- `int s1/0` => `ip nat inside`
- `int f0/1` => `ip nat out`
- `ping 10.0.0.1 source 1.1.4.4`

### domain
- if `no ip domain look` => change it to `ip domain look`
  - but if asked with unknown IPs/addresses, it will loop
- `ip name-server 8.8.8.8`
- `ping www.google.com`

### username / password
- `username master password admin@$` => `line c 0` => `password cisco` => `login local`
- `username user01 password router@$` => `line vty 0 4` => `login local` => `telnet (ip)`
- `enable password router@$`
- to encrypto all passwords so that it doesn't show on running-config
  - `service password-encryption

### banner / motd
- `banner mord` or `motd banner` => `exit` out all the way to check the message of the day or banner

### stp - spanning-tree protocol
- select root switch => select root port on other switches => select one port per one segment => other ports without any roles become alternate ports and be logically blocked
- 3 switches w/ vlan 10 => check spanning-tree vlan 10 brief
- block 20s => listen 15s => learn 15s => forward
- if root or designated port, it becomes listening status immediately
- between SWs
  - `spanning uplink` => blocked link is unblocked immediately, which is 30s
  - `spanning backbonefast` on all switches => max age is gone, which is 20s
  - `int f1/1` => `spanning portfast` => looping may occur
- `spanning-tree portfast` => saves 30s for ports for end devices
- SW1) `sp vlan 10 priority 0`
- SW2) `sp vlan 10 priority 4092`
- SW3) `sp vlan 10 priority 8192` => makes the neighboring switch's port to be blocked
- `sh sp vlan 10 br` => shows the spanning-tree information
- SW4) `int f1/2` => `span vlan 10 cost 5` => makes the final cost to be lower

### vtp
- SW1) `vtp domain VTP22` => `vtp mode server
- Sw2-4) `vtp mode client`
- SW1) vlan setting => trunking mode
- SW2-4) trunking setting => access mode for SW4
- SW4) `spanning-tree portfast` on access mode ports => `span port fast bpduguard`
- if vlan doesn't update, update any vlan then erase
- SW1) `span vlan 10 root primary diameter 4` => I am root out of 4 switches => default is 7 switch based config, so in this case, the timings are shortened around aging, etc.
- SW3) `span vlan 10 root second diameter 4` => becomes the number 2 for vlan 10
- SW2) `span vlan 20 root secondary dimater 4` => becomes the number 2 for vlan 20
- for new (but old) switches to be joined in an existing network, put this switch in transparent mode to make the revision information as 0
- if you want to change from 'transparent' to 'server' mode, the port must be shut down before doing so. Or else, the server will learn clients' vlan info. Then, 
  - SW1) `vtp m t` => revision becomes 0 => `int f1/1` => `sh` => `vtp m s` => increase revision num => if it's higher than clients' num => make vlan config => `no sh` on the interface

### vtp + ripv2
- SW1-4) vtp transparent mode
- SW1-3) vlan setting => access mode
- SW1-3) config ip address => next hop ping test
- SW4) vlan setting => access mode
- PC1-2) `ip default-gateway (gateway)` => must exist for end devices
- SW1) router rip => passive int on lo0
- SW2,3) router rip => `neighbor (ip address)` ip address of destination so that it doesn't go to end devices => to make it unicast. if not, it sends multicast due to switch in the middle

### HSRP - hot standby routing protocol
- SW3) `int vlan 234` => `standby 234 ip 1.1.234.100` => `st 234 priority 110` default value is 100 => `st 234 preempt delay minimum 30` how long to make the change effective since it cannot run before rip sends out new information => `st 234 track f1/3 20` subtract 20 to make it lower than 100. higher number is more prior number
- SW2) `int vlan 234` => `st 234 ip 1.1.234.100` => `st 234 preempt`
- SW2,3) `show standby` to check standby configs
- PC1-2) `ip default-g 1.1.234.100` => ping to the vGW => `clear arp` => `clear mac` to erase cached info => `traceroute 1.1.100.100`

### mhsrp - multiple hot standby routing protocol
- SW1-4) `vtp m t` => config vlan => `sp portf` for access ports => `sp portf bpduguard` on config mode
- SW1-3) `int f1/1` => `no sw` to change L2 port to L3 port => config ip address
- SW2,3) config int vlan 10,20 and ip address
- SW1-3) next hop ping tests
- SW1-3) config routing => include neighbor config for SW2,3
- SW3) `int vlan 10` => `st 10 ip 1.1.10.100` => `st 10 pri 100` => `st 10 pre d m 30` => `st 10 tr f1/3 20` => `int vlan 20` => `st 20 ip 1.1.20.100` => `st 20 pre`
- SW2) opposite config

### vrrp
- sw4) `vtp m s`
- sw2,3) `vtp m c`
- sw4) vlan setting => `sp portf` for reducing 30s forward delay => `sp portf b` for no bpdu on portfast interfaces => `int range f1/2 , f1/4` => trunking
- sw2,3) trunking => check if vlan info has been received => if not, increase revision num for sw4
- sw1-3) ip add for `sw m a` and `no sw` and svi => next hop test => `router rip` => include `nei (destination router ip add)` => `pass default` => `no pass vlan 10` and `20` and port to SW1
- SW3) `track 1 int f1/3 line-protocol` => `int vl 10` => `vrrp 10 track 1 decrement 20 `=> `vrrp 10 pri 110` => `vrrp 10 pre d m 30` =>

### etherchannel
- dsw,asw) vtp setting => vlan setting
- asw1,2) `sw m a` => `sp portf` => `sp portf b`
- dsw,asw) use `cdp` to check what are connected => trunking => `channel-group 5 mode on`
- bb,dsw,asw) ip add + vGW
- bb,dsw) ip routing + router rip
- dsw) mhsrp
- pc) `ip default-g (vGW add)` => `ip add (ip add)`

- CentOS server dhcp
  - `rpm -qa + grep dhcp` => check firewall => turn off dhcp firewall => `ps -ef | grep dnsmasq` to kill dns => `systemctl disable dnsmasq` to disable when rebooting => `vim /etc/dhcp/dhcpd.conf`

```bash
subnet 192.168.50.0 netmask 255.255.255.0 {
}

subnet 192.168.10.64 netmask 255.255.255.224 {
        option routers 192.168.10.92;
        option domain-name "kedu.edu";
        option domain-name-servers 192.168.50.101;
        range dynamic-bootp 192.168.10.65 192.168.10.91;
        default-lease-time 10000;
        max-lease-time 50000;
}
```
  - `systemctl start dhcpd.service`
- dsw1,2) vlan interfaces => `ip help (dhcp server ip add)`
- linux
  - `vim /etc/resolv.conf` => add nameserver ip => `route` to check kernel ip routing table

### 180v
- SW1-4
  - vtp => vlan => trunking => check vlans => check trunk => check ether summ
  - L3 `no sw` => ip add
- R1-3
  - fram relay global => clock rate
  - ip add for SW facing ports
  - fram relay per router

### eigrp + rip
- fram map => ping test
  - R1) R1->R2-3
  - R2) R2->R1,R3
  - R3) R3->R1-2,R4 
  - R4) R4->R3
- eigrp => needs to have neighbor R in the eigrp topology
  - R1) `net 25.25.1.1 0.0.0.0` == 25.25.1.0 0.0.0.255 => `net 25.25.123.1 0.0.0.0`
  - R2) `net 25.25.2.2 0.0.0.0` => `net 25.25.123.2 0.0.0.0` => `int s1/0.234` => `no ip sp eigrp 25`
  - R3) `net 25.25.3.3 0.0.0.0` => `net 25.25.123.3 0.0.0.0`
- rip
  - R3) `net 150.1.0.0` => no need for passive since diff subnet
  - R4) rip config
- connecting different networks
  - R3) to bring rip info to eigrp network `router ei 25` => redistribute rip metric bandwidth delay reliability load mtu `r rip m 1544 2000 255 1 1500`
  - R3) `ip prefix-list HOP1 permit 25.25.123.0/24` => `ip prefix-list HOP2 permit 25.25.2.0/24` => `ip pre HOP3 permit 25.25.1.0/24` => `route-map EIGRP_NET 10` => `match ip add prefix HOP1` => `set metric 1` => `route-map EIGRP_NET 20` => `match ip add prefix HOP2` => `set metric 2` => `route-map EIGRP_NET 30` => `match ip add prefix HOP3` => `set metric 3` => `show route-map`=> `router rip` => `redistribute digrp 25 route-map EIGRP_NET` => `sh ip route` D EX 170 are externals 
- R3) `int s1/0.34` => `ip summary-address rip 25.25.0.0 255.255.0.0` => R4) `clear ip ro\*` => `sho ip route` to check if routing table has been updated with abstracted addresses
- R3) `router ei 25` => `distribute-list prefix OVERWRITE out rip` = `ip prefix-list OVERWRITE deny 25.25.0.0/16` => `ip prefix-list OVERWRITE permit 0.0.0.0/0 le 32`

### Debugging
- (serial interface) cdp run => int s1/0 => cdp en
- (in case of mis config of int) => no... => no ip add => sh => exit => no int s1/0.23 => int s1/0.233
- `clear arp` `clear mac`
- for multipoint and main interface cannot ping selves => `fram map ip 14.14.11.1 102` this is to self ping test to check if ports are on or off

#### show
- sh ip int br
- sh fram map
- sh run int s1/0
- sh int tr
- sh ip protocol	: shows the protocol information
- sh clock
- sh ip access		: shows permits and access list 
- sh ip nat tr		: shows nat translation
- sh ip prefix		: shows prefix-list per hops
- sh route-map
- `sh spanning vlan 10 brief`
- sh arp		: show mac address table
- sh fram map
- sh fram pvc | in DLCI
- sh vlan-s		: show vlan-switch
- sh int trunk		: show interface trunk
- sh cdp n		: sh cdp (cisco discovery protocol) neighbor
- sh ip route		: show ip route
- sh controller s1/0	: check sex of interface ports
- sh fram lmi 		: CCITT => q933a
- sh ip nat trans	: show ip nat info
- sh run | begin nat	: show ip nat config
- sh vtp status`
- sh standby		: show standby vGW
- `sh ether summary` 	: show ether channel summary
- `sh route-map`
- `sh ip rip database`
- `sh ip protocol`
- `sh ip eigrp topology summary`

#### debug
- debug arp 		: ARP packet debug on <=> no ...
- deb ip pack 		: debugging on 
- debug ppp (options)	: debug on for ppp
- debug ip rip		: uses broadcast
- un all		: undebug all
- `deb sp event` => kill interface => watch event logs
- `deb sp bpdu`
- `deb ip dhcp server events` => `deb dhcp`
