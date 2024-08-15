- persistente Speicherung von Daten
- Datenbankmodelle: relationales (Tabelle)
- hierarchische 
- No-SQL
- Key-Value

#### Entity Relationship Modell (ER-Modell)

![[Einfaches-ER-Modell-1024x576.png]]

Das Entity-Relationship-Model (ER-Model) ist ein konzeptionelles Datenmodell, das verwendet wird, um die Struktur einer Datenbank zu definieren. Es besteht aus verschiedenen grundlegenden Elementen: Entitäten, Attribute, Beziehungen, Kardinalitäten und Schlüssel. Hier ist eine Beschreibung dieser Elemente anhand eines Beispiels einer Universitätsdatenbank:

1. **Entitäten (Entities)**:
   - **Definition**: Entitäten sind Objekte oder Dinge in der realen Welt, die voneinander unterscheidbar sind und über die Daten gespeichert werden sollen.
   - **Entitätstyp**: eine Abstrakte Klasse oder Kategorie, die eine Menge an Entitäten zusammenfasst
   - **Beispiel**: In einer Universitätsdatenbank könnten die Entitäten "Student", "Kurs" und "Professor" sein.

2. **Attribute**:
   - **Definition**: Attribute sind Eigenschaften oder Merkmale, die zur Beschreibung einer Entität verwendet werden.
   - **Beispiel**:
     - Für die Entität "Student" könnten die Attribute "Matrikelnummer", "Name", "Geburtsdatum" und "Adresse" sein.
     - Für die Entität "Kurs" könnten die Attribute "Kurs-ID", "Kursname" und "ECTS-Punkte" sein.
     - Für die Entität "Professor" könnten die Attribute "Professoren-ID", "Name" und "Fachbereich" sein.

3. **Beziehungen (Relationships)**:
   - **Definition**: Beziehungen beschreiben, wie Entitäten miteinander in Verbindung stehen.
   - **Beispiel**: Eine Beziehung könnte "belegt" sein, die beschreibt, dass ein Student einen Kurs belegt. Eine weitere Beziehung könnte "lehrt" sein, die beschreibt, dass ein Professor einen Kurs lehrt.

4. **Kardinalitäten (Cardinalities)**:
   - **Definition**: Kardinalitäten geben an, wie viele Instanzen einer Entität mit wie vielen Instanzen einer anderen Entität in Beziehung stehen können.
   - **Beispiel**:
     - Ein Student kann mehrere Kurse belegen, und jeder Kurs kann von mehreren Studenten belegt werden (Viele-zu-Viele-Beziehung).
     - Ein Professor kann mehrere Kurse lehren, aber jeder Kurs wird nur von einem Professor gelehrt (Eins-zu-Viele-Beziehung).

5. **Schlüssel (Keys)**:
   - **Definition**: Schlüssel sind Attribute oder Kombinationen von Attributen, die dazu dienen, Entitäten eindeutig zu identifizieren.
   - **Beispiel**:
     - Die "Matrikelnummer" eines Studenten ist ein Primärschlüssel, der jeden Studenten eindeutig identifiziert.
     - Die "Kurs-ID" ist ein Primärschlüssel für Kurse.
     - Die "Professoren-ID" ist ein Primärschlüssel für Professoren.

**Beispiel eines ER-Diagramms für eine Universitätsdatenbank:**

- Entitäten: Student, Kurs, Professor
- Attribute:
  - Student: Matrikelnummer (Primärschlüssel), Name, Geburtsdatum, Adresse
  - Kurs: Kurs-ID (Primärschlüssel), Kursname, ECTS-Punkte
  - Professor: Professoren-ID (Primärschlüssel), Name, Fachbereich
- Beziehungen:
  - "belegt" zwischen Student und Kurs
  - "lehrt" zwischen Professor und Kurs
