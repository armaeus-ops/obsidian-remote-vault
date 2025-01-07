Eine JSON-Datei ist eine Textdatei, die Daten im Format von JavaScript Object Notation (JSON) speichert. JSON ist ein standardisiertes Datenformat, das vor allem verwendet wird, um strukturierte Informationen zwischen Webanwendungen und Servern auszutauschen. Der Vorteil von JSON ist, dass es leicht zu lesen und für viele Programmiersprachen einfach zu verarbeiten ist.

Aufbau einer JSON-Datei:

- Schlüssel-Wert-Paaren (ähnlich wie in Python-Dictionaries oder JavaScript-Objekten),
- Arrays (Listen) und
- Werten, die z. B. Text, Zahlen, Wahrheitswerte, Arrays oder Objekte sein können.

Beispiel für eine JSON-Datei, die Daten über eine Person enthält:

![](https://ilias.bs-technik-rostock.de:7777/data/bstechnikhro/mobs/mm_83170/JSON.png?il_wac_token=885f3fcd7567593dc2f74e96d952985d87cbe16b&il_wac_ttl=3&il_wac_ts=1731999402)

Im Beispiel oben haben wir:

- Schlüssel wie `"name"`, `"alter"`, und `"adresse"`.
- Werte wie Text (`"Max Mustermann"`), Zahlen (`30`), Listen (`["Programmieren", "Lesen", "Wandern"]`), und geschachtelte Objekte (die `"adresse"`).

Verwendung von JSON-Dateien:

JSON-Dateien werden häufig in Webanwendungen und APIs verwendet, um Daten zwischen verschiedenen Systemen auszutauschen. Sie bieten eine einfache Möglichkeit, strukturierte Informationen zu speichern, die von Programmiersprachen wie JavaScript, Python, Java, und vielen anderen interpretiert werden können.

Vorteile von JSON:

1. Einfaches Format: JSON ist textbasiert und kann von Menschen gelesen und verstanden werden.
2. Kompatibilität: JSON ist weit verbreitet und wird von vielen Programmiersprachen und Systemen unterstützt.
3. Flexibilität: JSON ermöglicht verschachtelte Datenstrukturen (Objekte innerhalb von Objekten, Arrays etc.), die komplexe Datensätze leicht abbilden.

JSON-Dateien haben üblicherweise die Endung `.json`.

Aufgabe:  
  
- Erweitern Sie die obige Datei um relevante Daten für Berufsschüler.
```
{
  "name": "Max Mustermann",
  "alter": 30,
  "email": "max@beispiel.de",
  "interessen": ["Programmieren", "Lesen", "Wandern"],
  "adresse": {
    "strasse": "Musterstraße 1",
    "stadt": "Musterstadt",
    "postleitzahl": 12345
  },
  "berufsschule": {
    "name": "Berufsschule Musterstadt",
    "studiengang": "Fachinformatiker Anwendungsentwicklung",
    "klassenname": "FI22B",
    "jahrgang": 2022,
    "abschlussdatum": "2025-07-15"
  },
  "praktika": [
    {
      "unternehmen": "IT Solutions GmbH",
      "startdatum": "2023-03-01",
      "enddatum": "2023-06-30",
      "bereich": "Softwareentwicklung"
    },
    {
      "unternehmen": "WebDev AG",
      "startdatum": "2024-02-01",
      "enddatum": "2024-04-30",
      "bereich": "Frontend-Entwicklung"
    }
  ],
  "fertigkeiten": ["Java", "HTML", "CSS", "JavaScript", "Git"],
  "zertifikate": [
    {
      "titel": "Java OCA Zertifizierung",
      "ausgestellt_von": "Oracle",
      "datum": "2023-09-15"
    },
    {
      "titel": "Grundlagen von Git",
      "ausgestellt_von": "Udemy",
      "datum": "2022-11-10"
    }
  ]
}

```



----

# XML

Eine XML-Datei ist eine Textdatei, die Daten im eXtensible Markup Language (XML)-Format speichert. XML ist eine Markup-Sprache, die entwickelt wurde, um Daten strukturiert und plattformunabhängig zu speichern und auszutauschen. Anders als JSON (das hauptsächlich für die Datenübertragung entwickelt wurde), wurde XML ursprünglich als universelles Format für strukturierte Dokumente geschaffen, aber wird heute in vielen Anwendungsbereichen eingesetzt.

Aufbau einer XML-Datei:

XML-Dokumente bestehen aus hierarchischen Elementen mit sogenannten Tags und Attributen. Ein XML-Dokument hat eine baumartige Struktur, in der die Elemente ineinander verschachtelt sein können.

Hier ein Beispiel einer XML-Datei, die Daten über eine Person enthält:

![](https://ilias.bs-technik-rostock.de:7777/data/bstechnikhro/mobs/mm_83171/XML.png?il_wac_token=f4948d613b01bfbd8761eb2ee63095eba293ef2f&il_wac_ttl=3&il_wac_ts=1731999586)

Bestandteile einer XML-Datei:

- Elemente: Ein XML-Dokument besteht aus Elementen, die durch öffnende und schließende Tags gekennzeichnet sind, z. B. `<Name>Max Mustermann</Name>`.
- Attribute: Elemente können Attribute enthalten, die zusätzliche Informationen bereitstellen, z. B. `<Person Geschlecht="männlich">`.
- Wurzelelement: Jedes XML-Dokument hat genau ein Wurzelelement, in diesem Beispiel `<Person>`, das alle anderen Elemente umschließt.

Verwendung von XML-Dateien:

XML-Dateien werden oft in verschiedenen Anwendungsbereichen verwendet, etwa:

1. Webservices und APIs: Früher war XML das Standardformat für den Datenaustausch in APIs (z. B. SOAP-basierte Webservices).
2. Konfigurationsdateien: Viele Anwendungen, insbesondere in Java, .NET und Android, verwenden XML für Konfigurationen.
3. Dokumentenmanagement: XML wird häufig für standardisierte Dokumente und formale Beschreibungen (z. B. Microsoft Office-Dateiformate wie .docx, .xlsx) verwendet.

Unterschiede zu JSON:

- Markup-Struktur: XML verwendet eine Tag-Struktur ähnlich wie HTML, was es eher dokumentenorientiert macht.
- Flexibilität bei Datentypen: JSON hat spezielle Datentypen (wie Listen und Wahrheitswerte), während XML alles als Text speichert, was eine zusätzliche Verarbeitung erfordert.
- Lesbarkeit: JSON ist oft einfacher zu lesen und hat weniger "Overhead" durch Tags, weshalb es in modernen Anwendungen und APIs oft bevorzugt wird.

Vorteile von XML:

1. Flexibilität: XML eignet sich sowohl für einfache als auch für komplexe, stark verschachtelte Datenstrukturen.
2. Selbstbeschreibend: Die Struktur und Hierarchie der Daten sind leicht durch die Tags erkennbar.
3. Standardisierung und Validierung: XML-Dokumente können mit Hilfe von DTDs (Document Type Definitions) oder XML-Schemas validiert werden, was die Datenqualität und -integrität sicherstellt.

XML-Dateien haben die Dateiendung `.xml`.

Aufgabe:  
  
- Erweitern Sie die obige Datei um relevante Daten für den nächsten Urlaub.
```
<Person>
  <Name>Max Mustermann</Name>
  <Alter>30</Alter>
  <Email>max@beispiel.de</Email>
  <Interessen>
    <Interesse>Programmieren</Interesse>
    <Interesse>Lesen</Interesse>
    <Interesse>Wandern</Interesse>
  </Interessen>
  <Adresse>
    <Strasse>Musterstraße 1</Strasse>
    <Stadt>Musterstadt</Stadt>
    <Postleitzahl>12345</Postleitzahl>
  </Adresse>
  <Urlaub>
    <Reiseziel>Mallorca</Reiseziel>
    <Reisedauer>
      <Startdatum>2024-06-15</Startdatum>
      <Enddatum>2024-06-22</Enddatum>
    </Reisedauer>
    <Unterkunft>
      <Name>Hotel Paradiso</Name>
      <Adresse>
        <Strasse>Playa Boulevard 45</Strasse>
        <Stadt>Palma</Stadt>
        <Postleitzahl>07001</Postleitzahl>
      </Adresse>
      <Zimmer>Deluxe Suite mit Meerblick</Zimmer>
    </Unterkunft>
    <Aktivitäten>
      <Aktivität>Wanderung im Tramuntana-Gebirge</Aktivität>
      <Aktivität>Besuch der Kathedrale von Palma</Aktivität>
      <Aktivität>Schnorcheln in Cala Mondragó</Aktivität>
    </Aktivitäten>
    <Reisebudget>1500 EUR</Reisebudget>
  </Urlaub>
</Person>
```
