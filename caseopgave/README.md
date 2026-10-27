# Case opgave - Vindkraft Drift A/S

## Netværks Topologi

![image](topology.png)

## IP-Plan

### WAN-forbindelser mellem routerne

| Enhed | Interface | VLAN | IP-adresse | Netværksadresse | Subnetmaske | Forbindelse |
| --- | --- | --- | --- | --- | --- | --- |
| R1 | Gi0/0/0.912 | 912 | 10.255.12.1 | 10.255.12.0/30 | 255.255.255.252 | R1–R2 |
| R2 | Gi0/0/0.912 | 912 | 10.255.12.2 | 10.255.12.0/30 | 255.255.255.252 | R2–R1 |
| R1 | Gi0/0/0.913 | 913 | 10.255.13.1 | 10.255.13.0/30 | 255.255.255.252 | R1–R3 |
| R3 | Gi0/0/0.913 | 913 | 10.255.13.2 | 10.255.13.0/30 | 255.255.255.252 | R3–R1 |
| R2 | Gi0/0/0.923 | 923 | 10.255.23.1 | 10.255.23.0/30 | 255.255.255.252 | R2–R3, telemetrivej |
| R3 | Gi0/0/0.923 | 923 | 10.255.23.2 | 10.255.23.0/30 | 255.255.255.252 | R3–R2, telemetrivej |

### Managementnet

| Enhed | Interface | VLAN | IP-adresse | Netværksadresse | Subnetmaske | Gateway | Kommentar |
| --- | --- | --- | --- | --- | --- | --- | --- |
| R1 | Gi0/0/1.10 | 10 | 10.10.10.1 | 10.10.10.0/24 | 255.255.255.0 | – | HQ-management, gateway |
| SW1 | VLAN 10 (SVI) | 10 | 10.10.10.2 | 10.10.10.0/24 | 255.255.255.0 | 10.10.10.1 | SW1-management |
| R2 | Gi0/0/1.20 | 20 | 10.20.20.1 | 10.20.20.0/24 | 255.255.255.0 | – | Filial-management, gateway |
| SW2 | VLAN 20 (SVI) | 20 | 10.20.20.2 | 10.20.20.0/24 | 255.255.255.0 | 10.20.20.1 | SW2-management |
| R3 | Gi0/0/1.30 | 30 | 10.30.30.1 | 10.30.30.0/24 | 255.255.255.0 | – | DC-management, gateway |
| SW3 | VLAN 30 (SVI) | 30 | 10.30.30.2 | 10.30.30.0/24 | 255.255.255.0 | 10.30.30.1 | SW3-management |

### Tjenestenet

| Enhed/funktion | Interface | VLAN | IP-adresse | Netværksadresse | Subnetmaske | Gateway | Kommentar |
| --- | --- | --- | --- | --- | --- | --- | --- |
| R1 | Gi0/0/1.110 | 110 | 10.10.110.1 | 10.10.110.0/24 | 255.255.255.0 | – | HQ-driftsledelse |
| HQ-admin-PC | Netkort | 110 | 10.10.110.10 | 10.10.110.0/24 | 255.255.255.0 | 10.10.110.1 | Test af managementadgang |
| R2 | Gi0/0/1.120 | 120 | 10.20.120.1 | 10.20.120.0/24 | 255.255.255.0 | – | Sensornet, gateway |
| Sensor-test | Netkort | 120 | 10.20.120.10 | 10.20.120.0/24 | 255.255.255.0 | 10.20.120.1 | Kilde til telemetritest |
| R3 | Gi0/0/1.130 | 130 | 10.30.130.1 | 10.30.130.0/24 | 255.255.255.0 | – | DC-dataopsamling |
| DC-samler | Netkort | 130 | 10.30.130.10 | 10.30.130.0/24 | 255.255.255.0 | 10.30.130.1 | Destination for telemetri |

![image](zhej.jpg)