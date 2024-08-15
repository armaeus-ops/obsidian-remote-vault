Um die Netzadresse, die Broadcast-Adresse sowie die erste und letzte gültige IP-Adresse eines gegebenen Netzwerks zu berechnen, folgst du diesen Schritten:

1. **Netzadresse berechnen**:
    - Die Netzadresse erhältst du, indem du die gegebene IP-Adresse und die Subnetzmaske bitweise UND verknüpfst.
       ![[Tabelle - UND.png]]
2. **Broadcast-Adresse berechnen**:
    - Die Broadcast-Adresse erhältst du, indem du die Netzadresse nimmst und alle Host-Bits auf 1 setzt. Dies kannst du berechnen, indem du die Subnetzmaske invertierst (alle Bits umdrehst) und eine bitweise ODER-Verknüpfung mit der Netzadresse machst.
      ![[Tabelle - ODER.png]]
3. **Erste gültige IP-Adresse**:
    - Die erste gültige IP-Adresse ist die Netzadresse plus 1.
4. **Letzte gültige IP-Adresse**:
    - Die letzte gültige IP-Adresse ist die Broadcast-Adresse minus 1.

### Beispiel

Gegeben sei die IP-Adresse 192.168.1.10 mit der Subnetzmaske 255.255.255.240.

Schritt 1: Netzadresse berechnen

1. Konvertiere die IP-Adresse und die Subnetzmaske in Binärformat:[[Binär in Dezimal umrechnen (IP-Adressen)]]
    
    - IP-Adresse: 192.168.1.10 -> 11000000.10101000.00000001.00001010
    - Subnetzmaske: 255.255.255.240 -> 11111111.11111111.11111111.11110000
2. Führe eine bitweise UND-Verknüpfung durch:[[Tabelle - UND.png]]
    
    - Netzadresse: 11000000.10101000.00000001.00000000 -> 192.168.1.0

Schritt 2: Broadcast-Adresse berechnen

1. Invertiere die Subnetzmaske:
    
    - Invertierte Subnetzmaske: 00000000.00000000.00000000.00001111
2. Führe eine bitweise ODER-Verknüpfung zwischen der Netzadresse und der invertierten Subnetzmaske durch:[[Tabelle - ODER.png]]
    
    - Netzadresse: 11000000.10101000.00000001.00000000
    - Invertierte Subnetzmaske: 00000000.00000000.00000000.00001111
    - Broadcast-Adresse: 11000000.10101000.00000001.00001111 -> 192.168.1.15

Schritt 3: Erste gültige IP-Adresse

- Die erste gültige IP-Adresse ist die Netzadresse plus 1:
    - Netzadresse: 192.168.1.0
    - Erste gültige IP-Adresse: 192.168.1.1

Schritt 4: Letzte gültige IP-Adresse

- Die letzte gültige IP-Adresse ist die Broadcast-Adresse minus 1:
    - Broadcast-Adresse: 192.168.1.15
    - Letzte gültige IP-Adresse: 192.168.1.14

### Zusammenfassung für das Beispiel

- Netzadresse: 192.168.1.0
- Broadcast-Adresse: 192.168.1.15
- Erste gültige IP-Adresse: 192.168.1.1
- Letzte gültige IP-Adresse: 192.168.1.14