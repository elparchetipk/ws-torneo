# 🔍 API Filters y Paginación Avanzada - Día 3.2

**Objetivo:** Implementar filtros dinámicos y paginación optimizada en menos de 20 minutos

---

## 🎯 **Objetivo de la Sesión**

Los competidores deben poder crear **sistemas de filtrado dinámico** con paginación inteligente, búsqueda full-text, y ordenamientos complejos que manejen grandes volúmenes de datos deportivos en **menos de 20 minutos**.

---

## 🚀 **Arquitectura de Filtros Deportivos**

### **Casos de Filtrado WorldSkills**

```text
🏆 FILTROS PARA SISTEMA DEPORTIVO

Players Endpoint:
├─ Basic: name, position, team, age_range
├─ Advanced: height_range, nationality, injury_status
├─ Relations: team.tournament, team.category
└─ Performance: stats.goals, stats.assists, ranking

Matches Endpoint:
├─ Basic: date_range, teams, status, round
├─ Advanced: venue, referee, attendance_range
├─ Relations: tournament.category, teams.players_count
└─ Results: score_range, duration, weather_conditions

Tournaments Endpoint:
├─ Basic: name, status, category, dates
├─ Advanced: location, format, prize_money
├─ Relations: teams_count, matches_played
└─ Statistics: goals_average, attendance_total
```

---

## ⏱️ **Ejercicio 1: Query Builder Dinámico (25 minutos)**

### **Paso 1: Base Filter Class (8 minutos)**

```bash
# CRONÓMETRO: INICIAR ⏰

# Crear estructura de filtros
mkdir -p app/Http/Filters
touch app/Http/Filters/BaseFilter.php
touch app/Http/Filters/PlayerFilter.php
touch app/Http/Filters/MatchFilter.php
touch app/Http/Filters/TournamentFilter.php
```

```php
// app/Http/Filters/BaseFilter.php
<?php

namespace App\Http\Filters;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Http\Request;

abstract class BaseFilter
{
    protected $request;
    protected $builder;
    protected $filters = [];

    public function __construct(Request $request)
    {
        $this->request = $request;
    }

    public function apply(Builder $builder)
    {
        $this->builder = $builder;

        foreach ($this->getFilters() as $filter => $value) {
            if (method_exists($this, $filter)) {
                $this->$filter($value);
            }
        }

        return $this->builder;
    }

    public function getFilters()
    {
        return array_filter($this->request->only(array_keys($this->filters())));
    }

    abstract public function filters(): array;

    // Helper methods para filtros comunes
    protected function filterByDateRange($field, $value)
    {
        if (isset($value['from'])) {
            $this->builder->whereDate($field, '>=', $value['from']);
        }

        if (isset($value['to'])) {
            $this->builder->whereDate($field, '<=', $value['to']);
        }
    }

    protected function filterByNumericRange($field, $value)
    {
        if (isset($value['min']) && is_numeric($value['min'])) {
            $this->builder->where($field, '>=', $value['min']);
        }

        if (isset($value['max']) && is_numeric($value['max'])) {
            $this->builder->where($field, '<=', $value['max']);
        }
    }

    protected function filterBySearch($fields, $value)
    {
        $searchTerm = trim($value);

        if (empty($searchTerm)) {
            return;
        }

        $this->builder->where(function ($query) use ($fields, $searchTerm) {
            foreach ($fields as $field) {
                if (str_contains($field, '.')) {
                    // Relación
                    [$relation, $relationField] = explode('.', $field);
                    $query->orWhereHas($relation, function ($subQuery) use ($relationField, $searchTerm) {
                        $subQuery->where($relationField, 'LIKE', "%{$searchTerm}%");
                    });
                } else {
                    // Campo directo
                    $query->orWhere($field, 'LIKE', "%{$searchTerm}%");
                }
            }
        });
    }

    protected function filterByRelationExists($relation, $exists = true)
    {
        if ($exists) {
            $this->builder->has($relation);
        } else {
            $this->builder->doesntHave($relation);
        }
    }
}
```

### **Paso 2: Player Filter Completo (10 minutos)**

