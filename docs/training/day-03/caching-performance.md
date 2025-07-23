# ⚡ Caching y Optimización de Performance - Día 3.3

**Objetivo:** Implementar estrategias de cache y optimización para APIs de alto rendimiento en menos de 18 minutos

---

## 🎯 **Objetivo de la Sesión**

Los competidores deben poder implementar **sistemas de cache multi-nivel** con invalidación inteligente, query optimization, y monitoring de performance que soporten miles de usuarios concurrentes en **menos de 18 minutos**.

---

## 🏃‍♂️ **Arquitectura de Performance Deportiva**

### **Estrategias de Cache WorldSkills**

```text
🏆 CACHE LAYERS PARA SISTEMA DEPORTIVO

Application Cache:
├─ Tournament Standings (5 min TTL)
├─ Player Statistics (15 min TTL)
├─ Match Results (1 hour TTL)
└─ Team Rankings (30 min TTL)

Database Cache:
├─ Complex Queries (Query Cache)
├─ Relationship Counts (Eager Loading)
├─ Aggregations (Summary Tables)
└─ Full-Text Search (Indexed)

HTTP Cache:
├─ API Responses (ETag + Last-Modified)
├─ Static Resources (CDN)
├─ Conditional Requests (304 Not Modified)
└─ Compression (Gzip)

Redis Cache:
├─ Session Data (User preferences)
├─ Rate Limiting (API throttling)
├─ Real-time Data (Live scores)
└─ Background Jobs (Queue processing)
```

---

## ⏱️ **Ejercicio 1: Redis Cache Setup (20 minutos)**

### **Paso 1: Configurar Redis (5 minutos)**

```bash
# CRONÓMETRO: INICIAR ⏰

# Instalar Redis (si no está instalado)
sudo apt-get install redis-server -y

# Configurar Laravel para Redis
composer require predis/predis

# Verificar conexión Redis
redis-cli ping
```

```php
// config/cache.php - Verificar configuración
'default' => env('CACHE_DRIVER', 'redis'),

'stores' => [
    'redis' => [
        'driver' => 'redis',
        'connection' => 'cache',
        'lock_connection' => 'default',
    ],

    'tournaments' => [
        'driver' => 'redis',
        'connection' => 'cache',
        'prefix' => 'tournament_cache',
    ],

    'players' => [
        'driver' => 'redis',
        'connection' => 'cache',
        'prefix' => 'player_cache',
    ],
],
```

```php
// config/database.php - Configurar conexiones Redis
'redis' => [
    'client' => env('REDIS_CLIENT', 'predis'),

    'default' => [
        'url' => env('REDIS_URL'),
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'password' => env('REDIS_PASSWORD'),
        'port' => env('REDIS_PORT', '6379'),
        'database' => env('REDIS_DB', '0'),
    ],

    'cache' => [
        'url' => env('REDIS_URL'),
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'password' => env('REDIS_PASSWORD'),
        'port' => env('REDIS_PORT', '6379'),
        'database' => env('REDIS_CACHE_DB', '1'),
    ],

    'sessions' => [
        'url' => env('REDIS_URL'),
        'host' => env('REDIS_HOST', '127.0.0.1'),
        'password' => env('REDIS_PASSWORD'),
        'port' => env('REDIS_PORT', '6379'),
        'database' => env('REDIS_SESSION_DB', '2'),
    ],
],
```

### **Paso 2: Cache Service (10 minutos)**

