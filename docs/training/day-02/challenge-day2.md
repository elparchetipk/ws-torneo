# 🏆 Desafío Final - Día 2

**Duración:** 2 horas cronometradas  
**Objetivo:** Sistema completo con relaciones, autenticación y testing

---

## 🎯 **Descripción del Desafío**

Crear un sistema completo de gestión de **Torneo de Tenis Femenino** que incluya:

- **Relaciones complejas** entre múltiples entidades
- **Autenticación segura** con roles y permisos
- **Testing automatizado** con cobertura > 80%
- **Frontend React** completamente integrado

### **Entidades del Sistema**

```
🎾 TORNEO DE TENIS FEMENINO - RELACIONES

Players (Jugadoras)
├─ id, name, country_id, ranking, birth_date
├─ belongsTo → Country
├─ hasMany → MatchParticipations
└─ morphMany → Statistics

Countries (Países)
├─ id, name, iso_code, flag_url
├─ hasMany → Players
└─ hasMany → Tournaments

Tournaments (Torneos)
├─ id, name, country_id, start_date, end_date, prize_money
├─ belongsTo → Country (host)
├─ hasMany → Matches
└─ belongsToMany → Players (registrations)

Matches (Partidos)
├─ id, tournament_id, round, court, match_date, status
├─ belongsTo → Tournament
├─ hasMany → MatchParticipations (2 jugadoras por partido)
├─ hasOne → MatchResult
└─ morphMany → Statistics

MatchParticipations (Participaciones en Partidos)
├─ id, match_id, player_id, is_winner, sets_won
├─ belongsTo → Match
└─ belongsTo → Player

MatchResults (Resultados)
├─ id, match_id, player1_sets, player2_sets, duration_minutes
├─ belongsTo → Match
└─ hasMany → SetResults

Statistics (Estadísticas Polimórficas)
├─ id, statisticable_id, statisticable_type, stat_type, value
└─ morphTo → Statisticable (Player | Match)
```

---

## ⏱️ **Cronograma del Desafío**

### **Fase 1: Modelado y Relaciones (45 minutos)**

#### **Setup y Migraciones (15 minutos)**

```bash
# ⏰ CRONÓMETRO: INICIAR
composer create-project laravel/laravel tennis-tournament-api
cd tennis-tournament-api

# Configurar SQLite
touch database/database.sqlite
echo "DB_CONNECTION=sqlite" > .env.local
echo "DB_DATABASE=$(pwd)/database/database.sqlite" >> .env.local

# Frontend paralelo
cd ..
pnpm create vite tennis-tournament-frontend --template react
cd tennis-tournament-frontend
pnpm install
pnpm add axios react-router-dom @tanstack/react-query
```

```bash
# Crear modelos con migraciones
php artisan make:model Country -mcr
php artisan make:model Player -mcr
php artisan make:model Tournament -mcr
php artisan make:model Match -mcr
php artisan make:model MatchParticipation -mcr
php artisan make:model MatchResult -mcr
php artisan make:model Statistic -mcr
```

#### **Migraciones Complejas (20 minutos)**

```php
// database/migrations/xxxx_create_countries_table.php
public function up()
{
    Schema::create('countries', function (Blueprint $table) {
        $table->id();
        $table->string('name')->unique();
        $table->string('iso_code', 3)->unique();
        $table->string('flag_url')->nullable();
        $table->timestamps();
    });
}
```

```php
// database/migrations/xxxx_create_players_table.php
public function up()
{
    Schema::create('players', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->foreignId('country_id')->constrained()->onDelete('cascade');
        $table->integer('ranking')->unsigned()->nullable();
        $table->date('birth_date');
        $table->enum('hand', ['right', 'left'])->default('right');
        $table->decimal('prize_money', 12, 2)->default(0);
        $table->timestamps();

        $table->index(['ranking']);
        $table->index(['country_id', 'ranking']);
    });
}
```

