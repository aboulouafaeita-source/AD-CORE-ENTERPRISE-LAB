# Implementation Steps

# =========================================================
# Implementation Steps - AD + ASA + EVE-NG Lab
# =========================================================
# This block documents the full implementation flow used
# in this lab. It is stored in one code block for simple
# copy/paste into GitHub documentation.
# =========================================================

# 1) Build the topology in EVE-NG
# - Cisco vIOS router
# - Cisco ASA firewall
# - SW1
# - SW2
# - Windows Server 2022 Core
# - Windows client
# - Optional VPC nodes
#
# Topology:
# R1 Gi0/0 -> ASA Ethernet0
# ASA Ethernet1 -> SW1
# SW1 -> SW2
# Windows Server -> SW1
# Windows client -> SW1 or SW2
# Additional clients/VPCs -> SW1/SW2

# 2) Configure the IP plan
# WAN:
# - R1 GigabitEthernet0/0 = 10.0.0.1/30
# - ASA Ethernet0 outside = 10.0.0.2/30
#
# LAN:
# - ASA Ethernet1 inside = 192.168.10.1/24
# - Windows Server Core = 192.168.10.10/24
# - Windows client = 192.168.10.20/24
#
# DNS:
# - Preferred DNS for all domain members = 192.168.10.10

# 3) Configure the router
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

# 4) Configure the ASA firewall
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

# 5) Configure Windows Server Core network settings
# Windows Server values:
# - Hostname = SRV-DC1
# - IP address = 192.168.10.10
# - Subnet mask = 255.255.255.0
# - Default gateway = 192.168.10.1
# - DNS server = 192.168.10.10

ipconfig /all
Get-NetAdapter
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 192.168.10.10
ping 192.168.10.1
ping 192.168.10.10

# 6) Install Active Directory Domain Services
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools

# 7) Promote the server to Domain Controller
Install-ADDSForest -DomainName "lab.local"

# The server restarts automatically after promotion

# 8) Verify domain installation
Get-ADDomain

# Expected:
# - Domain name = lab.local
# - Domain controller = SRV-DC1.lab.local

# 9) Create a domain user
New-ADUser -Name "user1" -SamAccountName user1 -AccountPassword (ConvertTo-SecureString "User123Aa" -AsPlainText -Force) -Enabled $true

# If the account already exists, verify and fix it
Get-ADUser user1
Get-ADUser user1 -Properties Enabled
Set-ADAccountPassword -Identity user1 -Reset -NewPassword (ConvertTo-SecureString "User_123Aa" -AsPlainText -Force)
Enable-ADAccount -Identity user1
Get-ADUser user1 -Properties Enabled

# Expected:
# Enabled : True

# 10) Configure the Windows client
# Client values:
# - IP address = 192.168.10.20
# - Subnet mask = 255.255.255.0
# - Default gateway = 192.168.10.1
# - DNS server = 192.168.10.10

# Validation on client
ping 192.168.10.10
nslookup lab.local
ping srv-dc1.lab.local

# Expected:
# - lab.local resolves correctly
# - srv-dc1.lab.local resolves to 192.168.10.10
# - ping succeeds

# 11) Join the Windows client to the domain
# On the client:
# - Open System Properties
# - Go to Computer Name
# - Select Change
# - Choose Domain
# - Enter: lab.local
#
# Use credentials:
# - LAB\Administrator
#
# Expected message:
# Welcome to the lab.local domain.

# 12) Restart the client after domain join

# 13) Log in with the domain user
# Username:
# LAB\user1
#
# Use the password configured for user1

# Validation on the client after login
whoami
echo %userdomain%
echo %logonserver%

# Expected:
# - lab\user1
# - LAB
# - \\SRV-DC1

# 14) Optional future improvements
# - Create Organizational Units
# - Move users to Employees OU
# - Move computers to Workstations OU
# - Create and link GPOs
# - Restrict Control Panel for Employees
# - Add more clients
# - Add screenshots and troubleshooting notes

# 15) Final outcome
# - Working EVE-NG topology
# - Cisco router and ASA firewall configured
# - Windows Server 2022 Core promoted to Domain Controller
# - DNS hosted on the DC
# - Windows client joined to the domain
# - AD authentication tested successfully
