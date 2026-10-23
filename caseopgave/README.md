# Case opgave - Vindkraft Drift A/S

## IP-Plan

### WAN-forbindelser

| VLAN | Netværk | Routere |
| --- | --- | --- |
| 912 | 10.255.12.0/30 | R1 – R2 |
| 913 | 10.255.13.0/30 | R1 – R3 |
| 923 | 10.255.23.0/30 | R2 – R3 |

### Managementnet

| Site | VLAN | Netværk | Gateway |
| --- | --- | --- | --- |
| HQ | 10 | 10.10.10.0/24 | 10.10.10.1 |
| Filial | 20 | 10.20.20.0/24 | 10.20.20.1 |
| Datacenter | 30 | 10.30.30.0/24 | 10.30.30.1 |

### Nye netværk

| Funktion | VLAN | Netværk | Gateway |
| --- | --- | --- | --- |
| Driftsledelse | 110 | 10.10.110.0/24 | 10.10.110.1 |
| Sensorer | 120 | 10.20.120.0/24 | 10.20.120.1 |
| Dataopsamling | 130 | 10.30.130.0/24 | 10.30.130.1 |