- Kardinalitäten:
  - "belegt": Viele Studenten belegen viele Kurse (Viele-zu-Viele)
  - "lehrt": Ein Professor lehrt viele Kurse, aber jeder Kurs wird nur von einem Professor gelehrt (Eins-zu-Viele)

Dieses ER-Modell stellt die grundlegenden Strukturen und Beziehungen dar, die in einer Universitätsdatenbank vorkommen könnten, und hilft dabei, die Datenbank effizient zu planen und zu gestalten.

Das Entity-Relationship-Model (ER-Modell) eignet sich besonders für verschiedene Zwecke in der Datenbankentwicklung und -verwaltung. Hier sind einige spezifische Anwendungen und Vorteile des ER-Modells:

1. **Datenbankentwurf**:
   - **Zweck**: Das ER-Modell wird verwendet, um die logische Struktur einer Datenbank zu entwerfen. Es hilft dabei, die verschiedenen Entitäten, deren Attribute und die Beziehungen zwischen den Entitäten klar und strukturiert darzustellen.
   - **Vorteil**: Dadurch wird sichergestellt, dass alle relevanten Daten berücksichtigt und korrekt miteinander in Beziehung gesetzt werden, was die Grundlage für die physische Datenbankerstellung bildet.

2. **Dokumentation und Kommunikation**:
   - **Zweck**: Das ER-Modell dient als visuelles und dokumentarisches Werkzeug, um die Datenbankstruktur zu erklären und zu kommunizieren.
   - **Vorteil**: Es erleichtert die Kommunikation zwischen Entwicklern, Datenbankadministratoren und anderen Stakeholdern, da es eine klare und verständliche Darstellung der Datenbankkomponenten bietet.

3. **Anforderungsanalyse**:
   - **Zweck**: Bei der Erhebung der Anforderungen hilft das ER-Modell, die notwendigen Daten und deren Beziehungen zu identifizieren.
   - **Vorteil**: Dies ermöglicht eine präzise und vollständige Erfassung der Benutzeranforderungen und stellt sicher, dass die Datenbank alle notwendigen Funktionen abdeckt.

4. **Systemintegration und Datenmigration**:
   - **Zweck**: Beim Integrieren mehrerer Systeme oder bei der Migration von Daten von einem System auf ein anderes kann das ER-Modell helfen, die Datenstrukturen und -beziehungen klar zu definieren und abzugleichen.
   - **Vorteil**: Dies minimiert das Risiko von Datenverlust oder -inkonsistenzen und stellt sicher, dass die Daten korrekt in die neue Struktur übertragen werden.

5. **Datenbankoptimierung und -erweiterung**:
   - **Zweck**: Das ER-Modell kann verwendet werden, um bestehende Datenbanken zu analysieren und zu optimieren sowie um zukünftige Erweiterungen zu planen.
   - **Vorteil**: Es hilft dabei, ineffiziente Strukturen zu identifizieren und zu verbessern und ermöglicht eine vorausschauende Planung für zukünftige Erweiterungen und Anpassungen der Datenbank.

6. **Bildung und Training**:
   - **Zweck**: Das ER-Modell ist ein wertvolles Lehrmittel, um grundlegende Konzepte der Datenbankentwicklung und -verwaltung zu vermitteln.
   - **Vorteil**: Studierende und neue Mitarbeiter können anhand von ER-Modellen leicht verstehen, wie Datenbanken strukturiert und wie Beziehungen zwischen Daten konzipiert werden.

**Zusammenfassung:**
Das ER-Modell ist besonders nützlich für den logischen Datenbankentwurf, die Dokumentation und Kommunikation der Datenbankstruktur, die Anforderungsanalyse, die Systemintegration und Datenmigration, die Datenbankoptimierung und -erweiterung sowie für Bildungszwecke. Es bietet eine klare und strukturierte Methode, um komplexe Daten und deren Beziehungen zu visualisieren und zu verstehen, was zu effizienteren und effektiveren Datenbanklösungen führt.