```php
// app/Services/CacheService.php
<?php

namespace App\Services;

use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\Redis;
use Carbon\Carbon;

class CacheService
{
    protected $defaultTtl = 900; // 15 minutos

    protected $ttlConfig = [
        'tournament_standings' => 300,    // 5 minutos
        'player_statistics' => 900,      // 15 minutos
        'match_results' => 3600,         // 1 hora
        'team_rankings' => 1800,         // 30 minutos
        'live_scores' => 60,             // 1 minuto
        'user_preferences' => 86400,     // 24 horas
    ];

    public function remember($key, $callback, $ttl = null, $store = 'default')
    {
        $ttl = $ttl ?? $this->getTtl($key);
        $store = $store === 'default' ? config('cache.default') : $store;

        return Cache::store($store)->remember($key, $ttl, $callback);
    }

    public function rememberForever($key, $callback, $store = 'default')
    {
        $store = $store === 'default' ? config('cache.default') : $store;

        return Cache::store($store)->rememberForever($key, $callback);
    }

    public function invalidate($pattern, $store = 'default')
    {
        if ($store === 'redis' || config('cache.default') === 'redis') {
            return $this->invalidateRedisPattern($pattern, $store);
        }

        // Para otros drivers, usar tags si están disponibles
        return Cache::store($store)->flush();
    }

    protected function invalidateRedisPattern($pattern, $store = 'default')
    {
        $prefix = config("cache.stores.{$store}.prefix", config('cache.prefix'));
        $fullPattern = $prefix ? "{$prefix}:{$pattern}" : $pattern;

        $keys = Redis::keys($fullPattern);

        if (!empty($keys)) {
            Redis::del($keys);
            return count($keys);
        }

        return 0;
    }

    public function getTtl($key)
    {
        foreach ($this->ttlConfig as $pattern => $ttl) {
            if (str_contains($key, $pattern)) {
                return $ttl;
            }
        }

        return $this->defaultTtl;
    }

    public function cachePlayerStatistics($playerId)
    {
        $key = "player_stats:{$playerId}";

        return $this->remember($key, function () use ($playerId) {
            return \App\Models\Player::with([
                'statistics' => function ($query) {
                    $query->selectRaw('
                        player_id,
                        SUM(goals) as total_goals,
                        SUM(assists) as total_assists,
                        SUM(yellow_cards) as total_yellow_cards,
                        SUM(red_cards) as total_red_cards,
                        AVG(rating) as avg_rating,
                        COUNT(*) as matches_played
                    ')->groupBy('player_id');
                }
            ])->find($playerId);
        }, $this->ttlConfig['player_statistics']);
    }

    public function cacheTournamentStandings($tournamentId)
    {
        $key = "tournament_standings:{$tournamentId}";

        return $this->remember($key, function () use ($tournamentId) {
            return \App\Models\Team::where('tournament_id', $tournamentId)
                ->withCount([
                    'homeMatches as matches_played' => function ($query) {
                        $query->whereNotNull('result');
                    },
                    'awayMatches as away_matches' => function ($query) {
                        $query->whereNotNull('result');
                    }
                ])
                ->selectRaw('
                    teams.*,
                    COALESCE(stats.points, 0) as points,
                    COALESCE(stats.wins, 0) as wins,
                    COALESCE(stats.draws, 0) as draws,
                    COALESCE(stats.losses, 0) as losses,
                    COALESCE(stats.goals_for, 0) as goals_for,
                    COALESCE(stats.goals_against, 0) as goals_against,
                    (COALESCE(stats.goals_for, 0) - COALESCE(stats.goals_against, 0)) as goal_difference
                ')
                ->leftJoin('team_statistics as stats', 'teams.id', '=', 'stats.team_id')
                ->orderBy('points', 'desc')
                ->orderBy('goal_difference', 'desc')
                ->orderBy('goals_for', 'desc')
                ->get();
        }, $this->ttlConfig['tournament_standings']);
    }

    public function cacheMatchResults($matchId)
    {
        $key = "match_result:{$matchId}";

        return $this->remember($key, function () use ($matchId) {
            return \App\Models\Match::with([
                'homeTeam:id,name,logo',
                'awayTeam:id,name,logo',
                'referee:id,name',
                'events' => function ($query) {
                    $query->orderBy('minute')->with('player:id,name');
                }
            ])->find($matchId);
        }, $this->ttlConfig['match_results']);
    }

    public function cacheLiveScores()
    {
        $key = "live_scores";

        return $this->remember($key, function () {
            return \App\Models\Match::with(['homeTeam:id,name', 'awayTeam:id,name'])
                ->where('status', 'in_progress')
                ->orWhere('status', 'half_time')
                ->select('id', 'home_team_id', 'away_team_id', 'result', 'minute', 'status', 'match_date')
                ->get();
        }, $this->ttlConfig['live_scores']);
    }

    public function invalidatePlayerCache($playerId)
    {
        $patterns = [
            "player_stats:{$playerId}",
            "player_details:{$playerId}",
            "team_players:*", // Si el jugador cambió de equipo
        ];

        foreach ($patterns as $pattern) {
            $this->invalidate($pattern);
        }
    }

    public function invalidateTournamentCache($tournamentId)
    {
        $patterns = [
            "tournament_standings:{$tournamentId}",
            "tournament_matches:{$tournamentId}",
            "tournament_stats:{$tournamentId}",
        ];

        foreach ($patterns as $pattern) {
            $this->invalidate($pattern);
        }
    }

    public function warmupCache($tournamentId)
    {
        // Pre-cargar cache con datos frecuentemente accedidos
        $this->cacheTournamentStandings($tournamentId);
        $this->cacheLiveScores();

        // Estadísticas de equipos
        $teams = \App\Models\Team::where('tournament_id', $tournamentId)->pluck('id');
        foreach ($teams as $teamId) {
            $this->remember("team_stats:{$teamId}", function () use ($teamId) {
                return \App\Models\Team::with(['players', 'matches'])->find($teamId);
            });
        }
    }
}
```

### **Paso 3: Cache Middleware (5 minutos)**