```php
// app/Http/Filters/PlayerFilter.php
<?php

namespace App\Http\Filters;

class PlayerFilter extends BaseFilter
{
    public function filters(): array
    {
        return [
            'search' => 'search',
            'team_id' => 'teamId',
            'team_ids' => 'teamIds',
            'position' => 'position',
            'positions' => 'positions',
            'age_range' => 'ageRange',
            'height_range' => 'heightRange',
            'weight_range' => 'weightRange',
            'nationality' => 'nationality',
            'foot' => 'foot',
            'injury_status' => 'injuryStatus',
            'tournament_id' => 'tournamentId',
            'category' => 'category',
            'jersey_number' => 'jerseyNumber',
            'created_at' => 'createdAt',
            'has_stats' => 'hasStats',
            'goals_range' => 'goalsRange',
            'assists_range' => 'assistsRange',
        ];
    }

    public function search($value)
    {
        $this->filterBySearch([
            'name',
            'team.name',
            'team.tournament.name'
        ], $value);
    }

    public function teamId($value)
    {
        $this->builder->where('team_id', $value);
    }

    public function teamIds($value)
    {
        $teamIds = is_array($value) ? $value : explode(',', $value);
        $this->builder->whereIn('team_id', $teamIds);
    }

    public function position($value)
    {
        $this->builder->where('position', $value);
    }

    public function positions($value)
    {
        $positions = is_array($value) ? $value : explode(',', $value);
        $this->builder->whereIn('position', $positions);
    }

    public function ageRange($value)
    {
        $this->builder->when(isset($value['min']), function ($query) use ($value) {
            $maxBirthDate = now()->subYears($value['min'])->format('Y-m-d');
            $query->whereDate('birth_date', '<=', $maxBirthDate);
        })->when(isset($value['max']), function ($query) use ($value) {
            $minBirthDate = now()->subYears($value['max'] + 1)->addDay()->format('Y-m-d');
            $query->whereDate('birth_date', '>=', $minBirthDate);
        });
    }

    public function heightRange($value)
    {
        $this->filterByNumericRange('height', $value);
    }

    public function weightRange($value)
    {
        $this->filterByNumericRange('weight', $value);
    }

    public function nationality($value)
    {
        $this->builder->where('nationality', $value);
    }

    public function foot($value)
    {
        $this->builder->where('foot', $value);
    }

    public function injuryStatus($value)
    {
        switch ($value) {
            case 'injured':
                $this->builder->whereNotNull('injury_date')
                             ->whereNull('recovery_date');
                break;
            case 'recovered':
                $this->builder->whereNotNull('injury_date')
                             ->whereNotNull('recovery_date');
                break;
            case 'never_injured':
                $this->builder->whereNull('injury_date');
                break;
            case 'fit':
                $this->builder->where(function ($query) {
                    $query->whereNull('injury_date')
                          ->orWhere(function ($subQuery) {
                              $subQuery->whereNotNull('recovery_date')
                                      ->where('recovery_date', '<=', now());
                          });
                });
                break;
        }
    }

    public function tournamentId($value)
    {
        $this->builder->whereHas('team', function ($query) use ($value) {
            $query->where('tournament_id', $value);
        });
    }

    public function category($value)
    {
        $this->builder->whereHas('team.tournament', function ($query) use ($value) {
            $query->where('category', $value);
        });
    }

    public function jerseyNumber($value)
    {
        $this->builder->where('jersey_number', $value);
    }

    public function createdAt($value)
    {
        $this->filterByDateRange('created_at', $value);
    }

    public function hasStats($value)
    {
        $this->filterByRelationExists('statistics', filter_var($value, FILTER_VALIDATE_BOOLEAN));
    }

    public function goalsRange($value)
    {
        $this->builder->whereHas('statistics', function ($query) use ($value) {
            if (isset($value['min'])) {
                $query->where('goals', '>=', $value['min']);
            }
            if (isset($value['max'])) {
                $query->where('goals', '<=', $value['max']);
            }
        });
    }

    public function assistsRange($value)
    {
        $this->builder->whereHas('statistics', function ($query) use ($value) {
            if (isset($value['min'])) {
                $query->where('assists', '>=', $value['min']);
            }
            if (isset($value['max'])) {
                $query->where('assists', '<=', $value['max']);
            }
        });
    }
}
```

### **Paso 3: Match Filter (7 minutos)**

