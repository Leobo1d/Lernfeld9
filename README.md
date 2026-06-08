# Lernfeld9
Lernfeld 9 - Dokumentation und Projektdateien

IPAM-Adressierungsplan IPv6 und Einrichtungsdokumentation

Projekt: 4-Standorte-Netz (HH, HL, B, M)
Basis: Cisco ISR4331 (Router), Cisco 3650 (L3-Switch)

==================================================
0) ANFORDERUNGEN UND UMSETZUNGSRAHMEN


0.1 Allgemeine Netzwerkanforderungen
- Alle Infrastrukturgeräte, Server und Clients werden nativ mit IPv6 betrieben (kein Dual Stack).
- Transportnetze (Router-zu-Router) verwenden ULA (fd00:...).
- Standortnetze verwenden GUA aus dem PI-Space 2001:db8::/32.
- HH und HL erhalten je zwei VLANs mit jeweils eigenem globalen Präfix.
- B und M erhalten je mindestens ein VLAN für den jeweiligen Server mit eigenem globalen Präfix.
- Die Standortnetze werden jeweils über Router + L3-Switch realisiert.
- Clients und Server erhalten globale Unicast-Adressen.
- Die Adressvergabe erfolgt initial statisch.

0.2 Routing-Anforderungen (phasenweise)
- Basisanforderung: Erreichbarkeit zunächst mit statischen Routen sicherstellen.
- Must-have: Manülle Routingeinträge durch OSPFv3 ersetzen.
- OSPFv3-Vorgabe: Area 0, Prozess-ID 42, Router-IDs 1.1.1.1 bis 4.4.4.4.

0.3 Anforderungen zur Absicherung
- Banner auf Routern und Switches konfigurieren.
- Passwörter setzen und absichern.
- SSH-Zugänge einrichten.
- SSH-Zugriff nur von Clients des Standorts Hamburg erlauben.

0.4 IPv6 ACL-Anforderungen
- Clients aus HH dürfen Webserver per HTTP und HTTPS erreichen.
- Clients aus HL dürfen Webserver nur per HTTPS erreichen.
- ACL-Konfigurationen müssen dokumentiert werden.

0.5 DHCPv6-Anforderungen
- Must-have: DHCPv6 im Modus "SLAAC with DHCPv6-Server" auf Routern.
- Should-have: DHCPv6 am Standort Hamburg im Modus "Stateful".
- DHCPv6-Zuweisungen und Pools werden in der IPAM dokumentiert.

==================================================
1) ZIELBILD UND DESIGN-ENTSCHEIDUNGEN
==================================================

Ziel:
- Das Netzwerk soll reproduzierbar, störungsarm und nachvollziehbar aufgebaut werden.
- Die Konfiguration soll mit der IPAM konsistent bleiben.
- Jede Entscheidung soll technisch begründet sein (Wartbarkeit, Skalierbarkeit, Fehlersicherheit).

Design-Entscheidungen:
- VLAN-Standortnetze als GUA aus 2001:db8::/32.
- Transportnetze (Transit) als ULA aus fd00:db8::/32.
- Router-zu-Switch als Routed Port (kein Trunk).
- Routing in zwei Phasen: zürst statische Routen, danach Migration auf OSPFv3.
- Initial statische Adressvergabe, danach DHCPv6-Erweiterung gemäss Vorgaben.

Warum sinnvoll:
- Klare Trennung zwischen Nutznetzen (GUA) und Infrastruktur (ULA).
- Gute Lesbarkeit der Adressen durch Standort- und VLAN-Hextet.
- Skalierbar und wartbar bei späteren Erweiterungen.
- Phasenmodell erfüllt sowohl Basisanforderungen (statisch) als auch Must-have (OSPFv3).

==================================================
2) TOPOLOGIE, PORTPLAN UND VLAN-ZUORDNUNG
==================================================