```php
// app/Http/Middleware/CacheResponse.php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Cache;

class CacheResponse
{
    public function handle(Request $request, Closure $next, $ttl = 300)
    {
        // Solo cachear GET requests
        if ($request->method() !== 'GET') {
            return $next($request);
        }

        // Generar clave de cache basada en URL y parámetros
        $cacheKey = $this->generateCacheKey($request);

        // Verificar si ya existe en cache
        if (Cache::has($cacheKey)) {
            $cachedResponse = Cache::get($cacheKey);

            return response($cachedResponse['content'], $cachedResponse['status'])
                ->withHeaders($cachedResponse['headers'])
                ->header('X-Cache', 'HIT')
                ->header('X-Cache-Key', $cacheKey);
        }

        // Ejecutar request
        $response = $next($request);

        // Solo cachear respuestas exitosas
        if ($response->status() === 200) {
            $cacheData = [
                'content' => $response->getContent(),
                'status' => $response->status(),
                'headers' => $response->headers->all(),
            ];

            Cache::put($cacheKey, $cacheData, (int) $ttl);

            $response->header('X-Cache', 'MISS')
                    ->header('X-Cache-Key', $cacheKey)
                    ->header('X-Cache-TTL', $ttl);
        }

        return $response;
    }

    protected function generateCacheKey($request)
    {
        $url = $request->url();
        $params = $request->query();

        // Normalizar parámetros
        ksort($params);

        // Incluir user ID si está autenticado (para cache por usuario)
        $userId = $request->user()?->id ?? 'guest';

        return 'response_cache:' . hash('sha256', $url . serialize($params) . $userId);
    }
}
```

---

## ⏱️ **Ejercicio 2: Query Optimization (15 minutos)**

### **Paso 1: Eager Loading Service (8 minutos)**

