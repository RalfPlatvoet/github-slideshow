# PPI Strategy Navigator – Projektkontext

## Übersicht

Mobile-first Progressive Web App (PWA) für Management-Consultants. Läuft vollständig offline nach dem ersten Laden. Alle Daten werden lokal im `localStorage` gespeichert, kein Backend erforderlich.

**Deployment-Ziel:** `platvoet.org/strategy-navigator/`
**Branch:** `claude/ppi-strategy-navigator-pwa-ZaXaW`

---

## Branding

| Element | Wert |
|---|---|
| Name | PPI Strategy Navigator |
| Short Name | Strategy Nav |
| Primärfarbe (Navy) | `#1B2A4A` |
| Akzentfarbe (Gold) | `#C9A84C` |
| Heading-Font | Playfair Display (Google Fonts) |
| Body-Font | system-ui |

---

## Dateistruktur

```
strategy-navigator/
├── index.html       # Gesamte App (HTML + CSS + JS, all-in-one)
├── sw.js            # Service Worker (Cache-First, vollständig offline)
├── manifest.json    # PWA-Manifest (standalone, navy BG, gold theme)
├── icon-192.png     # App-Icon 192×192 (navy + gold Diamant)
├── icon-512.png     # App-Icon 512×512 (navy + gold Diamant)
└── kontext.md       # Diese Datei
```

---

## Technologie

- **Vanilla JS** — keine Abhängigkeiten außer Google Fonts
- **Service Worker** — Cache-First für alle Assets, Offline-Fallback auf `index.html`
- **localStorage** — Notizen (`notes_<id>`) und Active-Status (`active_<id>`) pro Framework
- **SVG** — Framework-Netzwerk, rein programmatisch generiert
- **PWA** — `manifest.json`, `apple-mobile-web-app-capable`, `safe-area-inset` für iPhone-Notch

---

## Navigation

Vier Tabs in der Bottom Tab Bar:

| Tab | Screen | Beschreibung |
|---|---|---|
| Library | `#screen-library` | 3×3-Grid aller 9 Frameworks |
| Workflow | `#screen-workflow` | 6 Guided-Workflow-Karten |
| Network | `#screen-network` | SVG-Netzwerk-Visualisierung |
| Analysis | `#screen-analysis` | Aktive Frameworks + Export |

Zusätzliche Overlays (schieben von rechts rein):
- **Detail-Screen** (`#detail`) — Framework-Detail, öffnet sich aus Library, Network oder Workflow
- **Workflow-Detail** (`#workflow-detail`) — Workflow-Stepper, öffnet sich aus dem Workflow-Tab

---

## Implementierte Features

### P1 — Framework Library
9 Strategy Frameworks als Karten-Grid:

| Framework | Kategorie |
|---|---|
| Porter's Five Forces | Competitive |
| VRIO Analysis | Resource |
| Balanced Scorecard | Execution |
| Wardley Mapping | Innovation |
| Business Model Canvas | Innovation |
| McKinsey 7-S | Execution |
| SWOT Analysis | Competitive |
| Ansoff Matrix | Innovation |
| Blue Ocean Strategy | Innovation |

Kategorie-Badges farbcodiert: Navy (Competitive), Grün (Resource), Orange (Execution), Lila (Innovation).

### P2 — Erweitertes Framework-Detail

Jedes Framework hat:
- **Meta-Zeile:** Autor + Jahr, Aufwandsstufe (Quick/Medium/Deep), Output-Typ, typische Projekttypen
- **Application Guide:** "When to use", "When not to use", 3 Common Pitfalls
- **Key Questions:** 4–5 Leitfragen
- **Related Frameworks:** Tippbare Karten mit Beziehungstyp und Erklärungstext
- **Active Toggle:** Fügt Framework zu "My Analysis" hinzu
- **Notizen:** Textarea mit Auto-Save (400ms Debounce) in `localStorage`
- **Export Button:** `navigator.share()` → Fallback auf Clipboard-Copy + Toast

### P3 — Guided Workflow

6 vordefinierte Framework-Sequenzen:

