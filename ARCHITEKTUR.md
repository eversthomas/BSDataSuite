# BSDataSuite – Architektur

---

## Verzeichnisstruktur

```
site/modules/BSDataSuite/
│
├── BSDataSuite.module              # Hauptklasse, Install/Uninstall, Router
├── BSDataSuite.info.php            # Modul-Metadaten
├── README.md
│
├── /src                            # PHP-Klassen
│   ├── BSRouter.php                # View-Router
│   ├── BSInstaller.php             # Fields, Templates, Seiten anlegen/entfernen
│   ├── BSApi.php                   # REST-API Handler
│   ├── BSClaudeApi.php             # Claude API (Phase 10, Stub von Anfang an)
│   └── /tools
│       ├── BSKanban.php
│       ├── BSCalendar.php
│       ├── BSWiki.php
│       ├── BSBookmarks.php
│       ├── BSProjects.php
│       ├── BSContacts.php
│       ├── BSDatabase.php          # Flexible Datenbank
│       └── BSMedia.php
│
├── /views                          # PHP-Templates für die Views
│   ├── dashboard.php
│   ├── kanban.php
│   ├── calendar.php
│   ├── wiki.php
│   ├── bookmarks.php
│   ├── projects.php
│   ├── contacts.php
│   ├── database.php
│   └── media.php
│
├── /assets
│   ├── /css
│   │   └── bsdatasuite.css
│   └── /js
│       └── bsdatasuite.js          # Eigenes JS, Tool-übergreifend
│
├── /libs                           # Alle externen Libs lokal, nie CDN
│   ├── /sortable
│   ├── /fullcalendar
│   ├── /tiptap
│   ├── /tabulator
│   ├── /jsonforms
│   └── /dropzone
│
└── /api                            # REST-API Endpoint
    └── index.php
```

---

## Namenskonventionen

| Was | Konvention | Beispiel |
|---|---|---|
| Modulname | PascalCase | `BSDataSuite` |
| PHP-Klassenname | PascalCase | `BSDataSuite`, `BSKanban` |
| Field-Präfix | `bsds_` | `bsds_status`, `bsds_due_date` |
| Template-Präfix | `bsds_` | `bsds_kanban_item` |
| Seiten-Name | `bsds-` | `bsds-kanban`, `bsds-root` |
| CSS-Klassen | `bsds-` | `bsds-board`, `bsds-card` |
| JS-Namespace | `BSDS` | `BSDS.kanban.init()` |

---

## Hauptklasse BSDataSuite.module

```php
<?php namespace ProcessWire;

class BSDataSuite extends Process {

    public static function getModuleInfo() {
        return [
            'title'     => 'BSDataSuite',
            'version'   => '0.1.0',
            'summary'   => 'Personal data suite – Kanban, Kalender, Wiki, CRM und mehr',
            'author'    => 'BezugsSysteme',
            'icon'      => 'dashboard',
            'autoload'  => true,
            'singular'  => true,
            'requires'  => 'ProcessWire>=3.0.0',
            'page'      => [
                'name'   => 'bsdatasuite',
                'title'  => 'BSDataSuite',
                'parent' => 'admin',
            ],
        ];
    }

    public function init() {
        // Assets einbinden
        $this->config->scripts->add($this->urls->BSDataSuite . 'libs/sortable/sortable.min.js');
        $this->config->scripts->add($this->urls->BSDataSuite . 'assets/js/bsdatasuite.js');
        $this->config->styles->add($this->urls->BSDataSuite . 'assets/css/bsdatasuite.css');

        // Autoloader für /src
        spl_autoload_register(function($class) {
            $file = __DIR__ . '/src/' . $class . '.php';
            if(file_exists($file)) require_once($file);
            $file = __DIR__ . '/src/tools/' . $class . '.php';
            if(file_exists($file)) require_once($file);
        });
    }

    public function ___execute() {
        $router = new BSRouter($this);
        return $router->route();
    }

    public function ___install() {
        $installer = new BSInstaller($this);
        $installer->install();
        parent::___install();
    }

    public function ___uninstall() {
        $installer = new BSInstaller($this);
        $installer->uninstall();
        parent::___uninstall();
    }
}
```