```php
// app/Services/QueryOptimizationService.php
<?php

namespace App\Services;

use Illuminate\Database\Eloquent\Builder;
use Illuminate\Support\Facades\DB;

class QueryOptimizationService
{
    protected $relationMap = [
        'players' => [
            'team' => ['team:id,name,tournament_id'],
            'statistics' => ['statistics'],
            'tournament' => ['team.tournament:id,name,category'],
            'full' => ['team.tournament', 'statistics', 'injuries']
        ],
        'matches' => [
            'teams' => ['homeTeam:id,name', 'awayTeam:id,name'],
            'tournament' => ['homeTeam.tournament:id,name'],
            'full' => ['homeTeam.tournament', 'awayTeam.tournament', 'referee', 'events.player']
        ],
        'tournaments' => [
            'teams' => ['teams:id,name,tournament_id'],
            'matches' => ['matches:id,home_team_id,away_team_id,match_date,status'],
            'full' => ['teams.players', 'matches.homeTeam', 'matches.awayTeam']
        ]
    ];

    public function optimizePlayerQuery(Builder $query, array $includes = [])
    {
        // Relaciones por defecto para evitar N+1
        $defaultIncludes = ['team:id,name,tournament_id'];

        // Agregar includes solicitados
        $allIncludes = array_merge($defaultIncludes, $this->resolveIncludes('players', $includes));

        return $query->with($allIncludes);
    }

    public function optimizeMatchQuery(Builder $query, array $includes = [])
    {
        $baseIncludes = [
            'homeTeam:id,name,logo',
            'awayTeam:id,name,logo'
        ];

        $allIncludes = array_merge($baseIncludes, $this->resolveIncludes('matches', $includes));

        return $query->with($allIncludes);
    }

    public function addSubquerySelects(Builder $query, $entity)
    {
        switch ($entity) {
            case 'teams':
                return $query->addSelect([
                    'players_count' => \App\Models\Player::selectRaw('count(*)')
                        ->whereColumn('team_id', 'teams.id'),
                    'matches_played' => \App\Models\Match::selectRaw('count(*)')
                        ->where(function ($q) {
                            $q->whereColumn('home_team_id', 'teams.id')
                              ->orWhereColumn('away_team_id', 'teams.id');
                        })
                        ->whereNotNull('result'),
                    'total_goals' => \App\Models\Match::selectRaw('
                        COALESCE(SUM(
                            CASE
                                WHEN home_team_id = teams.id THEN JSON_EXTRACT(result, "$.home")
                                WHEN away_team_id = teams.id THEN JSON_EXTRACT(result, "$.away")
                                ELSE 0
                            END
                        ), 0)
                    ')->where(function ($q) {
                        $q->whereColumn('home_team_id', 'teams.id')
                          ->orWhereColumn('away_team_id', 'teams.id');
                    })->whereNotNull('result')
                ]);

            case 'players':
                return $query->addSelect([
                    'goals_total' => \App\Models\PlayerStatistic::selectRaw('COALESCE(SUM(goals), 0)')
                        ->whereColumn('player_id', 'players.id'),
                    'assists_total' => \App\Models\PlayerStatistic::selectRaw('COALESCE(SUM(assists), 0)')
                        ->whereColumn('player_id', 'players.id'),
                    'matches_played' => \App\Models\PlayerStatistic::selectRaw('COUNT(*)')
                        ->whereColumn('player_id', 'players.id'),
                    'avg_rating' => \App\Models\PlayerStatistic::selectRaw('ROUND(AVG(rating), 2)')
                        ->whereColumn('player_id', 'players.id')
                ]);

            default:
                return $query;
        }
    }

    public function resolveIncludes($entity, array $requested)
    {
        $resolved = [];
        $map = $this->relationMap[$entity] ?? [];

        foreach ($requested as $include) {
            if (isset($map[$include])) {
                $resolved = array_merge($resolved, $map[$include]);
            } elseif (in_array($include, array_keys($map))) {
                $resolved[] = $include;
            }
        }

        return array_unique($resolved);
    }

    public function buildTournamentStandingsQuery($tournamentId)
    {
        return DB::table('teams')
            ->where('tournament_id', $tournamentId)
            ->leftJoin(DB::raw('(
                SELECT
                    team_id,
                    SUM(points) as points,
                    SUM(wins) as wins,
                    SUM(draws) as draws,
                    SUM(losses) as losses,
                    SUM(goals_for) as goals_for,
                    SUM(goals_against) as goals_against
                FROM (
                    -- Home matches
                    SELECT
                        home_team_id as team_id,
                        CASE
                            WHEN JSON_EXTRACT(result, "$.home") > JSON_EXTRACT(result, "$.away") THEN 3
                            WHEN JSON_EXTRACT(result, "$.home") = JSON_EXTRACT(result, "$.away") THEN 1
                            ELSE 0
                        END as points,
                        CASE WHEN JSON_EXTRACT(result, "$.home") > JSON_EXTRACT(result, "$.away") THEN 1 ELSE 0 END as wins,
                        CASE WHEN JSON_EXTRACT(result, "$.home") = JSON_EXTRACT(result, "$.away") THEN 1 ELSE 0 END as draws,
                        CASE WHEN JSON_EXTRACT(result, "$.home") < JSON_EXTRACT(result, "$.away") THEN 1 ELSE 0 END as losses,
                        JSON_EXTRACT(result, "$.home") as goals_for,
                        JSON_EXTRACT(result, "$.away") as goals_against
                    FROM matches
                    WHERE result IS NOT NULL

                    UNION ALL

                    -- Away matches
                    SELECT
                        away_team_id as team_id,
                        CASE
                            WHEN JSON_EXTRACT(result, "$.away") > JSON_EXTRACT(result, "$.home") THEN 3
                            WHEN JSON_EXTRACT(result, "$.away") = JSON_EXTRACT(result, "$.home") THEN 1
                            ELSE 0
                        END as points,
                        CASE WHEN JSON_EXTRACT(result, "$.away") > JSON_EXTRACT(result, "$.home") THEN 1 ELSE 0 END as wins,
                        CASE WHEN JSON_EXTRACT(result, "$.away") = JSON_EXTRACT(result, "$.home") THEN 1 ELSE 0 END as draws,
                        CASE WHEN JSON_EXTRACT(result, "$.away") < JSON_EXTRACT(result, "$.home") THEN 1 ELSE 0 END as losses,
                        JSON_EXTRACT(result, "$.away") as goals_for,
                        JSON_EXTRACT(result, "$.home") as goals_against
                    FROM matches
                    WHERE result IS NOT NULL
                ) as match_stats
                GROUP BY team_id
            ) as team_stats'), 'teams.id', '=', 'team_stats.team_id')
            ->select([
                'teams.id',
                'teams.name',
                'teams.logo',
                DB::raw('COALESCE(team_stats.points, 0) as points'),
                DB::raw('COALESCE(team_stats.wins, 0) as wins'),
                DB::raw('COALESCE(team_stats.draws, 0) as draws'),
                DB::raw('COALESCE(team_stats.losses, 0) as losses'),
                DB::raw('COALESCE(team_stats.goals_for, 0) as goals_for'),
                DB::raw('COALESCE(team_stats.goals_against, 0) as goals_against'),
                DB::raw('(COALESCE(team_stats.goals_for, 0) - COALESCE(team_stats.goals_against, 0)) as goal_difference'),
                DB::raw('(COALESCE(team_stats.wins, 0) + COALESCE(team_stats.draws, 0) + COALESCE(team_stats.losses, 0)) as matches_played')
            ])
            ->orderBy('points', 'desc')
            ->orderBy('goal_difference', 'desc')
            ->orderBy('goals_for', 'desc');
    }

    public function explainQuery(Builder $query)
    {
        $sql = $query->toSql();
        $bindings = $query->getBindings();

        // Reemplazar ? con valores reales para EXPLAIN
        foreach ($bindings as $binding) {
            $sql = preg_replace('/\?/', "'" . $binding . "'", $sql, 1);
        }

        return DB::select("EXPLAIN " . $sql);
    }
}
```

