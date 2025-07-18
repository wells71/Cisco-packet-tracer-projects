# Types of CLI modes
1. User exec mode - default mode:                                               Switch>
2. Priviledged exec mode - enabled by 'en' command in user exec mode:           Switch#
3. Configuration mode - enabled by 'conf t' command in priviledged exec mode:   Switch(config)

# 1. USER EXEC MODE

en - enable
ping
tracert etc

# 2. PRIVILEDGED EXEC MODE 

conf t - configuration

# 3. CONFIGURATION MODE

- hostname [name] - change hostname

- line console 0 - configure settings for console connections

        password [entry] - set password to entry

        login - tell device to prompt for password

        exec-timeout [num] [number] - lock user out after num minutes and number seconds of inactivity

- banner motd #message#

- no ip domain-lookup - disable ip domain lookup

- service password-encryption - encrypt all stored passwords

## Setting up SSH
1. Setup a username and password
    username [name] password [pass]

2. Setup domain name
    ip domain-name cisco.com

3. Generate keys
    crypto key generate rsa general-keys modulus 1024

## Set up Access list
- access-list 2 permit 192.168.10.0 0.0.0.255

- access-list 2 deny any

To clear an acl:
    - no access-list 2

View ACLs
    - do show access-lists

## Set up virtual (remote) CLI access
- line vty 0 15 (all 16 vty lines)

- login local (use local username and password - local to switch)

- transport input ssh (set remote access mode to ssh)

- access-class 2 in - limits incoming connections to rules in acl 2

## Save configs
- do wr - save running-config to startup-config

## Setup VLANS
- vlan 20
- name Management

## Setup Trunks
- int range g1/0/2-6
- switchport mode trunk

## Setup access ports
- int g1/0/10
- switchport access vlan [vlanNumber]

## Setup STP & BPDU guard
int range fa0/3-24 - access ports
spanning-tree portfast
spanning-tree bpduguard enable

do wr

## Setup EtherChannel on Multilayer switches
int range g1/0/21-23
channel-group 1 mode active/passive
exit

int Port-channel 1
switchport mode trunk
do write

## Multilayer Switch configuration
- ip routing
- on desired port:
    - no switchport
    - assign ip addresses

## Firewalls
- consist of 3 kinda ports - dmz, inside net, outside network
- for inside:
    - nameif INSIDE0, 1, 2 etc
    - security-level 100 (fully trust)

# HSRP & DHCP Helper (MLSW)
- int vlan 10

- ip addr 192.168.10.3 255.255.255.0
- ip helper-address 10.20.20.5 (dhcp server)
- ip helper-address 10.20.20.6
- standby 10 ip 192.168.10.1

## OSPF
- route ospf
- router-id 2.1.2.1
    - network 10.20.20.32 0.0.0.3 area 0
    - network 192.168.10.0 0.0.0.255 area 0
    - network 172.16.0.0 0.0.255.255 area 0
    - network 10.10.0.0 0.0.255.255 area 0

# firewall static routes to isp
- route OUTSIDE 0.0.0.0 0.0.0.0 105.100.50.1

# NAT
create object networks. each obj net represents a subnet
- object network INSIDE1-OUTSIDE (name it)
- subnet 192.168.10.0 255.255.255.0 (define subnet)
- nat (INSIDE1,OUTSIDE) dynamic interface (bind to NAT)

now, assuming MLSW1 fails, we accomodate a route from the same 192.168.10.0 network:
- object network INSIDE1a-OUTSIDE
- subnet 192.168.10.0 255.255.255.0
- nat (INSIDE2,OUTSIDE) dynamic interface

# ipsec

crypto ikev1 policy 10
encryption 3des
hash sha

authentication pre-share

group 2
lifetime 86400
exit

tunnel-group 105.100.50.2 type ipsec-l2l
tunnel-group 105.100.50.2 ipsec-attributes
ikev1 pre-shared-key cisco

crypto ipsec ikev1 transform-set TSET esp-3des esp-sha-hmac

access-list VPN-ACL extended permit ip 172.17.0.0 255.255.0.0 192.168.10.0 255.255.255.0
access-list VPN-ACL extended permit ip 10.11.0.0 255.255.0.0 192.168.10.0 255.255.255.0

access-list VPN-ACL extended permit ip 172.17.0.0 255.255.0.0 172.16.0.0 255.255.0.0
access-list VPN-ACL extended permit ip 10.11.0.0 255.255.0.0 172.16.0.0 255.255.0.0

access-list VPN-ACL extended permit ip 172.17.0.0 255.255.0.0 10.10.0.0 255.255.0.0
access-list VPN-ACL extended permit ip 10.11.0.0 255.255.0.0 10.10.0.0 255.255.0.0

access-list VPN-ACL extended permit ip 172.17.0.0 255.255.0.0 10.20.20.0 255.255.255.224
access-list VPN-ACL extended permit ip 10.11.0.0 255.255.0.0 10.20.20.0 255.255.255.224

crypto map CMAP 10 set peer 105.100.50.2
crypto map CMAP 10 set ikev1 transform-set TSET
crypto map CMAP 10 match address VPN-ACL

crypto map CMAP interface OUTSIDE
crypto ikev1 enable OUTSIDE

wr mem