# 📚 Documentación Automática de APIs - Día 3.4

**Objetivo:** Generar documentación interactiva profesional con OpenAPI/Swagger en menos de 12 minutos

---

## 🎯 **Objetivo de la Sesión**

Los competidores deben poder crear **documentación automática completa** con especificaciones OpenAPI, ejemplos interactivos, y validación de contratos que cumplan estándares profesionales en **menos de 12 minutos**.

---

## 📖 **Arquitectura de Documentación WorldSkills**

### **Componentes de Documentación Profesional**

```text
🏆 DOCUMENTACIÓN PARA SISTEMA DEPORTIVO

OpenAPI Specification:
├─ Resource Definitions (Player, Match, Tournament)
├─ Request/Response Schemas
├─ Authentication Methods (Sanctum)
└─ Error Response Formats

Interactive Examples:
├─ CRUD Operations para todas las entidades
├─ Filtros y Paginación
├─ Casos de Uso Complejos
└─ Error Scenarios

Validation Contracts:
├─ Request Schema Validation
├─ Response Schema Validation
├─ API Contract Testing
└─ Breaking Change Detection

Documentation Portal:
├─ Swagger UI Integrado
├─ Postman Collections
├─ Code Examples (JS, PHP, cURL)
└─ Versioning Support
```

---

## ⏱️ **Ejercicio 1: OpenAPI Setup (15 minutos)**

### **Paso 1: Instalar Swagger (3 minutos)**

```bash
# CRONÓMETRO: INICIAR ⏰

# Instalar L5-Swagger
composer require darkaonline/l5-swagger

# Publicar configuración
php artisan vendor:publish --provider "L5Swagger\L5SwaggerServiceProvider"

# Generar documentación inicial
php artisan l5-swagger:generate
```

```php
// config/l5-swagger.php - Configuración principal
return [
    'default' => 'default',
    'documentations' => [
        'default' => [
            'api' => [
                'title' => 'WorldSkills Tournament API',
                'description' => 'Complete API for tournament management system',
                'version' => '1.0.0',
            ],
            'routes' => [
                'api' => 'api/documentation',
            ],
            'paths' => [
                'use_absolute_path' => env('L5_SWAGGER_USE_ABSOLUTE_PATH', true),
                'docs_json' => 'api-docs.json',
                'docs_yaml' => 'api-docs.yaml',
                'format_to_use_for_docs' => env('L5_FORMAT_TO_USE_FOR_DOCS', 'json'),
                'annotations' => [
                    base_path('app'),
                ],
            ],
        ],
    ],
    'defaults' => [
        'routes' => [
            'docs' => 'docs',
            'oauth2_callback' => 'api/oauth2-callback',
            'middleware' => [
                'api' => [],
                'asset' => [],
                'docs' => [],
                'oauth2_callback' => [],
            ],
            'group_options' => [],
        ],
        'paths' => [
            'docs' => storage_path('api-docs'),
            'views' => base_path('resources/views/vendor/l5-swagger'),
            'base' => env('L5_SWAGGER_BASE_PATH', null),
            'swagger_ui_assets_path' => env('L5_SWAGGER_UI_ASSETS_PATH', 'vendor/swagger-api/swagger-ui/dist/'),
            'excludes' => [],
        ],
        'scanOptions' => [
            'analyser' => null,
            'analysis' => null,
            'processors' => [
                new \OpenApi\Processors\DocBlockDescriptions(),
                new \OpenApi\Processors\MergeIntoOpenApi(),
                new \OpenApi\Processors\MergeIntoComponents(),
                new \OpenApi\Processors\ExpandClasses(),
                new \OpenApi\Processors\ExpandInterfaces(),
                new \OpenApi\Processors\ExpandTraits(),
                new \OpenApi\Processors\ExpandEnums(),
                new \OpenApi\Processors\AugmentSchemas(),
                new \OpenApi\Processors\AugmentProperties(),
                new \OpenApi\Processors\BuildPaths(),
                new \OpenApi\Processors\AugmentParameters(),
                new \OpenApi\Processors\AugmentRefs(),
                new \OpenApi\Processors\MergeJsonContent(),
                new \OpenApi\Processors\MergeXmlContent(),
                new \OpenApi\Processors\OperationId(),
                new \OpenApi\Processors\CleanUnmerged(),
            ],
        ],
        'securityDefinitions' => [
            'securitySchemes' => [
                'sanctum' => [
                    'type' => 'http',
                    'scheme' => 'bearer',
                    'bearerFormat' => 'JWT',
                    'description' => 'Enter token in format (Bearer <token>)',
                ],
            ],
            'security' => [
                ['sanctum' => []],
            ],
        ],
    ],
];
```

### **Paso 2: Controller Base Documentation (7 minutos)**