### **Paso 4: Database Indexing (7 minutos)**

```php
// database/migrations/xxxx_add_performance_indexes.php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

class AddPerformanceIndexes extends Migration
{
    public function up()
    {
        // Índices para Players
        Schema::table('players', function (Blueprint $table) {
            $table->index(['team_id', 'position']); // Filtros comunes
            $table->index(['birth_date']); // Para cálculos de edad
            $table->index(['jersey_number', 'team_id']); // Unicidad
            $table->index(['created_at']); // Ordenamiento temporal
        });

        // Índices para Matches
        Schema::table('matches', function (Blueprint $table) {
            $table->index(['match_date', 'status']); // Filtros de fecha y estado
            $table->index(['home_team_id', 'away_team_id']); // Búsquedas por equipo
            $table->index(['tournament_id', 'round']); // Filtros de torneo
            $table->index(['referee_id']); // Filtros por árbitro
            $table->index(['venue']); // Búsquedas por venue
        });

        // Índices para Teams
        Schema::table('teams', function (Blueprint $table) {
            $table->index(['tournament_id', 'category']); // Filtros de torneo
            $table->index(['name']); // Búsquedas por nombre
        });

        // Índices para Player Statistics
        Schema::table('player_statistics', function (Blueprint $table) {
            $table->index(['player_id', 'match_id']); // Relación principal
            $table->index(['goals', 'assists']); // Ordenamiento por stats
            $table->index(['rating']); // Filtros por rating
        });

        // Índices para Full-Text Search
        if (DB::getDriverName() === 'mysql') {
            DB::statement('ALTER TABLE players ADD FULLTEXT(name)');
            DB::statement('ALTER TABLE teams ADD FULLTEXT(name)');
            DB::statement('ALTER TABLE tournaments ADD FULLTEXT(name, description)');
        }

        // Índices JSON para resultados (MySQL 5.7+)
        if (DB::getDriverName() === 'mysql') {
            DB::statement('ALTER TABLE matches ADD INDEX idx_home_goals ((JSON_EXTRACT(result, "$.home")))');
            DB::statement('ALTER TABLE matches ADD INDEX idx_away_goals ((JSON_EXTRACT(result, "$.away")))');
        }
    }

    public function down()
    {
        Schema::table('players', function (Blueprint $table) {
            $table->dropIndex(['team_id', 'position']);
            $table->dropIndex(['birth_date']);
            $table->dropIndex(['jersey_number', 'team_id']);
            $table->dropIndex(['created_at']);
        });

        Schema::table('matches', function (Blueprint $table) {
            $table->dropIndex(['match_date', 'status']);
            $table->dropIndex(['home_team_id', 'away_team_id']);
            $table->dropIndex(['tournament_id', 'round']);
            $table->dropIndex(['referee_id']);
            $table->dropIndex(['venue']);
        });

        Schema::table('teams', function (Blueprint $table) {
            $table->dropIndex(['tournament_id', 'category']);
            $table->dropIndex(['name']);
        });

        Schema::table('player_statistics', function (Blueprint $table) {
            $table->dropIndex(['player_id', 'match_id']);
            $table->dropIndex(['goals', 'assists']);
            $table->dropIndex(['rating']);
        });

        if (DB::getDriverName() === 'mysql') {
            DB::statement('ALTER TABLE players DROP INDEX name');
            DB::statement('ALTER TABLE teams DROP INDEX name');
            DB::statement('ALTER TABLE tournaments DROP INDEX name');
            DB::statement('ALTER TABLE matches DROP INDEX idx_home_goals');
            DB::statement('ALTER TABLE matches DROP INDEX idx_away_goals');
        }
    }
}
```

---

## ⏱️ **Ejercicio 3: HTTP Caching (10 minutos)**

### **Paso 1: ETag y Last-Modified (6 minutos)**

