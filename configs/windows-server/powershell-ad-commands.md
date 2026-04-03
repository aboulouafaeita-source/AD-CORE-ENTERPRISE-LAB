# Windows Server Core - PowerShell AD Commands

# =========================================================
# Windows Server Core - PowerShell AD Commands
# Lab: Cisco ASA + EVE-NG + Windows Server Core + AD DS
# Domain: lab.local
# Server: SRV-DC1
# IP: 192.168.10.10/24
# Gateway: 192.168.10.1
# DNS: 192.168.10.10
# =========================================================

# Install AD DS role
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools

# Promote server to Domain Controller
Install-ADDSForest -DomainName "lab.local"

# Verify domain installation
Get-ADDomain

# Check current network settings
ipconfig /all
Get-NetAdapter

# Set DNS server on Server Core
Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 192.168.10.10

# Test connectivity
ping 192.168.10.1
ping 192.168.10.10

# Create a user
New-ADUser -Name "user1" -SamAccountName user1 -AccountPassword (ConvertTo-SecureString "User@123Aa!" -AsPlainText -Force) -Enabled $true

# Verify user
Get-ADUser user1
Get-ADUser user1 -Properties Enabled

# Reset user password
Set-ADAccountPassword -Identity user1 -Reset -NewPassword (ConvertTo-SecureString "User@123Aa!" -AsPlainText -Force)

# Enable user account
Enable-ADAccount -Identity user1

# Verify enabled status
Get-ADUser user1 -Properties Enabled

# Create Organizational Units (optional future step)
New-ADOrganizationalUnit -Name "Workstations" -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Users" -Path "DC=lab,DC=local"
New-ADOrganizationalUnit -Name "IT" -Path "OU=Users,DC=lab,DC=local"
New-ADOrganizationalUnit -Name "Employees" -Path "OU=Users,DC=lab,DC=local"

# Move user to Employees OU (optional future step)
Get-ADUser user1 | Move-ADObject -TargetPath "OU=Employees,OU=Users,DC=lab,DC=local"

# List computers in domain
Get-ADComputer -Filter *

# Move a client computer to Workstations OU (optional future step)
Get-ADComputer "ovi-PC" | Move-ADObject -TargetPath "OU=Workstations,DC=lab,DC=local"

# Create a GPO (optional future step)
New-GPO -Name "Employees - Restrict Control Panel"

# Link a GPO to an OU (optional future step)
New-GPLink -Name "Employees - Restrict Control Panel" -Target "OU=Employees,OU=Users,DC=lab,DC=local"

# Useful AD and DNS verification commands
Get-Service ADWS
Resolve-DnsName lab.local
Resolve-DnsName srv-dc1.lab.local
nslookup lab.local
nslookup srv-dc1.lab.local
