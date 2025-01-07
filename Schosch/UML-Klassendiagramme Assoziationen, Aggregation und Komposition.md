In UML (Unified Modeling Language) Diagrammen werden **Assoziationen**, **Aggregation** und **Komposition** verwendet, um Beziehungen zwischen Klassen oder Objekten zu modellieren. Diese Konzepte sind zentrale Bestandteile der objektorientierten Modellierung und helfen dabei, die Art und Weise zu beschreiben, wie Klassen miteinander verbunden sind. 

Hier sind die Definitionen und die Darstellung dieser Konzepte in einem Klassendiagramm:

---

### 1. **Assoziation**

#### Bedeutung:
- Eine **Assoziation** beschreibt eine **allgemeine Beziehung** zwischen zwei Klassen. Sie stellt eine einfache Verbindung zwischen Objekten dar, bei der es keine besonderen Besitzverhältnisse oder hierarchischen Strukturen gibt.
- Es wird auch beschrieben, wie eine Klasse mit einer anderen interagiert.

#### Darstellung:
- In UML wird eine Assoziation durch eine **gerade Linie** zwischen zwei Klassen dargestellt.
- Optional kann eine **Kardinalität** (z.B. 1..*, 0..1) an den Enden der Linie hinzugefügt werden, um anzugeben, wie viele Instanzen der einen Klasse mit Instanzen der anderen Klasse verbunden sind.

#### Beispiel:
```plaintext
+------------+        +------------+
|  Person    |--------|  Adresse   |
+------------+        +------------+
```
In diesem Beispiel hat die Klasse `Person` eine Assoziation zur Klasse `Adresse`, die eine einfache Verbindung darstellt, ohne Besitzverhältnisse.

---

### 2. **Aggregation**

#### Bedeutung:
- **Aggregation** beschreibt eine **"Teil-Ganzes"-Beziehung**, bei der ein Objekt aus mehreren Objekten besteht, aber diese Objekte auch eigenständig existieren können. Es ist eine **schwache** Form der Komposition.
- Ein Objekt in einer Aggregation kann ohne das "ganze" Objekt existieren, was eine lockerere Bindung darstellt.

#### Darstellung:
- In UML wird Aggregation durch eine **Linie** mit einer **hohlen Raute** an dem Ende der Linie dargestellt, das die "Ganzes"-Seite zeigt.

#### Beispiel:
```plaintext
+----------------+        +------------------+
|  Auto          |<>------|  Rad             |
+----------------+        +------------------+
```
In diesem Beispiel ist `Auto` das "ganzes" Objekt, und `Rad` ist ein "Teil"-Objekt. Ein `Rad` kann auch ohne ein `Auto` existieren, daher handelt es sich um eine Aggregation.

---

### 3. **Komposition**

#### Bedeutung:
- **Komposition** ist eine **stärkere Form der Aggregation** und beschreibt eine **"Teil-Ganzes"-Beziehung**, bei der das Teil ohne das Ganze **nicht existieren kann**. Wenn das "ganze" Objekt gelöscht wird, werden auch alle zugehörigen "Teile" gelöscht.
- Komposition impliziert eine **starke Besitzbeziehung**.

#### Darstellung:
- In UML wird Komposition durch eine **Linie** mit einer **gefüllten Raute** an dem Ende der Linie dargestellt, das das "Ganze" zeigt.

#### Beispiel:
```plaintext
+----------------+        +------------------+
|  Haus          |<>------|  Zimmer          |
+----------------+        +------------------+
```
In diesem Beispiel ist `Haus` das "ganzes" Objekt, und `Zimmer` ist ein "Teil"-Objekt. Ein `Zimmer` kann ohne ein `Haus` nicht existieren – wenn das `Haus` zerstört wird, verschwinden auch die `Zimmer`.

---

### Zusammenfassung der Unterschiede:

| Beziehung     | Bedeutung                                      | Darstellung in UML  |
|---------------|------------------------------------------------|---------------------|
| **Assoziation** | Allgemeine Beziehung zwischen Klassen        | Gerade Linie        |
| **Aggregation** | "Teil-Ganzes"-Beziehung, aber Teile können ohne das Ganze existieren | Hohle Raute         |
| **Komposition** | Stärkere "Teil-Ganzes"-Beziehung, bei der Teile ohne das Ganze nicht existieren können | Gefüllte Raute      |

---

### Beispiel mit allen Beziehungen:

```plaintext
+----------------+        +------------------+
|  Firma         |<>------|  Abteilung       |
+----------------+        +------------------+
          |
          | (Aggregation)
          |
+----------------+        +------------------+
|  Mitarbeiter   |<>------|  Adresse         |
+----------------+        +------------------+
```

- **Firma und Abteilung**: **Aggregation**, da eine `Abteilung` ohne die `Firma` existieren kann.
- **Firma und Mitarbeiter**: **Komposition**, da `Mitarbeiter` ohne `Firma` nicht existieren kann.
- **Mitarbeiter und Adresse**: **Assoziation**, da es sich um eine einfache Beziehung handelt.

Diese Darstellungen und das Verständnis der verschiedenen Beziehungstypen helfen, die Struktur und die Interaktion zwischen Objekten oder Klassen in objektorientierten Systemen zu modellieren.