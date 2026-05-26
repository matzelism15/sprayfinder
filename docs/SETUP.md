# Setup-Notizen

Notizen für mich selbst (oder dich, falls du das nochmal so aufsetzen willst).
Keine Anleitung für Endnutzer.

---

## 1. Was wo liegt

| Was | Wo |
|---|---|
| Lokales Arbeitsverzeichnis | `~/Documents/sprayfinder/` |
| GitHub-Repo | https://github.com/matzelism15/sprayfinder |
| Live-Seite | https://matzelism15.github.io/sprayfinder/ |
| GoatCounter (Analytics) | `https://sprayfinder.goatcounter.com/` *(siehe Abschnitt 4)* |

Repo-Inhalt:

```
sprayfinder/
├── index.html          ← die komplette App in einer Datei
├── README.md           ← Nutzer-Doku
├── .gitignore          ← nur .DS_Store
└── docs/
    ├── SETUP.md        ← dieses Dokument
    └── DEVELOPER.md    ← Code-Doku
```

## 2. Hosting: GitHub Pages

GitHub Pages serviert `index.html` aus dem `main`-Branch, Root-Pfad.

### Aktivieren (einmalig, ist schon gemacht)

Per `gh` CLI:

```bash
gh api -X POST repos/matzelism15/sprayfinder/pages \
  -f 'source[branch]=main' \
  -f 'source[path]=/'
```

Oder per Web-UI: Repo → **Settings** → **Pages** → Source: *Deploy from branch*, Branch: `main`, Folder: `/ (root)`.

### Wie ein Update live geht

Datei ändern → committen → pushen. Pages baut automatisch in ~30–60 s neu.

```bash
cd ~/Documents/sprayfinder
# index.html bearbeiten
git add index.html
git commit -m "Was wurde geändert"
git push
```

Build-Status checken:

```bash
gh api repos/matzelism15/sprayfinder/pages/builds/latest --jq '.status'
# erwartete Werte: "queued" → "building" → "built"
```

## 3. Git-Identität für dieses Repo

Lokal nur für dieses Repo gesetzt (nicht global):

```bash
git config user.name  "Matthias Nigmann"
git config user.email "matthias.nigmann@gmail.com"
```

## 4. Analytics: GoatCounter (geplant / noch nicht eingebaut)

GoatCounter ist eine datenschutzfreundliche Alternative zu Google Analytics:
keine Cookies, keine IP-Speicherung, DSGVO-freundlich, free für Hobby.

### Anmeldung

1. https://www.goatcounter.com/signup
2. Site code: `sprayfinder` → URL wird `sprayfinder.goatcounter.com`
3. Domain: `matzelism15.github.io`
4. Plan: **Personal (free)** — reicht bis ~100.000 Pageviews/Monat

### Einbau in `index.html`

Direkt vor `</body>` einfügen (oder vor `</head>`):

```html
<script data-goatcounter="https://sprayfinder.goatcounter.com/count"
        async src="//gc.zgo.at/count.js"></script>
```

Danach committen + pushen — Pages baut neu und ab dem nächsten Aufruf zählt es.

### Dashboard

`https://sprayfinder.goatcounter.com` — zeigt Pageviews, Referrer, Land, Browser,
**aber keine personenbezogenen Daten** (keine IPs, keine Namen).

## 5. Häufige Befehle (Cheat Sheet)

```bash
# Lokale Vorschau (einfach Datei im Browser öffnen)
open ~/Documents/sprayfinder/index.html

# Änderung pushen
cd ~/Documents/sprayfinder && git add -A && git commit -m "msg" && git push

# Pages-Status prüfen
gh api repos/matzelism15/sprayfinder/pages/builds/latest --jq '.status'

# Repo im Browser öffnen
gh repo view matzelism15/sprayfinder --web

# Aktuellen Traffic anschauen (Repo, nicht Pages — Pages hat keine Stats von GitHub)
open "https://github.com/matzelism15/sprayfinder/graphs/traffic"
```

## 6. Wenn was kaputt geht

| Problem | Lösung |
|---|---|
| Pages baut nicht | `gh api repos/matzelism15/sprayfinder/pages/builds/latest` zeigt Fehler |
| Live-Seite zeigt alte Version | Browser-Cache leeren (Cmd+Shift+R), oder ~2 Min warten |
| `git push` will Passwort | `gh auth login` (sollte schon erledigt sein) |
| Domain als 404 | Pages braucht 1–2 Min nach erstmaligem Aktivieren |