```php
// app/Http/Filters/MatchFilter.php
<?php

namespace App\Http\Filters;

class MatchFilter extends BaseFilter
{
    public function filters(): array
    {
        return [
            'search' => 'search',
            'team_id' => 'teamId',
            'teams' => 'teams',
            'date_range' => 'dateRange',
            'status' => 'status',
            'round' => 'round',
            'rounds' => 'rounds',
            'venue' => 'venue',
            'referee_id' => 'refereeId',
            'tournament_id' => 'tournamentId',
            'has_result' => 'hasResult',
            'score_range' => 'scoreRange',
            'attendance_range' => 'attendanceRange',
            'duration_range' => 'durationRange',
            'weather' => 'weather',
        ];
    }

    public function search($value)
    {
        $this->filterBySearch([
            'venue',
            'homeTeam.name',
            'awayTeam.name',
            'referee.name'
        ], $value);
    }

    public function teamId($value)
    {
        $this->builder->where(function ($query) use ($value) {
            $query->where('home_team_id', $value)
                  ->orWhere('away_team_id', $value);
        });
    }

    public function teams($value)
    {
        $teamIds = is_array($value) ? $value : explode(',', $value);

        $this->builder->where(function ($query) use ($teamIds) {
            $query->whereIn('home_team_id', $teamIds)
                  ->orWhereIn('away_team_id', $teamIds);
        });
    }

    public function dateRange($value)
    {
        $this->filterByDateRange('match_date', $value);
    }

    public function status($value)
    {
        $this->builder->where('status', $value);
    }

    public function round($value)
    {
        $this->builder->where('round', $value);
    }

    public function rounds($value)
    {
        $rounds = is_array($value) ? $value : explode(',', $value);
        $this->builder->whereIn('round', $rounds);
    }

    public function venue($value)
    {
        $this->builder->where('venue', 'LIKE', "%{$value}%");
    }

    public function refereeId($value)
    {
        $this->builder->where('referee_id', $value);
    }

    public function tournamentId($value)
    {
        $this->builder->whereHas('homeTeam', function ($query) use ($value) {
            $query->where('tournament_id', $value);
        });
    }

    public function hasResult($value)
    {
        $hasResult = filter_var($value, FILTER_VALIDATE_BOOLEAN);

        if ($hasResult) {
            $this->builder->whereNotNull('result');
        } else {
            $this->builder->whereNull('result');
        }
    }

    public function scoreRange($value)
    {
        $this->builder->where(function ($query) use ($value) {
            if (isset($value['min_goals'])) {
                $query->whereRaw("JSON_EXTRACT(result, '$.home') + JSON_EXTRACT(result, '$.away') >= ?", [$value['min_goals']]);
            }

            if (isset($value['max_goals'])) {
                $query->whereRaw("JSON_EXTRACT(result, '$.home') + JSON_EXTRACT(result, '$.away') <= ?", [$value['max_goals']]);
            }

            if (isset($value['goal_difference'])) {
                $query->whereRaw("ABS(JSON_EXTRACT(result, '$.home') - JSON_EXTRACT(result, '$.away')) = ?", [$value['goal_difference']]);
            }
        });
    }

    public function attendanceRange($value)
    {
        $this->filterByNumericRange('attendance', $value);
    }

    public function durationRange($value)
    {
        $this->filterByNumericRange('duration_minutes', $value);
    }

    public function weather($value)
    {
        $this->builder->where('weather_conditions', 'LIKE', "%{$value}%");
    }
}
```

---

## ⏱️ **Ejercicio 2: Paginación Inteligente (20 minutos)**

### **Paso 1: Custom Paginator (12 minutos)**

