# Case opgave - Vindkraft Drift A/S

Vi er driftsteamet hos Vindkraft Drift A/S: hovedkontoret (HQ) styre driften, filialen indsamler data fra sensorerne, og datacentret (DC) gemmer alt data om driften.

## Netværks Topologi

![image](topologiV3.png)

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

### Tunnelnet (IPsec)

| Enhed | Interface | IP-adresse | Netværksadresse | Subnetmaske | Kommentar |
| --- | --- | --- | --- | --- | --- |
| RT02 | Tunnel0 | 10.255.100.1 | 10.255.100.0/29 | 255.255.255.248 | IPsec-tunnel mod RT03 |
| RT03 | Tunnel0 | 10.255.100.2 | 10.255.100.0/29 | 255.255.255.248 | IPsec-tunnel mod RT02 |

Tunnelnettet bruger kun to adresser (.1 og .2). Vi har valgt en /29, så der er plads til flere endepunkter senere (.3-.6), fx hvis RT01 skal med. Nettet ligger sammen med de andre infrastrukturnet i 10.255.x.x og overlapper ikke med noget. Det er ikke med i OSPF eller BGP.

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
- **Tunnelnet /29:** eget net til IPsec-tunnelen, så tunnelen ikke deler adresser med WAN eller tjenestenet.

## 2. Design og teknologivalg

### Overblik

| Protokol | Hvor | Hvorfor |
| --- | --- | --- |
| **OSPF** (proces 1, area 0) | Links 912 og 913 (RT01 er hub), management-net, HQ-admin, loopbacks | Hurtig, automatisk IGP som bærer reservevejen RT02-RT01-RT03 og giver fuld reachability |
| **eBGP** (RT02 AS 65002 ↔ RT03 AS 65003) | Kun link 923 | Primærvejen (direkte RT02-RT03). eBGP har administrativ distance 20 mod OSPF 110, så BGP vinder automatisk, når linket er oppe |
| **Redistribution** | På grænseroutere RT02 og RT03 | Binder de to domæner sammen, så filial og DC også er kendt i OSPF-domænet (RT01) |

RT01 kører kun OSPF: den er transit/reservevej, og domænegrænsen mellem OSPF og BGP ligger på RT02 og RT03.

| Router | Router-ID | BGP AS |
| --- | --- | --- |
| RT01 | 10.255.255.1 | – |
| RT02 | 10.255.255.2 | 65002 |
| RT03 | 10.255.255.3 | 65003 |

### V01: BGP + OSPF og kontrolleret redistribution

Afgrænsning ("hvilke links og præfikser håndterer de to protokoller"):

- **BGP** håndterer kun link 923 og kun fire præfikser: sensornettet (10.20.120.0/24), samlernettet (10.30.130.0/24), de to loopbacks (RT02/RT03 /32) og testpræfikset (10.30.250.0/24).
- **OSPF** håndterer links 912/913, alle management-net, HQ-admin og loopbacks.

Redistribution er **bevidst envejs: BGP → OSPF**, plus `connected → OSPF` for eget servicenet:

| Router | connected → OSPF (RM-CONN-TO-OSPF) | BGP → OSPF (RM-BGP-TO-OSPF) |
| --- | --- | --- |
| RT02 | Sensornet 10.20.120.0/24, metric 10, tag 10 | Kun samlernet 10.30.130.0/24, metric 50, tag 65. Testpræfiks 10.30.250.0/24 **afvises** (deny 5) |
| RT03 | Samlernet 10.30.130.0/24, metric 10, tag 10 | Kun sensornet 10.20.120.0/24, metric 50, tag 65 |

Filterpolitik og forhindring af route feedback:

- **Prefix-lister + route-maps** begrænser redistributionen til de to nødvendige præfikser. Alt andet (WAN, management, loopbacks, testpræfiks) rammer den implicitte deny.
- **Envejs-redistribution:** HQ-net og andre OSPF-ruter føres aldrig ind i BGP. Ellers ville RT03 lære HQ-net via BGP (AD 20) og vælge en suboptimal vej via RT02.
- **BGP in/out prefix-lister** (`PL-BGP-IN`/`PL-BGP-OUT`) sørger for, at hver router kun sender sine egne præfikser og kun modtager de forventede, så egne præfikser ikke kommer retur.
- **Metrikker:** RT01 modtager samme præfiks fra to routere (metric 10 fra ejeren, metric 50 fra grænseroutere). RT01 vælger altid ejeren, så der ikke opstår loops.
- **Tags** (10 = lokalt oprindelse, 65 = kom fra BGP) er mærkater, der gør det let at se herkomst i `show ip route`/OSPF-databasen. Skulle man senere redistribuere i den anden retning, kan man filtrere på dem.
- **Testpræfikset** 10.30.250.0/24 er med vilje i BGP (seq 15 i `PL-BGP-OUT`/`PL-BGP-IN`), så vi kan vise, at det findes i BGP, men ikke bliver redistribueret til OSPF.