| Workflow | Frameworks | Aufwand |
|---|---|---|
| Market Entry | Porter's → SWOT → Ansoff → BMC | 2–3 Tage |
| Competitive Strategy | Porter's → SWOT → VRIO → Blue Ocean | 2–4 Tage |
| Org. Transformation | McKinsey 7-S → VRIO → Balanced Scorecard | 3–5 Tage |
| M&A Integration | Porter's → VRIO → McKinsey 7-S → BSC | 4–6 Tage |
| Business Model Innovation | Blue Ocean → BMC → Wardley Mapping | 3–5 Tage |
| Technology Strategy | Wardley → VRIO → BMC → BSC | 3–4 Tage |

Jeder Schritt zeigt: Input needed, Output produced, Fortschrittskreis (grün wenn Notizen vorhanden), direkter "Open Framework" Link.

Fortschrittsbalken auf der Picker-Karte: Gold-Segmente = Schritte mit Notizen.

### P4 — Framework Network (SVG-Visualisierung)

- 9 Knoten im Kreislayout (Wardley im Zentrum), 16 Kanten
- Kanten farbcodiert nach Beziehungstyp:
  - Grün `#1a6b52` = Leads to (PRECEDES)
  - Navy `#1B5296` = Pair with (COMBINES)
  - Lila `#7b5fa8` = Validates
  - Orange `#c47b2a` = Contrasts
- Knoten antippen: Verbundene bleiben voll sichtbar, andere auf 20% Opacity, Kanten leuchten auf
- Info-Panel schiebt von unten: Kategorie, Name, Kurzbeschreibung, alle Relationen mit Erklärungstext
- "Open Framework" öffnet direkt den Detail-Screen
- Hintergrund antippen → Reset

---

## Beziehungsmatrix (alle 16 Kanten)

| Von | Zu | Typ |
|---|---|---|
| Porter's Five Forces | SWOT | PRECEDES |
| Porter's Five Forces | Blue Ocean | PRECEDES |
| Porter's Five Forces | VRIO | VALIDATES |
| VRIO | McKinsey 7-S | COMBINES |
| VRIO | Balanced Scorecard | PRECEDES |
| Balanced Scorecard | McKinsey 7-S | VALIDATES |
| Balanced Scorecard | Business Model Canvas | COMBINES |
| Wardley Mapping | Business Model Canvas | COMBINES |
| Wardley Mapping | Porter's Five Forces | COMBINES |
| Wardley Mapping | VRIO | VALIDATES |
| Business Model Canvas | Ansoff Matrix | COMBINES |
| Business Model Canvas | Balanced Scorecard | PRECEDES |
| McKinsey 7-S | Blue Ocean | VALIDATES |
| SWOT | Ansoff Matrix | PRECEDES |
| SWOT | Blue Ocean | CONTRASTS |
| Ansoff Matrix | Blue Ocean | CONTRASTS |

---

## localStorage-Schema

```
notes_<framework-id>    → string (Notiztext, beliebige Länge)
active_<framework-id>   → "1" | "0"
```

Framework-IDs: `porters-five-forces`, `vrio`, `balanced-scorecard`, `wardley-mapping`, `business-model-canvas`, `mckinsey-7s`, `swot`, `ansoff`, `blue-ocean`

---

## Export-Format

```
PPI Strategy Navigator – Export [Datum]

[Framework Name] ★ Active
[Vollständige Notizen]
---
```

`navigator.share()` wenn verfügbar (iOS Native Share Sheet), sonst Clipboard-Copy + Toast "Copied to clipboard".

---

## Offene Punkte / Ideen

- [ ] Icons als echte SVG-Vektordateien ersetzen (aktuell: via Python generierte PNGs)
- [ ] Filtering der Library nach Kategorie
- [ ] Search / Suche über Framework-Namen und Beschreibungen
- [ ] Dark Mode
- [ ] Mehrsprachigkeit (DE/EN)
- [ ] Projekt-Container: mehrere Analysen parallel speichern
- [ ] PDF-Export (via `window.print()` mit Print-CSS)