```php
// database/migrations/xxxx_create_tournaments_table.php
public function up()
{
    Schema::create('tournaments', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->foreignId('country_id')->constrained()->onDelete('cascade');
        $table->date('start_date');
        $table->date('end_date');
        $table->decimal('prize_money', 12, 2);
        $table->enum('surface', ['clay', 'grass', 'hard', 'carpet']);
        $table->enum('status', ['upcoming', 'ongoing', 'completed'])->default('upcoming');
        $table->timestamps();

        $table->index(['start_date', 'status']);
        $table->check('end_date > start_date');
    });
}
```

```php
// database/migrations/xxxx_create_matches_table.php
public function up()
{
    Schema::create('matches', function (Blueprint $table) {
        $table->id();
        $table->foreignId('tournament_id')->constrained()->onDelete('cascade');
        $table->string('round'); // "Round 1", "Quarterfinal", "Semifinal", "Final"
        $table->string('court')->nullable();
        $table->dateTime('match_date');
        $table->enum('status', ['scheduled', 'live', 'finished', 'cancelled'])->default('scheduled');
        $table->timestamps();

        $table->index(['tournament_id', 'round']);
        $table->index(['match_date', 'status']);
    });
}
```

```php
// database/migrations/xxxx_create_match_participations_table.php
public function up()
{
    Schema::create('match_participations', function (Blueprint $table) {
        $table->id();
        $table->foreignId('match_id')->constrained()->onDelete('cascade');
        $table->foreignId('player_id')->constrained()->onDelete('cascade');
        $table->boolean('is_winner')->default(false);
        $table->integer('sets_won')->unsigned()->default(0);
        $table->timestamps();

        // Un partido solo puede tener 2 participaciones
        $table->unique(['match_id', 'player_id']);
        $table->index(['player_id']);
    });
}
```

```php
// database/migrations/xxxx_create_match_results_table.php
public function up()
{
    Schema::create('match_results', function (Blueprint $table) {
        $table->id();
        $table->foreignId('match_id')->constrained()->onDelete('cascade');
        $table->json('set_scores'); // [{"player1": 6, "player2": 4}, {"player1": 6, "player2": 2}]
        $table->integer('duration_minutes')->unsigned();
        $table->timestamps();

        $table->unique('match_id');
    });
}
```

```php
// database/migrations/xxxx_create_statistics_table.php
public function up()
{
    Schema::create('statistics', function (Blueprint $table) {
        $table->id();
        $table->morphs('statisticable');
        $table->enum('stat_type', [
            'aces', 'double_faults', 'first_serve_percentage',
            'winners', 'unforced_errors', 'break_points_saved'
        ]);
        $table->decimal('value', 8, 2);
        $table->timestamps();

        $table->unique(['statisticable_id', 'statisticable_type', 'stat_type']);
        $table->index(['statisticable_type', 'stat_type']);
    });
}
```

#### **Tabla Pivot para Registraciones (10 minutos)**

```bash
php artisan make:migration create_tournament_player_table
```

```php
// database/migrations/xxxx_create_tournament_player_table.php
public function up()
{
    Schema::create('tournament_player', function (Blueprint $table) {
        $table->id();
        $table->foreignId('tournament_id')->constrained()->onDelete('cascade');
        $table->foreignId('player_id')->constrained()->onDelete('cascade');
        $table->timestamp('registered_at')->useCurrent();
        $table->timestamps();

        $table->unique(['tournament_id', 'player_id']);
    });
}
```

### **Fase 2: Modelos con Relaciones (30 minutos)**

#### **Modelos Principales (25 minutos)**

```php
// app/Models/Country.php
class Country extends Model
{
    protected $fillable = ['name', 'iso_code', 'flag_url'];

    public function players()
    {
        return $this->hasMany(Player::class);
    }

    public function tournaments()
    {
        return $this->hasMany(Tournament::class);
    }

    public function scopeWithActivePlayers($query)
    {
        return $query->with(['players' => function($q) {
            $q->whereNotNull('ranking')->orderBy('ranking');
        }]);
    }

    public function getFlagAttribute($value)
    {
        return $value ?: "https://flagcdn.com/w320/{$this->iso_code}.png";
    }
}
```