```php
// app/Http/Controllers/Controller.php - Anotaciones base
<?php

namespace App\Http\Controllers;

use Illuminate\Foundation\Auth\Access\AuthorizesRequests;
use Illuminate\Foundation\Bus\DispatchesJobs;
use Illuminate\Foundation\Validation\ValidatesRequests;
use Illuminate\Routing\Controller as BaseController;

/**
 * @OA\Info(
 *     title="WorldSkills Tournament API",
 *     version="1.0.0",
 *     description="Complete RESTful API for managing tournaments, teams, players, and matches. Built for WorldSkills 2025 Web Technologies competition.",
 *     @OA\Contact(
 *         email="admin@worldskills-tournament.com",
 *         name="API Support"
 *     ),
 *     @OA\License(
 *         name="MIT",
 *         url="https://opensource.org/licenses/MIT"
 *     )
 * )
 *
 * @OA\Server(
 *     url=L5_SWAGGER_CONST_HOST,
 *     description="Local Development Server"
 * )
 *
 * @OA\Server(
 *     url="https://api.worldskills-tournament.com",
 *     description="Production Server"
 * )
 *
 * @OA\SecurityScheme(
 *     securityScheme="sanctum",
 *     type="http",
 *     scheme="bearer",
 *     bearerFormat="JWT",
 *     description="Enter token in format (Bearer <token>)"
 * )
 *
 * @OA\Tag(
 *     name="Authentication",
 *     description="User authentication and authorization"
 * )
 *
 * @OA\Tag(
 *     name="Players",
 *     description="Player management operations"
 * )
 *
 * @OA\Tag(
 *     name="Teams",
 *     description="Team management operations"
 * )
 *
 * @OA\Tag(
 *     name="Matches",
 *     description="Match scheduling and results"
 * )
 *
 * @OA\Tag(
 *     name="Tournaments",
 *     description="Tournament management and standings"
 * )
 *
 * @OA\Tag(
 *     name="Statistics",
 *     description="Performance statistics and analytics"
 * )
 */
class Controller extends BaseController
{
    use AuthorizesRequests, DispatchesJobs, ValidatesRequests;
}
```

### **Paso 3: Schema Definitions (5 minutos)**

```php
// app/Http/Resources/PlayerResource.php - Con documentación OpenAPI
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\JsonResource;

/**
 * @OA\Schema(
 *     schema="Player",
 *     type="object",
 *     title="Player",
 *     description="Player model",
 *     required={"id", "name", "position", "jersey_number", "team_id"},
 *     @OA\Property(
 *         property="id",
 *         type="integer",
 *         format="int64",
 *         description="Unique identifier for the player",
 *         example=1
 *     ),
 *     @OA\Property(
 *         property="name",
 *         type="string",
 *         description="Full name of the player",
 *         maxLength=100,
 *         example="Carlos Rodríguez"
 *     ),
 *     @OA\Property(
 *         property="position",
 *         type="string",
 *         enum={"GK", "DEF", "MID", "FWD"},
 *         description="Player position on the field",
 *         example="MID"
 *     ),
 *     @OA\Property(
 *         property="jersey_number",
 *         type="integer",
 *         minimum=1,
 *         maximum=99,
 *         description="Jersey number (unique within team)",
 *         example=10
 *     ),
 *     @OA\Property(
 *         property="birth_date",
 *         type="string",
 *         format="date",
 *         description="Date of birth in YYYY-MM-DD format",
 *         example="1995-03-15"
 *     ),
 *     @OA\Property(
 *         property="height",
 *         type="number",
 *         format="float",
 *         minimum=1.5,
 *         maximum=2.2,
 *         description="Height in meters",
 *         example=1.78
 *     ),
 *     @OA\Property(
 *         property="weight",
 *         type="number",
 *         format="float",
 *         minimum=45,
 *         maximum=150,
 *         description="Weight in kilograms",
 *         example=75.5
 *     ),
 *     @OA\Property(
 *         property="foot",
 *         type="string",
 *         enum={"right", "left", "both"},
 *         description="Preferred foot",
 *         example="right"
 *     ),
 *     @OA\Property(
 *         property="nationality",
 *         type="string",
 *         maxLength=50,
 *         description="Player nationality",
 *         example="Colombian"
 *     ),
 *     @OA\Property(
 *         property="team_id",
 *         type="integer",
 *         description="ID of the team the player belongs to",
 *         example=1
 *     ),
 *     @OA\Property(
 *         property="team",
 *         ref="#/components/schemas/Team",
 *         description="Team information"
 *     ),
 *     @OA\Property(
 *         property="statistics",
 *         ref="#/components/schemas/PlayerStatistics",
 *         description="Player performance statistics"
 *     ),
 *     @OA\Property(
 *         property="created_at",
 *         type="string",
 *         format="date-time",
 *         description="Record creation timestamp",
 *         example="2024-01-15T10:30:00Z"
 *     ),
 *     @OA\Property(
 *         property="updated_at",
 *         type="string",
 *         format="date-time",
 *         description="Record last update timestamp",
 *         example="2024-01-15T10:30:00Z"
 *     )
 * )
 *
 * @OA\Schema(
 *     schema="PlayerStatistics",
 *     type="object",
 *     @OA\Property(property="goals", type="integer", description="Total goals scored", example=15),
 *     @OA\Property(property="assists", type="integer", description="Total assists", example=8),
 *     @OA\Property(property="yellow_cards", type="integer", description="Yellow cards received", example=3),
 *     @OA\Property(property="red_cards", type="integer", description="Red cards received", example=0),
 *     @OA\Property(property="matches_played", type="integer", description="Total matches played", example=22),
 *     @OA\Property(property="average_rating", type="number", format="float", description="Average match rating", example=7.8)
 * )
 *
 * @OA\Schema(
 *     schema="Team",
 *     type="object",
 *     @OA\Property(property="id", type="integer", example=1),
 *     @OA\Property(property="name", type="string", example="Real Madrid"),
 *     @OA\Property(property="logo", type="string", format="url", example="https://example.com/logo.png"),
 *     @OA\Property(property="founded_year", type="integer", example=1902),
 *     @OA\Property(property="tournament_id", type="integer", example=1)
 * )
 */
class PlayerResource extends JsonResource
{
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'name' => $this->name,
            'position' => $this->position,
            'jersey_number' => $this->jersey_number,
            'birth_date' => $this->birth_date?->format('Y-m-d'),
            'age' => $this->birth_date ? $this->birth_date->age : null,
            'height' => $this->height,
            'weight' => $this->weight,
            'foot' => $this->foot,
            'nationality' => $this->nationality,
            'team_id' => $this->team_id,
            'team' => $this->whenLoaded('team', new TeamResource($this->team)),
            'statistics' => $this->whenLoaded('statistics', $this->statistics),
            'created_at' => $this->created_at?->toISOString(),
            'updated_at' => $this->updated_at?->toISOString(),
        ];
    }
}
```