2.1 Router-zu-Router (gegeben)
- Router-HH Se0/1/0 <-> Router-B  Se0/1/0
- Router-HH Se0/1/1 <-> Router-HL Se0/1/1
- Router-HH Se0/2/0 <-> Router-M  Se0/2/0
- Router-M  Se0/1/0 <-> Router-HL Se0/1/0
- Router-M  Se0/1/1 <-> Router-B  Se0/1/1
- Router-HL Se0/2/0 <-> Router-B  Se0/2/0

2.2 Router-zu-Switch (gegeben)
- Router-HH Gi0/0/0 <-> SW-HH-01 Gi1/0/1
- Router-HL Gi0/0/0 <-> SW-HL-01 Gi1/0/1
- Router-B  Gi0/0/0 <-> SW-B-01  Gi1/0/1
- Router-M  Gi0/0/0 <-> SW-M-01  Gi1/0/1

2.3 Access-Ports je Standort

SW-HH-01:
- VLAN 10: Gi1/0/2 bis Gi1/0/11
- VLAN 20: Gi1/0/12 bis Gi1/0/24

SW-HL-01:
- VLAN 30: Gi1/0/2
- VLAN 40: Gi1/0/12

SW-B-01:
- VLAN 50: Gi1/0/2 (Server)

SW-M-01:
- VLAN 60: Gi1/0/2 (Server)

==================================================
3) ADRESSIERUNGSLOGIK
==================================================

GUA (Standort-/VLAN-Netze):
- PI-Space: 2001:db8::/32
- Schema: 2001:db8:SSSS:VVVV::/64
  - SSSS = Standortkennung
  - VVVV = VLAN-ID
  - Host-ID = Interface Identifier im Host-Anteil

Standortkennungen (SSSS):
- HH = 0100
- HL = 0200
- B  = 0300
- M  = 0400

ULA (Transport-/Transit-Netze):
- ULA-Block: fd00:db8::/32
- Verwendet für Router-zu-Router und Router-zu-L3-Switch-Transit

Gateway-Konvention:
- SVI-Gateway in VLAN-Netzen: ::1

Host-/Server-ID-Empfehlung:
- Infrastruktur: ::1 bis ::20
- Server: ::10
- Clients: ab ::100

==================================================
4) VLAN-NETZE (GUA)
==================================================

HH:
- VLAN 10: 2001:db8:0100:0010::/64
  - Gateway (SVI): 2001:db8:0100:0010::1
- VLAN 20: 2001:db8:0100:0020::/64
  - Gateway (SVI): 2001:db8:0100:0020::1

HL:
- VLAN 30: 2001:db8:0200:0030::/64
  - Gateway (SVI): 2001:db8:0200:0030::1
- VLAN 40: 2001:db8:0200:0040::/64
  - Gateway (SVI): 2001:db8:0200:0040::1

B:
- VLAN 50: 2001:db8:0300:0050::/64
  - Gateway (SVI): 2001:db8:0300:0050::1

M:
- VLAN 60: 2001:db8:0400:0060::/64
  - Gateway (SVI): 2001:db8:0400:0060::1

==================================================
5) TRANSPORTNETZE (ULA)
==================================================

5.1 Router-zu-Router Seriellinks (/127)

- Link HH <-> B: fd00:db8:0103:0000::/127
  - Router-HH Se0/1/0: fd00:db8:0103::0/127
  - Router-B  Se0/1/0: fd00:db8:0103::1/127

- Link HH <-> HL: fd00:db8:0102:0000::/127
  - Router-HH Se0/1/1: fd00:db8:0102::0/127
  - Router-HL Se0/1/1: fd00:db8:0102::1/127

- Link HH <-> M: fd00:db8:0104:0000::/127
  - Router-HH Se0/2/0: fd00:db8:0104::0/127
  - Router-M  Se0/2/0: fd00:db8:0104::1/127

- Link M <-> HL: fd00:db8:0204:0000::/127
  - Router-M  Se0/1/0: fd00:db8:0204::1/127
  - Router-HL Se0/1/0: fd00:db8:0204::0/127

- Link M <-> B: fd00:db8:0304:0000::/127
  - Router-M Se0/1/1: fd00:db8:0304::1/127
  - Router-B Se0/1/1: fd00:db8:0304::0/127

