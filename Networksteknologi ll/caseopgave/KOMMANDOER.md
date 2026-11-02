# Show-kommandoer til fremvisningen

SSH-adresser: RT01 10.10.10.1, SW01 10.10.10.2, RT02 10.20.20.1, SW02 10.20.20.2, RT03 10.30.30.1, SW03 10.30.30.2.

## 0. Grundtjek (før vi starter)

| Enhed | Kommando | Hvad det viser |
| --- | --- | --- |
| Alle routere | `show ip interface brief` | Alle interfaces og IP-adresser. Alt skal være `up/up`, og Tunnel0 på RT02/RT03 |
| Alle routere | `show ip route` | Hele routingtabellen |
| RT02 | `show ip bgp summary` | At BGP-naboen er oppe |
| Alle routere | `show ip ospf neighbor` | At OSPF-naboerne er i `FULL` |
| RT02 | `show bfd neighbors` | At BFD-sessionen er `Up` |
| RT02 | `show crypto session detail` | At IPsec er `UP-ACTIVE` |

## V01: BGP, OSPF og redistribution

| Enhed | Kommando | Hvad det viser |
| --- | --- | --- |
| RT02 | `show ip bgp summary` | BGP-naboen 10.255.23.2 (AS 65003). Et tal i sidste kolonne betyder Established |
| RT03 | `show ip bgp summary` | BGP-naboen 10.255.23.1 (AS 65002) |
| RT01 | `show ip ospf neighbor` | OSPF-naboer (RT02 og RT03) i `FULL` |
| RT02 og RT03 | `show ip ospf neighbor` | En OSPF-nabo (RT01) |
| RT02 | `show ip bgp` | Alle BGP-ruter, også testpræfikset 10.30.250.0 |
| RT02 | `show ip bgp 10.30.250.0` | Testpræfikset **findes** i BGP, men ikke OSPF |
| RT01 | `show ip route 10.30.250.0` | `Subnet not in table`: testpræfikset er **ikke** redistribueret |
| RT01 | `show ip route 10.30.130.0` | Samlernettet som `O E2` med tag 10 (kom ind via redistribution) |
| RT01 | `show ip route 10.20.120.0` | Sensornettet som `O E2` |
| RT01 | `show ip ospf database external` | De eksterne LSA'er. Kun sensor- og samlernet, med tag 10 og 65 |
| RT02 og RT03 | `show ip protocols` | Hvilke protokoller kører, og at redistribution bruger et route-map |
| RT02 | `show route-map RM-CONN-TO-OSPF` | Reglerne: kun sensornettet, metrik 10, tag 10 |
| RT02 | `show route-map RM-BGP-TO-OSPF` | Reglerne: deny testpræfiks, permit samlernet med metrik 50 og tag 65 |
| RT02 | `show ip prefix-list` | Alle prefix-listerne |
| RT02 | `show ip prefix-list detail` | Prefix-listerne med hit-tæller (hvor mange gange en regel er brugt) |
| RT02 | `show ip bgp neighbors 10.255.23.2 advertised-routes` | Hvad RT02 sender til RT03 |
| RT02 | `show ip bgp neighbors 10.255.23.2 routes` | Hvad RT02 modtager fra RT03 |

## V02: Fejltest (shutdown/no shutdown)

Start en lang ping fra sensor-PC: `ping 10.30.130.10 -t`

| Enhed | Kommando | Hvad det viser |
| --- | --- | --- |
| RT02 | `show ip route 10.255.255.3` | **Før:** `B` via 10.255.23.2 (direkte). **Under:** `O` via 10.255.12.1 (via RT01) |
| RT02 | `traceroute 10.255.255.3 source Loopback0` | **Før:** RT03 direkte. **Under:** RT01 og så RT03 |
| RT02 | `show ip route 10.30.130.0` | Ruten til samlernettet før, under og efter |
| RT02 | `show ip bgp summary` | **Under:** BGP-naboen er ikke Established. **Efter:** tilbage |
| RT02 | `show bfd neighbors` | **Under:** BFD-sessionen er væk. **Efter:** `Up` |
| RT02 | `show ip interface brief` | Gi0/0/0.923 er `administratively down` under fejlen |
| RT02 | `show logging` | Tidsstempler for, hvornår BFD, BGP og OSPF skiftede |
| RT02 | `show crypto ipsec sa \| include #pkts` | Tællerne stiger stadig under fejlen: telemetrien går krypteret via RT01 |

Selve fejltesten på RT02:
```
configure terminal
interface GigabitEthernet0/0/0.923
 shutdown
```

## V03: PBR

| Enhed | Kommando | Hvad det viser |
| --- | --- | --- |
| RT02 | `show ip policy` | Hvilket interface der bruger PBR (Gi0/0/1.120 med RM-PBR-SENSOR) |
| RT02 | `show route-map RM-PBR-SENSOR` | Reglerne (next-hop 10.255.100.2, Null0) og `Policy routing matches`, altså antal pakker, der har ramt policyen |
| RT02 | `show access-lists ACL-PBR-SENSOR` | Hvilken trafik PBR rammer (sensornet til samlernet) |
| RT03 | `show route-map RM-PBR-DC` | Samme for svartrafikken fra samleren |
| RT02 | `show ip route 10.30.130.0` | Almindelig rute (BGP), men trafikken går i tunnelen på grund af PBR |
| RT02 | `ping 10.30.30.1 source 10.20.20.1` | Kontroltrafik, som virker uden at ramme PBR (tælleren ændrer sig ikke) |

