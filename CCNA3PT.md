

# 711 Networking Practical test

- [711 Networking Practical test](#711-networking-practical-test)
  - [Basic Settings for Router](#basic-settings-for-router)
  - [Basic Settings for Switch](#basic-settings-for-switch)
  - [VLAN](#vlan)
    - [Show commands](#show-commands)
    - [Router on stick](#router-on-stick)
  - [OSPFv2](#ospfv2)
  - [EIGRP](#eigrp)
    - [EIGRP for IPv4](#eigrp-for-ipv4)
    - [EIGRP for IPv6](#eigrp-for-ipv6)
  - [ACL](#acl)
    - [Standard ACL](#standard-acl)
      - [Standard numbered Access-List](#standard-numbered-access-list)
      - [Named Access-List](#named-access-list)
      - [show commands for ACL](#show-commands-for-acl)
    - [Extended ACL](#extended-acl)
  - [NAT](#nat)
    - [Configuring static NAT](#configuring-static-nat)
    - [Configuring dynamic NAT](#configuring-dynamic-nat)
  - [Static/Default route](#staticdefault-route)
  - [SSH Config](#ssh-config)
  - [Port Security Config](#port-security-config)
  - [PPP Auth](#ppp-auth)
  - [BGP](#bgp)
  - [DHCP Config](#dhcp-config)
    - [IPv4 DHCP Config](#ipv4-dhcp-config)
    - [IPv6 DHCP Config](#ipv6-dhcp-config)

---

## Basic Settings for Router
```
router(config)# hostname R1
R1(config)# no ip domain lookup
R1(config)# enable secret class
R1(config)# line console 0
R1(config-line)# password cisco
R1(config-line)# login
R1(config)# login synchronous
R1(config)# line vty 0 4
R1(config-line)# password cisco
R1(config-line)# login
R1(config)# service password-encryption
R1(config)# banner motd $ Authorized Users Only! $
R1# copy running-config startup-config
```

## Basic Settings for Switch
```
switch(config)# hostname S1
S1(config)# no ip domain lookup
S1(config)# enable secret class
S1(config)# line console 0
S1(config-line)# password cisco
S1(config-line)# login
S1(config)# line vty 0 15
S1(config-line)# password cisco
S1(config-line)# login
S1(config)# service password-encryption
S1(config)# banner motd $ Authorized Users Only! $
S1# copy running-config startup-config
```
---

## VLAN

Creating vlan
```
S2(config)#vlan 10
S2(config-vlan)#name Faculty/Staff
```
Assigning vlan to int
```
S2(config)# interface f0/11
S2(config-if)# switchport mode access
S2(config-if)# switchport access vlan 10
```
Assinging voice vlan to int
```
S3(config)# interface f0/11
S3(config-if)# mls qos trust cos
S3(config-if)# switchport voice vlan 150
```

### Show commands 
```
S2# show vlan brief
```

### Router on stick

on router
```
R1(config)#interface g0/0
R1(config-if)#no shutdown
R1(config)# int g0/0.10
R1(config-subif)# encapsulation dot1Q 10
R1(config-subif)# ip address 172.17.10.1 255.255.255.0
R(config)# [no] ip classless
```

 on switch
```
S1(config)#int g0/1
S1(config-if)#switchport mode trunk
```

on trunk port in switch
```
S1(config)#int g0/1
S1(config-if)#switchport mode trunk
```

 to change **native vlan**
 ```
 Switch(config-if)# switchport trunk native vlan 10
 ```

  to specify what **vlans allowed**
 ```
 Switch(config-if)# switchport trunk allowed vlan 10,20,30
 ```

---

## OSPFv2

Configuring **OSPF**
```
R1(config)# router ospf 56
R1(config-router)# router-id 1.1.1.1
R1(config-router)# network 10.53.0.0 0.0.0.3 area 0
```

Configure the interface G0/0/1 OSPF **priority to 50 to ensure R1 is the *Designated Router.***
```
R1(config)# interface g0/0/1
R1(config-if)# ip ospf priority 50
```

Configure the **OSPF timers** on the G0/0/1 of each router for a ***hello timer of 30 seconds***
```
R1(config)# interface g0/0/1
R1(config-if)# ip ospf hello-interval 30
```


OSPF to treat R2 Loopback 1 like a **point-to-point** network
```
R2(config)# interface loopback 1
R2(config-if)# ip ospf network point-to-point
```

Configure **passive interface** on OSPF.
```
R2(config)# router ospf 56
R2(config-router)# passive-interface loopback 1
```

Configure OSPF to **propagate the default route** in OSPF routing updates.
```
R2(config)# router ospf 1
R2(config-router)# default-information originate
```

Change the **reference *bandwidth* on each router to 1Gbs** (1Gbs = 1000Mbs) & Have to change on both routers to be *consistant*
```
R1(config)# router ospf 56
R1(config-router)# auto-cost reference-bandwidth 1000
```

Following commands help **verifying** OSPF configurations
```
show ip interface brief
show ip route
show ip route ospf
show ip ospf neighbor
show ip protocols
show ip ospf
show ip ospf interface g0/0/1
```

---

## EIGRP 

### EIGRP for IPv4

Configuring *EIGRP on routers* 
> **NOTE** Need to be configured on all routers
```
R1(config)# router eigrp 1
R1(config-router)# eigrp router-id 1.1.1.1
R1(config-router)# network 172.16.1.0 0.0.0.3

```

Configuring *passive interfaces*
```
R1(config)# router eigrp 1
R1(config-router)# passive-interface g0/0
```

Disabling *auto summary*
```
R1(config)# router eigrp 1
R1(config-router)# no auto-summary
```

Configuring *bandwidth*
> **NOTE** need to be configured both ends of the connection.
```
R2(config)# interface s0/0/0
R2(config-if)# bandwidth 2000
```

### EIGRP for IPv6

Enable *ipv6* routing
```
R1(config)# ipv6 unicast-routing
```

Configuring *ipv6 addresses* on each interfaces in router
> Normally link local address automatically configured unless changed.
> Clock rate command is optional
```
int g0/0/0
ipv6 add 2001:sd8:sgfsfs:1::1/64
ipv6 add fe80::1 link-local
clock rate 64000
```

Enable *EIGRP for IPv6* routing
```
R1(config)# ipv6 router eigrp 1
R1(config-rtr)# no shutdown
R1(config-rtr)# eigrp router-id 1.1.1.1
```

Configuring EIGRP in each *interface*
```
R1(config)# int g0/0
R1(config-if)# ipv6 eigrp 
```

Configuring passive interfaces
```
R1(config)# ipv6 router eigrp 1
R1(config-rtr)# passive-interface g0/
```

---

## ACL
### Standard ACL
#### Standard numbered Access-List
Creating standard Access List 
```
R3(config)# access-list 1 remark Allow R1 LANs Access
R3(config)# access-list 1 permit 192.168.10.0 0.0.0.255
R3(config)# access-list 1 permit 192.168.20.0 0.0.0.255
R3(config)# access-list 1 deny an
```

Assigning access list to interfaces
```
R3(config)# interface g0/0/0
R3(config-if)# ip access-group 1 out
```

#### Named Access-List
Creating *named* access list
```
R1(config)# ip access-list standard BRANCH-OFFICE-POLICY
R1(config-std-nacl)# permit host 192.168.30.3
R1(config-std-nacl)# permit 192.168.40.0 0.0.0.255
R1(config-std-nacl)# end
```

Assigning named access list to interfaces
```
R1(config)# interface g0/0/0
R1(config-if)# ip access-group BRANCH-OFFICE-POLICY ou
```

#### show commands for ACL

To verify configurations
```markdown
R3# show access-lists 1
R3# show access-lists
R3# show ip access-lists
R3# show ip interface g0/0/0
R3# show ip interface
```

### Extended ACL
Creating Extended ACL
> Have to input `permit ip any any` at the end if want to permit other becasue in default there is `deny any` on ACL
```
RT1(config)# ip access-list extended acl-name
RT1(config-ext-nacl)# deny tcp host 172.31.1.101 host 64.101.255.254 eq 80
RT1(config-ext-nacl)# deny icmp host 172.31.1.103 host 64.103.255.254
RT1(config-ext-nacl)# permit ip any any
RT1(config)# interface g0/0
RT1(config-f)# ip access-group ACL in
```

---

## NAT
### Configuring static NAT
```
R(config)# ip nat inside source static 192.168.1.10 209.73.69.10
```

### Configuring dynamic NAT
```
R(config)# ip nat pool NAT-POOL 209.73.69.0 209.73.69.9 netmask 255.255.255.0
R(config)# ip access-list standard NAT-ACL
R(config-std-nacl)# permit 192.168.1.0 255.255.255.0
R(config-std-nacl)# deny any
R(config)# ip nat inside source list NAT-ACL pool NAT-POOL overload
```

---

## Static/Default route
assigning **default route**
> Can also use the ip address of the interface
```
R2(config)# ip route 0.0.0.0 0.0.0.0 Serial0/1/0

R(config)# ip route 192.168.20.0 255.255.255.0 serial0/1
```

---

## SSH Config
Description
```
R(config)# hostname {hostname}
R(config)# username {username} password {password}
R(config)# ip domain-name {domain-name}
R(config)# crypto key generate rsa
R(config)# ip ssh version 2
R(config)# line vty 0 15
R(config-line)# login local
R(config-line)# transport input ssh

```

Actual configuration
```
R(config)# hostname R1
R(config)# username Cisco password Password-123
R(config)# ip domain-name anything.com
R(config)# crypto key generate rsa
R(config)# ip ssh version 2
R(config)# line vty 0 15
R(config-line)# login local
R(config-line)# transport input ssh
```

---

## Port Security Config
Description
```
S(config)# interface {interface | range {interface}{interface-start}-{interface-end}}
S(config-if)# switchport mode access
S(config-if)# switchport port-security
S(config-if)# switchport port-security maximum {number}
S(config-if)# switchport port-security mac-address sticky
S(config-if)# switchport port-security violation {restrict | protect | shutdown}
S(config-if)# switchport protected
S(config-if)# spanning-tree bpduguard enable
S(config-if)# shutdown
S(config-if)# no shutdown
S# show port-security interface {interface}

```

Actual configuration
```
S(config)# interface interface range f0/1-24,g0/1-2
S(config-if)# switchport mode access
S(config-if)# switchport port-security
S(config-if)# switchport port-security maximum 25
S(config-if)# switchport port-security mac-address sticky
S(config-if)# switchport port-security violation shutdown
S(config-if)# switchport protected
S(config-if)# spanning-tree bpduguard enable
S(config-if)# shutdown
S(config-if)# no shutdown
S# show port-security interface f0/1
```

---

## PPP Auth

Desription
```
R(config)# username {pap-neighbor-username | chap-neighbor-hostname} password {neighbor-password}
R(config)# interface {interface-to-neighbor}
R(config-if)# encapsulation ppp
R(config-if)# ppp authentication {pap | chap | pap chap | chap pap}
R(config-if (pap))# ppp pap sent-username {your-username} password {your-password}
```

Actual configuration
```
R(config)# username R2 password R2Password
R(config)# interface S0/0/0
R(config-if)# encapsulation ppp
R(config-if)# ppp authentication pap
R(config-if)# ppp pap sent-username R1 password R1Password
```

---

## BGP
Desription
```
R(config)# router bgp {autonomous-system-number}
R(config-router)# neighbor {neighbor-ip-address} remote-as {neighbor-autonomous-system-number}
R(config-router)# network {internal-network} [mask {subnet-mask}]
```

Actual configuration
```
R(config)# router bgp 65000
R(config-router)# neighbor 209.165.201.1 remote-as 65001
R(config-router)# network 198.133.219.0 mask 255.255.255.0
```

---

## DHCP Config
### IPv4 DHCP Config
Desription
```
R1(config)# ip dhcp excluded {excluded-start-ip} {excluded-end-ip}
R1(config)# ip dhcp pool {pool-name}
R1(dhcp-config)# network {allowed-network-address} {subnet-mask}
R1(dhcp-config)# default-router {default-gateway}
R1(dhcp-config)# dns-server {dns-server-address (up to 4)}
R1(dhcp-config)# domain-name {domain-name}
R1(dhcp-config)# lease-time {days}

R3(config)# interface {interface}
R3(config-if)# ip helper-address {dhcp-server-address}

```

Actual configuration
```
R1# show ip dhcp binding

R1(config)# ip dhcp excluded 172.16.2.1 172.16.2.6
R1(config)# ip dhcp pool R1-DHCP
R1(dhcp-config)# network 172.16.2.0 255.255.255.128
R1(dhcp-config)# default-router 172.16.2.1
R1(dhcp-config)# dns-server 140.198.8.14
R1(dhcp-config)# domain-name anything.com
R1(dhcp-config)# lease-time 5

R3(config)# interface fastethernet 0/1
R3(config-if)# ip helper-address 192.168.15.2
```

---

### IPv6 DHCP Config
Desription
> Stateless Address Auto Config (SLAAC)
```
R(config)# ipv6 unicast routing
R(config)# ipv6 dhcp pool {pool-name}
R(dhcpv6-config)# dns-server {ipv6-dns-server-address}
R(dhcpv6-config)# domain-name {domain-name}
R(config)# interface {interface}
R(config-if)# ipv6 address {ip-address}{prefix-length}
R(config-if)# ipv6 dhcp server {pool-name}
R(config-if)# ipv6 nd other-config-flag
```
> Stateful Address Auto Config
```
R1(config)# ipv6 unicast routing
R1(config)# ipv6 dhcp pool {pool-name}
R1(dhcpv6-config)# address prefix {ipv6-prefix} [lifetime {preferred-lifetime} {valid-lifetime}]
R1(dhcpv6-config)# dns-server {ipv6-dns-server-address}
R1(dhcpv6-config)# domain-name {domain-name}
R1(config)# interface {interface}
R1(config-if)# ipv6 address {ipv6-address}
R1(config-if)# ipv6 dhcp server {pool-name}
R1(config-if)# ipv6 nd managed-config-flag

```

Actual configuration
```
R3(config)# interface fastethernet 0/1
R3(config-if)# ip dhcp relay destination 2001:A123:7CA1::15

R# show ipv6 dhcp pool
R# show ipv6 dhcp binding

R(config)# ipv6 unicast routing
R(config)# ipv6 dhcp pool LAN-10-STATELESS
R(dhcpv6-config)# dns-server 2001:345:ACAD:F::5
R(dhcpv6-config)# domain-name cisco.com

R(config)# interface g1/1
R(config-if)# ipv6 address 2001:A1B5:C13:10::1/64
R(config-if)# ipv6 dhcp server LAN-10-STATELESS
R(config-if)# ipv6 nd other-config-flag
```
---