# Sprühfarben-Finder

Vom Entwurf zur richtigen Dose — eine Browser-App, die ein Bild analysiert und passende
Sprühdosen vorschlägt. Funktioniert komplett im Browser, ohne Anmeldung, ohne Upload.

**👉 Direkt im Browser öffnen:** https://matzelism15.github.io/sprayfinder/

---

## Was die App macht

Du lädst ein Foto oder einen Entwurf hoch, und der Sprühfarben-Finder

1. erkennt die wichtigsten Farben im Bild (du wählst wie viele — 2 bis 16),
2. sucht aus über 1.000 echten Dosenfarben die ähnlichste pro Farbgruppe heraus,
3. zeigt dir eine Vorschau, wie dein Motiv mit diesen Dosenfarben aussehen würde,
4. listet die nötigen Dosen mit **Hersteller, Farbname und Code** auf,
5. lässt dich jede vorgeschlagene Farbe manuell durch eine andere ersetzen,
6. zeigt die prozentuale Verteilung jeder Farbe im Bild,
7. erlaubt dir, die Vorschau als PNG zu speichern.

## Unterstützte Hersteller

- **Montana Cans** — BLACK und GOLD Serien
- **Montana Colors (MTN)** — 94er und Hardcore
- **Molotow** — Premium, FLAME Orange, FLAME Blue
- **Loop Colors**

Stand der Farbdatenbank: Frühjahr 2026.

## Anleitung

### 1. Bild hochladen
Klicke auf das Upload-Feld links oder zieh ein Bild per Drag & Drop hinein.
Unterstützt werden JPG, PNG und WEBP.

### 2. Anzahl der Zielfarben wählen
Der Schieberegler legt fest, wie viele Farbgruppen die App im Bild sucht.
- **2–4** für stark stilisierte Motive (Schablonen, Logos)
- **6–8** für typische Graffiti-Entwürfe
- **10+** für detailreiche Bilder mit Verläufen

### 3. Hersteller auswählen
Hak nur die Linien an, die du tatsächlich besitzt oder kaufen willst.
So bekommst du keine Empfehlungen für Dosen, die du nicht nutzen kannst.

### 4. „Farben analysieren" klicken
Die App rechnet wenige Sekunden und zeigt:
- **Original** vs **Vorschau in Dosenfarben** (Seite an Seite)
- Eine Verteilungsleiste mit den Prozentanteilen
- Eine Liste empfohlener Dosen mit Match-Qualität (grün = sehr gut, gelb = ok, rot = nur Annäherung)

### 5. Farben anpassen (optional)
- **Im Vorschaubild klicken** → öffnet eine Liste ähnlicher Dosen zum Tauschen
- **Im Listen-Bereich** den Farb-Picker nutzen
- **Vorschau vergrößern** über das Symbol oben rechts im Vorschaubild

### 6. Speichern
- **Vorschau speichern (PNG)** für deine Unterlagen
- Die **Einkaufsliste** unten zeigt alle nötigen Dosen mit Code

## Datenschutz

- Alle Bilder werden **ausschließlich lokal im Browser** verarbeitet
- Es wird **nichts hochgeladen, nichts gespeichert, nichts versendet**
- Funktioniert auch komplett **offline**, sobald die Seite einmal geladen ist
- Du kannst `index.html` einfach speichern und ohne Internet weiter nutzen

## Lokal nutzen

Falls du die App ohne GitHub Pages nutzen willst:

1. `index.html` aus dem Repo herunterladen
2. Datei doppelklicken — fertig

Keine Abhängigkeiten, kein Build, kein Server nötig.

## Weiterführende Dokumentation

- [`docs/SETUP.md`](docs/SETUP.md) — Wie dieses Repo + Hosting aufgesetzt sind (Notizen für mich selbst)
- [`docs/DEVELOPER.md`](docs/DEVELOPER.md) — Code-Struktur, Farb-Algorithmus, neue Hersteller hinzufügen

## Lizenz

Frei nutzbar für persönliche und kommerzielle Zwecke. Hersteller- und Farbnamen
sind Marken der jeweiligen Eigentümer; sie werden hier nur zur Referenz genannt.