```php
// app/Http/Controllers/Api/CachedController.php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Cache;

class CachedController extends Controller
{
    protected function respondWithETag($data, $lastModified = null)
    {
        $etag = '"' . hash('sha256', json_encode($data)) . '"';
        $lastModified = $lastModified ?? now();

        // Verificar If-None-Match header
        if (request()->header('If-None-Match') === $etag) {
            return response('', 304);
        }

        // Verificar If-Modified-Since header
        $ifModifiedSince = request()->header('If-Modified-Since');
        if ($ifModifiedSince && strtotime($ifModifiedSince) >= $lastModified->timestamp) {
            return response('', 304);
        }

        return response()->json($data)
            ->header('ETag', $etag)
            ->header('Last-Modified', $lastModified->format('D, d M Y H:i:s \G\M\T'))
            ->header('Cache-Control', 'public, max-age=300'); // 5 minutos
    }

    protected function respondWithConditionalCache($data, $key, $ttl = 300)
    {
        $cacheData = Cache::get($key);

        if ($cacheData) {
            // Verificar si el cliente tiene la versión cached
            $clientETag = request()->header('If-None-Match');

            if ($clientETag && $clientETag === $cacheData['etag']) {
                return response('', 304)
                    ->header('Cache-Control', 'public, max-age=' . $ttl);
            }

            return response()->json($cacheData['data'])
                ->header('ETag', $cacheData['etag'])
                ->header('Last-Modified', $cacheData['last_modified'])
                ->header('Cache-Control', 'public, max-age=' . $ttl)
                ->header('X-Cache', 'HIT');
        }

        // Generar nueva respuesta
        $etag = '"' . hash('sha256', json_encode($data)) . '"';
        $lastModified = now()->format('D, d M Y H:i:s \G\M\T');

        // Guardar en cache
        Cache::put($key, [
            'data' => $data,
            'etag' => $etag,
            'last_modified' => $lastModified
        ], $ttl);

        return response()->json($data)
            ->header('ETag', $etag)
            ->header('Last-Modified', $lastModified)
            ->header('Cache-Control', 'public, max-age=' . $ttl)
            ->header('X-Cache', 'MISS');
    }
}

// Ejemplo en PlayerController
class PlayerController extends CachedController
{
    public function show($id)
    {
        $cacheKey = "player_details:{$id}";

        $player = Cache::remember($cacheKey, 900, function () use ($id) {
            return Player::with(['team', 'statistics'])->findOrFail($id);
        });

        return $this->respondWithConditionalCache(
            new PlayerResource($player),
            "player_response:{$id}",
            900
        );
    }

    public function standings($tournamentId)
    {
        $cacheKey = "standings:{$tournamentId}";

        // Obtener última modificación basada en últimos partidos
        $lastMatchUpdate = Match::whereHas('homeTeam', function ($q) use ($tournamentId) {
            $q->where('tournament_id', $tournamentId);
        })->whereNotNull('result')
          ->latest('updated_at')
          ->value('updated_at');

        $standings = Cache::remember($cacheKey, 300, function () use ($tournamentId) {
            return app(QueryOptimizationService::class)
                ->buildTournamentStandingsQuery($tournamentId)
                ->get();
        });

        return $this->respondWithETag($standings, $lastMatchUpdate);
    }
}
```

### **Paso 2: Response Compression (4 minutos)**

```php
// app/Http/Middleware/CompressResponse.php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class CompressResponse
{
    public function handle(Request $request, Closure $next)
    {
        $response = $next($request);

        // Solo comprimir respuestas de texto/JSON grandes
        $contentType = $response->headers->get('Content-Type', '');
        $contentLength = strlen($response->getContent());

        if ($this->shouldCompress($request, $contentType, $contentLength)) {
            $compressed = gzencode($response->getContent(), 6);

            if ($compressed !== false && strlen($compressed) < $contentLength) {
                $response->setContent($compressed);
                $response->headers->set('Content-Encoding', 'gzip');
                $response->headers->set('Content-Length', strlen($compressed));
            }
        }

        return $response;
    }

    protected function shouldCompress($request, $contentType, $contentLength)
    {
        // Verificar que el client soporte gzip
        $acceptEncoding = $request->header('Accept-Encoding', '');
        if (!str_contains($acceptEncoding, 'gzip')) {
            return false;
        }

        // Solo comprimir contenido de texto y JSON > 1KB
        $compressibleTypes = [
            'application/json',
            'text/html',
            'text/plain',
            'text/css',
            'application/javascript'
        ];

        $isCompressible = false;
        foreach ($compressibleTypes as $type) {
            if (str_contains($contentType, $type)) {
                $isCompressible = true;
                break;
            }
        }

        return $isCompressible && $contentLength > 1024; // > 1KB
    }
}

// Registrar en Kernel.php
protected $middleware = [
    // ... otros middlewares
    \App\Http\Middleware\CompressResponse::class,
];
```

---

## ⏱️ **Ejercicio 4: Performance Monitoring (10 minutos)**

### **Paso 1: Performance Middleware (6 minutos)**