---

## BSInstaller.php – Field- und Template-Grundstock

```php
<?php namespace ProcessWire;

class BSInstaller {

    protected $module;

    public function __construct(BSDataSuite $module) {
        $this->module = $module;
    }

    // Universeller Field-Grundstock – lieber zu viel als zu wenig
    protected $fieldDefinitions = [

        // Inhalt
        'bsds_body'         => ['type' => 'FieldtypeTextarea',  'label' => 'Inhalt',              'contentType' => 1],
        'bsds_summary'      => ['type' => 'FieldtypeText',      'label' => 'Zusammenfassung'],

        // Zeit
        'bsds_date_start'   => ['type' => 'FieldtypeDatetime',  'label' => 'Startdatum'],
        'bsds_date_end'     => ['type' => 'FieldtypeDatetime',  'label' => 'Enddatum'],
        'bsds_due_date'     => ['type' => 'FieldtypeDatetime',  'label' => 'Fälligkeitsdatum'],
        'bsds_reminder'     => ['type' => 'FieldtypeDatetime',  'label' => 'Erinnerung'],

        // Status & Klassifizierung
        'bsds_status'       => ['type' => 'FieldtypeOptions',   'label' => 'Status'],
        'bsds_priority'     => ['type' => 'FieldtypeOptions',   'label' => 'Priorität'],
        'bsds_visibility'   => ['type' => 'FieldtypeOptions',   'label' => 'Sichtbarkeit'],

        // Verknüpfungen (Page References)
        'bsds_tags'         => ['type' => 'FieldtypePage',      'label' => 'Tags',                'derefAsPage' => 0],
        'bsds_project'      => ['type' => 'FieldtypePage',      'label' => 'Projekt',             'derefAsPage' => 1],
        'bsds_contacts'     => ['type' => 'FieldtypePage',      'label' => 'Kontakte',            'derefAsPage' => 0],
        'bsds_related'      => ['type' => 'FieldtypePage',      'label' => 'Verwandte Seiten',    'derefAsPage' => 0],

        // Medien
        'bsds_images'       => ['type' => 'FieldtypeImage',     'label' => 'Bilder'],
        'bsds_files'        => ['type' => 'FieldtypeFile',      'label' => 'Dateien'],

        // Extras
        'bsds_url'          => ['type' => 'FieldtypeURL',       'label' => 'URL'],
        'bsds_color'        => ['type' => 'FieldtypeText',      'label' => 'Farbe (Hex)'],
        'bsds_meta'         => ['type' => 'FieldtypeTextarea',  'label' => 'Meta (JSON)'],
        'bsds_schema'       => ['type' => 'FieldtypeTextarea',  'label' => 'Schema (JSON)'],
        'bsds_position'     => ['type' => 'FieldtypeInteger',   'label' => 'Position/Reihenfolge'],
    ];

    // Seitenstruktur
    protected $pageStructure = [
        ['name' => 'bsds-root',      'title' => 'BSDataSuite',  'parent' => '/',         'template' => 'admin'],
        ['name' => 'bsds-kanban',    'title' => 'Kanban',       'parent' => 'bsds-root', 'template' => 'admin'],
        ['name' => 'bsds-calendar',  'title' => 'Kalender',     'parent' => 'bsds-root', 'template' => 'admin'],
        ['name' => 'bsds-wiki',      'title' => 'Wiki',         'parent' => 'bsds-root', 'template' => 'admin'],
        ['name' => 'bsds-bookmarks', 'title' => 'Bookmarks',    'parent' => 'bsds-root', 'template' => 'admin'],
        ['name' => 'bsds-projects',  'title' => 'Projekte',     'parent' => 'bsds-root', 'template' => 'admin'],
        ['name' => 'bsds-contacts',  'title' => 'Kontakte',     'parent' => 'bsds-root', 'template' => 'admin'],
        ['name' => 'bsds-database',  'title' => 'Datenbanken',  'parent' => 'bsds-root', 'template' => 'admin'],
        ['name' => 'bsds-media',     'title' => 'Mediathek',    'parent' => 'bsds-root', 'template' => 'admin'],
        ['name' => 'bsds-tags',      'title' => 'Tags',         'parent' => 'bsds-root', 'template' => 'admin'],
    ];

    public function install() {
        $this->createFields();
        $this->createTemplates();
        $this->createPages();
    }

    public function uninstall() {
        $this->removePages();
        $this->removeTemplates();
        $this->removeFields();
    }

    protected function createFields() { /* ... */ }
    protected function createTemplates() { /* ... */ }
    protected function createPages() { /* ... */ }
    protected function removePages() { /* ... */ }
    protected function removeTemplates() { /* ... */ }
    protected function removeFields() { /* ... */ }
}
```