- Link HL <-> B: fd00:db8:0203:0000::/127
  - Router-HL Se0/2/0: fd00:db8:0203::0/127
  - Router-B  Se0/2/0: fd00:db8:0203::1/127

5.2 Router-zu-L3-Switch Transit (/127)

- HH Transit: fd00:db8:0100:ff00::/127
  - Router-HH Gi0/0/0:   fd00:db8:0100:ff00::0/127
  - SW-HH-01 Gi1/0/1 L3: fd00:db8:0100:ff00::1/127

- HL Transit: fd00:db8:0200:ff00::/127
  - Router-HL Gi0/0/0:   fd00:db8:0200:ff00::0/127
  - SW-HL-01 Gi1/0/1 L3: fd00:db8:0200:ff00::1/127

- B Transit: fd00:db8:0300:ff00::/127
  - Router-B Gi0/0/0:   fd00:db8:0300:ff00::0/127
  - SW-B-01 Gi1/0/1 L3: fd00:db8:0300:ff00::1/127

- M Transit: fd00:db8:0400:ff00::/127
  - Router-M Gi0/0/0:   fd00:db8:0400:ff00::0/127
  - SW-M-01 Gi1/0/1 L3: fd00:db8:0400:ff00::1/127

==================================================
6) UPLINK- UND ROUTING-KONZEPT
==================================================

L3-Uplink/Trunk-Entscheidung:
- Zwischen Router und L3-Switch jeweils Routed Port (kein 802.1Q-Trunk).
- VLAN-Routing erfolgt auf den L3-Switches per SVI.
- Trunks sind nur nötig, wenn weitere Access-Switches angebunden werden.

Routing-Entscheidung:
- Phase 1 (Basis): Statische Routen für alle entfernten Standortpräfixe.
- Phase 2 (Must-have): Migration auf OSPFv3 (Ablosung statischer Routen).
- OSPFv3 auf allen ULA-Transitinterfaces (Router-Router und Router-Switch-Transit).
- VLAN/GUA-Netze auf den L3-Switches announcen.
- OSPFv3-Parameter: Area 0, Prozess-ID 42.
- Router-ID-Plan:
  - Router-HH = 1.1.1.1
  - Router-HL = 2.2.2.2
  - Router-B  = 3.3.3.3
  - Router-M  = 4.4.4.4

Mögliche Standort-Zusammenfassung:
- HH: 2001:db8:0100::/48
- HL: 2001:db8:0200::/48
- B:  2001:db8:0300::/48
- M:  2001:db8:0400::/48

Warum sinnvoll:
- OSPFv3 bietet bei mehreren Pfaden schnellere Konvergenz als statische Routen.
- Zusammenfassungen halten Routingtabellen übersichtlich.
- Die initiale statische Phase ist gut kontrollierbar und erleichtert die Erstinbetriebnahme.

==================================================
7) EINRICHTUNGS-VORGEHEN (SCHRITT FUR SCHRITT)
==================================================

1) Basis-Setup aller Geräte
- Hostname, Management-Zugang, lokaler Admin, SSH, Banner, Zeitsynchronisation.
- Ohne Basis-Setup sind Fehlersuche, Remote-Zugriff und Change-Nachvollziehbarkeit schwierig.

1a) Grundabsicherung direkt mit umsetzen
- Login-Banner konfigurieren.
- Enable-/User-Passwörter und SSH-Hardening umsetzen.
- VTY-Zugriff per IPv6-ACL auf Hamburg-Clientnetze beschränken.
- So wird die Sicherheitsanforderung von Beginn an eingehalten.

2) Physik und Interface-Beschriftung
- Interfaces laut Portplan mit description versehen.
- Shutdown/No Shutdown bewusst setzen.
- Einheitliche Bezeichnungen verhindern Zuordnungsfehler.

3) Layer-2/VLAN-Setup auf den L3-Switches
- VLANs anlegen: HH(10,20), HL(30,40), B(50), M(60).
- Access-Ports gemäss Plan zuweisen.
- Erst stabile VLAN-Struktur, dann Layer-3.

