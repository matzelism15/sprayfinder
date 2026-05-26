# Entwickler-Dokumentation

Für Leute, die den Code anpassen oder erweitern wollen.

---

## Datei-Struktur

Alles steckt in einer einzigen Datei: `index.html` (~2.150 Zeilen). Bewusst so —
kein Build, keine Dependencies, läuft als statisches File.

Die Datei besteht aus drei Abschnitten:

| Zeilen | Was |
|---|---|
| `1–232` | CSS (im `<style>`-Block) |
| `233–355` | HTML-Markup |
| `357–2143` | JavaScript (im `<script>`-Block) — Farbdatenbank + Logik |

### JS-Layout im Detail

```
Zeile 361   const PAINTS = [ … ]     // ~1.250 Dosenfarben
Zeile 1615  const BRANDS = [ … ]     // 8 unterstützte Linien
Zeile 1620+ Farb-Math (sRGB→Lab, ΔE)
Zeile 1643  function kmeans()        // Cluster-Algorithmus
Zeile 1691+ DOM-Setup, Event-Handler
Zeile 1732  function analyze()       // Haupt-Pipeline
Zeile 1799+ Render-Funktionen
Zeile 1941+ Modal (Farbtausch-Dialog)
Zeile 1991+ Vollbild-Vorschau + Chip-Overlay
```

## Datenmodell

### `PAINTS` — die Farb-Datenbank

Flaches Array aus Objekten, ein Eintrag pro Dose:

```js
{
  b: "MTN 94",          // Brand (muss mit einem Eintrag in BRANDS übereinstimmen)
  c: "RV-1021",         // Hersteller-Code
  n: "Light Yellow",    // Farbname
  h: "#fedf00"          // Hex-Farbwert
}
```

### `BRANDS` — die Hersteller-Linien

```js
const BRANDS = [
  "Montana BLACK", "Montana GOLD",
  "MTN 94",        "MTN Hardcore",
  "Molotow Premium","Molotow FLAME Orange","Molotow FLAME Blue",
  "Loop Colors"
];
```

Reihenfolge bestimmt die Anzeige im UI-Filter.

## Farbabgleich — wie matched die App?

Die Pipeline in `analyze()` (Zeile 1732):

### Schritt 1: Bild herunterskalieren

```js
const SAMPLE = 160;   // längste Seite auf 160 px
```

Sehr kleines Sampling, weil k-means sonst zu langsam wird und das visuelle Ergebnis
sich nicht spürbar verbessert.

### Schritt 2: Pixel → CIE Lab

Jeder gesampelte Pixel wird konvertiert:

```
sRGB → linear RGB  (srgbToLin)
linear RGB → XYZ   (D65 white point)
XYZ → CIE Lab      (rgbToLab)
```

**Warum Lab?** Im Lab-Raum entspricht der euklidische Abstand näherungsweise dem,
was Menschen als „Farbunterschied" wahrnehmen. In RGB ist das nicht so —
z.B. dunkle Grüntöne sind in RGB nahe, sehen aber sehr unterschiedlich aus.

### Schritt 3: k-means Clustering

```js
function kmeans(labs, k, iters = 14)
```

Findet die `K` dominanten Lab-Cluster im Bild. Initialisierung: ersten K Samples;
Iterationen: 14 (empirisch genug für stabile Cluster bei ≤16k Punkten).

### Schritt 4: Cluster → nächste Dose

Für jedes Cluster-Zentrum wird die Dose mit minimaler **ΔE76**-Distanz aus den
aktiven Hersteller-Linien gesucht:

```js
function deltaE(a, b) {  // CIE76 – schnell, ausreichend
  const dL = a[0]-b[0], da = a[1]-b[1], db = a[2]-b[2];
  return Math.sqrt(dL*dL + da*da + db*db);
}
```

Hinweis: ΔE94 oder ΔE2000 wären präziser, aber CIE76 ist deutlich schneller und
für diese Use-Case (Vorschläge, keine Druckfreigabe) reicht es klar aus.

### Schritt 5: Match-Qualität als Label

Die Distanz wird in drei Klassen gemappt (Funktion `matchTag`):

| ΔE-Wert | Label | Farbe |
|---|---|---|
| < 5  | „Sehr gut" | grün |
| < 12 | „OK"      | gelb |
| ≥ 12 | „Annäherung" | rot |

## Eine neue Hersteller-Linie hinzufügen

### 1. Linie in `BRANDS` eintragen

```js
const BRANDS = [
  "Montana BLACK", "Montana GOLD",
  "MTN 94", "MTN Hardcore",
  "Molotow Premium","Molotow FLAME Orange","Molotow FLAME Blue",
  "Loop Colors",
  "Neue Marke X"   // ← hier
];
```

### 2. Farben in `PAINTS` ergänzen

Direkt unten an das Array anhängen, mit demselben Brand-String wie in `BRANDS`:

```js
{b:"Neue Marke X", c:"NX-001", n:"Pure White", h:"#FFFFFF"},
{b:"Neue Marke X", c:"NX-002", n:"Coal Black", h:"#1a1a1a"},
// …
```

**Wichtig:**
- `b` muss exakt mit dem `BRANDS`-Eintrag übereinstimmen (case-sensitive)
- `h` muss ein valider 6-stelliger Hex-Code sein (`#RRGGBB`)
- `c` und `n` sind reine Anzeige-Strings, dürfen Sonderzeichen enthalten

### 3. Testen

`index.html` im Browser öffnen → neue Linie sollte als Checkbox im Filter erscheinen.
Mit Test-Bild prüfen, ob Vorschläge plausibel sind.

## Eine einzelne Farbe ändern

Suchen + ersetzen. Beispiel: Hex-Code für `RV-1021` korrigieren:

```bash
grep -n "RV-1021" index.html
# zeigt die Zeile, dort den h-Wert anpassen
```

## Algorithmus tunen

| Was tunen | Wo | Default | Was passiert |
|---|---|---|---|
| Sampling-Auflösung | `SAMPLE` in `analyze()` | 160 | höher = präziser, langsamer |
| k-means-Iterationen | Default-Parameter von `kmeans()` | 14 | höher = stabilere Cluster |
| Match-Schwellen | `matchTag()` | 5 / 12 | strenger oder lockerer |
| Farbraum | `rgbToLab` / `deltaE` | CIE76 | bei Bedarf auf ΔE2000 wechseln |

## Lokale Entwicklung

Keine Tools nötig:

```bash
open ~/Documents/sprayfinder/index.html
```

Live-Reload bei jeder Änderung: Browser neu laden (Cmd+R). Falls Browser-Cache
nervt: Hard-Reload mit Cmd+Shift+R oder DevTools öffnen + „Disable cache" aktivieren.

## Bekannte Einschränkungen

- **K=16 ist hartes Limit** im UI. Mehr macht visuell kaum Sinn und wird langsam.
- **Datenbank ist Snapshot** — Hersteller ändern Sortimente. Stand: Frühjahr 2026.
- **Keine RAL/Pantone-Mappings.** Nur Dosenfarben sind hinterlegt.
- **ΔE76 statt ΔE2000.** Bei extrem ähnlichen Tönen (z.B. zwei sehr dunkle Blautöne)
  kann die Reihenfolge der Treffer minimal danebenliegen.

## Pull Requests willkommen

Wenn du eine Hersteller-Linie pflegst und deren aktuelle Farbdaten teilen willst:
PR mit Quelle in der Commit-Message (z.B. Link zur Hersteller-Seite).