---

## ⏱️ **Ejercicio 2: API Endpoint Documentation (20 minutos)**

### **Paso 1: Player Controller Documentation (12 minutos)**

```php
// app/Http/Controllers/Api/PlayerController.php - Documentación completa
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Http\Requests\StorePlayerRequest;
use App\Http\Requests\UpdatePlayerRequest;
use App\Http\Resources\PlayerResource;
use App\Models\Player;
use App\Http\Filters\PlayerFilter;
use Illuminate\Http\Request;

class PlayerController extends Controller
{
    /**
     * @OA\Get(
     *     path="/api/players",
     *     summary="Get list of players",
     *     description="Retrieve a paginated list of players with advanced filtering, searching, and sorting capabilities",
     *     operationId="getPlayers",
     *     tags={"Players"},
     *     security={{"sanctum": {}}},
     *     @OA\Parameter(
     *         name="page",
     *         in="query",
     *         description="Page number for pagination",
     *         required=false,
     *         @OA\Schema(type="integer", minimum=1, default=1, example=1)
     *     ),
     *     @OA\Parameter(
     *         name="per_page",
     *         in="query",
     *         description="Number of items per page (max 100)",
     *         required=false,
     *         @OA\Schema(type="integer", minimum=1, maximum=100, default=15, example=15)
     *     ),
     *     @OA\Parameter(
     *         name="search",
     *         in="query",
     *         description="Search term for player name, team name, or tournament name",
     *         required=false,
     *         @OA\Schema(type="string", maxLength=100, example="Carlos")
     *     ),
     *     @OA\Parameter(
     *         name="team_id",
     *         in="query",
     *         description="Filter by specific team ID",
     *         required=false,
     *         @OA\Schema(type="integer", example=1)
     *     ),
     *     @OA\Parameter(
     *         name="position",
     *         in="query",
     *         description="Filter by player position",
     *         required=false,
     *         @OA\Schema(type="string", enum={"GK", "DEF", "MID", "FWD"}, example="MID")
     *     ),
     *     @OA\Parameter(
     *         name="age_range[min]",
     *         in="query",
     *         description="Minimum age filter",
     *         required=false,
     *         @OA\Schema(type="integer", minimum=16, maximum=50, example=18)
     *     ),
     *     @OA\Parameter(
     *         name="age_range[max]",
     *         in="query",
     *         description="Maximum age filter",
     *         required=false,
     *         @OA\Schema(type="integer", minimum=16, maximum=50, example=35)
     *     ),
     *     @OA\Parameter(
     *         name="sort_by",
     *         in="query",
     *         description="Field to sort by",
     *         required=false,
     *         @OA\Schema(type="string", enum={"name", "position", "jersey_number", "birth_date", "height", "weight", "created_at"}, default="name", example="name")
     *     ),
     *     @OA\Parameter(
     *         name="sort_direction",
     *         in="query",
     *         description="Sort direction",
     *         required=false,
     *         @OA\Schema(type="string", enum={"asc", "desc"}, default="asc", example="asc")
     *     ),
     *     @OA\Parameter(
     *         name="include",
     *         in="query",
     *         description="Relations to include (comma-separated)",
     *         required=false,
     *         @OA\Schema(type="string", example="team,statistics,injuries")
     *     ),
     *     @OA\Response(
     *         response=200,
     *         description="Successful operation",
     *         @OA\JsonContent(
     *             type="object",
     *             @OA\Property(
     *                 property="data",
     *                 type="array",
     *                 @OA\Items(ref="#/components/schemas/Player")
     *             ),
     *             @OA\Property(
     *                 property="meta",
     *                 type="object",
     *                 @OA\Property(property="current_page", type="integer", example=1),
     *                 @OA\Property(property="from", type="integer", example=1),
     *                 @OA\Property(property="to", type="integer", example=15),
     *                 @OA\Property(property="per_page", type="integer", example=15),
     *                 @OA\Property(property="total", type="integer", example=247),
     *                 @OA\Property(property="last_page", type="integer", example=17),
     *                 @OA\Property(property="has_more_pages", type="boolean", example=true),
     *                 @OA\Property(property="total_teams", type="integer", example=12),
     *                 @OA\Property(
     *                     property="position_counts",
     *                     type="object",
     *                     @OA\Property(property="GK", type="integer", example=24),
     *                     @OA\Property(property="DEF", type="integer", example=96),
     *                     @OA\Property(property="MID", type="integer", example=84),
     *                     @OA\Property(property="FWD", type="integer", example=43)
     *                 ),
     *                 @OA\Property(
     *                     property="age_stats",
     *                     type="object",
     *                     @OA\Property(property="average", type="number", format="float", example=24.5),
     *                     @OA\Property(property="minimum", type="integer", example=18),
     *                     @OA\Property(property="maximum", type="integer", example=35)
     *                 )
     *             ),
     *             @OA\Property(
     *                 property="links",
     *                 type="object",
     *                 @OA\Property(property="first", type="string", format="url", example="http://localhost:8000/api/players?page=1"),
     *                 @OA\Property(property="last", type="string", format="url", example="http://localhost:8000/api/players?page=17"),
     *                 @OA\Property(property="prev", type="string", format="url", nullable=true, example=null),
     *                 @OA\Property(property="next", type="string", format="url", example="http://localhost:8000/api/players?page=2"),
     *                 @OA\Property(property="self", type="string", format="url", example="http://localhost:8000/api/players?page=1")
     *             ),
     *             @OA\Property(
     *                 property="filters",
     *                 type="object",
     *                 description="Active filters applied to the query",
     *                 example={"search": "Carlos", "position": "MID"}
     *             ),
     *             @OA\Property(
     *                 property="sorting",
     *                 type="object",
     *                 @OA\Property(property="sort_by", type="string", example="name"),
     *                 @OA\Property(property="sort_direction", type="string", example="asc")
     *             )
     *         )
     *     ),
     *     @OA\Response(
     *         response=401,
     *         description="Unauthenticated",
     *         @OA\JsonContent(ref="#/components/schemas/Error")
     *     ),
     *     @OA\Response(
     *         response=422,
     *         description="Validation error",
     *         @OA\JsonContent(ref="#/components/schemas/ValidationError")
     *     )
     * )
     */
    public function index(Request $request, PlayerFilter $filter)
    {
        // Implementation here...
    }

    /**
     * @OA\Post(
     *     path="/api/players",
     *     summary="Create a new player",
     *     description="Create a new player with comprehensive validation including team limits, position constraints, and document uploads",
     *     operationId="createPlayer",
     *     tags={"Players"},
     *     security={{"sanctum": {}}},
     *     @OA\RequestBody(
     *         required=true,
     *         description="Player data",
     *         @OA\MediaType(
     *             mediaType="multipart/form-data",
     *             @OA\Schema(
     *                 type="object",
     *                 required={"name", "team_id", "position", "jersey_number", "birth_date", "foot"},
     *                 @OA\Property(property="name", type="string", maxLength=100, description="Full name of the player", example="Carlos Rodríguez"),
     *                 @OA\Property(property="team_id", type="integer", description="ID of the team", example=1),
     *                 @OA\Property(property="position", type="string", enum={"GK", "DEF", "MID", "FWD"}, description="Player position", example="MID"),
     *                 @OA\Property(property="jersey_number", type="integer", minimum=1, maximum=99, description="Jersey number (unique within team)", example=10),
     *                 @OA\Property(property="birth_date", type="string", format="date", description="Date of birth", example="1995-03-15"),
     *                 @OA\Property(property="height", type="number", format="float", minimum=1.5, maximum=2.2, description="Height in meters", example=1.78),
     *                 @OA\Property(property="weight", type="number", format="float", minimum=45, maximum=150, description="Weight in kilograms", example=75.5),
     *                 @OA\Property(property="foot", type="string", enum={"right", "left", "both"}, description="Preferred foot", example="right"),
     *                 @OA\Property(property="nationality", type="string", maxLength=50, description="Player nationality", example="Colombian"),
     *                 @OA\Property(
     *                     property="emergency_contact",
     *                     type="object",
     *                     required={"name", "phone", "relationship"},
     *                     @OA\Property(property="name", type="string", maxLength=100, example="María Rodríguez"),
     *                     @OA\Property(property="phone", type="string", description="Phone in E.164 format", example="+573001234567"),
     *                     @OA\Property(property="relationship", type="string", enum={"parent", "spouse", "sibling", "guardian", "other"}, example="parent")
     *                 ),
     *                 @OA\Property(
     *                     property="documents",
     *                     type="array",
     *                     description="Player documents (max 5 files, 2MB each)",
     *                     @OA\Items(type="string", format="binary")
     *                 )
     *             )
     *         )
     *     ),
     *     @OA\Response(
     *         response=201,
     *         description="Player created successfully",
     *         @OA\JsonContent(
     *             type="object",
     *             @OA\Property(property="data", ref="#/components/schemas/Player"),
     *             @OA\Property(property="message", type="string", example="Player created successfully")
     *         )
     *     ),
     *     @OA\Response(
     *         response=422,
     *         description="Validation error",
     *         @OA\JsonContent(ref="#/components/schemas/ValidationError")
     *     )
     * )
     */
    public function store(StorePlayerRequest $request)
    {
        // Implementation here...
    }

    /**
     * @OA\Get(
     *     path="/api/players/{id}",
     *     summary="Get player by ID",
     *     description="Retrieve detailed information about a specific player including statistics and related data",
     *     operationId="getPlayer",
     *     tags={"Players"},
     *     security={{"sanctum": {}}},
     *     @OA\Parameter(
     *         name="id",
     *         in="path",
     *         required=true,
     *         description="Player ID",
     *         @OA\Schema(type="integer", minimum=1, example=1)
     *     ),
     *     @OA\Parameter(
     *         name="include",
     *         in="query",
     *         description="Relations to include",
     *         required=false,
     *         @OA\Schema(type="string", example="team,statistics,injuries,transfers")
     *     ),
     *     @OA\Response(
     *         response=200,
     *         description="Successful operation",
     *         @OA\JsonContent(
     *             type="object",
     *             @OA\Property(property="data", ref="#/components/schemas/Player")
     *         ),
     *         @OA\Header(
     *             header="ETag",
     *             description="Entity tag for caching",
     *             @OA\Schema(type="string", example="\"abc123\"")
     *         ),
     *         @OA\Header(
     *             header="Last-Modified",
     *             description="Last modification date",
     *             @OA\Schema(type="string", example="Mon, 15 Jan 2024 10:30:00 GMT")
     *         )
     *     ),
     *     @OA\Response(
     *         response=404,
     *         description="Player not found",
     *         @OA\JsonContent(ref="#/components/schemas/Error")
     *     ),
     *     @OA\Response(
     *         response=304,
     *         description="Not modified (cached response)"
     *     )
     * )
     */
    public function show($id, Request $request)
    {
        // Implementation here...
    }
}
```

