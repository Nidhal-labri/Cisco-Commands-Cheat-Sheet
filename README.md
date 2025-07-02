# Cisco Commands Cheat Sheet

A curated list of essential Cisco IOS commands for routers and switches. This cheat sheet is designed for students, network engineers, and anyone preparing for Cisco certifications like CCNA.

---

## 🔐 Modes Overview (Router/Switch)

<img alt="image_2025-07-02_16-58-45" src="https://github.com/user-attachments/assets/23564e4e-aaa8-44bd-af16-8a229dc7719e" />



- **User EXEC Mode**: Limited command access, used for basic checks.
  - Prompt: `>`
- **Privileged EXEC Mode**: Access to all device commands.
  - Entered with: `enable`
  - Prompt: `#`
- **Global Configuration Mode**: Used to make configuration changes.
  - Entered with: `configure terminal`

---

## 📌 Common Router & Switch Basic Commands

```bash
enable
configure terminal
no ip domain-lookup              # Disable DNS resolution for mistyped commands
hostname S1/R1    
banner motd $ Authorized Access Only! $
exit
```

---

## 🔒 Basic Security

```bash
enable
enable secret class
line console 0
 password cisco
 exec-timeout 5                  # Auto-logout after 5 minutes
 login
exit
line vty 0 4                     # Set VTY (remote access) password
 password cisco
 exec-timeout 5
 login
exit
service password-encryption
end
```

---

## ⚙️ Configuration Management

```bash
copy running-config startup-config   # Save config
show running-config                 # Active config
show startup-config                 # Saved config
erase startup-config                # Wipe saved config
reload                              # Reboot device
```

---

## 🌐 Interface Configuration

```bash
enable
configure terminal
interface fa0/1
ip address 192.168.1.1 255.255.255.0
no shutdown
exit
```

> ⚠️ *Switches typically assign IPs to VLAN interfaces, not physical ports.*

---

## 🔐 SSH Configuration

```bash
enable
configure terminal
hostname S1/R1
ip domain-name nidhal.com
username admin secret nidhal123     # Create local user
crypto key generate rsa
 1024                               # Key size (in bits) can be 1024 or 2048
ip ssh version 2                    # Use SSH version 2
ip ssh authentication-retries 3     # Optional: retry attempts
ip ssh time-out 60                  # Optional: SSH timeout
line vty 0 15
 transport input ssh                # Allow only SSH
 login local                        # Authenticate using local user
 exec-timeout 5 0                   # Auto-logout after 5 min
 privilege level 15                 # Full admin access
exit
logging monitor debugging           # Optional: enable session logging
end
```

### ✅ SSH Verification

```bash
show ip ssh                         # SSH config and version
show ssh                            # Active SSH sessions
show run | include ssh              # SSH-related config
show users                          # Connected users
debug ip ssh                        # Debug SSH (use with care)
```

---

## 📦 DHCP Configuration

```bash
enable
configure terminal
ip dhcp excluded-address 192.168.1.1 192.168.1.10   # Reserved addresses
ip dhcp excluded-address 192.168.1.254              # Reserve single IP
ip dhcp pool LAN-POOL
 network 192.168.1.0 255.255.255.0
 default-router 192.168.1.1
 dns-server 8.8.8.8 1.1.1.1
 domain-name nidhal.com
 lease 7                                            # Lease in days
exit
```

### DHCP Relay

```bash
interface vlan 10
 ip address 192.168.10.1 255.255.255.0
 ip helper-address 192.168.1.1
 no shutdown
exit
```

### ✅ DHCP Verification & Cleanup

```bash
show running-config | include dhcp
show ip dhcp binding               # DHCP leases
show ip dhcp pool                  # DHCP pool stats
debug ip dhcp server events       # Real-time DHCP logs
show ip dhcp server statistics     # DHCP packet stats
clear ip dhcp binding *            # Clear leases
clear ip dhcp conflict *           # Clear conflicts
```

---

## 🔀 VLAN Configuration (Switches)

```bash
configure terminal
vlan 10
 name Sales
exit
vlan 20
 name HR
exit
vlan 99
 name Management
exit

no vlan 20                         # Delete VLAN 20
```

### VLAN Port Assignments

```bash
interface FastEthernet0/1
 switchport mode access            # Access mode
 switchport access vlan 10         # Assign VLAN 10
exit

interface FastEthernet0/2
 switchport mode access
 switchport access vlan 20
 switchport voice vlan 30          # Optional for IP phones
exit
```

