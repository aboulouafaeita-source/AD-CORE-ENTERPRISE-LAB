 # AD + ASA + EVE-NG Lab

 ==========================================
# AD + ASA + EVE-NG Lab
 ==========================================
 
# Project type:
 Small enterprise-style lab built in EVE-NG

# Main components:
 - Cisco vIOS Router
 - Cisco ASA Firewall
 - SW1 / SW2
 - Windows Server 2022 Core
 - Windows client
 - Active Directory Domain Services
 - DNS

# Main objective:
 Build and document a small enterprise lab that provides:
 - Router and firewall connectivity
 - Internal LAN communication
 - Windows Server Core deployment
 - Active Directory domain controller
 - DNS resolution
 - Domain user creation
 - Domain join from a Windows client
   
 ==============================================
# TOPOLOGY
 ==============================================
### Designed topology
![Designed Topology](docs/screenshots/topology-diagram.png)

### EVE-NG implementation
![EVE-NG Topology](docs/screenshots/topology-eve-ng.png)
 
# Router -> ASA -> SW1 -> SW2 -> Clients
                    |
                     -> Windows Server Core

# Screenshots:
 - docs/screenshots/topology-diagram.png
 - docs/screenshots/topology-eve-ng.png

 ===============================================
# IP PLAN
# =========================================================
# WAN Segment
 - R1 GigabitEthernet0/0  = 10.0.0.1/30
 - ASA Ethernet0 outside  = 10.0.0.2/30

# LAN Segment
 - ASA Ethernet1 inside   = 192.168.10.1/24
 - SRV-DC1                = 192.168.10.10/24
 - Windows Client         = 192.168.10.20/24

# DNS
 - Preferred DNS for all domain members = 192.168.10.10

# Domain
 - FQDN    = lab.local
 - NetBIOS = LAB

 ============================================
# ROUTER CONFIGURATION
 ============================================
enable
configure terminal
hostname R1

interface GigabitEthernet0/0
 description TO-ASA-OUTSIDE
 ip address 10.0.0.1 255.255.255.252
 no shutdown

end
write memory

# Validation
show ip interface brief
ping 10.0.0.2

# Router config file:
 - configs/router/r1-base-config.txt

 =========================================================
# ASA CONFIGURATION
 =========================================================
enable
configure terminal
hostname FW1

interface Ethernet0
 nameif outside
 security-level 0
 ip address 10.0.0.2 255.255.255.252
 no shutdown

interface Ethernet1
 nameif inside
 security-level 100
 ip address 192.168.10.1 255.255.255.0
 no shutdown

route outside 0.0.0.0 0.0.0.0 10.0.0.1

object network LAN
 subnet 192.168.10.0 255.255.255.0
 nat (inside,outside) dynamic interface

write memory

# Validation
show interface ip brief
ping 10.0.0.1
ping 192.168.10.1

# ASA config file:
 - configs/asa/asa-base-config.txt
#
# Connectivity validation screenshot:
 - docs/screenshots/asa-connectivity.png

 ===========================================
# WINDOWS SERVER CORE CONFIGURATION
 ===========================================
# Hostname
 - SRV-DC1

# Network settings
 - IP address      = 192.168.10.10
 - Subnet mask     = 255.255.255.0
 - Default gateway = 192.168.10.1
 - DNS server      = 192.168.10.10

ipconfig /all
Get-NetAdapter
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 192.168.10.10
ping 192.168.10.1
ping 192.168.10.10

# Install AD DS role
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools

# Promote server to Domain Controller
Install-ADDSForest -DomainName "lab.local"

# Verify domain installation
Get-ADDomain

# Validation screenshot:
 - docs/screenshots/server-core-domain.png

# Windows Server commands file:
 - configs/windows-server/powershell-ad-commands.md

 =====================================
 # ACTIVE DIRECTORY USER CREATION
 =====================================
 
# Create user
New-ADUser -Name "user1" -SamAccountName user1 -AccountPassword (ConvertTo-SecureString "User@123Aa!" -AsPlainText -Force) -Enabled $true

# Verify user
Get-ADUser user1
Get-ADUser user1 -Properties Enabled

# If needed, reset password
Set-ADAccountPassword -Identity user1 -Reset -NewPassword (ConvertTo-SecureString "User@123Aa!" -AsPlainText -Force)

# Enable user account
Enable-ADAccount -Identity user1

# Verify enabled status
Get-ADUser user1 -Properties Enabled

# Validation screenshot:
 - docs/screenshots/user1-enabled.png

 ========================================
# WINDOWS CLIENT CONFIGURATION
 =========================================
 
# Client network settings
 - IP address      = 192.168.10.20
 - Subnet mask     = 255.255.255.0
 - Default gateway = 192.168.10.1
 - DNS server      = 192.168.10.10

# Validation on client
ping 192.168.10.10
nslookup lab.local
ping srv-dc1.lab.local

# DNS validation screenshot:
 - docs/screenshots/dns-resolution.png

 ======================================
# DOMAIN JOIN
 =====================================
 
# On the Windows client:
 - Open System Properties
 - Go to Computer Name
 - Select Change
 - Choose Domain
 - Enter: lab.local

# Use domain admin credentials:
 - LAB\Administrator

# Expected result:
 - Welcome to the lab.local domain.

 Restart the client after successful domain join.

# Domain login validation:
whoami
echo %userdomain%
echo %logonserver%

# Validation screenshot:
 - docs/screenshots/domain-login-validation.png

 ===========================================
# SCREENSHOTS
 ===========================================
 
# This repository includes the following screenshots:
 - docs/screenshots/topology-diagram.png
 - docs/screenshots/topology-eve-ng.png
 - docs/screenshots/asa-connectivity.png
 - docs/screenshots/server-core-domain.png
 - docs/screenshots/user1-enabled.png
 - docs/screenshots/domain-login-validation.png
 - docs/screenshots/dns-resolution.png
#
# Screenshots folder README:
 - docs/screenshots/README.md

 ===========================================
# PROJECT STRUCTURE
 ==========================================
 
# AD-CORE-ENTERPRISE-LAB/
 ├── README.md
 ├── LICENSE
 ├── .gitignore
 ├── configs/
 │   ├── asa/
 │   │   └── asa-base-config.txt
 │   ├── router/
 │   │   └── r1-base-config.txt
 │   └── windows-server/
 │       └── powershell-ad-commands.md
 └── docs/
     ├── ip-plan.md
     ├── implementation-steps.md
     └── screenshots/
         ├── README.md
         ├── topology-diagram.png
         ├── topology-eve-ng.png
         ├── asa-connectivity.png
         ├── server-core-domain.png
         ├── user1-enabled.png
         ├── domain-login-validation.png
         └── dns-resolution.png

 ===============================================
# NOTES
 ===============================================
 
# - This project documents a working small enterprise AD lab.
# - Windows Server was deployed as Server Core.
# - Active Directory and DNS were configured successfully.
# - A Windows client joined the lab.local domain successfully.
# - User authentication was validated through domain login.
# - OU and GPO steps can be added later as future improvements.

# =========================================================
# LICENSE
# =========================================================

# This repository includes documentation and configuration files only.
# It does not include ISO images, QEMU images, Cisco images,
# or any licensed VM files.
