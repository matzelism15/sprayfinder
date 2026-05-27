# Wie der Sprühfarben-Finder Farben matched

Eine technische Erklärung des Farbabgleichs zwischen einem hochgeladenen Bild und der Datenbank realer Sprühdosen.

---

## Die Grundidee

Die Anwendung sucht in einem Bild die wichtigsten Farbgruppen, übersetzt jede davon in einen Zahlenwert, der menschliche Farbwahrnehmung widerspiegelt, und findet anschließend diejenige Dose aus der Datenbank, deren Wert am dichtesten dran liegt.

So einfach das klingt — jeder einzelne Schritt löst ein konkretes Problem.

---

## 1. Das Bild verkleinern

Bevor irgendetwas analysiert wird, wird das Eingabebild auf eine längste Seitenlänge von **160 Pixel** heruntergerechnet.

**Warum?** Ein typisches Foto enthält mehrere Millionen Pixel. Ein vollständiger Vergleich aller Pixel würde Minuten dauern. 160 px reichen für die Aufgabe völlig aus, weil keine Bilddetails gesucht werden, sondern *dominante Farben*. Für die statistische Verteilung von Farben ist die Auflösung weitgehend irrelevant.

Konkret: aus einem 4000 × 3000 Pixel großen Bild werden etwa 19 000 Farbpunkte, die analysiert werden.

---

## 2. Pixel in den menschlichen Farbraum übersetzen

### Das Problem mit RGB

Computer speichern Farben üblicherweise als drei Werte (Rot, Grün, Blau, je 0–255). Dieser Farbraum ist jedoch **nicht wahrnehmungstreu**:

Beispiel:

| Vergleich | RGB-Distanz | Visueller Eindruck |
|---|---|---|
| `#00FF00` (grell-grün) gegen `#00CC00` (dunkler grün) | 51 | Sehr deutlich verschieden |
| `#000033` (dunkelblau) gegen `#000000` (schwarz) | 51 | Kaum wahrnehmbar |

Gleiche RGB-Distanz, völlig unterschiedlicher Eindruck. Damit lässt sich kein sinnvoller Farbabgleich bauen.

### Die Lösung: CIE Lab

Der Lab-Farbraum wurde von der *Commission internationale de l'éclairage* genau dafür entworfen: Gleiche Zahlendistanz entspricht annähernd gleichem wahrgenommenem Unterschied. Drei Achsen:

| Achse | Bedeutung |
|---|---|
| **L** (0–100) | Helligkeit — schwarz unten, weiß oben |
| **a** | Rot-Grün-Achse — negativ = grün, positiv = rot |
| **b** | Blau-Gelb-Achse — negativ = blau, positiv = gelb |

Jeder Pixel des verkleinerten Bildes wird umgerechnet:

```
sRGB → linear RGB → XYZ → Lab
```

Dieser Umrechnungspfad ist mathematisch eindeutig festgelegt und entspricht der internationalen Norm.

---

## 3. Die dominanten Farben finden (k-means-Clustering)

Nach Schritt 2 liegen etwa 19 000 Lab-Farbpunkte vor — aber das Bild enthält selten 19 000 unterschiedliche Farben. Die meisten Pixel sind sehr ähnlich (Himmel, Hintergrund, Hautpartien, Schatten).

Der Nutzer wählt eine Zielanzahl an Farbgruppen — typisch 2 bis 16. Daraus extrahiert der Algorithmus k-means die dominanten Farben:

1. **Initialisierung:** *K* Startpunkte werden im Lab-Raum verteilt
2. **Zuordnung:** Jeder Pixel wird dem nächstgelegenen Startpunkt zugeordnet
3. **Verschiebung:** Jeder Startpunkt wandert in den Mittelpunkt seiner zugeordneten Pixel
4. **Wiederholung:** Schritte 2 und 3 werden 14 Mal iteriert

Anschaulich: K Magneten wandern in einer Wolke aus farbigen Punkten umher, bis sich für jeden ein eigenes Einzugsgebiet stabilisiert hat.