```php
// app/Models/Player.php
class Player extends Model
{
    protected $fillable = [
        'name', 'country_id', 'ranking', 'birth_date', 'hand', 'prize_money'
    ];

    protected $casts = [
        'birth_date' => 'date',
        'prize_money' => 'decimal:2',
    ];

    public function country()
    {
        return $this->belongsTo(Country::class);
    }

    public function tournaments()
    {
        return $this->belongsToMany(Tournament::class)
                   ->withPivot('registered_at')
                   ->withTimestamps();
    }

    public function matchParticipations()
    {
        return $this->hasMany(MatchParticipation::class);
    }

    public function statistics()
    {
        return $this->morphMany(Statistic::class, 'statisticable');
    }

    // Scopes
    public function scopeTopRanked($query, $limit = 10)
    {
        return $query->whereNotNull('ranking')
                    ->orderBy('ranking')
                    ->limit($limit);
    }

    public function scopeByCountry($query, $countryId)
    {
        return $query->where('country_id', $countryId);
    }

    // Accessors
    public function getAgeAttribute()
    {
        return $this->birth_date->age;
    }

    public function getWinsAttribute()
    {
        return $this->matchParticipations()->where('is_winner', true)->count();
    }

    public function getLossesAttribute()
    {
        return $this->matchParticipations()->where('is_winner', false)->count();
    }
}
```

```php
// app/Models/Match.php
class Match extends Model
{
    protected $fillable = [
        'tournament_id', 'round', 'court', 'match_date', 'status'
    ];

    protected $casts = [
        'match_date' => 'datetime',
    ];

    public function tournament()
    {
        return $this->belongsTo(Tournament::class);
    }

    public function participations()
    {
        return $this->hasMany(MatchParticipation::class);
    }

    public function players()
    {
        return $this->belongsToMany(Player::class, 'match_participations')
                   ->withPivot(['is_winner', 'sets_won']);
    }

    public function result()
    {
        return $this->hasOne(MatchResult::class);
    }

    public function statistics()
    {
        return $this->morphMany(Statistic::class, 'statisticable');
    }

    // Scopes
    public function scopeFinished($query)
    {
        return $query->where('status', 'finished');
    }

    public function scopeToday($query)
    {
        return $query->whereDate('match_date', today());
    }

    public function scopeWithPlayers($query)
    {
        return $query->with(['players.country']);
    }

    // Métodos de negocio
    public function getWinnerAttribute()
    {
        return $this->participations()->where('is_winner', true)->first()?->player;
    }

    public function getLoserAttribute()
    {
        return $this->participations()->where('is_winner', false)->first()?->player;
    }
}
```

#### **Ejecutar Migraciones (5 minutos)**

```bash
php artisan migrate

# Verificar estructura
php artisan schema:dump
```

### **Fase 3: Autenticación con Sanctum (25 minutos)**

#### **Setup Sanctum (5 minutos)**

```bash
composer require laravel/sanctum
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
php artisan migrate
```

#### **AuthController Completo (15 minutos)**

```php
// app/Http/Controllers/AuthController.php
class AuthController extends Controller
{
    public function register(Request $request)
    {
        $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|string|email|max:255|unique:users',
            'password' => 'required|string|min:8|confirmed',
            'role' => 'sometimes|in:admin,organizer,viewer'
        ]);

        $user = User::create([
            'name' => $request->name,
            'email' => $request->email,
            'password' => bcrypt($request->password),
            'role' => $request->role ?? 'viewer',
        ]);

        $token = $user->createToken('auth-token')->plainTextToken;

        return response()->json([
            'user' => $user->only(['id', 'name', 'email', 'role']),
            'token' => $token,
        ], 201);
    }

    public function login(Request $request)
    {
        $request->validate([
            'email' => 'required|email',
            'password' => 'required',
        ]);

        if (!Auth::attempt($request->only('email', 'password'))) {
            return response()->json(['message' => 'Invalid credentials'], 401);
        }

        $user = User::where('email', $request->email)->first();

        // Revoke previous tokens
        $user->tokens()->delete();

        $token = $user->createToken('auth-token')->plainTextToken;

        return response()->json([
            'user' => $user->only(['id', 'name', 'email', 'role']),
            'token' => $token,
        ]);
    }

    public function logout(Request $request)
    {
        $request->user()->currentAccessToken()->delete();

        return response()->json(['message' => 'Logged out successfully']);
    }

    public function me(Request $request)
    {
        return response()->json([
            'user' => $request->user()->only(['id', 'name', 'email', 'role'])
        ]);
    }
}
```