### V02: Telemetri med automatisk reservevej

- **Normalt:** sensortrafik RT02 → RT03 krypteres i IPsec-tunnelen (V06). Tunnelen er bygget mellem loopbacks, og ruten til loopbacken går direkte over VLAN 923 (BGP-rute, AD 20).
- **Ved fejl** (`shutdown` på RT02 Gi0/0/0.923) forsvinder BGP-ruten, og OSPF-ruten via RT01 overtager automatisk. Tunnelen bliver oppe, fordi endepunkterne er de samme, og trafikken går nu RT02 → RT01 → RT03. Ingen manuel indgriben.
- **Tilbage:** `no shutdown` genopretter BGP, som igen slår OSPF på administrativ distance.
- **BFD** på 923 (100 ms × 3 = ca. 300 ms) sørger for, at **RT03** også opdager fejlen hurtigt. Uden BFD ville RT03 vente på BGP hold-time (9 s), fordi SW01-porten forbliver oppe, når kun RT02's subinterface lukkes.

### V03: Målrettet styring af sensortrafik (PBR)

Policy-based routing sender **kun** telemetri ind i tunnelen. Al anden trafik (management, HQ osv.) bruger almindelig routing.

| Router | Route-map | Rammer | Next-hop |
| --- | --- | --- | --- |
| RT02 | `RM-PBR-SENSOR` | Sensornet → samlernet | 10.255.100.2 (RT03 i tunnelen) |
| RT03 | `RM-PBR-DC` | Samlernet → sensornet (svartrafik) | 10.255.100.1 (RT02 i tunnelen) |