Ergebnis: *K* Lab-Werte, die je eine dominante Bildfarbe repräsentieren. Zusätzlich bekannt: wie viele Pixel jeweils zugeordnet wurden — daraus ergibt sich die prozentuale Verteilung, die in der Anwendung als Balkendiagramm dargestellt wird.

---

## 4. Die passende Dose finden

Die Datenbank enthält rund 1 250 Dosenfarben verschiedener Hersteller (Montana Cans, Montana Colors, Molotow Premium, Loop Colors). Jede Dose ist bereits vorab in ihren Lab-Wert umgerechnet.

Für jede der *K* dominanten Bildfarben wird die nächstgelegene Dose gesucht:

```
Für jede Dose der vom Nutzer ausgewählten Hersteller:
  Berechne den Abstand zwischen Bildfarbe und Dosenfarbe.
Wähle die Dose mit dem kleinsten Abstand.
```

Als Abstand dient **ΔE** (Delta-E) — die offiziell standardisierte perzeptuelle Farbdistanz. Die hier verwendete Formel ist CIE76:

```
ΔE = √( (L₁ − L₂)² + (a₁ − a₂)² + (b₁ − b₂)² )
```

Mathematisch der Satz des Pythagoras im dreidimensionalen Lab-Raum. Genau hier zahlt sich der Umweg über den Lab-Farbraum aus: der ΔE-Wert korreliert mit dem tatsächlichen visuellen Unterschied.

---

## 5. Qualität des Treffers bewerten

Der ΔE-Wert wird nicht nur als Zahl ausgegeben, sondern in drei Qualitätsklassen übersetzt, die sich an Standardwerten der Farbforschung orientieren:

| ΔE-Wert | Wahrnehmung | Anzeige |
|---|---|---|
| < 2 | Nicht unterscheidbar (kommt selten vor) | — |
| < 5 | Sehr ähnlich, nur Fachpersonal erkennt Unterschied | **Sehr gut** |
| < 12 | Deutlich erkennbar verschieden, gleiche Farbfamilie | **OK** |
| ≥ 12 | Klar verschieden, nur Annäherung | **Annäherung** |

Diese Labels erscheinen rechts neben jeder Empfehlung in der Anwendung.

---

## Konkretes Beispiel

Beispielpixel im Eingabebild: `#E2231A` (knalliges Rot).

| Schritt | Wert |
|---|---|
| RGB | (226, 35, 26) |
| Umrechnung in Lab | L = 49, a = 70, b = 53 |
| Zuordnung durch k-means | Cluster #3, Zentrum bei L = 51, a = 68, b = 51 |
| Nächste Dose in der Datenbank | **MTN 94 RV-3020 „Tomato Red"** mit Lab (50, 71, 53) |
| ΔE-Berechnung | √((50−51)² + (71−68)² + (53−51)²) ≈ 3,7 |
| Bewertung | **Sehr gut** |

---

## Wie die Hex-Werte der Datenbank entstehen

Die in der Anwendung hinterlegten Hex-Werte sind keine spektrometrischen Messungen, sondern aus mehreren öffentlich zugänglichen Quellen kuratiert. Vier typische Wege:

**Quelle 1 — Direkter Hex vom Hersteller.**
Montana Cans, Montana Colors und teilweise Loop Colors veröffentlichen auf ihren Webseiten zu jeder Dose einen RGB- oder Hex-Wert. Beispiel: die Produktseite zu MTN 94 RV-1021 enthält den Wert direkt. Dieser wird unverändert übernommen.

**Quelle 2 — Über RAL-Konvertierung.**
Viele Dosen orientieren sich an RAL-Industriefarben (etwa `RV-7016` → RAL 7016). Für RAL existieren offiziell standardisierte sRGB-Konvertierungen, aus denen sich der Hex-Wert eindeutig ableiten lässt.