```php
// app/Http/Resources/PaginatedCollection.php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\ResourceCollection;

class PaginatedCollection extends ResourceCollection
{
    protected $resourceClass;
    protected $meta = [];

    public function __construct($resource, $resourceClass = null, $meta = [])
    {
        parent::__construct($resource);
        $this->resourceClass = $resourceClass;
        $this->meta = $meta;
    }

    public function toArray($request)
    {
        return [
            'data' => $this->resourceClass
                ? $this->resourceClass::collection($this->collection)
                : $this->collection,
            'meta' => array_merge([
                'current_page' => $this->currentPage(),
                'from' => $this->firstItem(),
                'to' => $this->lastItem(),
                'per_page' => $this->perPage(),
                'total' => $this->total(),
                'last_page' => $this->lastPage(),
                'path' => $this->path(),
                'has_more_pages' => $this->hasMorePages(),
            ], $this->meta),
            'links' => [
                'first' => $this->url(1),
                'last' => $this->url($this->lastPage()),
                'prev' => $this->previousPageUrl(),
                'next' => $this->nextPageUrl(),
                'self' => $this->url($this->currentPage()),
            ],
            'filters' => $this->getActiveFilters($request),
            'sorting' => $this->getActiveSorting($request),
        ];
    }

    protected function getActiveFilters($request)
    {
        $filters = [];
        $filterParams = ['search', 'team_id', 'position', 'age_range', 'height_range', 'status', 'date_range'];

        foreach ($filterParams as $param) {
            if ($request->has($param) && !empty($request->get($param))) {
                $filters[$param] = $request->get($param);
            }
        }

        return $filters;
    }

    protected function getActiveSorting($request)
    {
        return [
            'sort_by' => $request->get('sort_by', 'id'),
            'sort_direction' => $request->get('sort_direction', 'asc'),
        ];
    }
}
```

### **Paso 2: Controller con Filtros y Paginación (8 minutos)**

```php
// app/Http/Controllers/Api/PlayerController.php - Método index mejorado
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use App\Http\Filters\PlayerFilter;
use App\Http\Resources\PlayerResource;
use App\Http\Resources\PaginatedCollection;
use App\Models\Player;
use Illuminate\Http\Request;

class PlayerController extends Controller
{
    public function index(Request $request, PlayerFilter $filter)
    {
        // Validar parámetros de entrada
        $validated = $request->validate([
            'page' => 'sometimes|integer|min:1',
            'per_page' => 'sometimes|integer|min:1|max:100',
            'sort_by' => 'sometimes|string|in:name,position,jersey_number,birth_date,height,weight,created_at',
            'sort_direction' => 'sometimes|string|in:asc,desc',
            'include' => 'sometimes|string',

            // Filtros específicos
            'search' => 'sometimes|string|max:100',
            'team_id' => 'sometimes|integer|exists:teams,id',
            'team_ids' => 'sometimes|string',
            'position' => 'sometimes|string|in:GK,DEF,MID,FWD',
            'positions' => 'sometimes|string',
            'age_range' => 'sometimes|array',
            'age_range.min' => 'sometimes|integer|min:16|max:50',
            'age_range.max' => 'sometimes|integer|min:16|max:50',
            'height_range' => 'sometimes|array',
            'nationality' => 'sometimes|string|max:50',
            'injury_status' => 'sometimes|string|in:injured,recovered,never_injured,fit',
            'tournament_id' => 'sometimes|integer|exists:tournaments,id',
            'has_stats' => 'sometimes|boolean',
        ]);

        // Query base con relaciones eager loading
        $query = Player::query();

        // Aplicar relaciones solicitadas
        $includes = $this->parseIncludes($request->get('include', ''));
        if (!empty($includes)) {
            $query->with($includes);
        } else {
            // Relaciones por defecto para evitar N+1
            $query->with(['team', 'statistics']);
        }

        // Aplicar filtros
        $query = $filter->apply($query);

        // Aplicar ordenamiento
        $sortBy = $validated['sort_by'] ?? 'name';
        $sortDirection = $validated['sort_direction'] ?? 'asc';

        // Ordenamientos especiales
        switch ($sortBy) {
            case 'age':
                $query->orderBy('birth_date', $sortDirection === 'asc' ? 'desc' : 'asc');
                break;
            case 'team_name':
                $query->join('teams', 'players.team_id', '=', 'teams.id')
                      ->orderBy('teams.name', $sortDirection)
                      ->select('players.*');
                break;
            default:
                $query->orderBy($sortBy, $sortDirection);
        }

        // Paginación
        $perPage = min($validated['per_page'] ?? 15, 100);
        $players = $query->paginate($perPage);

        // Agregar metadata adicional
        $meta = [
            'total_teams' => $this->getTotalTeamsCount($filter),
            'position_counts' => $this->getPositionCounts($filter),
            'age_stats' => $this->getAgeStatistics($filter),
        ];

        return new PaginatedCollection($players, PlayerResource::class, $meta);
    }

    protected function parseIncludes($include)
    {
        if (empty($include)) {
            return [];
        }

        $allowedIncludes = [
            'team',
            'team.tournament',
            'statistics',
            'injuries',
            'transfers',
            'contracts'
        ];

        $requestedIncludes = explode(',', $include);

        return array_intersect($requestedIncludes, $allowedIncludes);
    }

    protected function getTotalTeamsCount(PlayerFilter $filter)
    {
        $query = Player::query();
        $query = $filter->apply($query);

        return $query->distinct('team_id')->count('team_id');
    }

    protected function getPositionCounts(PlayerFilter $filter)
    {
        $query = Player::query();
        $query = $filter->apply($query);

        return $query->selectRaw('position, COUNT(*) as count')
                     ->groupBy('position')
                     ->pluck('count', 'position')
                     ->toArray();
    }

    protected function getAgeStatistics(PlayerFilter $filter)
    {
        $query = Player::query();
        $query = $filter->apply($query);

        $stats = $query->selectRaw('
            AVG(YEAR(CURDATE()) - YEAR(birth_date)) as avg_age,
            MIN(YEAR(CURDATE()) - YEAR(birth_date)) as min_age,
            MAX(YEAR(CURDATE()) - YEAR(birth_date)) as max_age
        ')->first();

        return [
            'average' => round($stats->avg_age ?? 0, 1),
            'minimum' => $stats->min_age ?? 0,
            'maximum' => $stats->max_age ?? 0,
        ];
    }
}
```