```php
// app/Http/Middleware/PerformanceMonitor.php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Log;
use Illuminate\Support\Facades\DB;

class PerformanceMonitor
{
    protected $thresholds = [
        'response_time' => 1000, // 1 segundo
        'memory_usage' => 50,    // 50MB
        'query_count' => 20,     // 20 queries
    ];

    public function handle(Request $request, Closure $next)
    {
        $startTime = microtime(true);
        $startMemory = memory_get_usage(true);

        // Contar queries
        DB::enableQueryLog();

        $response = $next($request);

        $endTime = microtime(true);
        $endMemory = memory_get_usage(true);
        $queries = DB::getQueryLog();

        $metrics = [
            'url' => $request->fullUrl(),
            'method' => $request->method(),
            'response_time' => round(($endTime - $startTime) * 1000, 2), // ms
            'memory_usage' => round(($endMemory - $startMemory) / 1024 / 1024, 2), // MB
            'peak_memory' => round(memory_get_peak_usage(true) / 1024 / 1024, 2), // MB
            'query_count' => count($queries),
            'status_code' => $response->status(),
            'user_id' => auth()->id(),
            'timestamp' => now()->toISOString(),
        ];

        // Agregar headers de debug
        $response->headers->set('X-Response-Time', $metrics['response_time'] . 'ms');
        $response->headers->set('X-Memory-Usage', $metrics['memory_usage'] . 'MB');
        $response->headers->set('X-Query-Count', $metrics['query_count']);

        // Log si excede umbrales
        $this->checkThresholds($metrics, $queries);

        // Almacenar métricas para análisis
        $this->storeMetrics($metrics);

        return $response;
    }

    protected function checkThresholds($metrics, $queries)
    {
        $warnings = [];

        if ($metrics['response_time'] > $this->thresholds['response_time']) {
            $warnings[] = "Slow response: {$metrics['response_time']}ms";
        }

        if ($metrics['memory_usage'] > $this->thresholds['memory_usage']) {
            $warnings[] = "High memory usage: {$metrics['memory_usage']}MB";
        }

        if ($metrics['query_count'] > $this->thresholds['query_count']) {
            $warnings[] = "Too many queries: {$metrics['query_count']}";

            // Log las queries lentas
            $slowQueries = array_filter($queries, function ($query) {
                return $query['time'] > 100; // > 100ms
            });

            if (!empty($slowQueries)) {
                Log::warning('Slow queries detected', [
                    'url' => $metrics['url'],
                    'queries' => $slowQueries
                ]);
            }
        }

        if (!empty($warnings)) {
            Log::warning('Performance threshold exceeded', [
                'warnings' => $warnings,
                'metrics' => $metrics
            ]);
        }
    }

    protected function storeMetrics($metrics)
    {
        // Almacenar en Redis para análisis en tiempo real
        $key = 'performance_metrics:' . date('Y-m-d:H');

        \Illuminate\Support\Facades\Redis::lpush($key, json_encode($metrics));
        \Illuminate\Support\Facades\Redis::expire($key, 86400); // 24 horas

        // Actualizar contadores globales
        \Illuminate\Support\Facades\Redis::hincrby('performance_stats', 'total_requests', 1);
        \Illuminate\Support\Facades\Redis::hincrby('performance_stats', 'total_response_time', $metrics['response_time']);
    }
}
```

### **Paso 2: Performance Dashboard (4 minutos)**