### **Paso 2: Error Schema Definitions (8 minutos)**

```php
// app/Http/Resources/ErrorResource.php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\JsonResource;

/**
 * @OA\Schema(
 *     schema="Error",
 *     type="object",
 *     title="Error Response",
 *     description="Standard error response format",
 *     required={"message", "status_code"},
 *     @OA\Property(
 *         property="message",
 *         type="string",
 *         description="Human-readable error message",
 *         example="The requested resource was not found"
 *     ),
 *     @OA\Property(
 *         property="status_code",
 *         type="integer",
 *         description="HTTP status code",
 *         example=404
 *     ),
 *     @OA\Property(
 *         property="error_code",
 *         type="string",
 *         description="Application-specific error code",
 *         example="RESOURCE_NOT_FOUND"
 *     ),
 *     @OA\Property(
 *         property="timestamp",
 *         type="string",
 *         format="date-time",
 *         description="Error occurrence timestamp",
 *         example="2024-01-15T10:30:00Z"
 *     ),
 *     @OA\Property(
 *         property="path",
 *         type="string",
 *         description="API endpoint where error occurred",
 *         example="/api/players/999"
 *     ),
 *     @OA\Property(
 *         property="trace_id",
 *         type="string",
 *         description="Unique identifier for error tracking",
 *         example="abc123-def456-ghi789"
 *     )
 * )
 *
 * @OA\Schema(
 *     schema="ValidationError",
 *     type="object",
 *     title="Validation Error Response",
 *     description="Response format for validation errors",
 *     required={"message", "errors"},
 *     @OA\Property(
 *         property="message",
 *         type="string",
 *         description="General error message",
 *         example="The given data was invalid."
 *     ),
 *     @OA\Property(
 *         property="errors",
 *         type="object",
 *         description="Field-specific validation errors",
 *         additionalProperties={
 *             "type": "array",
 *             "items": {"type": "string"}
 *         },
 *         example={
 *             "name": {"The name field is required."},
 *             "jersey_number": {"The jersey number has already been taken for this team."},
 *             "birth_date": {"The player must be at least 16 years old."}
 *         }
 *     ),
 *     @OA\Property(
 *         property="status_code",
 *         type="integer",
 *         description="HTTP status code",
 *         example=422
 *     )
 * )
 *
 * @OA\Schema(
 *     schema="SuccessResponse",
 *     type="object",
 *     @OA\Property(
 *         property="message",
 *         type="string",
 *         description="Success message",
 *         example="Operation completed successfully"
 *     ),
 *     @OA\Property(
 *         property="status_code",
 *         type="integer",
 *         description="HTTP status code",
 *         example=200
 *     ),
 *     @OA\Property(
 *         property="timestamp",
 *         type="string",
 *         format="date-time",
 *         example="2024-01-15T10:30:00Z"
 *     )
 * )
 */
class ErrorResource extends JsonResource
{
    public function toArray($request)
    {
        return [
            'message' => $this->resource['message'] ?? 'An error occurred',
            'status_code' => $this->resource['status_code'] ?? 500,
            'error_code' => $this->resource['error_code'] ?? null,
            'timestamp' => now()->toISOString(),
            'path' => $request->getPathInfo(),
            'trace_id' => $request->header('X-Trace-Id', uniqid()),
        ];
    }
}
```