#### **Rutas Protegidas (5 minutos)**

```php
// routes/api.php
Route::post('/auth/register', [AuthController::class, 'register']);
Route::post('/auth/login', [AuthController::class, 'login']);

Route::middleware('auth:sanctum')->group(function () {
    Route::post('/auth/logout', [AuthController::class, 'logout']);
    Route::get('/auth/me', [AuthController::class, 'me']);

    // Admin routes
    Route::middleware('role:admin')->group(function () {
        Route::apiResource('countries', CountryController::class);
        Route::apiResource('tournaments', TournamentController::class);
        Route::apiResource('players', PlayerController::class);
        Route::apiResource('matches', MatchController::class);
    });

    // Organizer routes
    Route::middleware('role:admin,organizer')->group(function () {
        Route::post('matches/{match}/result', 'MatchController@storeResult');
        Route::patch('matches/{match}/status', 'MatchController@updateStatus');
    });

    // Viewer routes (read-only)
    Route::get('tournaments', 'TournamentController@index');
    Route::get('players', 'PlayerController@index');
    Route::get('matches', 'MatchController@index');
    Route::get('rankings', 'PlayerController@rankings');
});

// Public routes
Route::get('/public/tournaments/current', 'TournamentController@current');
Route::get('/public/players/top-ranked', 'PlayerController@topRanked');
Route::get('/public/matches/today', 'MatchController@today');
```

### **Fase 4: Testing Automatizado (20 minutos)**

#### **Database Factories (10 minutos)**

```php
// database/factories/PlayerFactory.php
class PlayerFactory extends Factory
{
    public function definition()
    {
        $femaleNames = [
            'Serena Williams', 'Maria Sharapova', 'Steffi Graf', 'Monica Seles',
            'Venus Williams', 'Justine Henin', 'Kim Clijsters', 'Martina Hingis'
        ];

        return [
            'name' => $this->faker->randomElement($femaleNames),
            'country_id' => Country::factory(),
            'ranking' => $this->faker->numberBetween(1, 100),
            'birth_date' => $this->faker->dateTimeBetween('-35 years', '-18 years'),
            'hand' => $this->faker->randomElement(['right', 'left']),
            'prize_money' => $this->faker->randomFloat(2, 10000, 5000000),
        ];
    }

    public function topRanked()
    {
        return $this->state(['ranking' => $this->faker->numberBetween(1, 10)]);
    }
}
```

#### **Feature Tests Críticos (10 minutos)**