- Begge route-maps har `set interface Null0`. Hvis tunnelen er nede, bliver pakken droppet i stedet for at gå i klartekst.
- Policyen ligger både på interfacet trafikken kommer ind på (`ip policy`, til de rigtige PC'er) og som `ip local policy` (så tests fra routeren også rammer den).
- Vi bruger PBR og ikke en statisk rute, fordi en statisk rute (AD 1) ville slå BGP, og så stopper redistributionen fra V01.

### V04/V05: Segmentering og adgangskontrol

| ACL | Placering | Formål |
| --- | --- | --- |
| `ACL-SENSOR-IN` | RT02 Gi0/0/1.120 inbound | Første forsvarslinje ved kilden: sensorer må kun nå samlernettet (og pinge deres egen gateway) |
| `ACL-SENSOR-GUARD` | RT01 Gi0/0/0.912 inbound | Beskytter HQ (inkl. management) mod sensornet, også når trafik kommer via reservevejen |
| `ACL-SENSOR-GUARD` | RT03 Gi0/0/0.913 og .923 inbound | Beskytter DC-management og andre DC-net på både primær- og reservevej |
| `ACL-SENSOR-GUARD` | RT03 Tunnel0 inbound | Samme kontrol på trafikken, der kommer ud af tunnelen |

Reglerne i guard-ACL'erne (i rækkefølge): kontrolplan (OSPF/BGP/BFD) tillades altid, derefter sensor → samlernet tilladt, sensor → HQ/filial/DC/transport nægtet, og til sidst alt andet tilladt (`permit ip any any`).

Vi har bevidst lagt kontrollen **flere steder**, så den også holder på reservevejen (kravet i V02/V05).

### V07: Stabil og loopfri omskiftning

| Optimering | Virkning |
| --- | --- |
| BFD på link 923 (100 ms × 3) | Fejl opdages på under ét sekund i begge ender |
| BGP timers 3/9 og `fall-over bfd` | BGP-sessionen falder med det samme ved BFD-fejl |
| OSPF point-to-point + hello 1 / dead 3 på 912/913 | Ingen DR/BDR-valg, hurtig naboopdagelse |
| eBGP (AD 20) slår OSPF (AD 110) | Deterministisk primær/reserve uden ekstra konfiguration |
| Envejs-redistribution, prefix-lister og forskellige metrikker (10 vs 50) | Ingen route feedback og ingen loops. De tilladte præfikser er uændrede |

Sikker tilbageførsel: `no shutdown` på RT02 Gi0/0/0.923. BFD og BGP genetableres, og BGP-ruten (AD 20) erstatter OSPF-ruten. Tunnelen kører hele tiden.

## 3. Test og fremvisning

**Test-kilder:** PC'erne (sensor-test, DC-samler, HQ-admin) er de rigtige tests. Tests fra routeren bruger `source`, fx `ping 10.30.130.10 source 10.20.120.1` på RT02. Husk, at en test fra routeren ikke passerer routerens egen inbound-ACL, men den passerer RT01/RT03's guard-ACL'er.

### Fremvisningsrækkefølge

1. Forklar topologi, IP-plan og teknologivalg (afsnit 1 og 2).
2. **Normal drift** (V01, V02 før).
3. **V01:** redistribution og testpræfiks.
4. **V05:** adgangskontrol (positiv og to negative tests).
5. **V03:** PBR og upåvirket kontroltrafik.
6. **V06:** IPsec (tællere før og efter).
7. **V02:** fejltest (før, under, efter), og vis at IPsec stadig virker.
8. **V04:** segmentering og management-adgang.
9. **V07:** forklar optimeringen med output fra fejltesten.
10. **Gemt konfiguration og tilbageførsel.**

### V01: BGP + OSPF og redistribution

| Hvor | Kommando | Forventet |
| --- | --- | --- |
| RT02 | `show ip bgp summary` | Nabo 10.255.23.2 (AS 65003) i Established |
| RT01 | `show ip ospf neighbor` | To naboer (RT02 og RT03) i Full |
| RT02 og RT03 | `show ip ospf neighbor` | En nabo hver (RT01) i Full |
| RT02 | `show ip bgp 10.30.250.0` | Testpræfikset **findes** i BGP |
| RT01 | `show ip route 10.30.250.0` | `Network not in table`, altså **ikke** redistribueret |
| RT01 | `show ip route 10.30.130.0` | `O E2` med tag 10 via RT03 (redistribueret) |
| RT02 | `show route-map RM-BGP-TO-OSPF` og `show ip prefix-list` | Policy-matches, deny-hits på testpræfiks |
| RT01 | `show ip ospf database external` | Kun sensor- og samlernet (tag 10/65) |

**Præfiks før og efter redistribution:**

1. På RT02: `router ospf 1`, `no redistribute connected subnets route-map RM-CONN-TO-OSPF`, `no redistribute bgp 65002 subnets route-map RM-BGP-TO-OSPF`.
2. På RT03: samme med `65003`.
3. På RT01: `show ip route 10.30.130.0` viser, at præfikset **mangler** (før).
4. Sæt de fire linjer tilbage. Vent kort, og `show ip route 10.30.130.0` viser nu `O E2` (efter).

### V05: Adgangskontrol (fra sensor-test-PC)

| Test | Kommando | Forventet |
| --- | --- | --- |
| Positiv | `ping 10.30.130.10` | Succes |
| Negativ 1 | `ping 10.10.110.10` (HQ-admin) | Afvist |
| Negativ 2 | `ping 10.10.10.1` (HQ-management) | Afvist |

Vis bagefter `show access-lists` (RT02 `ACL-SENSOR-IN`, RT01/RT03 `ACL-SENSOR-GUARD`), så tællerne på permit- og deny-linjerne tydeligt stiger, og `show ip interface Gi0/0/1.120` (viser `Inbound access list is ACL-SENSOR-IN`).

### V03: PBR og upåvirket kontroltrafik

| Test | Kommando | Forventet |
| --- | --- | --- |
| Policy-match | Ping fra sensor-test-PC til 10.30.130.10, derefter RT02: `show route-map RM-PBR-SENSOR` | Match-tæller stiger |
| Svartrafik | RT03: `show route-map RM-PBR-DC` | Match-tæller stiger |
| Upåvirket kontroltrafik | RT02: `ping 10.30.30.1 source 10.20.20.1` | Virker, og PBR-tælleren **ændrer sig ikke** |

### V06: IPsec

| Trin | Kommando | Forventet |
| --- | --- | --- |
| 1. SA'er | RT02: `show crypto ikev2 sa` og `show crypto session detail` | IKEv2-SA i `READY`, session `UP-ACTIVE` mod 10.255.255.3 |
| 2. Før | RT02: `show crypto ipsec sa \| include #pkts` | Notér encaps/decaps-tællerne |
| 3. Skab trafik | Sensor-test-PC: `ping 10.30.130.10 -n 50` | Ping virker |
| 4. Efter | RT02 og RT03: `show crypto ipsec sa \| include #pkts` | Encaps stiger på RT02 og decaps på RT03 |
| 5. Underlay | RT02: `show ip route 10.255.255.3` | Ruten til endepunktet går via transportnettet (BGP/OSPF), **ikke** via Tunnel0 |

### V02: Telemetri med automatisk reservevej

Tunnelen skjuler hoppene, så et traceroute fra sensor-PC'en viser kun tunnelen (10.255.100.2). Vejen ser vi i stedet på tunnelens endepunkter med `traceroute 10.255.255.3 source Loopback0` på RT02.

| Fase | Handling | Kommandoer og forventet |
| --- | --- | --- |
| **Før** | – | RT02: `show ip route 10.255.255.3` viser `B` via 10.255.23.2. `traceroute 10.255.255.3 source Loopback0`: RT02 → RT03. Ping fra sensor-PC virker. |
| **Under** | RT02: `conf t`, `interface Gi0/0/0.923`, `shutdown` | RT02: `show ip route 10.255.255.3` viser `O` via 10.255.12.1. Traceroute: RT02 → **RT01** → RT03. Ping fra sensor-PC virker stadig, og IPsec-tællerne stiger. Gentag de to negative tests (stadig afvist). `show bfd neighbors` viser, at BFD-sessionen er faldet. |
| **Efter** | RT02: `no shutdown` på Gi0/0/0.923 | BGP kommer tilbage (`show ip bgp summary`), ruten er `B` via RT03 igen, traceroute går direkte. |

Husk: kun `shutdown` på subinterfacet Gi0/0/0.923 på RT02, **ikke** en switchport eller SW01.

### V04: Segmentering og management

- `show vlan brief`, `show interfaces trunk` og `show ip interface brief` på switche og routere.
- Vis subnetmasker, gateways og adskilte VLAN for sensor (120), samler (130), HQ-admin (110) og management (10/20/30).
- Vis fortsat management-adgang, fx SSH fra HQ-admin-PC til 10.20.20.2 (SW02) og 10.30.30.2 (SW03).

### V07: Optimering

- Vis `show ip route 10.255.255.3` på RT02 før, under og efter fejlen (se V02).
- Vis `show bfd neighbors`, `show ip ospf interface` (hello 1 / dead 3, point-to-point) og `show ip bgp neighbors 10.255.23.1 | include BFD` (på RT02: 10.255.23.2).
- Forklar virkning (hurtig omskiftning), hvorfor der ikke opstår loops (envejs-redistribution, filtre, metrikker) og sikker tilbageførsel (`no shutdown`).

### Gemt konfiguration og tilbageførsel

- **Vis gemt konfiguration:** `show startup-config` (eller `show running-config` sammenlignet med startup).
- **Tilbagefør en konkret ændring (eksempler):**
  - Fjern adgangskontrol på sensornettet: `interface Gi0/0/1.120`, `no ip access-group ACL-SENSOR-IN in`.
  - Fjern PBR: `no ip policy route-map RM-PBR-SENSOR` på samme interface samt `no ip local policy route-map RM-PBR-SENSOR`.
  - Slå redistribution fra: `no redistribute bgp 65002 subnets route-map RM-BGP-TO-OSPF`.
- **Genskab fra backup:** `copy flash:RT02-basis.cfg running-config` lægger konfigurationen oven på den kørende, så nye linjer skal evt. fjernes manuelt. Ved fuld genskabelse bruges `configure replace flash:RT02-basis.cfg`. Fuld nulstilling er ikke et krav.

## 4. Typiske spørgsmål og svar

| Spørgsmål | Kort svar |
| --- | --- |
| Hvorfor kører RT01 kun OSPF? | RT01 er kun transit og reservevej. Domænegrænsen ligger på RT02 og RT03, så BGP er kun nødvendigt dér |
| Hvorfor BGP på 923 og OSPF på 912/913? | BGP (AD 20) er primærvejen og slår automatisk OSPF (AD 110). Fejler 923, overtager OSPF via RT01 uden manuel indgriben |
| Hvorfor envejs-redistribution? | HQ-net i BGP ville give suboptimale ruter (AD 20) og risiko for route feedback. Filial og DC skal kun være synlige i OSPF |
| Hvad er testpræfikset til? | Det viser, at filtreringen virker: præfikset er i BGP, men bliver ikke redistribueret til OSPF |
| Hvad bruges tags til? | De mærker, om en rute er lokal (10) eller kom fra BGP (65), så herkomst kan ses og evt. filtreres senere |
| Hvorfor BFD? | Uden BFD venter RT03 op til 9 sekunder (BGP hold-time), fordi SW01-porten er oppe, når kun RT02's subinterface lukkes |
| Hvorfor ACL flere steder? | Kontrollen skal også holde på reservevejen via RT01, så HQ og DC er beskyttet uanset hvilken vej trafikken tager |
| Hvorfor ikke ACL på RT02 alene? | Tests fra routeren passerer ikke egen inbound-ACL, og flere lag dækker begge veje |
| Svagheden ved PBR? | Den ser kun lokal tilstand, ikke fejl hos naboen. IP SLA/track (V08) ville løse det |
| Hvorfor en tunnel (VTI) og ikke crypto map? | Tunnelen er bygget mellem loopbacks og er uafhængig af det fysiske link, så den overlever fejltesten. En crypto map skulle ligge på hvert WAN-link, og trafik via RT01 kunne gå ukrypteret |
| Hvorfor eget tunnelnet 10.255.100.0/29? | Så tunnelen ikke deler adresser med WAN eller tjenestenet. /29 giver plads til flere endepunkter senere |
| Hvad sker der, hvis tunnelen falder? | PBR har `set interface Null0`, så telemetrien droppes i stedet for at gå i klartekst |
| Hvordan viser vi, at trafikken er krypteret? | Encaps/decaps-tællerne i `show crypto ipsec sa` stiger, når vi sender trafik |

## 5. Kendte begrænsninger

- PBR reagerer ikke på fejl hos naboen, når linket er oppe lokalt. V08 (IP SLA/track) er ikke implementeret.
- Tags er kun mærkater. Route feedback forhindres af envejs-redistribution og prefix-lister, ikke af tag-matching.
- Konfigurationerne er skrevet til at blive pastet ind. `crypto key generate rsa` kan spørge om overskrivning, hvis der findes en nøgle i forvejen.
- IPsec kræver, at routerens sikkerhedslicens er aktiv. Tjek med `show version` eller `show license summary`, hvis tunnelen ikke kommer op.
- Pre-shared key er et laboratorie-valg og står i klartekst i konfigurationen.

## 6. V06: IPsec

IPsec-krypteret site-to-site VPN mellem filial (RT02) og DC (RT03) til telemetritrafik.

| Parameter | Valg | Hvorfor |
| --- | --- | --- |
| Type | VTI (`tunnel mode ipsec ipv4`) | Én tunnel, uafhængig af fysisk link. GRE alene er ikke nok (ingen kryptering) |
| Endepunkter | Loopback0: RT02 10.255.255.2, RT03 10.255.255.3 | Nås via transportnettet (BGP normalt, OSPF via RT01 ved fejl), og ruten afhænger ikke af tunnelen |
| Tunnelnet | 10.255.100.0/29 (RT02 .1, RT03 .2) | Eget net til tunnelen |
| IKE | IKEv2: AES-256, SHA-256, DH 14, pre-shared key | Nøgleforhandling |
| IPsec | ESP tunnel mode: AES-256, SHA-256 | Kryptering og integritet |
| Trafik ind i tunnelen | PBR (`RM-PBR-SENSOR` på RT02, `RM-PBR-DC` på RT03) | Kun telemetri krypteres, og routingen er uændret |
| MTU | `ip mtu 1400`, `ip tcp adjust-mss 1360` | Undgår fragmentering af krypteret trafik |

Sådan opfylder vi kravene:

- **Endepunkterne kan nås under WAN-fejlen:** loopbackerne ligger i både BGP og OSPF, så ruten skifter automatisk til RT01 (V02).
- **Ruten afhænger ikke af tunnelen:** der kører ingen routingprotokol over Tunnel0, og tunnelnettet er ikke annonceret.
- **Kontrollen på sensortrafikken holder:** ACL'erne rammer ikke ESP-pakkerne (kilde og destination er loopbackerne), og trafik der kommer ud af tunnelen på RT03 bliver kontrolleret af `ACL-SENSOR-GUARD` på Tunnel0.