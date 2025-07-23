# 🔗 Relaciones Eloquent Avanzadas - Día 2.1

**Objetivo:** Dominar modelado de datos deportivos con relaciones complejas

---

## 🎯 **Objetivo de la Sesión**

Los competidores deben poder modelar relaciones **One-to-Many**, **Many-to-Many** y **Polymorphic** para un sistema deportivo completo en **menos de 20 minutos**, implementando todas las restricciones de integridad referencial.

---

## 📋 **Entidades del Sistema de Torneo**

```
🏆 SISTEMA DE TORNEO FEMENINO SUDAMERICANO

Teams (Equipos)
├─ País, técnico, jugadoras
├─ One-to-Many: Team hasMany Players
└─ Many-to-Many: Team belongsToMany Matches (pivot: home/away)

Players (Jugadoras)
├─ Nombre, posición, edad
├─ Many-to-One: Player belongsTo Team
└─ Polymorphic: Player morphMany Comments

Matches (Partidos)
├─ Fecha, estadio, resultado
├─ Many-to-Many: Match belongsToMany Teams
└─ One-to-Many: Match hasMany Goals

Goals (Goles)
├─ Minuto, tipo, jugadora
├─ Many-to-One: Goal belongsTo Player
└─ Many-to-One: Goal belongsTo Match

Comments (Comentarios) - Polymorphic
├─ Texto, autor, timestamp
└─ Morphs: comentarios en Teams, Players, Matches
```

---

## ⏱️ **EJERCICIO 1: Relaciones One-to-Many (30 minutos)**

### **Paso 1: Modelos Base (10 minutos)**

```bash
# Crear modelos con migraciones
php artisan make:model Team -m
php artisan make:model Player -m
php artisan make:model Match -m
php artisan make:model Goal -m
```

### **Paso 2: Migraciones con Constraints (15 minutos)**

```php
// database/migrations/xxxx_create_teams_table.php
public function up()
{
    Schema::create('teams', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->string('country', 3); // ISO code: ARG, BRA, COL
        $table->string('coach');
        $table->integer('founded_year');
        $table->string('logo_url')->nullable();
        $table->timestamps();

        // Índices para búsquedas rápidas
        $table->index('country');
        $table->unique(['name', 'country']);
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
        $table->integer('jersey_number');
        $table->enum('position', ['GK', 'DF', 'MF', 'FW']);
        $table->date('birth_date');
        $table->integer('goals')->default(0);
        $table->integer('assists')->default(0);

        // Foreign Key con constraint
        $table->foreignId('team_id')
              ->constrained()
              ->onDelete('cascade')
              ->onUpdate('cascade');

        $table->timestamps();

        // Índices estratégicos
        $table->index(['team_id', 'position']);
        $table->unique(['team_id', 'jersey_number']);
    });
}
```

### **Paso 3: Relaciones en Modelos (5 minutos)**

```php
// app/Models/Team.php
class Team extends Model
{
    protected $fillable = [
        'name', 'country', 'coach', 'founded_year', 'logo_url'
    ];

    // One-to-Many: Un equipo tiene muchas jugadoras
    public function players()
    {
        return $this->hasMany(Player::class);
    }

    // Scope para filtrar por país
    public function scopeFromCountry($query, $country)
    {
        return $query->where('country', $country);
    }

    // Query builder optimizado
    public function playersWithStats()
    {
        return $this->players()
                   ->select(['id', 'name', 'position', 'goals', 'assists', 'team_id'])
                   ->orderBy('goals', 'desc');
    }

    // Accessor para bandera
    public function getFlagUrlAttribute()
    {
        return "https://flagcdn.com/w40/{$this->country}.png";
    }
}
```

