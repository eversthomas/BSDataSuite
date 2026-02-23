# BSDataSuite – Projektplan

> **BS** steht für [BezugsSysteme](https://bezugssysteme.de)  
> Ein ProcessWire-Modul das Kanban, Kalender, Wiki, CRM, Bookmarks, Mediathek und flexible Datenbanken als integrierte Suite im PW-Backend bereitstellt.

---

## Rahmenbedingungen

- Einzelentwickler, Cursor AI als Unterstützung
- Lokale ProcessWire Installation als Entwicklungsumgebung
- **Ein einziges Modul** – keine Abhängigkeiten zu anderen Modulen
- Open Source von Anfang an sauber strukturiert
- PHP OOP, ProcessWire-Konventionen durchgehend
- REST-API und Claude-API-Schnittstellen werden von Anfang an mitgedacht

---

## Externe Bibliotheken

| Bibliothek | Verwendung | Einbindung |
|---|---|---|
| Sortable.js | Kanban drag & drop | lokal |
| FullCalendar.js | Kalender-View | lokal |
| TipTap | Wiki / Rich Text Editor | lokal |
| Tabulator.js | Tabellen, Datenbank-Views, CRM, Projekte | lokal |
| JSON Forms | Formular-Generierung für flexible Datenbank | lokal |
| Dropzone.js | Mediathek Upload | lokal |

Alle Libs leben lokal im Modul-Ordner – keine CDN-Abhängigkeit.

---

## Kernprinzip

**Alles ist eine ProcessWire-Seite.** Kanban-Karte, Kalender-Termin, Wiki-Eintrag, Kontakt, Bookmark – alles sind PW-Seiten mit demselben universellen Field-Grundstock. Der Charakter (Kanban, Kalender, etc.) entsteht durch die View, nicht durch unterschiedliche Datenspeicher.

---

## Phasen

---

### Phase 1 – Fundament
*Ohne das läuft nichts. Muss felsenfest sein.*

- [ ] `BSDataSuite.module` mit korrektem PW-Modul-Header
- [ ] Namespace, Autoloading, `init()`, `install()`, `uninstall()`
- [ ] Eigener Menüeintrag im PW-Backend via Process-Klasse
- [ ] Router-Logik (`___execute()` leitet auf Views weiter)
- [ ] Alle `bsds_`-Fields programmatisch in `install()` anlegen
- [ ] Templates anlegen und Fields zuweisen
- [ ] Seitenstruktur anlegen (Parent-Seiten für jedes Tool)
- [ ] `uninstall()` räumt alles sauber auf
- [ ] Dashboard – "Was möchtest du tun?" Startseite
- [ ] CSS/JS Grundlage, alle Libs eingebunden
- [ ] REST-API Grundstruktur mit Authentifizierung via API-Key
- [ ] `BSClaudeApi.php` als Stub (leer, aber Klasse existiert)
- [ ] Footer mit BS / BezugsSysteme Erklärung

**Testcheck Phase 1:**
- Modul installiert ohne Fehler
- Modul deinstalliert sauber (keine verwaisten Fields/Templates)
- Dashboard lädt im PW-Backend
- REST-Endpoint antwortet mit 401 ohne Key, 200 mit Key

---

### Phase 2 – Kanban
*Erstes echtes Tool, setzt das Muster für alle weiteren.*

- [ ] Boards anlegen (PW-Seiten)
- [ ] Spalten konfigurierbar pro Board
- [ ] Karten erstellen via AJAX → PW-API legt Seite an
- [ ] Drag & drop via Sortable.js (Status-Änderung wird gespeichert)
- [ ] Karten einem Projekt zuweisbar (vorbereitet)
- [ ] REST-Endpoint `/bsds-api/kanban/`

**Testcheck Phase 2:**
- Board anlegen, Karte erstellen, zwischen Spalten verschieben, löschen
- Status-Änderung wird in PW-Seite gespeichert
- API gibt Board-Daten als JSON aus

---

### Phase 3 – Kalender
*Zweites Tool, erstes das FullCalendar intensiv nutzt.*

- [ ] Mehrere Kalender anlegbar, kategorisierbar, Farbe pro Kalender
- [ ] Termine erstellen via FullCalendar-Interface → PW-API
- [ ] iCal-Export pro Kalender
- [ ] Termine mit Projekten verknüpfbar (vorbereitet)
- [ ] REST-Endpoint `/bsds-api/calendar/`

**Testcheck Phase 3:**
- Kalender anlegen, Termin erstellen und bearbeiten
- iCal-Feed in externem Client abonnierbar
- API-Ausgabe korrekt

---

### Phase 4 – Wiki
*Nutzt PW's Seitenbaum am stärksten.*

- [ ] Seiten-Hierarchie als Wiki-Struktur (PW-nativ)
- [ ] TipTap als Editor
- [ ] Kategorien
- [ ] Interne Verlinkung zwischen Wiki-Einträgen via PW-Hook (`[[Seitenname]]`-Syntax)
- [ ] Suchfunktion via `$pages->find()`
- [ ] REST-Endpoint `/bsds-api/wiki/`

**Testcheck Phase 4:**
- Eintrag anlegen, verlinken, kategorisieren
- Suche findet Inhalte
- API-Ausgabe korrekt

---

### Phase 5 – Bookmarks
*Einfachstes Tool – gute Verschnaufpause.*

- [ ] URL speichern, Titel automatisch fetchen via PHP
- [ ] Tags, Projekt-Zuordnung
- [ ] Listenansicht mit Filter (Tabulator.js)
- [ ] REST-Endpoint `/bsds-api/bookmarks/`
- [ ] Browser-Extension (optionaler Bonus, später)

**Testcheck Phase 5:**
- Bookmark anlegen, Titel wird automatisch geholt
- Tags setzen, filtern
- API-Ausgabe korrekt

---

### Phase 6 – Projekte & Aufgaben
*Verbindet erstmals mehrere Tools miteinander.*

- [ ] Projekte anlegen
- [ ] Aufgaben pro Projekt
- [ ] Meilensteine mit Datum (erscheinen im Kalender)
- [ ] Kanban-Karten und Wiki-Einträge einem Projekt zuweisbar
- [ ] Tabulator.js für Projektübersicht
- [ ] REST-Endpoint `/bsds-api/projects/`

**Testcheck Phase 6:**
- Projekt anlegen, Aufgaben erstellen
- Meilenstein erscheint im Kalender
- Kanban-Karte einem Projekt zuweisen

---

### Phase 7 – CRM Light
*Erste wirklich relationale Struktur.*

- [ ] Kontakte anlegen
- [ ] Notizen pro Kontakt als Unterseiten
- [ ] Nächster-Schritt-Datum (erscheint im Kalender)
- [ ] Projekt-Zuordnung
- [ ] Tabulator.js für Kontaktliste
- [ ] REST-Endpoint `/bsds-api/contacts/`

**Testcheck Phase 7:**
- Kontakt anlegen, Notiz hinzufügen
- Nächster-Schritt-Datum erscheint im Kalender
- API-Ausgabe korrekt

---

### Phase 8 – Flexible Datenbank
*Ambitionierteste Phase – JSON Forms kommt zum Einsatz.*

- [ ] Neue Datensammlung definieren (Feldtypen wählen)
- [ ] JSON Schema als internes Format für Felddefinitionen
- [ ] Modul übersetzt Schema in PW-Fields und Template
- [ ] JSON Forms generiert Eingabeformular automatisch
- [ ] Tabulator.js für Tabellenansicht
- [ ] REST-Endpoint automatisch pro Datensammlung

**Testcheck Phase 8:**
- Datensammlung "Einrichtungen" mit 4 Feldern definieren
- Datensatz anlegen via generiertem Formular
- API-Endpoint automatisch verfügbar

---

### Phase 9 – Mediathek
*Nutzt PW-Boardmittel am stärksten.*

- [ ] Zentrale Medienbibliothek als PW-Seitenstruktur
- [ ] Dropzone.js für Upload
- [ ] Grid-Ansicht, Filter nach Typ/Datum/Tags
- [ ] Medien anderen Seiten zuweisbar (Kanban-Karte, Kontakt, etc.)
- [ ] REST-Endpoint `/bsds-api/media/`

**Testcheck Phase 9:**
- Upload, Bild einer Kanban-Karte und einem Kontakt zuweisen
- API-Ausgabe korrekt

---

### Phase 10 – Claude API Integration
*Kommt zuletzt, Schnittstelle war von Anfang an vorbereitet.*

- [ ] Internes Chat-Interface im Dashboard
- [ ] PHP macht API-Call direkt (kein SDK)
- [ ] BSDataSuite-Inhalte als Kontext mitgebbar
- [ ] Kontext-Auswahl: welche Tools soll Claude kennen?
- [ ] Nur intern, kein öffentlicher Zugriff

**Testcheck Phase 10:**
- Frage stellen, Antwort bezieht sich auf eigene BSDataSuite-Inhalte
- Kontext-Auswahl funktioniert

---

## Durchgehende Prinzipien

- Nach jeder Phase ist das Modul installierbar, nutzbar und deinstallierbar
- REST-API wächst mit jeder Phase mit
- Kein Tool bricht ein anderes
- Cursor-Aufgaben sind immer auf eine Phase oder einen Testcheck begrenzt
- PW-Konventionen haben immer Vorrang vor cleveren Abkürzungen