---

## BSRouter.php

```php
<?php namespace ProcessWire;

class BSRouter {

    protected $module;

    protected $allowed = [
        'dashboard', 'kanban', 'calendar', 'wiki',
        'bookmarks', 'projects', 'contacts', 'database', 'media'
    ];

    public function __construct(BSDataSuite $module) {
        $this->module = $module;
    }

    public function route() {
        $segment = wire('input')->urlSegment1 ?: 'dashboard';
        if(!in_array($segment, $this->allowed)) $segment = 'dashboard';

        $viewFile = dirname(__DIR__) . "/views/{$segment}.php";
        ob_start();
        include($viewFile);
        return ob_get_clean();
    }
}
```

---

## BSApi.php – REST von Anfang an

```php
<?php namespace ProcessWire;

class BSApi {

    protected $routes = [
        'kanban'     => 'handleKanban',
        'calendar'   => 'handleCalendar',
        'wiki'       => 'handleWiki',
        'bookmarks'  => 'handleBookmarks',
        'projects'   => 'handleProjects',
        'contacts'   => 'handleContacts',
        'database'   => 'handleDatabase',
        'media'      => 'handleMedia',
    ];

    public function handle($endpoint, $method, $data = []) {
        if(!$this->authenticate()) {
            return $this->response(401, 'Unauthorized');
        }
        if(!isset($this->routes[$endpoint])) {
            return $this->response(404, 'Endpoint not found');
        }
        return $this->{$this->routes[$endpoint]}($method, $data);
    }

    protected function authenticate() {
        $key = wire('input')->requestHeader('X-BSDS-Key');
        return $key === wire('modules')->get('BSDataSuite')->apiKey;
    }

    protected function pageToArray(Page $page) {
        // Universelle Umwandlung einer PW-Seite in JSON-fähiges Array
        return [
            'id'       => $page->id,
            'title'    => $page->title,
            'name'     => $page->name,
            'url'      => $page->url,
            'created'  => $page->created,
            'modified' => $page->modified,
            // bsds_ Fields werden von den einzelnen Handlern ergänzt
        ];
    }

    protected function response($code, $data) {
        http_response_code($code);
        header('Content-Type: application/json');
        echo json_encode(['status' => $code, 'data' => $data]);
        exit;
    }
}
```

---

## BSClaudeApi.php – Stub (Phase 10)

```php
<?php namespace ProcessWire;

class BSClaudeApi {

    protected $apiKey;
    protected $model    = 'claude-opus-4-6';
    protected $endpoint = 'https://api.anthropic.com/v1/messages';

    public function __construct() {
        $this->apiKey = wire('modules')->get('BSDataSuite')->claudeApiKey ?? '';
    }

    /**
     * Frage an Claude stellen mit optionalem BSDataSuite-Kontext
     * @param string $question
     * @param array  $tools  Welche Tools als Kontext mitgegeben werden ['kanban', 'wiki', ...]
     * @return string
     */
    public function ask(string $question, array $tools = []): string {
        // Wird in Phase 10 implementiert
        return '';
    }

    protected function buildContext(array $tools = []): string {
        // Sammelt Inhalte aus gewählten Tools und baut Kontext-String
        // Wird in Phase 10 implementiert
        return '';
    }

    protected function call(array $payload): array {
        // Direkter cURL-Call, kein SDK
        // Wird in Phase 10 implementiert
        return [];
    }
}
```