```php
// app/Models/Player.php
class Player extends Model
{
    protected $fillable = [
        'name', 'jersey_number', 'position', 'birth_date', 'team_id', 'goals', 'assists'
    ];

    protected $casts = [
        'birth_date' => 'date',
    ];

    // Many-to-One: Una jugadora pertenece a un equipo
    public function team()
    {
        return $this->belongsTo(Team::class);
    }

    // Accessor para edad
    public function getAgeAttribute()
    {
        return $this->birth_date->age;
    }

    // Scope para posición
    public function scopeByPosition($query, $position)
    {
        return $query->where('position', $position);
    }

    // Scope para goleadoras
    public function scopeTopScorers($query, $limit = 10)
    {
        return $query->where('goals', '>', 0)
                     ->orderBy('goals', 'desc')
                     ->limit($limit);
    }
}
```

---

## ⏱️ **EJERCICIO 2: Many-to-Many con Pivot (45 minutos)**

### **Paso 1: Tabla Pivot Personalizada (15 minutos)**

```bash
# Crear migración para tabla pivot
php artisan make:migration create_match_team_table
```

```php
// database/migrations/xxxx_create_match_team_table.php
public function up()
{
    Schema::create('match_team', function (Blueprint $table) {
        $table->id();

        // Foreign keys
        $table->foreignId('match_id')->constrained()->onDelete('cascade');
        $table->foreignId('team_id')->constrained()->onDelete('cascade');

        // Campos adicionales del pivot
        $table->enum('side', ['home', 'away']);
        $table->integer('score')->default(0);
        $table->json('formation')->nullable(); // 4-4-2, 3-5-2, etc.
        $table->timestamps();

        // Constraints únicos
        $table->unique(['match_id', 'team_id']);
        $table->unique(['match_id', 'side']); // Solo un home y un away por partido

        // Índices
        $table->index(['match_id', 'side']);
    });
}
```

### **Paso 2: Modelo Match con Relación (15 minutos)**

```php
// database/migrations/xxxx_create_matches_table.php
public function up()
{
    Schema::create('matches', function (Blueprint $table) {
        $table->id();
        $table->datetime('match_date');
        $table->string('stadium');
        $table->string('city');
        $table->enum('status', ['scheduled', 'playing', 'finished', 'cancelled'])
              ->default('scheduled');
        $table->integer('attendance')->nullable();
        $table->string('referee')->nullable();
        $table->timestamps();

        // Índices para queries comunes
        $table->index(['match_date', 'status']);
        $table->index('stadium');
    });
}
```

```php
// app/Models/Match.php
class Match extends Model
{
    protected $fillable = [
        'match_date', 'stadium', 'city', 'status', 'attendance', 'referee'
    ];

    protected $casts = [
        'match_date' => 'datetime',
    ];

    // Many-to-Many con pivot personalizado
    public function teams()
    {
        return $this->belongsToMany(Team::class, 'match_team')
                   ->withPivot(['side', 'score', 'formation'])
                   ->withTimestamps();
    }

    // Relación específica para equipo local
    public function homeTeam()
    {
        return $this->belongsToMany(Team::class, 'match_team')
                   ->wherePivot('side', 'home')
                   ->withPivot(['score', 'formation']);
    }

    // Relación específica para equipo visitante
    public function awayTeam()
    {
        return $this->belongsToMany(Team::class, 'match_team')
                   ->wherePivot('side', 'away')
                   ->withPivot(['score', 'formation']);
    }

    // Scope para partidos de hoy
    public function scopeToday($query)
    {
        return $query->whereDate('match_date', today());
    }

    // Scope para partidos en vivo
    public function scopeLive($query)
    {
        return $query->where('status', 'playing');
    }

    // Accessor para resultado
    public function getResultAttribute()
    {
        $teams = $this->teams()->withPivot('score')->get();
        if ($teams->count() !== 2) return null;

        $home = $teams->where('pivot.side', 'home')->first();
        $away = $teams->where('pivot.side', 'away')->first();

        return [
            'home' => ['team' => $home->name, 'score' => $home->pivot->score],
            'away' => ['team' => $away->name, 'score' => $away->pivot->score],
            'status' => $this->status
        ];
    }
}
```

### **Paso 3: Uso de Relaciones Many-to-Many (15 minutos)**

