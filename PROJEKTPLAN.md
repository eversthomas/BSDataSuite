# BSDataSuite – Projektplan

> **BS** steht für [BezugsSysteme](https://bezugssysteme.de)  
> Ein ProcessWire-Modul das Kanban, Kalender, Wiki, CRM, Bookmarks, Mediathek und flexible Datenbanken als integrierte Suite im PW-Backend bereitstellt.

---

## Rahmenbedingungen

- Einzelentwickler, Cursor AI als Unterstützung
- Lokale ProcessWire Installation als Entwicklungsumgebung
- **Ein einziges Modul** – keine Abhängigkeiten zu anderen Modulen, aber intern sauber modularisiert
- Open Source von Anfang an sauber strukturiert
- PHP OOP, ProcessWire-Konventionen durchgehend
- REST-API und Claude-API-Schnittstellen werden von Anfang an mitgedacht

---

## Externe Bibliotheken

| Bibliothek | Verwendung | Einbindung | Lizenz |
|---|---|---|---|
| Sortable.js | Kanban drag & drop | lokal | MIT ✓ |
| FullCalendar.js | Kalender-View | lokal | MIT ✓ |
| TipTap | Wiki / Rich Text Editor | lokal | MIT ✓ |
| Tabulator.js | Tabellen, Datenbank-Views, CRM, Projekte | lokal | MIT ✓ |
| JSON Forms | Formular-Generierung für flexible Datenbank | lokal | MIT ✓ |
| Dropzone.js | Mediathek Upload | lokal | MIT ✓ |

Alle Libs leben lokal im Modul-Ordner – keine CDN-Abhängigkeit. Jede Lib braucht ihre `LICENSE`-Datei im Lib-Unterordner. Verwendete Versionen werden in einer `DEPENDENCIES.md` dokumentiert.

---

## Kernprinzip

**Alles ist eine ProcessWire-Seite.** Kanban-Karte, Kalender-Termin, Wiki-Eintrag, Kontakt, Bookmark – alles sind PW-Seiten mit demselben universellen Field-Grundstock. Der Charakter (Kanban, Kalender, etc.) entsteht durch die View, nicht durch unterschiedliche Datenspeicher.

Ein Pflichtfeld `bsds_type` (zusätzlich zu getrennten Templates) sorgt für semantische Klarheit und Migrationsfähigkeit.

---

## Definition of Done – gilt für jede Phase

Neben den phasenspezifischen Testchecks müssen diese Kriterien nach jeder Phase erfüllt sein:

- **Install/Update/Uninstall:** keine Fatal Errors, keine PHP Notices im Debug-Mode
- **Security:** CSRF-Schutz für alle write-Actions, Permission-Checks überall
- **Data Integrity:** keine silent failures bei `$page->save()`
- **Logging:** Debug-Toggle, jede Phase erweitert das Logging
- **Backwards Compatibility:** Upgrade-Pfad von vorheriger Phase funktioniert

---

## Phasen

---

### Phase 1 – Fundament
*Ohne das läuft nichts. Muss felsenfest sein.*

**Modul-Grundstruktur**
- [ ] `BSDataSuite.module` mit korrektem PW-Modul-Header
- [ ] Namespace, Autoloading intern nach `/src/Domain/`, `/src/Api/`, `/src/Admin/`, `/src/Infrastructure/`
- [ ] `init()`, `install()`, `uninstall()`
- [ ] Eigener Menüeintrag im PW-Backend via Process-Klasse
- [ ] Router mappt auf Controller-Klassen (nicht eine große `execute()`-Methode)
- [ ] Jeder Controller hat `render()`, `handleAjax()`, `assets()`

**Fields, Templates, Seiten**
- [ ] Alle `bsds_`-Fields programmatisch in `install()` anlegen inkl. `bsds_type`
- [ ] Alle vom Modul erzeugten Fields/Templates werden mit `BSDS_MANAGED` markiert
- [ ] Registry-Eintrag in Modul-Config: welche Field/Template-IDs wurden angelegt
- [ ] Templates anlegen und Fields zuweisen
- [ ] Seitenstruktur anlegen (Parent-Seiten für jedes Tool)
- [ ] `uninstall()` im **safe mode**: entfernt nur BSDS_MANAGED-Assets die nachweislich leer sind – alles andere wird dokumentiert aber nicht gelöscht

**Sicherheit & Konfiguration**
- [ ] Konfigurationsscreen: API-Key, Debug-Toggle, Defaults
- [ ] API-Key wird **gehasht** gespeichert, Vergleich via `password_verify()`
- [ ] CSRF-Schutz für alle write-Actions von Anfang an
- [ ] Minimales Rechte/Rollen-Konzept: wer darf was sehen/ändern
- [ ] Logging-Grundstruktur mit Debug-Toggle

**Dashboard & REST-Stub**
- [ ] Dashboard – "Was möchtest du tun?" Startseite
- [ ] CSS/JS Grundlage, alle Libs eingebunden
- [ ] REST-API Grundstruktur: Authentifizierung, Scopes (`read`/`write`), Rate-Limit light, Audit-Log (Zeitpunkt, Endpoint, IP)
- [ ] `BSClaudeApi.php` als Stub (leer, aber Klasse existiert)
- [ ] Footer mit BS / BezugsSysteme Erklärung

**Testcheck Phase 1:**
- Modul installiert ohne Fehler, keine PHP Notices
- Modul deinstalliert im safe mode – mit Inhalten bleiben Strukturen, ohne Inhalte wird alles entfernt
- Dashboard lädt im PW-Backend
- REST-Endpoint: 401 ohne Key, 403 bei falschem Scope, 200 mit korrektem Key
- CSRF-Token wird bei write-Actions geprüft

---

### Phase 2 – Kanban
*Erstes echtes Tool, setzt das Muster für alle weiteren.*

- [ ] Boards anlegen (PW-Seiten)
- [ ] Spalten konfigurierbar pro Board
- [ ] Karten erstellen via AJAX → PW-API legt Seite an
- [ ] Drag & drop via Sortable.js (Status- und Positions-Änderung wird gespeichert)
- [ ] Positionslogik: explizites `bsds_position`-Field, optimistische Updates im Frontend
- [ ] Karten einem Projekt zuweisbar (vorbereitet)
- [ ] REST-Endpoint `/bsds-api/kanban/`

**Testcheck Phase 2:**
- Board anlegen, Karte erstellen, zwischen Spalten verschieben, löschen
- Status- und Positions-Änderung wird korrekt in PW-Seite gespeichert
- Parallele Drag&Drop-Aktion überschreibt nicht silent die andere
- API gibt Board-Daten als JSON aus
- Definition of Done erfüllt

---

### Phase 3 – Kalender
*Zweites Tool, erstes das FullCalendar intensiv nutzt.*

- [ ] Mehrere Kalender anlegbar, kategorisierbar, Farbe pro Kalender
- [ ] Termine erstellen via FullCalendar-Interface → PW-API
- [ ] All-Day vs. Timed Events explizit unterschieden
- [ ] Zeitzonen-Handling: UTC intern, Ausgabe lokalisiert
- [ ] iCal-Export pro Kalender
- [ ] Termine mit Projekten verknüpfbar (vorbereitet)
- [ ] REST-Endpoint `/bsds-api/calendar/`

**Testcheck Phase 3:**
- Kalender anlegen, Termin erstellen und bearbeiten
- All-Day und Timed Event verhalten sich korrekt
- iCal-Feed in externem Client abonnierbar
- API-Ausgabe korrekt
- Definition of Done erfüllt

---

### Phase 4 – Wiki
*Nutzt PW's Seitenbaum am stärksten.*

- [ ] Seiten-Hierarchie als Wiki-Struktur (PW-nativ)
- [ ] TipTap als Editor – Speicherformat: **JSON** (primär) + HTML (für Ausgabe)
- [ ] Kategorien
- [ ] Interne Verlinkung: unter der Haube ID-basiert (robust gegen Umbenennung), Anzeige als `[[Seitenname]]`
- [ ] Link-Auflösung via PW-Hook auf `Pages::saved` (Umbenennung aktualisiert Links)
- [ ] Suchfunktion via `$pages->find()`
- [ ] REST-Endpoint `/bsds-api/wiki/`

**Testcheck Phase 4:**
- Eintrag anlegen, verlinken, kategorisieren
- Seite umbenennen – Links bleiben gültig
- Suche findet Inhalte
- API-Ausgabe korrekt
- Definition of Done erfüllt

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
- Definition of Done erfüllt

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
- Definition of Done erfüllt

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
- Definition of Done erfüllt

---

### Phase 8a – Flexible Datenbank (JSON-basiert)
*Erst sicher, dann mächtig.*

- [ ] Neue Datensammlung definieren (Feldtypen wählen)
- [ ] Schema wird als JSON in `bsds_schema`-Field der Collection-Seite gespeichert
- [ ] Datensätze speichern ihre Daten in `bsds_meta` (JSON-Field) – **keine neuen PW-Fields**
- [ ] JSON Forms generiert Eingabeformular automatisch aus Schema
- [ ] Tabulator.js liest aus JSON-Field und rendert Tabellenansicht
- [ ] REST-Endpoint automatisch pro Datensammlung

**Testcheck Phase 8a:**
- Datensammlung "Einrichtungen" mit 4 Feldern definieren
- Datensatz anlegen via generiertem Formular
- Tabellen-Ansicht zeigt Daten korrekt
- API-Endpoint automatisch verfügbar
- Definition of Done erfüllt

---

### Phase 8b – Flexible Datenbank (PW-Fields) *(optional, später)*
*Nur wenn 8a sich in der Praxis als zu einschränkend erweist.*

- [ ] Schema → echte PW-Fields und Template generieren
- [ ] Migrations-Logik: bestehende JSON-Daten in neue Fields überführen
- [ ] Konflikt-Prüfung bei Field-Namen vor Anlage
- [ ] Rollback-Strategie bei fehlgeschlagener Migration dokumentiert

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
- Definition of Done erfüllt

---

### Phase 10 – Claude API Integration
*Kommt zuletzt, Schnittstelle war von Anfang an vorbereitet.*

- [ ] Internes Chat-Interface im Dashboard
- [ ] PHP macht API-Call direkt via cURL (kein SDK)
- [ ] BSDataSuite-Inhalte als Kontext mitgebbar
- [ ] Kontext-Auswahl: welche Tools soll Claude kennen?
- [ ] Daten-Minimierung: sensible Felder (E-Mail, Tel, private Notizen) werden vor Übergabe redacted
- [ ] Token-Budgeting: Kontext wird auf sinnvolle Größe gecapped und gechunkt
- [ ] Nur intern, kein öffentlicher Zugriff

**Testcheck Phase 10:**
- Frage stellen, Antwort bezieht sich auf eigene BSDataSuite-Inhalte
- Kontext-Auswahl funktioniert
- Redaction greift bei sensiblen Feldern
- Definition of Done erfüllt

---

## Durchgehende Prinzipien

- Nach jeder Phase ist das Modul installierbar, nutzbar und deinstallierbar
- REST-API wächst mit jeder Phase mit
- Kein Tool bricht ein anderes
- Cursor-Aufgaben sind immer auf eine Phase oder einen Testcheck begrenzt
- PW-Konventionen haben immer Vorrang vor cleveren Abkürzungen
- `uninstall()` vernichtet niemals Nutzerdaten ohne explizite Bestätigung
