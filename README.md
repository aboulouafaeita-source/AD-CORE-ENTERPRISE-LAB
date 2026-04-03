# AD + ASA + EVE-NG Lab

This project documents a small enterprise lab built in EVE-NG using:

- Cisco vIOS Router
- Cisco ASA Firewall
- Switches
- Windows Server 2022 Core
- Windows client joined to domain

## Objectives

- Build a small enterprise network
- Configure router and ASA
- Deploy Windows Server Core
- Promote the server to Domain Controller
- Configure DNS
- Create AD users
- Join a client machine to the domain

## Topology

Router -> ASA -> SW1 -> SW2 -> Clients
                       └-> Windows Server Core

## IP Plan

- R1 g0/0: 10.0.0.1/30
- ASA outside: 10.0.0.2/30
- ASA inside: 192.168.10.1/24
- SRV-DC1: 192.168.10.10/24
- Client: 192.168.10.20/24

- ## Topology

![EVE-NG Topology](docs/topology.png)

## Topology

![EVE-NG Topology](docs/eve-ng-topology.png)

## Notes

- No VLANs in the current version
- Windows Server is Server Core
- GitHub repo does not include ISO/QCOW2/vendor images