```php
// Crear un partido con equipos
$match = Match::create([
    'match_date' => '2025-07-25 20:00:00',
    'stadium' => 'Estadio Monumental',
    'city' => 'Buenos Aires',
    'referee' => 'María González'
]);

// Asociar equipos con información del pivot
$argentina = Team::where('country', 'ARG')->first();
$brasil = Team::where('country', 'BRA')->first();

$match->teams()->attach($argentina->id, [
    'side' => 'home',
    'score' => 2,
    'formation' => json_encode(['formation' => '4-4-2', 'captain' => 'Messi'])
]);

$match->teams()->attach($brasil->id, [
    'side' => 'away',
    'score' => 1,
    'formation' => json_encode(['formation' => '4-3-3', 'captain' => 'Marta'])
]);

// Consultar datos del pivot
$matchDetails = Match::with(['teams' => function($query) {
    $query->withPivot(['side', 'score', 'formation']);
}])->find($match->id);

foreach ($matchDetails->teams as $team) {
    echo "{$team->name} ({$team->pivot->side}): {$team->pivot->score} gols\n";
}
```

---

## ⏱️ **EJERCICIO 3: Relaciones Polymorphic (40 minutos)**

### **Paso 1: Modelo Comment Polymorphic (15 minutos)**

```bash
# Crear modelo para comentarios polimórficos
php artisan make:model Comment -m
```

```php
// database/migrations/xxxx_create_comments_table.php
public function up()
{
    Schema::create('comments', function (Blueprint $table) {
        $table->id();
        $table->text('content');
        $table->string('author');
        $table->string('author_role')->default('fan'); // fan, journalist, coach
        $table->integer('rating')->nullable(); // 1-5 stars

        // Campos polimórficos
        $table->morphs('commentable'); // commentable_id, commentable_type

        $table->timestamps();

        // Índices para queries polimórficas
        $table->index(['commentable_type', 'commentable_id']);
        $table->index(['created_at', 'rating']);
    });
}
```

```php
// app/Models/Comment.php
class Comment extends Model
{
    protected $fillable = [
        'content', 'author', 'author_role', 'rating',
        'commentable_id', 'commentable_type'
    ];

    // Relación polimórfica inversa
    public function commentable()
    {
        return $this->morphTo();
    }

    // Scope para comentarios recientes
    public function scopeRecent($query, $days = 7)
    {
        return $query->where('created_at', '>=', now()->subDays($days));
    }

    // Scope por tipo de autor
    public function scopeByAuthorRole($query, $role)
    {
        return $query->where('author_role', $role);
    }

    // Scope por rating
    public function scopeHighRated($query, $minRating = 4)
    {
        return $query->where('rating', '>=', $minRating);
    }
}
```

### **Paso 2: Añadir Relaciones Polymorphic a Modelos (15 minutos)**

```php
// Agregar a app/Models/Team.php
public function comments()
{
    return $this->morphMany(Comment::class, 'commentable');
}

public function recentComments()
{
    return $this->morphMany(Comment::class, 'commentable')
               ->recent()
               ->orderBy('created_at', 'desc');
}

// Agregar a app/Models/Player.php
public function comments()
{
    return $this->morphMany(Comment::class, 'commentable');
}

public function fanComments()
{
    return $this->morphMany(Comment::class, 'commentable')
               ->byAuthorRole('fan')
               ->orderBy('rating', 'desc');
}

// Agregar a app/Models/Match.php
public function comments()
{
    return $this->morphMany(Comment::class, 'commentable');
}

public function matchReports()
{
    return $this->morphMany(Comment::class, 'commentable')
               ->byAuthorRole('journalist')
               ->orderBy('created_at', 'desc');
}
```

### **Paso 3: Usar Relaciones Polymorphic (10 minutos)**