---

## ⏱️ **Ejercicio 3: Examples y Testing (10 minutos)**

### **Paso 1: API Examples Generator (6 minutos)**

```php
// app/Console/Commands/GenerateApiExamples.php
<?php

namespace App\Console\Commands;

use Illuminate\Console\Command;
use Illuminate\Support\Facades\File;

class GenerateApiExamples extends Command
{
    protected $signature = 'api:examples';
    protected $description = 'Generate API examples for documentation';

    public function handle()
    {
        $this->info('Generating API examples...');

        $examples = [
            'players' => $this->generatePlayerExamples(),
            'matches' => $this->generateMatchExamples(),
            'tournaments' => $this->generateTournamentExamples(),
        ];

        // Crear archivo de ejemplos
        $examplesPath = storage_path('api-docs/examples.json');
        File::put($examplesPath, json_encode($examples, JSON_PRETTY_PRINT));

        // Generar colección de Postman
        $this->generatePostmanCollection($examples);

        $this->info('API examples generated successfully!');
    }

    protected function generatePlayerExamples()
    {
        return [
            'create_player' => [
                'description' => 'Create a new player with complete information',
                'request' => [
                    'method' => 'POST',
                    'url' => '/api/players',
                    'headers' => [
                        'Authorization' => 'Bearer {token}',
                        'Content-Type' => 'multipart/form-data',
                    ],
                    'body' => [
                        'name' => 'Carlos Rodríguez',
                        'team_id' => 1,
                        'position' => 'MID',
                        'jersey_number' => 10,
                        'birth_date' => '1995-03-15',
                        'height' => 1.78,
                        'weight' => 75.5,
                        'foot' => 'right',
                        'nationality' => 'Colombian',
                        'emergency_contact' => [
                            'name' => 'María Rodríguez',
                            'phone' => '+573001234567',
                            'relationship' => 'parent'
                        ]
                    ]
                ],
                'response' => [
                    'status' => 201,
                    'body' => [
                        'data' => [
                            'id' => 1,
                            'name' => 'Carlos Rodríguez',
                            'position' => 'MID',
                            'jersey_number' => 10,
                            'birth_date' => '1995-03-15',
                            'age' => 29,
                            'height' => 1.78,
                            'weight' => 75.5,
                            'foot' => 'right',
                            'nationality' => 'Colombian',
                            'team_id' => 1,
                            'created_at' => '2024-01-15T10:30:00Z',
                            'updated_at' => '2024-01-15T10:30:00Z'
                        ],
                        'message' => 'Player created successfully'
                    ]
                ]
            ],
            'filter_players' => [
                'description' => 'Get players with advanced filtering',
                'request' => [
                    'method' => 'GET',
                    'url' => '/api/players?search=Carlos&position=MID&age_range[min]=20&age_range[max]=30&sort_by=name&include=team,statistics',
                    'headers' => [
                        'Authorization' => 'Bearer {token}',
                        'Accept' => 'application/json',
                    ]
                ],
                'response' => [
                    'status' => 200,
                    'body' => [
                        'data' => [
                            [
                                'id' => 1,
                                'name' => 'Carlos Rodríguez',
                                'position' => 'MID',
                                'team' => [
                                    'id' => 1,
                                    'name' => 'Real Madrid'
                                ],
                                'statistics' => [
                                    'goals' => 15,
                                    'assists' => 8,
                                    'matches_played' => 22
                                ]
                            ]
                        ],
                        'meta' => [
                            'current_page' => 1,
                            'total' => 1,
                            'per_page' => 15
                        ]
                    ]
                ]
            ]
        ];
    }

    protected function generateMatchExamples()
    {
        return [
            'create_match' => [
                'description' => 'Schedule a new match with validation',
                'request' => [
                    'method' => 'POST',
                    'url' => '/api/matches',
                    'body' => [
                        'home_team_id' => 1,
                        'away_team_id' => 2,
                        'match_date' => '2024-02-15T15:00:00Z',
                        'venue' => 'Santiago Bernabéu',
                        'round' => 'Quarterfinal',
                        'referee_id' => 1
                    ]
                ]
            ],
            'update_result' => [
                'description' => 'Update match result',
                'request' => [
                    'method' => 'PUT',
                    'url' => '/api/matches/1/result',
                    'body' => [
                        'result' => [
                            'home' => 3,
                            'away' => 1
                        ],
                        'events' => [
                            [
                                'type' => 'goal',
                                'player_id' => 1,
                                'minute' => 25,
                                'description' => 'Header from corner kick'
                            ]
                        ]
                    ]
                ]
            ]
        ];
    }

    protected function generateTournamentExamples()
    {
        return [
            'standings' => [
                'description' => 'Get tournament standings with statistics',
                'request' => [
                    'method' => 'GET',
                    'url' => '/api/tournaments/1/standings'
                ],
                'response' => [
                    'status' => 200,
                    'body' => [
                        'data' => [
                            [
                                'id' => 1,
                                'name' => 'Real Madrid',
                                'points' => 15,
                                'wins' => 5,
                                'draws' => 0,
                                'losses' => 1,
                                'goals_for' => 18,
                                'goals_against' => 4,
                                'goal_difference' => 14
                            ]
                        ]
                    ]
                ]
            ]
        ];
    }

    protected function generatePostmanCollection($examples)
    {
        $collection = [
            'info' => [
                'name' => 'WorldSkills Tournament API',
                'description' => 'Complete API collection for tournament management',
                'schema' => 'https://schema.getpostman.com/json/collection/v2.1.0/collection.json'
            ],
            'variable' => [
                [
                    'key' => 'baseUrl',
                    'value' => 'http://localhost:8000',
                    'type' => 'string'
                ],
                [
                    'key' => 'token',
                    'value' => '',
                    'type' => 'string'
                ]
            ],
            'item' => []
        ];

        foreach ($examples as $resource => $resourceExamples) {
            $folder = [
                'name' => ucfirst($resource),
                'item' => []
            ];

            foreach ($resourceExamples as $exampleName => $example) {
                $request = [
                    'name' => str_replace('_', ' ', ucwords($exampleName, '_')),
                    'request' => [
                        'method' => $example['request']['method'],
                        'header' => [],
                        'url' => [
                            'raw' => '{{baseUrl}}' . $example['request']['url'],
                            'host' => ['{{baseUrl}}'],
                            'path' => explode('/', trim($example['request']['url'], '/'))
                        ]
                    ],
                    'response' => []
                ];

                // Agregar headers
                if (isset($example['request']['headers'])) {
                    foreach ($example['request']['headers'] as $key => $value) {
                        $request['request']['header'][] = [
                            'key' => $key,
                            'value' => $value,
                            'type' => 'text'
                        ];
                    }
                }

                // Agregar body si existe
                if (isset($example['request']['body'])) {
                    $request['request']['body'] = [
                        'mode' => 'raw',
                        'raw' => json_encode($example['request']['body'], JSON_PRETTY_PRINT),
                        'options' => [
                            'raw' => [
                                'language' => 'json'
                            ]
                        ]
                    ];
                }

                $folder['item'][] = $request;
            }

            $collection['item'][] = $folder;
        }

        $postmanPath = storage_path('api-docs/postman-collection.json');
        File::put($postmanPath, json_encode($collection, JSON_PRETTY_PRINT));
    }
}
```

