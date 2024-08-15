

*Der Elektro-Automatik GmbH wurde die Netzwerkadresse 192.168.172.0/22 zugeteilt. Es wird grundsätzlich das TCP/IP Netzprotokoll eingesetzt. Aus Sicherheitsgründen sollen die drei Bereiche Verwaltung, Fertigung und Konstruktion in Subnetze verteilt werden. A) geben Sie die Netzadressen der Subnetze an und berücksichtigen Sie folgende Vorgaben: Es sollen insgesamt bis zu sechs gleich große Subnetze möglich sein. Die Nutzung des Adressraums soll auf eine möglichst hohe Anzahl von Hosts pro Subnetz optimiert werden. B) Geben Sie die Subnetzmaske für die Hosts in den Subnetzen an.*

1. **Identifikation des ursprünglichen Netzwerks und seiner Eigenschaften**:
   - Das zugeteilte Netzwerk ist 192.168.172.0/22.
   - Ein /22-Netzwerk hat eine Standard-Subnetzmaske von 255.255.252.0 [[Subnetzmaske aus CIDR-Notationen berechnen]]
   - Das bedeutet, dass das Netzwerk 1024 IP-Adressen umfasst (2^10, da 32 - 22 = 10 freie Bits für Hosts sind).

2. **Subnetting in 6 gleich große Subnetze**:
   - Da wir 6 gleich große Subnetze benötigen, schauen wir uns an, wie viele zusätzliche Bits wir ausleihen müssen, um 6 Subnetze zu erstellen.
   - Wir benötigen mindestens 3 zusätzliche Bits (2^3 = 8 Subnetze), weil 2^2 = 4 nicht ausreicht.
   - Dies ändert die Subnetzmaske von /22 zu /25 (22 + 3 = 25).

3. **Berechnung der neuen Subnetzmaske**:
   - Eine /25 Subnetzmaske bedeutet 255.255.255.128. [[Berechnen von Subnetzen aus IP und Subnetzmaske]]
   - Jedes /25 Subnetz enthält 128 IP-Adressen, davon 126 nutzbare Hosts (weil 2 Adressen für Netzwerk- und Broadcast-Adresse reserviert sind).

4. **Berechnung der Netzadressen der Subnetze**:
   - Die Netzadressen ändern sich in 128er-Schritten (da 128 Adressen pro Subnetz existieren).

Hier sind die Berechnungen im Detail:

### A) Netzadressen der Subnetze
1. **Erstes Subnetz**:
   - Netzadresse: 192.168.172.0
   - Hostbereich: 192.168.172.1 bis 192.168.172.126
   - Broadcast-Adresse: 192.168.172.127

2. **Zweites Subnetz**:
   - Netzadresse: 192.168.172.128
   - Hostbereich: 192.168.172.129 bis 192.168.172.254
   - Broadcast-Adresse: 192.168.172.255

3. **Drittes Subnetz**:
   - Netzadresse: 192.168.173.0
   - Hostbereich: 192.168.173.1 bis 192.168.173.126
   - Broadcast-Adresse: 192.168.173.127

4. **Viertes Subnetz**:
   - Netzadresse: 192.168.173.128
   - Hostbereich: 192.168.173.129 bis 192.168.173.254
   - Broadcast-Adresse: 192.168.173.255

5. **Fünftes Subnetz**:
   - Netzadresse: 192.168.174.0
   - Hostbereich: 192.168.174.1 bis 192.168.174.126
   - Broadcast-Adresse: 192.168.174.127

6. **Sechstes Subnetz**:
   - Netzadresse: 192.168.174.128
   - Hostbereich: 192.168.174.129 bis 192.168.174.254
   - Broadcast-Adresse: 192.168.174.255

### B) Subnetzmaske für die Hosts in den Subnetzen
- Die Subnetzmaske für die Hosts in allen Subnetzen lautet: 255.255.255.128 oder /25.

### Zusammenfassung:
- Die Netzadressen der sechs gleich großen Subnetze sind:
  - 192.168.172.0/25
  - 192.168.172.128/25
  - 192.168.173.0/25
  - 192.168.173.128/25
  - 192.168.174.0/25
  - 192.168.174.128/25

- Die Subnetzmaske für jedes Subnetz ist 255.255.255.128.

Mit diesen Einstellungen sind die drei Bereiche Verwaltung, Fertigung und Konstruktion sicher voneinander getrennt und die Adressierung ist effizient genutzt.