4) Layer-3 auf den Switches (SVI + IPv6 Routing)
- Pro VLAN SVI mit GUA ::1 konfigurieren.
- IPv6 Routing auf den L3-Switches aktivieren.
- Inter-VLAN-Routing erfolgt lokal am Standort.

5) Transitnetze (ULA) konfigurieren
- Router-zu-Router und Router-zu-Switch mit /127 aus ULA setzen.
- /127 reduziert Neighbor-Discovery-Fläche auf Punkt-zu-Punkt-Links.

6) Erreichbarkeit zürst mit statischen Routen herstellen
- Auf allen Routern statische IPv6-Routen zu den entfernten Standortpräfixen konfigurieren.
- Funktionstests (Ping/Traceroute) für jede Standortkombination durchführen.

7) Migration auf OSPFv3 (Must-have)
- OSPFv3 Prozess 42, Area 0 auf allen Routern aktivieren.
- Router-IDs 1.1.1.1 bis 4.4.4.4 nach Plan setzen.
- Nach stabiler Nachbarschaft statische Routen entfernen.
- Transit-Interfaces aktiv, Access-Interfaces wo sinnvoll passiv.
- VLAN-Netze am Standort announcen.

8) DHCPv6 umsetzen
- Must-have: SLAAC with DHCPv6-Server (M-Flag aus, O-Flag an) auf Router-Interfaces je Client-/Server-VLAN.
- Should-have: Stateful DHCPv6 in Hamburg (M-Flag an) für definierte VLANs.
- DHCPv6-Pools, DNS, Domain und Lease-Parameter dokumentieren.

9) IPv6 ACL für Webserverzugriffe umsetzen
- ACL-Regeln nach Herkunftsstandort trennen:
  - HH-Clients: HTTP(80) und HTTPS(443) zu Webservern erlauben.
  - HL-Clients: nur HTTPS(443) zu Webservern erlauben.
- Nicht benötigten Verkehr gem. Sicherheitskonzept sperren und protokollieren.

10) Sicherheits- und Stabilitätsmassnahmen
- Nicht genutzte Ports shutdown.
- Optional: RA-Guard/Port-Security an Access-Ports, Control-Plane-Protection am Router.

11) Verifikation und Abnahme
- show ipv6 interface brief
- show ipv6 route
- show ospfv3 neighbor
- show ipv6 dhcp pool / binding
- show access-lists (IPv6)
- ping ipv6 / traceroute ipv6 standortintern und standortübergreifend

==================================================
8) DOKUMENTATIONSSTANDARD UND ABNAHMEKRITERIEN
==================================================

Dokumentationsstandard während der Einrichtung:
- Jede änderung sofort in der IPAM markieren (Datum, Bearbeiter, Grund).
- Pro Link/VLAN eine Checkliste: konfiguriert, getestet, abgenommen.
- Abweichungen vom Plan im änderungsprotokoll dokumentieren.
- Nach jedem Standort-Abschnitt Konfigurationsbackup erstellen.
- Sicherheitskonfigurationen (Banner, SSH, VTY-ACL, IPv6-ACL) explizit versionieren.
- DHCPv6-Konfigurationen (Modus, Poolname, Prefix, DNS, Bindings) in der IPAM führen.

Minimale Abnahmekriterien (prüfungsreif):
- Alle geplanten Interfaces sind administrativ und operativ up.
- Alle VLAN-Gateways antworten auf IPv6 Ping.
- Jeder Standort erreicht jeden anderen Standort per IPv6.
- OSPFv3-Nachbarschaften sind auf allen geplanten Transitlinks stabil.
- IPAM, Portplan und reale Konfiguration sind konsistent.
- SSH-Zugriff ist nur aus Hamburg-Clientnetzen möglich.
- Webserverzugriffe entsprechen der ACL-Vorgabe (HH: HTTP/HTTPS, HL: nur HTTPS).
- DHCPv6 Must-have ist funktional; Should-have (Stateful in HH) ist dokumentiert, falls umgesetzt.

==================================================
ENDE
==================================================