```php
// tests/Feature/TournamentApiTest.php
class TournamentApiTest extends TestCase
{
    use RefreshDatabase;

    public function test_can_create_tournament_with_players()
    {
        $admin = User::factory()->admin()->create();
        $this->actingAs($admin, 'sanctum');

        $country = Country::factory()->create();
        $players = Player::factory()->count(4)->create();

        $tournamentData = [
            'name' => 'ATP Masters',
            'country_id' => $country->id,
            'start_date' => now()->addDays(30)->format('Y-m-d'),
            'end_date' => now()->addDays(37)->format('Y-m-d'),
            'prize_money' => 1000000,
            'surface' => 'hard',
            'player_ids' => $players->pluck('id')->toArray()
        ];

        $response = $this->postJson('/api/tournaments', $tournamentData);

        $response->assertStatus(201);
        $this->assertDatabaseHas('tournaments', [
            'name' => 'ATP Masters',
            'country_id' => $country->id
        ]);

        // Verify players are registered
        $tournament = Tournament::first();
        $this->assertEquals(4, $tournament->players()->count());
    }

    public function test_can_create_match_with_participations()
    {
        $admin = User::factory()->admin()->create();
        $this->actingAs($admin, 'sanctum');

        $tournament = Tournament::factory()->create();
        $players = Player::factory()->count(2)->create();

        $matchData = [
            'tournament_id' => $tournament->id,
            'round' => 'Round 1',
            'court' => 'Center Court',
            'match_date' => now()->addDays(5)->toISOString(),
            'player_ids' => $players->pluck('id')->toArray()
        ];

        $response = $this->postJson('/api/matches', $matchData);

        $response->assertStatus(201);

        $match = Match::first();
        $this->assertEquals(2, $match->participations()->count());
        $this->assertEquals(2, $match->players()->count());
    }

    public function test_viewer_cannot_create_tournaments()
    {
        $viewer = User::factory()->viewer()->create();
        $this->actingAs($viewer, 'sanctum');

        $response = $this->postJson('/api/tournaments', []);

        $response->assertStatus(403);
    }
}
```

### **Fase 5: Frontend Integration (20 minutos**

#### **React Context para Autenticación (10 minutos)**

```jsx
// src/contexts/AuthContext.jsx
import React, { createContext, useContext, useState, useEffect } from 'react';
import authService from '../services/authService';

const AuthContext = createContext();

export const useAuth = () => useContext(AuthContext);

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const token = localStorage.getItem('token');
    if (token) {
      authService.setToken(token);
      loadUser();
    } else {
      setLoading(false);
    }
  }, []);

  const loadUser = async () => {
    try {
      const response = await authService.me();
      setUser(response.user);
    } catch (error) {
      localStorage.removeItem('token');
    } finally {
      setLoading(false);
    }
  };

  const login = async (credentials) => {
    const response = await authService.login(credentials);
    setUser(response.user);
    localStorage.setItem('token', response.token);
    authService.setToken(response.token);
  };

  const logout = async () => {
    try {
      await authService.logout();
    } finally {
      setUser(null);
      localStorage.removeItem('token');
      authService.removeToken();
    }
  };

  const value = {
    user,
    login,
    logout,
    loading,
    isAuthenticated: !!user,
    isAdmin: () => user?.role === 'admin',
    isOrganizer: () => user?.role === 'organizer',
    canManage: () => ['admin', 'organizer'].includes(user?.role),
  };

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
};
```

#### **Componente Principal del Torneo (10 minutos)**

