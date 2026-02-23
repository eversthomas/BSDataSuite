# BSDataSuite

**BS** steht für [BezugsSysteme](https://bezugssysteme.de).

Ein ProcessWire-Modul, das Kanban, Kalender, Wiki, Bookmarks, Projektmanagement, CRM light, flexible Datenbank und Mediathek als integrierte Suite im PW-Backend bereitstellt – plus REST-API und geplante Claude-API-Integration.

---

## Übersicht

BSDataSuite ist **ein einziges Modul** ohne Abhängigkeiten zu anderen PW-Modulen. Alle Tools nutzen dasselbe Datenmodell: **Alles ist eine ProcessWire-Seite.** Kanban-Karte, Kalender-Termin, Wiki-Eintrag, Kontakt oder Bookmark sind PW-Seiten mit einem gemeinsamen Field-Grundstock (`bsds_`-Präfix). Der Charakter eines Tools entsteht durch die View, nicht durch getrennte Datenspeicher.

| Tool            | Beschreibung                          |
|-----------------|----------------------------------------|
| **Kanban**      | Boards, Spalten, Karten, Drag & Drop  |
| **Kalender**    | Termine, mehrere Kalender, iCal-Export|
| **Wiki**        | Hierarchische Einträge, Rich-Text     |
| **Bookmarks**   | URL-Sammlung mit Tags und Filter      |
| **Projekte**    | Projekte, Aufgaben, Meilensteine      |
| **CRM Light**   | Kontakte, Notizen, Nächste Schritte   |
| **Flexible DB** | JSON-Schema-basierte Datensammlungen  |
| **Mediathek**   | Zentrale Medienbibliothek, Upload     |

---

## Projektplan & Architektur

- **[PROJEKTPLAN.md](PROJEKTPLAN.md)** – Rahmenbedingungen, externe Bibliotheken, **10 Phasen** mit konkreten Aufgaben und Testchecks. Nach jeder Phase ist das Modul installierbar, nutzbar und deinstallierbar.
- **[ARCHITEKTUR.md](ARCHITEKTUR.md)** – Verzeichnisstruktur, Namenskonventionen, Hauptklassen (`BSDataSuite`, `BSRouter`, `BSInstaller`, `BSApi`, `BSClaudeApi`), Datenfluss und REST-Endpunkte.

---

## Phasen (Kurzüberblick)

| Phase | Inhalt |
|-------|--------|
| **1 – Fundament** | Modul-Grundstruktur, Fields/Templates/Seiten, Dashboard, REST-API-Stub, `BSClaudeApi`-Stub |
| **2 – Kanban** | Boards, Spalten, Karten, Sortable.js |
| **3 – Kalender** | FullCalendar, mehrere Kalender, iCal-Export |
| **4 – Wiki** | TipTap, Hierarchie, interne Verlinkung |
| **5 – Bookmarks** | URL-Fetch, Tags, Tabulator-Filter |
| **6 – Projekte** | Projekte, Aufgaben, Meilensteine, Verknüpfung mit Kanban/Wiki |
| **7 – CRM Light** | Kontakte, Notizen, Nächster-Schritt-Datum |
| **8 – Flexible Datenbank** | JSON Schema → PW-Fields, JSON Forms |
| **9 – Mediathek** | Dropzone, Grid-View, Zuordnung zu anderen Seiten |
| **10 – Claude API** | Internes Chat-Interface, BSDataSuite-Inhalte als Kontext |

---

## Technik

- **ProcessWire** 3.x, **PHP 8.x**
- **Modul-Typ:** Process-Modul (eigener Menüeintrag im Backend)
- **Externe Libs** (lokal im Modul, kein CDN): Sortable.js, FullCalendar, TipTap, Tabulator, JSON Forms, Dropzone
- **REST-API:** Endpoint `/bsds-api/{tool}/`, Authentifizierung per Header `X-BSDS-Key`

---

## Entwicklung

- Einzelentwickler mit Cursor AI
- Lokale ProcessWire-Installation
- Konventionen und Regeln für Cursor: [.cursorrules](.cursorrules)

---

## Lizenz & Autor

BezugsSysteme · Open Source, von Anfang an sauber strukturiert.