### Trunking

```bash
interface FastEthernet0/24
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,99
 switchport trunk native vlan 99
exit
```

### Management Interface (SVI)

```bash
interface vlan 99
 ip address 192.168.99.1 255.255.255.0
 no shutdown
exit
ip default-gateway 192.168.99.254   # Gateway for management access
end
```

### Inter-VLAN Routing (Router-on-a-Stick)

```bash
interface GigabitEthernet0/0/1.10
 encapsulation dot1Q 10             # Tag VLAN 10
 ip address 192.168.10.1 255.255.255.0
exit
interface GigabitEthernet0/0/1.20
 encapsulation dot1Q 20
 ip address 192.168.20.1 255.255.255.0
exit
interface GigabitEthernet0/0/1.99
 encapsulation dot1Q 99
 ip address 192.168.99.1 255.255.255.0
exit
interface GigabitEthernet0/0/1
 no shutdown
exit
```

### ✅ VLAN Verification

```bash
show vlan brief                    # VLANs and port mappings
show interfaces trunk              # Trunk info
show interfaces switchport         # Port mode and VLANs
show mac address-table vlan 10     # MACs in VLAN 10
show ip interface brief            # IP status for all interfaces
```

---

## 🚫 Access Control Lists (ACL)

### Syntax Overview

#### Standard Numbered ACL (1–99, 1300–1999)

```
access-list <1-99> {permit | deny} <source> <wildcard-mask>
```

#### Extended Numbered ACL (100–199, 2000–2699)

```
access-list <100-199> {permit | deny} <protocol> <source> <wildcard> [operator port] <destination> <wildcard> [operator port] [log]
```

#### Named ACL

```
ip access-list [standard | extended] NAME
 [permit | deny] ...
```

### Examples

#### 1. Standard Numbered ACL

```bash
access-list 10 permit 192.168.1.0 0.0.0.255   # Allow subnet
access-list 10 deny any                      # Block others
```

#### 2. Extended Numbered ACL

```bash
access-list 199 permit tcp 192.168.1.0 0.0.0.255 gt 1024 host 10.0.0.5 eq 22 established log   # Allow SSH to host
```

#### 3. Named ACLs

```bash
ip access-list standard BLOCK_INTERNET
 permit 192.168.10.0 0.0.0.255
 deny any
```

### 4. Applying ACLs

```bash
interface FastEthernet0/0
 ip access-group 101 in
exit

access-list 23 permit 192.168.1.0 0.0.0.255
line vty 0 4
 access-class 23 in               # Restrict VTY login
 login
```

### 5. Removing ACLs

```bash
no access-list 101
no ip access-group 101 in
```

### ✅ ACL Verification

```bash
show access-lists
show ip access-lists
show ip interface <int>          # See applied ACLs
debug ip packet                  # Packet match logs (careful)
```

> ⚠️ ACLs are processed top-down. First match wins. One ACL per direction/interface/protocol.
> Best practice: Standard ACLs should be applied closest to the destination, extended ACLs closest to the source.

---

## 🌍 NAT Configuration

### 1. Static NAT

```bash
ip nat inside source static 192.168.1.10 203.0.113.10
interface GigabitEthernet0/0
 ip nat inside
exit
interface GigabitEthernet0/1
 ip nat outside
exit
```

### 2. Dynamic NAT

```bash
ip nat pool PUBLIC_POOL 203.0.113.10 203.0.113.20 netmask 255.255.255.0
access-list 1 permit 192.168.1.0 0.0.0.255
ip nat inside source list 1 pool PUBLIC_POOL
interface GigabitEthernet0/0
 ip nat inside
exit
interface GigabitEthernet0/1
 ip nat outside
exit
```

### 3. NAT Overload (PAT)

```bash
access-list 1 permit 192.168.1.0 0.0.0.255
ip nat inside source list 1 interface GigabitEthernet0/1 overload
interface GigabitEthernet0/0
 ip nat inside
exit
interface GigabitEthernet0/1
 ip nat outside
exit
```

### ✅ NAT Verification

```bash
show ip nat translations          # NAT table
show ip nat statistics            # NAT stats
debug ip nat                      # Debug NAT (careful)
clear ip nat translation *        # Reset NAT table
```

---

## 📎 License

This cheat sheet is released under the MIT License. Feel free to copy, modify, and share with attribution.

---

## 🧠 Contributions

Contributions, corrections, and improvements are welcome. Open a pull request or create an issue!

---

**Made with 💻 by Nidhal**

