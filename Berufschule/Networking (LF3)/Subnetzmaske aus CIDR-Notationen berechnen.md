

Die Schreibweise von Subnetzmasken wie /22 wird als "CIDR-Notation" (Classless Inter-Domain Routing) bezeichnet. Diese Notation gibt an, wie viele Bits der Subnetzmaske den Netzwerkteil der IP-Adresse darstellen.

Für das Netzwerk 192.168.172.0/22:

1. Die CIDR-Notation ist /22.
2. Die entsprechende Subnetzmaske in Dotted-Decimal-Notation ist 255.255.252.0.

In der CIDR-Notation gibt die Zahl nach dem Schrägstrich die Anzahl der 1-Bits in der Subnetzmaske an. Für /22 sind die ersten 22 Bits der Subnetzmaske 1, und die verbleibenden 10 Bits sind 0. 

Um die Subnetzmaske in Dotted-Decimal-Notation zu berechnen:
- Die ersten 8 Bits (die ersten beiden Oktette) sind 255 (11111111).
- Die nächsten 8 Bits (das dritte Oktett) sind 255 (11111111).
- Die folgenden 6 Bits des vierten Oktetts sind 1, und die letzten 2 Bits sind 0, was 252 (11111100) ergibt.
- Das letzte Oktett ist 0 (00000000).

Somit ergibt sich die Subnetzmaske 255.255.252.0.