```jsx
// src/components/TournamentDashboard.jsx
import React, { useState, useEffect } from 'react';
import { useAuth } from '../contexts/AuthContext';
import tournamentService from '../services/tournamentService';

const TournamentDashboard = () => {
  const [tournaments, setTournaments] = useState([]);
  const [players, setPlayers] = useState([]);
  const [matches, setMatches] = useState([]);
  const [loading, setLoading] = useState(true);

  const { user, canManage } = useAuth();

  useEffect(() => {
    loadData();
  }, []);

  const loadData = async () => {
    try {
      const [tournamentsRes, playersRes, matchesRes] = await Promise.all([
        tournamentService.getAll(),
        tournamentService.getTopPlayers(),
        tournamentService.getTodayMatches(),
      ]);

      setTournaments(tournamentsRes.data);
      setPlayers(playersRes.data);
      setMatches(matchesRes.data);
    } catch (error) {
      console.error('Error loading data:', error);
    } finally {
      setLoading(false);
    }
  };

  if (loading) {
    return (
      <div className="flex justify-center items-center min-h-screen">
        <div className="animate-spin rounded-full h-32 w-32 border-b-2 border-blue-500"></div>
      </div>
    );
  }

  return (
    <div className="min-h-screen bg-gray-100">
      <header className="bg-white shadow">
        <div className="max-w-7xl mx-auto px-4 py-6">
          <div className="flex justify-between items-center">
            <h1 className="text-3xl font-bold text-gray-900">
              🎾 Tennis Tournament Dashboard
            </h1>
            <div className="flex items-center space-x-4">
              <span className="text-gray-600">
                Welcome, {user?.name} ({user?.role})
              </span>
              {canManage() && (
                <button className="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600">
                  Manage Tournament
                </button>
              )}
            </div>
          </div>
        </div>
      </header>

      <main className="max-w-7xl mx-auto px-4 py-8">
        <div className="grid grid-cols-1 lg:grid-cols-3 gap-8">
          {/* Active Tournaments */}
          <div className="lg:col-span-2">
            <div className="bg-white rounded-lg shadow p-6">
              <h2 className="text-xl font-semibold mb-4">Active Tournaments</h2>
              <div className="space-y-4">
                {tournaments.map((tournament) => (
                  <div
                    key={tournament.id}
                    className="border rounded-lg p-4">
                    <div className="flex justify-between items-start">
                      <div>
                        <h3 className="font-semibold text-lg">
                          {tournament.name}
                        </h3>
                        <p className="text-gray-600">
                          📍 {tournament.country?.name} • 🏟️{' '}
                          {tournament.surface} • 💰 $
                          {tournament.prize_money?.toLocaleString()}
                        </p>
                        <p className="text-sm text-gray-500">
                          {new Date(tournament.start_date).toLocaleDateString()}{' '}
                          -{new Date(tournament.end_date).toLocaleDateString()}
                        </p>
                      </div>
                      <span
                        className={`
                                                px-2 py-1 rounded text-xs font-medium
                                                ${
                                                  tournament.status ===
                                                  'ongoing'
                                                    ? 'bg-green-100 text-green-800'
                                                    : 'bg-blue-100 text-blue-800'
                                                }
                                            `}>
                        {tournament.status}
                      </span>
                    </div>
                  </div>
                ))}
              </div>
            </div>
          </div>

          {/* Top Players */}
          <div>
            <div className="bg-white rounded-lg shadow p-6">
              <h2 className="text-xl font-semibold mb-4">Top Rankings</h2>
              <div className="space-y-3">
                {players.slice(0, 10).map((player, index) => (
                  <div
                    key={player.id}
                    className="flex items-center space-x-3">
                    <span className="w-6 h-6 bg-blue-100 text-blue-800 rounded-full flex items-center justify-center text-sm font-medium">
                      {index + 1}
                    </span>
                    <div className="flex-1">
                      <p className="font-medium">{player.name}</p>
                      <p className="text-sm text-gray-600">
                        {player.country?.name} • #{player.ranking}
                      </p>
                    </div>
                    <div className="text-right">
                      <p className="text-sm font-medium">
                        ${player.prize_money?.toLocaleString()}
                      </p>
                    </div>
                  </div>
                ))}
              </div>
            </div>
          </div>
        </div>

        {/* Today's Matches */}
        <div className="mt-8">
          <div className="bg-white rounded-lg shadow p-6">
            <h2 className="text-xl font-semibold mb-4">Today's Matches</h2>
            <div className="overflow-x-auto">
              <table className="min-w-full divide-y divide-gray-200">
                <thead className="bg-gray-50">
                  <tr>
                    <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">
                      Time
                    </th>
                    <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">
                      Match
                    </th>
                    <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">
                      Court
                    </th>
                    <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">
                      Status
                    </th>
                  </tr>
                </thead>
                <tbody className="bg-white divide-y divide-gray-200">
                  {matches.map((match) => (
                    <tr key={match.id}>
                      <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-900">
                        {new Date(match.match_date).toLocaleTimeString()}
                      </td>
                      <td className="px-6 py-4 whitespace-nowrap">
                        <div className="text-sm font-medium text-gray-900">
                          {match.players?.[0]?.name} vs{' '}
                          {match.players?.[1]?.name}
                        </div>
                        <div className="text-sm text-gray-500">
                          {match.round}
                        </div>
                      </td>
                      <td className="px-6 py-4 whitespace-nowrap text-sm text-gray-900">
                        {match.court}
                      </td>
                      <td className="px-6 py-4 whitespace-nowrap">
                        <span
                          className={`
                                                    px-2 py-1 rounded text-xs font-medium
                                                    ${
                                                      match.status === 'live'
                                                        ? 'bg-red-100 text-red-800'
                                                        : match.status ===
                                                          'finished'
                                                        ? 'bg-green-100 text-green-800'
                                                        : 'bg-yellow-100 text-yellow-800'
                                                    }
                                                `}>
                          {match.status}
                        </span>
                      </td>
                    </tr>
                  ))}
                </tbody>
              </table>
            </div>
          </div>
        </div>
      </main>
    </div>
  );
};

export default TournamentDashboard;
```

