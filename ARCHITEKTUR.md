# BSDataSuite – Architektur

---

## Verzeichnisstruktur

```
site/modules/BSDataSuite/
│
├── BSDataSuite.module              # Hauptklasse, Install/Uninstall, Bootstrap
├── BSDataSuite.info.php            # Modul-Metadaten
├── README.md
├── DEPENDENCIES.md                 # Lib-Versionen dokumentiert
│
├── /src
│   ├── /Domain                     # Business-Logik pro Tool
│   │   ├── Kanban/
│   │   │   ├── KanbanService.php
│   │   │   └── KanbanBoard.php
│   │   ├── Calendar/
│   │   ├── Wiki/
│   │   ├── Bookmarks/
│   │   ├── Projects/
│   │   ├── Contacts/
│   │   ├── Database/
│   │   └── Media/
│   │
│   ├── /Api                        # REST-Controller
│   │   ├── BSApi.php               # Router + Auth
│   │   ├── BSApiResponse.php       # Einheitliche JSON-Responses
│   │   └── /Controllers
│   │       ├── KanbanApiController.php
│   │       ├── CalendarApiController.php
│   │       └── ...
│   │
│   ├── /Admin                      # Backend-Controller (Process-Views)
│   │   ├── BSRouter.php
│   │   └── /Controllers
│   │       ├── DashboardController.php
│   │       ├── KanbanController.php    # render(), handleAjax(), assets()
│   │       ├── CalendarController.php
│   │       └── ...
│   │
│   └── /Infrastructure             # Querschnittsthemen
│       ├── BSInstaller.php         # Fields, Templates, Seiten
│       ├── BSLogger.php            # Logging mit Debug-Toggle
│       ├── BSAuth.php              # CSRF, Permissions
│       └── BSClaudeApi.php         # Claude API (Phase 10, Stub)
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
│       └── bsdatasuite.js          # BSDS-Namespace, Tool-übergreifend
│
├── /libs                           # Alle externen Libs lokal, nie CDN
│   ├── /sortable                   # + LICENSE
│   ├── /fullcalendar               # + LICENSE
│   ├── /tiptap                     # + LICENSE
│   ├── /tabulator                  # + LICENSE
│   ├── /jsonforms                  # + LICENSE
│   └── /dropzone                   # + LICENSE
│
└── /api                            # REST-API Endpoint
    └── index.php
```

---

## Namenskonventionen

| Was | Konvention | Beispiel |
|---|---|---|
| Modulname | PascalCase | `BSDataSuite` |
| PHP-Klassenname | PascalCase | `KanbanService`, `BSInstaller` |
| Field-Präfix | `bsds_` | `bsds_status`, `bsds_due_date`, `bsds_type` |
| Template-Präfix | `bsds_` | `bsds_kanban_item` |
| Seiten-Name | `bsds-` | `bsds-kanban`, `bsds-root` |
| CSS-Klassen | `bsds-` | `bsds-board`, `bsds-card` |
| JS-Namespace | `BSDS` | `BSDS.kanban.init()` |
| Managed-Marking | `BSDS_MANAGED` | Field/Template description |

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
        // Autoloader für /src (alle Unterordner)
        spl_autoload_register(function($class) {
            $dirs = ['Domain', 'Api', 'Api/Controllers', 'Admin', 'Admin/Controllers', 'Infrastructure'];
            foreach($dirs as $dir) {
                $file = __DIR__ . "/src/{$dir}/{$class}.php";
                if(file_exists($file)) { require_once($file); return; }
            }
        });

        // Assets einbinden
        $this->config->scripts->add($this->urls->BSDataSuite . 'assets/js/bsdatasuite.js');
        $this->config->styles->add($this->urls->BSDataSuite . 'assets/css/bsdatasuite.css');
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
        $installer->uninstall(); // safe mode – siehe BSInstaller
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

    // Registry: welche IDs wurden angelegt (gespeichert in Modul-Config)
    protected $registry = ['fields' => [], 'templates' => [], 'pages' => []];

    // Universeller Field-Grundstock
    protected $fieldDefinitions = [
        // Typ-Pflichtfeld (semantische Klarheit + Migrationsfähigkeit)
        'bsds_type'         => ['type' => 'FieldtypeText',      'label' => 'Typ'],

        // Inhalt
        'bsds_body'         => ['type' => 'FieldtypeTextarea',  'label' => 'Inhalt', 'contentType' => 1],
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

        // Verknüpfungen
        'bsds_tags'         => ['type' => 'FieldtypePage',      'label' => 'Tags',             'derefAsPage' => 0],
        'bsds_project'      => ['type' => 'FieldtypePage',      'label' => 'Projekt',          'derefAsPage' => 1],
        'bsds_contacts'     => ['type' => 'FieldtypePage',      'label' => 'Kontakte',         'derefAsPage' => 0],
        'bsds_related'      => ['type' => 'FieldtypePage',      'label' => 'Verwandte Seiten', 'derefAsPage' => 0],

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

    public function install() {
        $this->createFields();
        $this->createTemplates();
        $this->createPages();
        $this->saveRegistry();
    }

    /**
     * Safe mode uninstall:
     * - Entfernt nur Assets die mit BSDS_MANAGED markiert sind
     * - Entfernt nur Seiten/Templates die nachweislich leer sind
     * - Alles andere: bleibt, wird im Log dokumentiert
     */
    public function uninstall() {
        $this->loadRegistry();
        $this->removePages();     // nur leere
        $this->removeTemplates(); // nur ohne Seiten
        $this->removeFields();    // nur ohne Templates
    }

    protected function markAsManaged(Field|Template $asset) {
        $asset->description = 'BSDS_MANAGED – ' . ($asset->description ?? '');
        $asset->save();
    }

    // createFields(), createTemplates(), createPages(),
    // removeFields(), removeTemplates(), removePages(),
    // saveRegistry(), loadRegistry() werden vollständig implementiert
}
```

---

## BSAuth.php – CSRF & Permissions

```php
<?php namespace ProcessWire;