---

## ⏱️ **Ejercicio 3: Búsqueda Full-Text (15 minutos)**

### **Paso 1: Search Service (10 minutos)**

```php
// app/Services/SearchService.php
<?php

namespace App\Services;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Support\Str;

class SearchService
{
    protected $weights = [
        'exact' => 10,
        'start' => 8,
        'word' => 6,
        'partial' => 3,
        'related' => 2,
    ];

    public function searchPlayers($query, $searchTerm)
    {
        if (empty(trim($searchTerm))) {
            return $query;
        }

        $searchTerm = trim($searchTerm);
        $words = explode(' ', $searchTerm);

        return $query->where(function ($q) use ($searchTerm, $words) {
            // Búsqueda exacta en nombre
            $q->orWhere('name', '=', $searchTerm)
              ->orWhere('name', 'LIKE', "{$searchTerm}%");

            // Búsqueda por palabras
            foreach ($words as $word) {
                if (strlen($word) >= 2) {
                    $q->orWhere('name', 'LIKE', "%{$word}%");
                }
            }

            // Búsqueda en equipo
            $q->orWhereHas('team', function ($teamQuery) use ($searchTerm, $words) {
                $teamQuery->where('name', 'LIKE', "%{$searchTerm}%");

                foreach ($words as $word) {
                    if (strlen($word) >= 2) {
                        $teamQuery->orWhere('name', 'LIKE', "%{$word}%");
                    }
                }
            });

            // Búsqueda en torneo
            $q->orWhereHas('team.tournament', function ($tournamentQuery) use ($searchTerm) {
                $tournamentQuery->where('name', 'LIKE', "%{$searchTerm}%");
            });

            // Búsqueda por número de camiseta si es numérico
            if (is_numeric($searchTerm)) {
                $q->orWhere('jersey_number', $searchTerm);
            }
        });
    }

    public function searchMatches($query, $searchTerm)
    {
        if (empty(trim($searchTerm))) {
            return $query;
        }

        $searchTerm = trim($searchTerm);

        return $query->where(function ($q) use ($searchTerm) {
            // Búsqueda en venue
            $q->orWhere('venue', 'LIKE', "%{$searchTerm}%");

            // Búsqueda en equipos
            $q->orWhereHas('homeTeam', function ($teamQuery) use ($searchTerm) {
                $teamQuery->where('name', 'LIKE', "%{$searchTerm}%");
            });

            $q->orWhereHas('awayTeam', function ($teamQuery) use ($searchTerm) {
                $teamQuery->where('name', 'LIKE', "%{$searchTerm}%");
            });

            // Búsqueda en árbitro
            $q->orWhereHas('referee', function ($refQuery) use ($searchTerm) {
                $refQuery->where('name', 'LIKE', "%{$searchTerm}%");
            });

            // Búsqueda por fecha si parece una fecha
            if ($this->isDateLike($searchTerm)) {
                $q->orWhereDate('match_date', $searchTerm);
            }
        });
    }

    public function searchWithScoring($query, $searchTerm, $type = 'players')
    {
        if (empty(trim($searchTerm))) {
            return $query;
        }

        $searchTerm = trim($searchTerm);
        $words = explode(' ', $searchTerm);

        // Agregar score calculado
        $scoreCase = "CASE ";

        switch ($type) {
            case 'players':
                // Nombre exacto
                $scoreCase .= "WHEN name = '{$searchTerm}' THEN {$this->weights['exact']} ";
                // Nombre empieza con
                $scoreCase .= "WHEN name LIKE '{$searchTerm}%' THEN {$this->weights['start']} ";
                // Contiene nombre
                $scoreCase .= "WHEN name LIKE '%{$searchTerm}%' THEN {$this->weights['partial']} ";
                break;

            case 'matches':
                // Venue exacto
                $scoreCase .= "WHEN venue = '{$searchTerm}' THEN {$this->weights['exact']} ";
                // Venue contiene
                $scoreCase .= "WHEN venue LIKE '%{$searchTerm}%' THEN {$this->weights['partial']} ";
                break;
        }

        $scoreCase .= "ELSE 1 END";

        return $query->selectRaw("*, ({$scoreCase}) as search_score")
                     ->where(function ($q) use ($searchTerm, $words, $type) {
                         if ($type === 'players') {
                             $this->searchPlayers($q, $searchTerm);
                         } elseif ($type === 'matches') {
                             $this->searchMatches($q, $searchTerm);
                         }
                     })
                     ->orderBy('search_score', 'desc');
    }

    protected function isDateLike($string)
    {
        return preg_match('/^\d{4}-\d{2}-\d{2}$/', $string) ||
               preg_match('/^\d{2}\/\d{2}\/\d{4}$/', $string);
    }
}
```