```php
// app/Http/Controllers/Api/PerformanceController.php
<?php

namespace App\Http\Controllers\Api;

use App\Http\Controllers\Controller;
use Illuminate\Support\Facades\Redis;
use Illuminate\Http\Request;

class PerformanceController extends Controller
{
    public function metrics(Request $request)
    {
        $period = $request->get('period', 'hour'); // hour, day, week

        $metrics = $this->getMetricsForPeriod($period);

        return response()->json([
            'period' => $period,
            'summary' => $this->calculateSummary($metrics),
            'trends' => $this->calculateTrends($metrics),
            'slow_endpoints' => $this->getSlowEndpoints($metrics),
            'recommendations' => $this->getRecommendations($metrics),
        ]);
    }

    protected function getMetricsForPeriod($period)
    {
        $keys = [];
        $now = now();

        switch ($period) {
            case 'hour':
                for ($i = 0; $i < 60; $i++) {
                    $keys[] = 'performance_metrics:' . $now->copy()->subMinutes($i)->format('Y-m-d:H:i');
                }
                break;
            case 'day':
                for ($i = 0; $i < 24; $i++) {
                    $keys[] = 'performance_metrics:' . $now->copy()->subHours($i)->format('Y-m-d:H');
                }
                break;
            case 'week':
                for ($i = 0; $i < 7; $i++) {
                    $keys[] = 'performance_metrics:' . $now->copy()->subDays($i)->format('Y-m-d');
                }
                break;
        }

        $allMetrics = [];
        foreach ($keys as $key) {
            $data = Redis::lrange($key, 0, -1);
            foreach ($data as $item) {
                $allMetrics[] = json_decode($item, true);
            }
        }

        return $allMetrics;
    }

    protected function calculateSummary($metrics)
    {
        if (empty($metrics)) {
            return null;
        }

        $responseTimes = array_column($metrics, 'response_time');
        $memoryUsages = array_column($metrics, 'memory_usage');
        $queryCounts = array_column($metrics, 'query_count');

        return [
            'total_requests' => count($metrics),
            'avg_response_time' => round(array_sum($responseTimes) / count($responseTimes), 2),
            'max_response_time' => max($responseTimes),
            'avg_memory_usage' => round(array_sum($memoryUsages) / count($memoryUsages), 2),
            'max_memory_usage' => max($memoryUsages),
            'avg_query_count' => round(array_sum($queryCounts) / count($queryCounts), 2),
            'max_query_count' => max($queryCounts),
            'error_rate' => $this->calculateErrorRate($metrics),
        ];
    }

    protected function getSlowEndpoints($metrics)
    {
        $endpoints = [];

        foreach ($metrics as $metric) {
            $url = parse_url($metric['url'], PHP_URL_PATH);

            if (!isset($endpoints[$url])) {
                $endpoints[$url] = [
                    'url' => $url,
                    'count' => 0,
                    'total_time' => 0,
                    'max_time' => 0,
                ];
            }

            $endpoints[$url]['count']++;
            $endpoints[$url]['total_time'] += $metric['response_time'];
            $endpoints[$url]['max_time'] = max($endpoints[$url]['max_time'], $metric['response_time']);
        }

        // Calcular promedio y ordenar
        foreach ($endpoints as &$endpoint) {
            $endpoint['avg_time'] = round($endpoint['total_time'] / $endpoint['count'], 2);
        }

        // Ordenar por tiempo promedio
        uasort($endpoints, function ($a, $b) {
            return $b['avg_time'] <=> $a['avg_time'];
        });

        return array_slice(array_values($endpoints), 0, 10);
    }

    protected function getRecommendations($metrics)
    {
        $recommendations = [];
        $summary = $this->calculateSummary($metrics);

        if ($summary['avg_response_time'] > 500) {
            $recommendations[] = [
                'type' => 'performance',
                'priority' => 'high',
                'message' => 'Average response time is high (' . $summary['avg_response_time'] . 'ms). Consider caching or query optimization.',
            ];
        }

        if ($summary['avg_query_count'] > 15) {
            $recommendations[] = [
                'type' => 'queries',
                'priority' => 'medium',
                'message' => 'High number of database queries (' . $summary['avg_query_count'] . '). Implement eager loading.',
            ];
        }

        if ($summary['avg_memory_usage'] > 30) {
            $recommendations[] = [
                'type' => 'memory',
                'priority' => 'medium',
                'message' => 'High memory usage (' . $summary['avg_memory_usage'] . 'MB). Review data loading patterns.',
            ];
        }

        return $recommendations;
    }

    protected function calculateErrorRate($metrics)
    {
        $total = count($metrics);
        $errors = count(array_filter($metrics, function ($m) {
            return $m['status_code'] >= 400;
        }));

        return $total > 0 ? round(($errors / $total) * 100, 2) : 0;
    }
}
```

---

## 🎯 **Comandos de Testing**

```bash
# Test de cache
php artisan tinker
>>> Cache::put('test', 'value', 60);
>>> Cache::get('test');

# Verificar Redis
redis-cli
> KEYS *
> GET laravel_cache:test

# Test de performance
curl -H "Accept: application/json" -w "@curl-format.txt" http://localhost:8000/api/players

# Archivo curl-format.txt:
#      time_namelookup:  %{time_namelookup}\n
#         time_connect:  %{time_connect}\n
#      time_appconnect:  %{time_appconnect}\n
#     time_pretransfer:  %{time_pretransfer}\n
#        time_redirect:  %{time_redirect}\n
#    time_starttransfer:  %{time_starttransfer}\n
#                      ----------\n
#          time_total:  %{time_total}\n
```

---

## 🎯 **Rúbrica de Evaluación**

| Criterio                   | Excelente (4)                      | Bueno (3)                 | Satisfactorio (2) | Insuficiente (1) |
| -------------------------- | ---------------------------------- | ------------------------- | ----------------- | ---------------- |
| **Redis Cache**            | Cache multicapa + invalidación     | Cache básico implementado | Solo cache simple | Sin cache        |
| **Query Optimization**     | Eager loading + subqueries         | Algunas optimizaciones    | Queries básicas   | N+1 problems     |
| **HTTP Caching**           | ETag + Last-Modified + Compression | Algún HTTP caching        | Headers básicos   | Sin HTTP cache   |
| **Performance Monitoring** | Métricas completas + alertas       | Monitoring básico         | Algunas métricas  | Sin monitoring   |
| **Database Indexes**       | Índices optimizados                | Algunos índices           | Índices básicos   | Sin índices      |
| **Tiempo**                 | < 18 minutos                       | 18-25 min                 | 25-35 min         | > 35 min         |

---

## 🔧 **Tips de Debugging**

```bash
# Ver queries ejecutadas
DB::enableQueryLog();
// ... tu código ...
dd(DB::getQueryLog());

# Cache debugging
Cache::put('debug', ['test' => 'data'], 60);
dd(Cache::get('debug'));

# Redis monitoring
redis-cli MONITOR

# Performance profiling
php artisan route:cache
php artisan config:cache
php artisan view:cache
```

---

**⚡ Con estas optimizaciones, tu API manejará miles de usuarios simultáneos con respuestas sub-segundo!**