class BSAuth {

    public function csrfToken(): string {
        return wire('session')->CSRF->getTokenName() . ':' . wire('session')->CSRF->getTokenValue();
    }

    public function verifyCsrf(): bool {
        return wire('session')->CSRF->hasValidToken();
    }

    public function checkPermission(string $permission): bool {
        return wire('user')->hasPermission($permission);
    }

    public function requirePermission(string $permission): void {
        if(!$this->checkPermission($permission)) {
            throw new WireException('Permission denied: ' . $permission, 403);
        }
    }
}
```

---

## Admin-Controller – einheitliches Interface

```php
<?php namespace ProcessWire;

abstract class BSAdminController {

    protected $module;
    protected $auth;

    public function __construct(BSDataSuite $module) {
        $this->module = $module;
        $this->auth   = new BSAuth();
    }

    abstract public function render(): string;
    abstract public function handleAjax(): void;
    abstract public function assets(): void;
}

// Beispiel: KanbanController
class KanbanController extends BSAdminController {

    public function render(): string {
        $this->auth->requirePermission('bsds-kanban-view');
        ob_start();
        include __DIR__ . '/../../views/kanban.php';
        return ob_get_clean();
    }

    public function handleAjax(): void {
        $this->auth->verifyCsrf() or throw new WireException('CSRF', 403);
        $action = wire('input')->post('action');
        // createCard, moveCard, deleteCard ...
    }

    public function assets(): void {
        wire('config')->scripts->add(
            wire('config')->urls->BSDataSuite . 'libs/sortable/sortable.min.js'
        );
    }
}
```

---

## BSRouter.php

```php
<?php namespace ProcessWire;

class BSRouter {

    protected $module;

    protected $map = [
        'dashboard' => DashboardController::class,
        'kanban'    => KanbanController::class,
        'calendar'  => CalendarController::class,
        'wiki'      => WikiController::class,
        'bookmarks' => BookmarksController::class,
        'projects'  => ProjectsController::class,
        'contacts'  => ContactsController::class,
        'database'  => DatabaseController::class,
        'media'     => MediaController::class,
    ];

    public function route(): string {
        $segment = wire('input')->urlSegment1 ?: 'dashboard';
        if(!isset($this->map[$segment])) $segment = 'dashboard';

        $isAjax = wire('config')->ajax;
        $class   = $this->map[$segment];
        $ctrl    = new $class($this->module);
        $ctrl->assets();

        if($isAjax) {
            $ctrl->handleAjax();
            exit; // handleAjax() gibt JSON aus und beendet
        }

        return $ctrl->render();
    }
}
```

---

## BSApi.php – REST

```php
<?php namespace ProcessWire;

class BSApi {

    protected $routes = [
        'kanban'     => KanbanApiController::class,
        'calendar'   => CalendarApiController::class,
        'wiki'       => WikiApiController::class,
        'bookmarks'  => BookmarksApiController::class,
        'projects'   => ProjectsApiController::class,
        'contacts'   => ContactsApiController::class,
        'database'   => DatabaseApiController::class,
        'media'      => MediaApiController::class,
    ];

    public function handle(string $endpoint, string $method, array $data = []): void {
        if(!$this->authenticate()) {
            BSApiResponse::error(401, 'Unauthorized');
        }
        if(!$this->checkScope($endpoint, $method)) {
            BSApiResponse::error(403, 'Forbidden');
        }
        if(!isset($this->routes[$endpoint])) {
            BSApiResponse::error(404, 'Not found');
        }

        $this->logRequest($endpoint, $method);

        $ctrl = new $this->routes[$endpoint]();
        $ctrl->handle($method, $data);
    }

