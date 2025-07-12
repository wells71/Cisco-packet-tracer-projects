# Types of modes
1. User exec mode - default mode: Switch>
2. Priviledged exec mode - enabled by 'en' command in user exec mode: Switch#
3. Configuration mode - enabled by 'conf t' command in priviledged exec mode: Switch(config)

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

        exec timeout [num] [number] - lock user out after num minutes and number seconds of inactivity

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

## 
- do wr - save running-config to startup-config