### **Paso 2: Contract Testing (4 minutos)**

```php
// tests/Feature/ApiContractTest.php
<?php

namespace Tests\Feature;

use Tests\TestCase;
use App\Models\User;
use App\Models\Player;
use App\Models\Team;
use Laravel\Sanctum\Sanctum;
use Illuminate\Foundation\Testing\RefreshDatabase;

class ApiContractTest extends TestCase
{
    use RefreshDatabase;

    protected function setUp(): void
    {
        parent::setUp();

        // Crear usuario autenticado
        $user = User::factory()->create();
        Sanctum::actingAs($user);
    }

    /** @test */
    public function players_index_returns_expected_schema()
    {
        // Arrange
        $team = Team::factory()->create();
        Player::factory()->count(3)->create(['team_id' => $team->id]);

        // Act
        $response = $this->getJson('/api/players');

        // Assert - Verificar estructura de respuesta
        $response->assertStatus(200)
                 ->assertJsonStructure([
                     'data' => [
                         '*' => [
                             'id',
                             'name',
                             'position',
                             'jersey_number',
                             'birth_date',
                             'team_id',
                             'created_at',
                             'updated_at'
                         ]
                     ],
                     'meta' => [
                         'current_page',
                         'from',
                         'to',
                         'per_page',
                         'total',
                         'last_page'
                     ],
                     'links' => [
                         'first',
                         'last',
                         'prev',
                         'next'
                     ]
                 ]);

        // Verificar tipos de datos
        $data = $response->json('data.0');
        $this->assertIsInt($data['id']);
        $this->assertIsString($data['name']);
        $this->assertContains($data['position'], ['GK', 'DEF', 'MID', 'FWD']);
    }

    /** @test */
    public function players_create_validates_input_schema()
    {
        $team = Team::factory()->create();

        // Test con datos inválidos
        $response = $this->postJson('/api/players', [
            'name' => '', // Requerido
            'position' => 'INVALID', // Enum inválido
            'jersey_number' => 100, // Fuera de rango
            'team_id' => 999 // No existe
        ]);

        $response->assertStatus(422)
                 ->assertJsonStructure([
                     'message',
                     'errors' => [
                         'name',
                         'position',
                         'jersey_number',
                         'team_id'
                     ]
                 ]);
    }

    /** @test */
    public function api_returns_consistent_error_format()
    {
        // Test 404 error
        $response = $this->getJson('/api/players/999');

        $response->assertStatus(404)
                 ->assertJsonStructure([
                     'message',
                     'status_code',
                     'timestamp',
                     'path'
                 ]);

        // Verificar valores esperados
        $this->assertEquals(404, $response->json('status_code'));
        $this->assertEquals('/api/players/999', $response->json('path'));
    }

    /** @test */
    public function api_includes_performance_headers()
    {
        $response = $this->getJson('/api/players');

        $response->assertStatus(200);

        // Verificar headers de performance
        $this->assertNotNull($response->headers->get('X-Response-Time'));
        $this->assertNotNull($response->headers->get('X-Memory-Usage'));
        $this->assertNotNull($response->headers->get('X-Query-Count'));
    }
}
```