---

## Datenprinzip – wie Tools Seiten anlegen

Jedes Tool folgt exakt demselben Muster. Kein Sonderweg.

```php
// Beispiel: Kanban-Karte via AJAX erstellen
public function createCard(array $data): Page {
    $p = new Page();
    $p->template      = 'bsds_kanban_item';
    $p->parent        = wire('pages')->get('name=bsds-kanban');
    $p->title         = $data['title'];
    $p->bsds_status   = $data['status']   ?? 'open';
    $p->bsds_project  = $data['project']  ?? null;
    $p->bsds_position = $data['position'] ?? 0;
    $p->save();
    return $p;
}

// Dasselbe Muster für Kalender-Termin, Wiki-Eintrag, Kontakt, Bookmark
// Immer: new Page() → Felder setzen → $p->save()
// Immer aufgerufen via AJAX aus der jeweiligen View
```

---

## Datenfluss (Request → Response)

```
Browser
  ↓ URL: /admin/bsdatasuite/kanban/
  ↓
BSDataSuite.module
  ↓ ___execute()
BSRouter
  ↓ segment = 'kanban'
views/kanban.php
  ↓ instanziiert BSKanban
  ↓ $pages->find('template=bsds_kanban_item') holt Daten
  ↓ Sortable.js rendert Board
  ↓ User verschiebt Karte → AJAX POST
BSKanban::updateCard()
  ↓ $page->bsds_status = 'done' → $page->save()
  ↓ JSON Response
Browser aktualisiert UI

Parallel:
  /bsds-api/kanban/ → BSApi::handleKanban() → JSON für externe Seiten
```

---

## REST-API Endpunkte (wächst pro Phase)

| Endpoint | Methoden | Beschreibung |
|---|---|---|
| `/bsds-api/kanban/` | GET, POST, PATCH, DELETE | Boards und Karten |
| `/bsds-api/calendar/` | GET, POST, PATCH, DELETE | Kalender und Termine |
| `/bsds-api/wiki/` | GET, POST, PATCH, DELETE | Wiki-Einträge |
| `/bsds-api/bookmarks/` | GET, POST, DELETE | Bookmarks |
| `/bsds-api/projects/` | GET, POST, PATCH, DELETE | Projekte und Aufgaben |
| `/bsds-api/contacts/` | GET, POST, PATCH, DELETE | Kontakte |
| `/bsds-api/database/{collection}/` | GET, POST, PATCH, DELETE | Flexible Datenbanken |
| `/bsds-api/media/` | GET, POST, DELETE | Mediathek |

Authentifizierung: `X-BSDS-Key: {apiKey}` Header

---

## Modul-Einstellungen (Module Config)

```php
// Felder die im Modul-Config-Formular erscheinen
protected $configFields = [
    'apiKey'       => '',   // REST-API Key
    'claudeApiKey' => '',   // Claude API Key (Phase 10)
    'adminTheme'   => '',   // optionales Theme
];
```

---

## Templates Übersicht

| Template-Name | Verwendung |
|---|---|
| `bsds_kanban_board` | Kanban Board (Parent) |
| `bsds_kanban_item` | Kanban Karte |
| `bsds_calendar` | Kalender (Parent) |
| `bsds_event` | Kalender-Termin |
| `bsds_wiki_category` | Wiki Kategorie |
| `bsds_wiki_entry` | Wiki Eintrag |
| `bsds_bookmark` | Bookmark |
| `bsds_project` | Projekt |
| `bsds_task` | Aufgabe |
| `bsds_contact` | Kontakt |
| `bsds_note` | Notiz (zu Kontakt, Projekt, etc.) |
| `bsds_collection` | Flexible Datenbank (Parent) |
| `bsds_record` | Datensatz in flexibler Datenbank |
| `bsds_media_item` | Mediathek-Eintrag |
| `bsds_tag` | Tag |
