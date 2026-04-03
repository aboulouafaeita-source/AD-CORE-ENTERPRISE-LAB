# IP Plan

## WAN
- R1 g0/0: 10.0.0.1/30
- ASA e0 (outside): 10.0.0.2/30

## LAN
- ASA e1 (inside): 192.168.10.1/24
- SRV-DC1: 192.168.10.10/24
- WIN-CLIENT: 192.168.10.20/24

## DNS
- Preferred DNS for clients: 192.168.10.10