---

## ⏱️ **Ejercicio 4: Documentation Portal (8 minutos)**

### **Paso 1: Custom Swagger UI (5 minutos)**

```php
// resources/views/vendor/l5-swagger/index.blade.php
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>{{config('l5-swagger.documentations.'.$documentation.'.api.title')}}</title>
    <link rel="stylesheet" type="text/css" href="{{ l5_swagger_asset($documentation, 'swagger-ui-bundle.css') }}" />
    <link rel="stylesheet" type="text/css" href="{{ l5_swagger_asset($documentation, 'swagger-ui-standalone-preset.css') }}" />
    <style>
        html {
            box-sizing: border-box;
            overflow: -moz-scrollbars-vertical;
            overflow-y: scroll;
        }
        *, *:before, *:after {
            box-sizing: inherit;
        }
        body {
            margin:0;
            background: #fafafa;
            font-family: "Segoe UI", Tahoma, Geneva, Verdana, sans-serif;
        }

        /* Custom styles for WorldSkills branding */
        .topbar {
            background: #0066cc !important;
        }

        .swagger-ui .topbar .link {
            color: white !important;
        }

        .swagger-ui .info .title {
            color: #0066cc;
        }

        /* Highlight different HTTP methods */
        .swagger-ui .opblock.opblock-post {
            border-color: #49cc90;
        }

        .swagger-ui .opblock.opblock-get {
            border-color: #61affe;
        }

        .swagger-ui .opblock.opblock-put {
            border-color: #fca130;
        }

        .swagger-ui .opblock.opblock-delete {
            border-color: #f93e3e;
        }

        /* Custom header */
        .api-header {
            background: linear-gradient(135deg, #0066cc 0%, #004499 100%);
            color: white;
            padding: 20px;
            text-align: center;
            margin-bottom: 20px;
        }

        .api-header h1 {
            margin: 0;
            font-size: 2.5em;
            font-weight: 300;
        }

        .api-header p {
            margin: 10px 0 0 0;
            font-size: 1.2em;
            opacity: 0.9;
        }

        .quick-links {
            background: white;
            padding: 15px;
            margin: 20px;
            border-radius: 8px;
            box-shadow: 0 2px 4px rgba(0,0,0,0.1);
        }

        .quick-links h3 {
            margin-top: 0;
            color: #0066cc;
        }

        .quick-links a {
            display: inline-block;
            margin: 5px 10px 5px 0;
            padding: 8px 16px;
            background: #f0f8ff;
            color: #0066cc;
            text-decoration: none;
            border-radius: 4px;
            border: 1px solid #e0e6ed;
            transition: all 0.2s;
        }

        .quick-links a:hover {
            background: #0066cc;
            color: white;
        }
    </style>
</head>

<body>
    <div class="api-header">
        <h1>🏆 WorldSkills Tournament API</h1>
        <p>Complete RESTful API for tournament management • Version {{ config('l5-swagger.documentations.'.$documentation.'.api.version') }}</p>
    </div>

    <div class="quick-links">
        <h3>📚 Quick Start</h3>
        <a href="#/Authentication/loginUser">🔐 Authentication</a>
        <a href="#/Players/getPlayers">👤 Players</a>
        <a href="#/Teams/getTeams">⚽ Teams</a>
        <a href="#/Matches/getMatches">🏟️ Matches</a>
        <a href="#/Tournaments/getTournaments">🏆 Tournaments</a>

        <h3>🛠️ Developer Tools</h3>
        <a href="{{ url('api-docs.json') }}" target="_blank">📄 OpenAPI JSON</a>
        <a href="{{ url('storage/api-docs/postman-collection.json') }}" target="_blank">📮 Postman Collection</a>
        <a href="{{ url('api/performance/metrics') }}" target="_blank">📊 Performance Metrics</a>
    </div>

    <div id="swagger-ui"></div>

    <script src="{{ l5_swagger_asset($documentation, 'swagger-ui-bundle.js') }}"></script>
    <script src="{{ l5_swagger_asset($documentation, 'swagger-ui-standalone-preset.js') }}"></script>
    <script>
        window.onload = function() {
            const ui = SwaggerUIBundle({
                url: "{!! $urlToDocs !!}",
                dom_id: '#swagger-ui',
                deepLinking: true,
                presets: [
                    SwaggerUIBundle.presets.apis,
                    SwaggerUIStandalonePreset
                ],
                plugins: [
                    SwaggerUIBundle.plugins.DownloadUrl
                ],
                layout: "StandaloneLayout",
                requestInterceptor: function(request) {
                    // Auto-add bearer token if available in localStorage
                    const token = localStorage.getItem('api_token');
                    if (token && !request.headers.Authorization) {
                        request.headers.Authorization = `Bearer ${token}`;
                    }
                    return request;
                },
                responseInterceptor: function(response) {
                    // Log performance metrics
                    console.log('API Response Time:', response.headers['x-response-time']);
                    console.log('Memory Usage:', response.headers['x-memory-usage']);
                    return response;
                },
                onComplete: function() {
                    // Custom completion logic
                    console.log('Swagger UI loaded successfully');

                    // Add token input helper
                    const authSection = document.querySelector('.auth-wrapper');
                    if (authSection) {
                        const tokenInput = document.createElement('div');
                        tokenInput.innerHTML = `
                            <div style="margin: 10px 0; padding: 10px; background: #f8f9fa; border-radius: 4px;">
                                <label>🔑 Quick Token Setup:</label>
                                <input type="text" id="quick-token" placeholder="Paste your token here" style="width: 300px; margin: 0 10px;">
                                <button onclick="setQuickToken()" style="padding: 5px 10px;">Apply</button>
                            </div>
                        `;
                        authSection.appendChild(tokenInput);
                    }
                }
            });

            // Helper function for quick token setup
            window.setQuickToken = function() {
                const token = document.getElementById('quick-token').value;
                if (token) {
                    localStorage.setItem('api_token', token);
                    location.reload();
                }
            };
        };
    </script>
</body>
</html>
```