---

## 🎯 **Rúbrica de Evaluación**

| Criterio              | Excelente (4)                              | Bueno (3)                        | Satisfactorio (2)   | Insuficiente (1)      |
| --------------------- | ------------------------------------------ | -------------------------------- | ------------------- | --------------------- |
| **Modelado de Datos** | Todas las relaciones, constraints, índices | Relaciones principales correctas | Relaciones básicas  | Estructura incompleta |
| **Autenticación**     | Sistema completo con roles y permisos      | Auth básico funcionando          | Login/logout simple | No implementado       |
| **Testing**           | Feature + Unit, cobertura > 80%            | Tests principales funcionando    | Algunos tests       | Sin tests             |
| **Frontend**          | Dashboard completo e integrado             | Componentes básicos conectados   | Frontend simple     | Sin frontend          |
| **APIs**              | CRUD completo con validaciones             | Endpoints principales            | APIs básicas        | APIs incompletas      |
| **Tiempo**            | Completado en 2 horas                      | 2h 15min                         | 2h 30min            | > 2h 30min            |

### **Puntaje Total: \_\_\_ / 100 puntos**

---

## 🏆 **Funcionalidad Mínima Requerida (≥ 85 puntos)**

### **Backend (50 puntos)**

- [ ] ✅ **Migraciones completas** con todas las relaciones (15 pts)
- [ ] ✅ **Modelos Eloquent** con relaciones y métodos (15 pts)
- [ ] ✅ **API REST** funcional con CRUD básico (10 pts)
- [ ] ✅ **Autenticación** con roles funcional (10 pts)

### **Testing (20 puntos)**

- [ ] ✅ **Database Factories** con datos realistas (5 pts)
- [ ] ✅ **Feature Tests** para endpoints principales (10 pts)
- [ ] ✅ **Unit Tests** para modelos críticos (5 pts)

### **Frontend (20 puntos)**

- [ ] ✅ **Dashboard funcional** mostrando datos (10 pts)
- [ ] ✅ **Autenticación integrada** con contexto (5 pts)
- [ ] ✅ **Componentes reutilizables** (5 pts)

### **Integración (10 puntos)**

- [ ] ✅ **Frontend ↔ Backend** comunicación fluida (5 pts)
- [ ] ✅ **Manejo de errores** y loading states (5 pts)

---

## 🔧 **Comandos de Verificación Final**

```bash
# Backend
php artisan migrate:status
php artisan test --filter TournamentApiTest
php artisan route:list | grep api

# Frontend
cd tennis-tournament-frontend
pnpm build
pnpm preview

# Integration test
curl -X POST http://localhost:8000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password"}'
```

---

## 💡 **Tips de Última Hora**

### **Si te quedas sin tiempo:**

1. **Prioriza backend** (relaciones + auth) = 70% del puntaje
2. **Factories simples** pero funcionales
3. **Frontend básico** que muestre datos
4. **Un test que funcione** es mejor que varios rotos

### **Puntos extra:**

- **Seeders** con datos de ejemplo
- **Validaciones** robustas
- **Error handling** profesional
- **UI responsiva** y atractiva

---

**🔥 ¡Este desafío valida si los competidores están listos para sistemas complejos de WorldSkills!**
