# Case opgave - Vindkraft Drift A/S

Vi er driftsteamet hos Vindkraft Drift A/S: hovedkontoret (HQ) styre driften, filialen indsamler data fra sensorerne, og datacentret (DC) gemmer alt data om driften.

## Netværks Topologi

![image](topology.png)

## 1. IP-Plan

### WAN-forbindelser

| Enhed | Interface | VLAN | IP-adresse | Netværksadresse | Subnetmaske | Forbindelse |
| --- | --- | --- | --- | --- | --- | --- |
| RT01 | Gi0/0/0.912 | 912 | 10.255.12.1 | 10.255.12.0/30 | 255.255.255.252 | RT01–RT02 |
| RT02 | Gi0/0/0.912 | 912 | 10.255.12.2 | 10.255.12.0/30 | 255.255.255.252 | RT02–RT01 |
| RT01 | Gi0/0/0.913 | 913 | 10.255.13.1 | 10.255.13.0/30 | 255.255.255.252 | RT01–RT03 |
| RT03 | Gi0/0/0.913 | 913 | 10.255.13.2 | 10.255.13.0/30 | 255.255.255.252 | RT03–RT01 |
| RT02 | Gi0/0/0.923 | 923 | 10.255.23.1 | 10.255.23.0/30 | 255.255.255.252 | RT02–RT03, telemetrivej |
| RT03 | Gi0/0/0.923 | 923 | 10.255.23.2 | 10.255.23.0/30 | 255.255.255.252 | RT03–RT02, telemetrivej |

### Managementnet

| Enhed | Interface | VLAN | IP-adresse | Netværksadresse | Subnetmaske | Gateway | Kommentar |
| --- | --- | --- | --- | --- | --- | --- | --- |
| RT01 | Gi0/0/1.10 | 10 | 10.10.10.1 | 10.10.10.0/24 | 255.255.255.0 | – | HQ-management, gateway |
| SW01 | VLAN 10 (SVI) | 10 | 10.10.10.2 | 10.10.10.0/24 | 255.255.255.0 | 10.10.10.1 | SW01-management |
| RT02 | Gi0/0/1.20 | 20 | 10.20.20.1 | 10.20.20.0/24 | 255.255.255.0 | – | Filial-management, gateway |
| SW02 | VLAN 20 (SVI) | 20 | 10.20.20.2 | 10.20.20.0/24 | 255.255.255.0 | 10.20.20.1 | SW02-management |
| RT03 | Gi0/0/1.30 | 30 | 10.30.30.1 | 10.30.30.0/24 | 255.255.255.0 | – | DC-management, gateway |
| SW03 | VLAN 30 (SVI) | 30 | 10.30.30.2 | 10.30.30.0/24 | 255.255.255.0 | 10.30.30.1 | SW03-management |

### Tjenestenet

| Enhed/funktion | Interface | VLAN | IP-adresse | Netværksadresse | Subnetmaske | Gateway | Kommentar |
| --- | --- | --- | --- | --- | --- | --- | --- |
| RT01 | Gi0/0/1.110 | 110 | 10.10.110.1 | 10.10.110.0/24 | 255.255.255.0 | – | HQ-driftsledelse |
| HQ-admin-PC | Netkort | 110 | 10.10.110.10 | 10.10.110.0/24 | 255.255.255.0 | 10.10.110.1 | Test af managementadgang |
| RT02 | Gi0/0/1.120 | 120 | 10.20.120.1 | 10.20.120.0/24 | 255.255.255.0 | – | Sensornet, gateway |
| Sensor-test | Netkort | 120 | 10.20.120.10 | 10.20.120.0/24 | 255.255.255.0 | 10.20.120.1 | Kilde til telemetritest |
| RT03 | Gi0/0/1.130 | 130 | 10.30.130.1 | 10.30.130.0/24 | 255.255.255.0 | – | DC-dataopsamling |
| DC-samler | Netkort | 130 | 10.30.130.10 | 10.30.130.0/24 | 255.255.255.0 | 10.30.130.1 | Destination for telemetri |

### Loopbacks

| Enhed | Interface | IP-adresse | Maske | Formål |
| --- | --- | --- | --- | --- |
| RT01 | Loopback0 | 10.255.255.1 | 255.255.255.255 (/32) | Router-ID (OSPF) |
| RT02 | Loopback0 | 10.255.255.2 | 255.255.255.255 (/32) | Router-ID (OSPF/BGP) og IPsec-endpoint |
| RT03 | Loopback0 | 10.255.255.3 | 255.255.255.255 (/32) | Router-ID (OSPF/BGP) og IPsec-endpoint |
| RT03 | Loopback250 | 10.30.250.1 | 255.255.255.0 (/24) | Uønsket testpræfiks til V01 (skal ikke redistribueres til OSPF) |

Loopback0-adresserne ligger i 10.255.255.0/24. Det er uden for de tre sites (10.10, 10.20, 10.30). Det overlapper hverken med WAN-/30-nettene (10.255.12/13/23.0) eller andre net.

![image](zhej.jpg)

### Switchporte og VLAN

| Switch | Port | Forbundet til | Type | VLAN |
| --- | --- | --- | --- | --- |
| SW01 | Gi1/0/1 | RT01 Gi0/0/0 | Trunk | 912, 913 |
| SW01 | Gi1/0/2 | RT02 Gi0/0/0 | Trunk | 912, 923 |
| SW01 | Gi1/0/3 | RT03 Gi0/0/0 | Trunk | 913, 923 |
| SW01 | Gi1/0/4 | RT01 Gi0/0/1 | Trunk | 10, 110 |
| SW01 | Gi1/0/5 | HQ-admin-PC | Access | 110 |
| SW02 | Gi1/0/1 | RT02 Gi0/0/1 | Trunk | 20, 120 |
| SW02 | Gi1/0/2 | Sensor-test | Access | 120 |
| SW03 | Gi1/0/1 | RT03 Gi0/0/1 | Trunk | 30, 130 |
| SW03 | Gi1/0/2 | DC-samler | Access | 130 |

### Hvorfor sådan her?

- **WAN /30:** punkt-til-punkt-links med router til router.
- **Tjenestenet /24:** alle tre tjenestenet er /24. Det giver plads til ca. 250 enheder pr. net, det passer til de eksisterende /24 managementnet og gør ACL-regler og prefix-lister enkle at læse og forklare.
- **Separate VLAN:** sensor (120), DC-samler (130) og HQ-admin (110) ligger i hvert deres VLAN, adskilt fra management (10/20/30). Så kan trafik mellem zonerne styres med Layer 3-regler på routerne (V04/V05).
- **Loopbacks:** unikke /32-adresser som matcher med router-ID på hver router og fungere som endpoints.