### **Paso 3: Generate Documentation (3 minutos)**

```bash
# Generar documentación
php artisan l5-swagger:generate

# Generar ejemplos
php artisan api:examples

# Probar documentación
curl http://localhost:8000/api/documentation
```

---

## 🎯 **URLs de Documentación**

```bash
# Swagger UI interactivo
http://localhost:8000/api/documentation

# OpenAPI JSON
http://localhost:8000/api-docs.json

# Postman Collection
http://localhost:8000/storage/api-docs/postman-collection.json

# Métricas de performance
http://localhost:8000/api/performance/metrics
```

---

## 🎯 **Rúbrica de Evaluación**

| Criterio             | Excelente (4)                     | Bueno (3)                  | Satisfactorio (2)     | Insuficiente (1)    |
| -------------------- | --------------------------------- | -------------------------- | --------------------- | ------------------- |
| **OpenAPI Spec**     | Esquemas completos + ejemplos     | Especificación básica      | Documentación parcial | Sin especificación  |
| **Interactive UI**   | Swagger personalizado + funcional | Swagger básico funcionando | UI estándar           | Sin interfaz        |
| **Examples**         | Ejemplos realistas + Postman      | Algunos ejemplos           | Ejemplos básicos      | Sin ejemplos        |
| **Contract Testing** | Tests de esquema + validación     | Algunos tests              | Tests básicos         | Sin testing         |
| **Customization**    | Branding + UX personalizada       | Algunas personalizaciones  | Configuración básica  | Sin personalización |
| **Tiempo**           | < 12 minutos                      | 12-18 min                  | 18-25 min             | > 25 min            |

---

## 🔧 **Commands Útiles**

```bash
# Regenerar docs después de cambios
php artisan l5-swagger:generate

# Validar OpenAPI spec
curl -X POST "https://validator.swagger.io/validator/debug" \
     -H "Content-Type: application/json" \
     -d @storage/api-docs/api-docs.json

# Test de documentación automático
php artisan test --group=contract
```

---

**📚 Con esta documentación interactiva, tu API tendrá la calidad profesional que exige WorldSkills!**