    protected function authenticate(): bool {
        $key = wire('input')->requestHeader('X-BSDS-Key');
        if(!$key) return false;
        return password_verify($key, wire('modules')->get('BSDataSuite')->apiKeyHash);
    }

    protected function checkScope(string $endpoint, string $method): bool {
        $scope = wire('input')->requestHeader('X-BSDS-Scope') ?: 'read';
        if(in_array($method, ['POST', 'PATCH', 'DELETE']) && $scope !== 'write') return false;
        return true;
    }

    protected function logRequest(string $endpoint, string $method): void {
        // Audit-Log: Zeitpunkt, Endpoint, Methode, IP
        $logger = new BSLogger();
        $logger->log('api', "{$method} {$endpoint} – " . wire('session')->getIP());
    }
}
```

---

## BSClaudeApi.php – Stub (Phase 10)

```php
<?php namespace ProcessWire;

class BSClaudeApi {

    protected string $apiKey   = '';
    protected string $model    = 'claude-opus-4-6';
    protected string $endpoint = 'https://api.anthropic.com/v1/messages';

    public function __construct() {
        $this->apiKey = wire('modules')->get('BSDataSuite')->claudeApiKey ?? '';
    }

    /**
     * Frage an Claude mit optionalem BSDataSuite-Kontext
     * @param string $question
     * @param array  $tools  z.B. ['kanban', 'wiki', 'projects']
     */
    public function ask(string $question, array $tools = []): string {
        // Phase 10
        return '';
    }

    protected function buildContext(array $tools = []): string {
        // Sammelt Inhalte aus gewählten Tools
        // Redaction: E-Mail, Tel, private Notizen werden gefiltert
        // Token-Budgeting: Kontext wird gecapped
        // Phase 10
        return '';
    }

    protected function call(array $payload): array {
        // Direkter cURL-Call, kein SDK
        // Phase 10
        return [];
    }
}
```

---

## Datenprinzip – wie Tools Seiten anlegen

```php
// Jedes Tool – immer dasselbe Muster
$p = new Page();
$p->template      = 'bsds_kanban_item';
$p->parent        = wire('pages')->get('name=bsds-kanban');
$p->title         = $sanitizer->text($data['title']);
$p->bsds_type     = 'kanban_item';      // Pflicht
$p->bsds_status   = $data['status']   ?? 'open';
$p->bsds_project  = $data['project']  ?? null;
$p->bsds_position = $data['position'] ?? 0;

try {
    $p->save();
} catch(WireException $e) {
    BSLogger::error('kanban', 'createCard failed: ' . $e->getMessage());
    BSApiResponse::error(500, 'Save failed');
}
```

---

## Datenfluss

```
Browser
  ↓ /admin/bsdatasuite/kanban/
BSDataSuite::___execute()
  ↓
BSRouter::route()
  ↓ segment = 'kanban'
KanbanController::assets()   → lädt Sortable.js
KanbanController::render()   → views/kanban.php → KanbanService → $pages->find()

User verschiebt Karte → AJAX POST
  ↓
KanbanController::handleAjax()
  ↓ CSRF prüfen → Permission prüfen
KanbanService::moveCard()
  ↓ $page->bsds_status = 'done' → $page->save()
  ↓ JSON Response

Parallel extern:
GET /bsds-api/kanban/  →  BSApi::handle()  →  KanbanApiController  →  JSON
```

---

## REST-Endpunkte

| Endpoint | Methoden | Scope |
|---|---|---|
| `/bsds-api/kanban/` | GET, POST, PATCH, DELETE | read / write |
| `/bsds-api/calendar/` | GET, POST, PATCH, DELETE | read / write |
| `/bsds-api/wiki/` | GET, POST, PATCH, DELETE | read / write |
| `/bsds-api/bookmarks/` | GET, POST, DELETE | read / write |
| `/bsds-api/projects/` | GET, POST, PATCH, DELETE | read / write |
| `/bsds-api/contacts/` | GET, POST, PATCH, DELETE | read / write |
| `/bsds-api/database/{collection}/` | GET, POST, PATCH, DELETE | read / write |
| `/bsds-api/media/` | GET, POST, DELETE | read / write |

Authentifizierung: `X-BSDS-Key: {key}` + `X-BSDS-Scope: read|write`

---

## Templates Übersicht

| Template | Verwendung |
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
| `bsds_note` | Notiz |
| `bsds_collection` | Flexible Datenbank (Parent) |
| `bsds_record` | Datensatz in flexibler Datenbank |
| `bsds_media_item` | Mediathek-Eintrag |
| `bsds_tag` | Tag |

Alle Templates haben `BSDS_MANAGED` in der Description.