## V04: Segmentering og management

| Enhed | Kommando | Hvad det viser |
| --- | --- | --- |
| SW01, SW02, SW03 | `show vlan brief` | VLAN-navne og hvilke porte, der er i hvilket VLAN |
| SW01, SW02, SW03 | `show interfaces trunk` | Trunk-portene og hvilke VLAN de tillader |
| RT01, RT02, RT03 | `show ip interface brief` | Alle subinterfaces, adresser og status |
| RT02 | `show ip interface GigabitEthernet0/0/1.120` | At sensor-interfacet har ACL'en `ACL-SENSOR-IN` og PBR |
| RT01, RT02, RT03 | `show running-config \| section interface` | Alle interfaces med masker og beskrivelser |

Management-adgang: SSH fra HQ-admin-PC til alle enheder.

## V05: Adgangskontrol

Fra sensor-PC: `ping 10.30.130.1` skal virke, `ping 10.10.110.1` og `ping 10.10.10.1` skal fejle.

| Enhed | Kommando | Hvad det viser |
| --- | --- | --- |
| RT02 | `show access-lists ACL-SENSOR-IN` | Reglerne med `(N matches)`. Permit-linjen stiger ved den positive test og deny-linjerne ved de negative |
| RT01 | `show access-lists ACL-SENSOR-GUARD` | Guard-reglerne på reservevejen. Hits får I ved at pinge fra RT02 med `source 10.20.120.1` |
| RT03 | `show access-lists ACL-SENSOR-GUARD` | Det samme på RT03 |
| RT01 | `show ip interface GigabitEthernet0/0/0.912` | At guard-ACL'en sidder på interfacet (`Inbound access list`) |
| RT03 | `show ip interface Tunnel0` | At guard-ACL'en også sidder på tunnelen |
| RT02 | `show logging` | Log af afviste pakker (deny-reglerne har `log`) |

Tests fra RT02 til reservevejen:
```
ping 10.10.110.10 source 10.20.120.1
ping 10.10.10.1 source 10.20.120.1
ping 10.30.130.10 source 10.20.120.1
```
De to første skal fejle (hits på RT01), den sidste skal virke.

## V06: IPsec

| Enhed | Kommando | Hvad det viser |
| --- | --- | --- |
| RT02 | `show crypto ikev2 sa` | IKE-SA'en (nøgleforhandlingen): `READY`, naboens adresse og algoritmerne |
| RT02 | `show crypto session detail` | Status `UP-ACTIVE`, og at der er to aktive IPsec-SA'er |
| RT02 | `show crypto ipsec sa` | IPsec-SA'erne med alle tællere |
| RT02 og RT03 | `show crypto ipsec sa \| include #pkts` | Kun tællerne: `encaps` (krypterede pakker sendt) og `decaps` (dekrypterede pakker modtaget) |
| RT02 | `show interface Tunnel0` | Tunnelen er `up/up`, med source, destination og `Tunnel protection via IPSec` |
| RT02 | `show ip route 10.255.255.3` | Ruten til tunnelens endepunkt går via WAN og ikke via tunnelen |

**Bevis for kryptering:** kør `show crypto ipsec sa | include #pkts` og notér tallene. Ping fra sensor-PC (`ping 10.30.130.10 -n 50`), og kør kommandoen igen. Encaps (RT02) og decaps (RT03) skal være steget.

## V07: Optimering

| Enhed | Kommando | Hvad det viser |
| --- | --- | --- |
| RT02 | `show bfd neighbors detail` | BFD med interval 100 ms og multiplier 3 |
| RT02 | `show ip ospf interface GigabitEthernet0/0/0.912` | Network type `POINT_TO_POINT` og `Hello 1, Dead 3` |
| RT02 | `show ip bgp neighbors 10.255.23.2` | BGP-timers, og at BFD er slået til på naboen |
| RT02 | `show ip protocols` | Redistribution med route-map, og OSPF/BGP-opsætning |
| RT02 | `show ip route 10.255.255.3` | Ruten før, under og efter fejlen (se V02) |
| RT02 | `show running-config \| section router bgp` | BGP-konfigurationen med `fall-over bfd` |

## Sådan læser I en rute

```
O E2  10.30.130.0/24 [110/10] via 10.255.13.2, GigabitEthernet0/0/0.913
```

- `O E2`: OSPF, eksternt (redistribueret) type 2
- `10.30.130.0/24`: netværket
- `[110/10]`: administrativ distance 110 og metrik 10
- `via 10.255.13.2`: næste router (next-hop)

`B` betyder BGP (`[20/0]`), `C` betyder tilsluttet direkte, og `L` er routerens egen adresse.