```php
// Crear comentarios para diferentes entidades
$team = Team::first();
$player = Player::first();
$match = Match::first();

// Comentario en equipo
$team->comments()->create([
    'content' => 'Excelente desempeño en el torneo',
    'author' => 'Juan Pérez',
    'author_role' => 'journalist',
    'rating' => 5
]);

// Comentario en jugadora
$player->comments()->create([
    'content' => 'Mejor jugadora del partido',
    'author' => 'María Silva',
    'author_role' => 'fan',
    'rating' => 5
]);

// Comentario en partido
$match->comments()->create([
    'content' => 'Partido muy disputado y emocionante',
    'author' => 'Carlos López',
    'author_role' => 'coach',
    'rating' => 4
]);

// Consultar comentarios polimórficos
$allComments = Comment::with('commentable')->recent()->get();

foreach ($allComments as $comment) {
    $type = class_basename($comment->commentable_type);
    $name = $comment->commentable->name ?? $comment->commentable->stadium;
    echo "Comentario en {$type}: {$name} - {$comment->content}\n";
}
```

---

## ⏱️ **EJERCICIO 4: Eager Loading y Optimización (35 minutos)**

### **Paso 1: N+1 Query Problem Identificado (10 minutos)**

```php
// ❌ PROBLEMA: N+1 Queries
$teams = Team::all(); // 1 query

foreach ($teams as $team) {
    echo $team->name . " tiene " . $team->players->count() . " jugadoras\n"; // N queries
}
// Total: 1 + N queries (si hay 10 equipos = 11 queries)
```

### **Paso 2: Eager Loading Básico (10 minutos)**

```php
// ✅ SOLUCIÓN: Eager Loading
$teams = Team::with('players')->get(); // 2 queries total

foreach ($teams as $team) {
    echo $team->name . " tiene " . $team->players->count() . " jugadoras\n"; // Sin queries extra
}

// Eager Loading múltiple
$matches = Match::with(['teams', 'comments'])->get();

// Eager Loading con constraints
$teams = Team::with(['players' => function ($query) {
    $query->where('goals', '>', 5)
          ->orderBy('goals', 'desc');
}])->get();

// Eager Loading polimórfico
$comments = Comment::with('commentable')->recent()->get();
```

### **Paso 3: Lazy Eager Loading (10 minutos)**

```php
// Cargar relaciones después si es necesario
$teams = Team::all();

// Si necesitas las jugadoras después
$teams->load('players');

// Con constraints
$teams->load(['players' => function ($query) {
    $query->byPosition('FW');
}]);

// Load múltiple
$teams->load(['players.comments', 'comments']);
```

### **Paso 4: Performance con Modelos (5 minutos)**

```php
// Usar select para campos específicos
$teams = Team::select(['id', 'name', 'country'])
             ->with(['players:id,name,team_id,goals'])
             ->get();

// Contar sin cargar modelos
$teamCounts = Team::withCount(['players', 'comments'])->get();

foreach ($teamCounts as $team) {
    echo "{$team->name}: {$team->players_count} jugadoras, ";
    echo "{$team->comments_count} comentarios\n";
}

// Exists en lugar de count para validaciones
$hasPlayers = Team::whereHas('players')->exists();
```

---

## 🧪 **EJERCICIO 5: Factories y Seeders (20 minutos)**

### **Paso 1: Database Factories (10 minutos)**

```bash
# Crear factories
php artisan make:factory TeamFactory
php artisan make:factory PlayerFactory
php artisan make:factory CommentFactory
```

```php
// database/factories/TeamFactory.php
class TeamFactory extends Factory
{
    public function definition()
    {
        $countries = ['ARG', 'BRA', 'COL', 'CHI', 'URU', 'PER', 'ECU', 'BOL', 'PAR', 'VEN'];

        return [
            'name' => 'Club ' . $this->faker->city(),
            'country' => $this->faker->randomElement($countries),
            'coach' => $this->faker->name(),
            'founded_year' => $this->faker->numberBetween(1900, 2020),
            'logo_url' => $this->faker->imageUrl(100, 100, 'sports'),
        ];
    }
}
```