**Quelle 3 — Auslesen aus Produktbildern.**
Wenn der Hersteller nur einen visuellen Swatch ohne Zahlenwert zeigt, wird die Farbe per Color-Picker aus dem Bild ausgelesen. Diese Methode ist die unzuverlässigste, weil Produktfotos nicht farbkalibriert sind.

**Quelle 4 — Manuelle Korrektur und Konsistenzprüfung.**
Nach dem Sammeln wird der Datensatz quergeprüft: Passt der Farbname zum Hex, sind ähnlich klingende Töne zwischen Herstellern in einem plausiblen Verhältnis, sind offensichtliche Fehler vorhanden.

### Warum ein Hex-Wert nie die exakte Realität trifft

Selbst bei sorgfältiger Kuratierung bleiben grundsätzliche Unschärfen:

| Faktor | Auswirkung |
|---|---|
| Monitor-Kalibrierung | Jeder Bildschirm zeigt denselben Hex-Wert leicht anders an. |
| Nicht farbkalibrierte Herstellerfotos | Auch offizielle Produktbilder sind selten unter normierten Lichtbedingungen entstanden. |
| Glanzgrad | Matte und glänzende Dose mit identischem Pigment haben denselben Hex, wirken aber sichtbar unterschiedlich. |
| Untergrund-Effekt | Pigment auf weißer gegen schwarze Wand erscheint anders. Der Hex codiert nur die Reinfarbe. |
| Chargenstreuung | Auch der Hersteller hat Pigment-Toleranzen — zwei Dosen derselben Sorte können messbar abweichen. |

Eine wirklich präzise Datenbank würde spektrometrische Messungen echter Dosenmuster erfordern. Solche Daten werden von Herstellern nicht öffentlich bereitgestellt.

### Konsequenz für die Anwendung

Die Empfehlungen sind **fundierte Annäherungen, keine Garantie**. Bei einem ΔE-Wert unter 5 („Sehr gut") liegt die vorgeschlagene Dose visuell sicher in der richtigen Farbfamilie — die exakte Nuance vor Ort weicht erfahrungsgemäß um wenige Prozent ab. Bei ΔE-Werten über 12 („Annäherung") existiert im ausgewählten Hersteller-Sortiment schlicht keine passende Dose, und der Vorschlag ist nur ein Notnagel.

Empfehlung für ernsthafte Projekte: vor dem Kauf einen Test-Spray auf weißem Papier anfertigen und unter realem Licht prüfen. Der Sprühfarben-Finder ersetzt diese Kontrolle nicht; er verkürzt nur die Vorauswahl von rund 1 250 Dosen auf eine Handvoll Kandidaten.

---

## Grenzen des Verfahrens

- **Verläufe** werden nicht als Verläufe erkannt. k-means arbeitet auf einer Punktwolke und ignoriert räumliche Zusammenhänge. Ein blauer Verlauf zu Lila wird in zwei separate Cluster aufgeteilt.
- **Druckqualität-Genauigkeit** wird nicht erreicht. Die hier verwendete ΔE76-Formel ist schnell, aber gröber als die heute übliche ΔE2000. Für die Anwendung — Vorschläge für Sprühdosen, nicht Druckvorstufe — reicht ΔE76 deutlich aus.
- **Glanzgrad und Textur** werden nicht abgebildet. Eine matte und eine glänzende Dose mit identischem Pigment haben denselben Hex-Wert und werden gleich behandelt.
- **Aktualität der Datenbank.** Hersteller ändern ihre Sortimente. Der aktuelle Stand entspricht dem Frühjahr 2026.

---

## Zusammenfassung in einem Satz

Bild auf 160 Pixel verkleinern → jeden Pixel in den Lab-Farbraum umrechnen → mit k-means die *K* dominanten Farben extrahieren → für jede die Dose aus der Datenbank mit minimaler ΔE-Distanz auswählen.

---

*Sprühfarben-Finder · https://matzelism15.github.io/sprayfinder/*