### **Paso 3: Integrar Search en Controller (5 minutos)**

```php
// Agregar al PlayerController
use App\Services\SearchService;

public function search(Request $request, SearchService $searchService)
{
    $validated = $request->validate([
        'q' => 'required|string|min:2|max:100',
        'type' => 'sometimes|string|in:exact,fuzzy,scored',
        'include_stats' => 'sometimes|boolean',
        'per_page' => 'sometimes|integer|min:1|max:50',
    ]);

    $query = Player::with(['team', 'statistics']);

    // Aplicar búsqueda según tipo
    switch ($validated['type'] ?? 'scored') {
        case 'exact':
            $query->where('name', 'LIKE', "%{$validated['q']}%");
            break;

        case 'fuzzy':
            $query = $searchService->searchPlayers($query, $validated['q']);
            break;

        case 'scored':
            $query = $searchService->searchWithScoring($query, $validated['q'], 'players');
            break;
    }

    $perPage = $validated['per_page'] ?? 10;
    $results = $query->paginate($perPage);

    return new PaginatedCollection($results, PlayerResource::class, [
        'search_term' => $validated['q'],
        'search_type' => $validated['type'] ?? 'scored',
    ]);
}
```

---

## ⏱️ **Ejercicio 4: Sorting Avanzado (10 minutos)**

### **Paso 1: Sortable Trait (6 minutos)**

```php
// app/Traits/Sortable.php
<?php

namespace App\Traits;

use Illuminate\Database\Eloquent\Builder;

trait Sortable
{
    public function scopeSortBy(Builder $query, $field, $direction = 'asc')
    {
        $direction = strtolower($direction) === 'desc' ? 'desc' : 'asc';

        // Campos permitidos para ordenamiento
        $allowedFields = $this->getSortableFields();

        if (!in_array($field, array_keys($allowedFields))) {
            return $query;
        }

        $sortConfig = $allowedFields[$field];

        // Si es un campo simple
        if (is_string($sortConfig)) {
            return $query->orderBy($sortConfig, $direction);
        }

        // Si es configuración compleja
        if (isset($sortConfig['relation'])) {
            return $this->sortByRelation($query, $sortConfig, $direction);
        }

        if (isset($sortConfig['raw'])) {
            return $query->orderByRaw($sortConfig['raw'] . ' ' . $direction);
        }

        return $query;
    }

    protected function sortByRelation(Builder $query, $config, $direction)
    {
        $relation = $config['relation'];
        $field = $config['field'];
        $table = $config['table'] ?? $relation;

        return $query->join($table, function ($join) use ($relation, $table) {
            $join->on($this->getTable() . '.' . $relation . '_id', '=', $table . '.id');
        })->orderBy($table . '.' . $field, $direction)
          ->select($this->getTable() . '.*');
    }

    abstract protected function getSortableFields(): array;
}
```