```php
// database/factories/PlayerFactory.php
class PlayerFactory extends Factory
{
    public function definition()
    {
        return [
            'name' => $this->faker->name(),
            'jersey_number' => $this->faker->unique()->numberBetween(1, 99),
            'position' => $this->faker->randomElement(['GK', 'DF', 'MF', 'FW']),
            'birth_date' => $this->faker->dateTimeBetween('-35 years', '-18 years'),
            'goals' => $this->faker->numberBetween(0, 20),
            'assists' => $this->faker->numberBetween(0, 15),
        ];
    }

    // Factory states
    public function goalkeeper()
    {
        return $this->state(['position' => 'GK', 'goals' => 0]);
    }

    public function striker()
    {
        return $this->state(['position' => 'FW', 'goals' => $this->faker->numberBetween(5, 20)]);
    }
}
```

### **Paso 2: Seeder Completo (10 minutos)**

```php
// database/seeders/TournamentSeeder.php
class TournamentSeeder extends Seeder
{
    public function run()
    {
        // Crear equipos con jugadoras
        $teams = Team::factory(10)->create();

        foreach ($teams as $team) {
            // Cada equipo tiene 1 portera, 4 defensas, 4 medios, 3 delanteras
            Player::factory()->goalkeeper()->create(['team_id' => $team->id, 'jersey_number' => 1]);
            Player::factory(4)->state(['position' => 'DF'])->create(['team_id' => $team->id]);
            Player::factory(4)->state(['position' => 'MF'])->create(['team_id' => $team->id]);
            Player::factory(3)->striker()->create(['team_id' => $team->id]);
        }

        // Crear partidos
        $matches = Match::factory(15)->create();

        // Asignar equipos a partidos
        foreach ($matches as $match) {
            $selectedTeams = $teams->random(2);

            $match->teams()->attach($selectedTeams[0]->id, [
                'side' => 'home',
                'score' => rand(0, 4),
                'formation' => json_encode(['formation' => '4-4-2'])
            ]);

            $match->teams()->attach($selectedTeams[1]->id, [
                'side' => 'away',
                'score' => rand(0, 4),
                'formation' => json_encode(['formation' => '4-3-3'])
            ]);
        }

        // Crear comentarios polimórficos
        foreach ($teams as $team) {
            Comment::factory(rand(1, 5))->create([
                'commentable_id' => $team->id,
                'commentable_type' => Team::class
            ]);
        }

        foreach (Player::limit(20)->get() as $player) {
            Comment::factory(rand(0, 3))->create([
                'commentable_id' => $player->id,
                'commentable_type' => Player::class
            ]);
        }
    }
}
```

---

## 🎯 **Desafío de Velocidad (20 minutos)**

### **Crear Sistema Completo de Ratings**

**Objetivo:** Implementar relaciones Many-to-Many para sistema de calificaciones

```php
// Entidades:
// - Users (pueden calificar)
// - Rateable (Teams, Players, Matches pueden ser calificados)
// - Ratings (tabla pivot con score y comentario)

// Tiempo límite: 20 minutos
// Incluir: migraciones, modelos, relaciones, factory, seeder
```

**Criterios de Evaluación:**

- ✅ Tabla pivot con campos adicionales (10 puntos)
- ✅ Relación polimórfica correcta (15 puntos)
- ✅ Scopes útiles en modelos (10 puntos)
- ✅ Factory y seeder funcionando (10 puntos)
- ✅ Queries optimizadas (eager loading) (15 puntos)

---

## 📊 **Métricas de Éxito**

Al completar esta sesión, verificar:

- [ ] ⏱️ **Many-to-Many** implementado en < 15 minutos
- [ ] 🔗 **Relación polimórfica** funcionando correctamente
- [ ] ⚡ **Eager loading** reduciendo queries > 80%
- [ ] 🏭 **Factories** generando datos consistentes
- [ ] 🌱 **Seeders** poblando base de datos completa

---

**🔥 ¡Relaciones complejas dominadas! Preparados para autenticación avanzada.**