### **Paso 2: Implementar en Player Model (4 minutos)**

```php
// En Player Model - agregar trait y método
use App\Traits\Sortable;

class Player extends Model
{
    use Sortable;

    protected function getSortableFields(): array
    {
        return [
            'name' => 'name',
            'position' => 'position',
            'jersey_number' => 'jersey_number',
            'age' => [
                'raw' => 'YEAR(CURDATE()) - YEAR(birth_date)'
            ],
            'height' => 'height',
            'weight' => 'weight',
            'team_name' => [
                'relation' => 'team',
                'field' => 'name',
                'table' => 'teams'
            ],
            'goals' => [
                'relation' => 'statistics',
                'field' => 'goals',
                'table' => 'player_statistics'
            ],
            'assists' => [
                'relation' => 'statistics',
                'field' => 'assists',
                'table' => 'player_statistics'
            ],
            'created_at' => 'created_at',
        ];
    }
}

// Usar en controller
$query->sortBy(
    $request->get('sort_by', 'name'),
    $request->get('sort_direction', 'asc')
);
```

---

## 🎯 **Ejemplos de URLs de API**

```bash
# Filtros básicos
GET /api/players?position=GK&team_id=1

# Filtros de rango
GET /api/players?age_range[min]=20&age_range[max]=30&height_range[min]=1.80

# Búsqueda y filtros combinados
GET /api/players?search=carlos&positions=MID,FWD&has_stats=true

# Paginación personalizada
GET /api/players?page=2&per_page=25&sort_by=goals&sort_direction=desc

# Includes de relaciones
GET /api/players?include=team,statistics,injuries

# Búsqueda avanzada
GET /api/players/search?q=Real Madrid&type=scored&include_stats=true

# Filtros de partidos
GET /api/matches?date_range[from]=2024-01-01&status=completed&has_result=true

# Filtros complejos de resultados
GET /api/matches?score_range[min_goals]=3&score_range[goal_difference]=1
```

---

## 🎯 **Rúbrica de Evaluación**

| Criterio          | Excelente (4)                 | Bueno (3)                    | Satisfactorio (2)             | Insuficiente (1)     |
| ----------------- | ----------------------------- | ---------------------------- | ----------------------------- | -------------------- |
| **Query Builder** | Filtros dinámicos complejos   | Filtros básicos funcionando  | Algunos filtros implementados | Filtros hardcodeados |
| **Paginación**    | Meta-data completa + links    | Paginación básica            | Solo límites                  | Sin paginación       |
| **Búsqueda**      | Full-text con scoring         | Búsqueda en múltiples campos | Búsqueda básica               | Like simple          |
| **Sorting**       | Múltiples campos + relaciones | Sorting básico               | Un campo                      | Sin sorting          |
| **Performance**   | Eager loading + N+1 evitado   | Algunas optimizaciones       | Queries básicas               | Queries ineficientes |
| **Tiempo**        | < 20 minutos                  | 20-25 min                    | 25-35 min                     | > 35 min             |

---

## 🔧 **Tips de Performance**

```php
// ✅ Bueno - Evitar N+1
$players = Player::with(['team', 'statistics'])->paginate(15);

// ❌ Malo - N+1 queries
$players = Player::paginate(15);
foreach ($players as $player) {
    echo $player->team->name; // Query por cada jugador
}

// ✅ Bueno - Contar relaciones
$teams = Team::withCount('players')->get();

// ✅ Bueno - Filtrar en DB, no en PHP
$players = Player::where('birth_date', '>', '1990-01-01')->get();

// ❌ Malo - Filtrar en PHP
$players = Player::all()->filter(fn($p) => $p->birth_date > '1990-01-01');
```

---

**🚀 Con estos filtros y paginación, las APIs manejarán miles de registros deportivos con performance profesional!**
