# 🧪 Guía de Testing Automatizado - Día 2.3

**Objetivo:** Implementar tests completos (Feature + Unit) en menos de 15 minutos

---

## 🎯 **Objetivo de la Sesión**

Los competidores deben poder crear una **test suite completa** que incluya Feature Tests para endpoints, Unit Tests para modelos y servicios, Database Factories para datos realistas, y alcanzar **> 80% de cobertura** en funcionalidades críticas, todo en **menos de 15 minutos**.

---

## 🧪 **Estrategia de Testing para WorldSkills**

### **Pirámide de Testing Optimizada**

```
🏆 Testing Strategy for Competition

Unit Tests (60%)
├─ Models: Relaciones, métodos, validaciones
├─ Services: Lógica de negocio pura
└─ Utilities: Helpers y transformers

Feature Tests (35%)
├─ API Endpoints: CRUD completo
├─ Authentication: Login, register, middleware
└─ Integration: Frontend ↔ Backend

E2E Tests (5%) - Solo si hay tiempo extra
├─ User flows críticos
└─ Happy path scenarios
```

---

## ⏱️ **Ejercicio 1: Setup Testing (5 minutos)**

### **Configuración Rápida**

```bash
# CRONÓMETRO: INICIAR ⏰

# 1. Verificar PHPUnit está instalado
./vendor/bin/phpunit --version

# 2. Configurar base de datos de testing
cp .env .env.testing
```

#### **Configurar .env.testing**

```bash
# .env.testing
APP_ENV=testing
DB_CONNECTION=sqlite
DB_DATABASE=:memory:

# Desactivar features innecesarias para testing
BROADCAST_DRIVER=log
CACHE_DRIVER=array
FILESYSTEM_DISK=local
QUEUE_CONNECTION=sync
SESSION_DRIVER=array
```

#### **Configurar phpunit.xml**

```xml
<!-- phpunit.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<phpunit xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:noNamespaceSchemaLocation="./vendor/phpunit/phpunit/phpunit.xsd"
         bootstrap="vendor/autoload.php"
         colors="true">
    <testsuites>
        <testsuite name="Unit">
            <directory suffix="Test.php">./tests/Unit</directory>
        </testsuite>
        <testsuite name="Feature">
            <directory suffix="Test.php">./tests/Feature</directory>
        </testsuite>
    </testsuites>
    <coverage processUncoveredFiles="true">
        <include>
            <directory suffix=".php">./app</directory>
        </include>
        <exclude>
            <directory suffix=".php">./app/Console</directory>
        </exclude>
    </coverage>
    <php>
        <env name="APP_ENV" value="testing"/>
        <env name="BCRYPT_ROUNDS" value="4"/>
        <env name="CACHE_DRIVER" value="array"/>
        <env name="DB_CONNECTION" value="sqlite"/>
        <env name="DB_DATABASE" value=":memory:"/>
        <env name="MAIL_MAILER" value="array"/>
        <env name="QUEUE_CONNECTION" value="sync"/>
        <env name="SESSION_DRIVER" value="array"/>
        <env name="TELESCOPE_ENABLED" value="false"/>
    </php>
</phpunit>
```

---

## ⏱️ **Ejercicio 2: Database Factories (10 minutos)**

### **Factories para Datos Realistas**

#### **CountryFactory**

```bash
php artisan make:factory CountryFactory
```

```php
// database/factories/CountryFactory.php
<?php

namespace Database\Factories;

use Illuminate\Database\Eloquent\Factories\Factory;

class CountryFactory extends Factory
{
    private static $southAmericanCountries = [
        ['name' => 'Argentina', 'iso_code' => 'ARG', 'confederation' => 'CONMEBOL'],
        ['name' => 'Brasil', 'iso_code' => 'BRA', 'confederation' => 'CONMEBOL'],
        ['name' => 'Chile', 'iso_code' => 'CHI', 'confederation' => 'CONMEBOL'],
        ['name' => 'Colombia', 'iso_code' => 'COL', 'confederation' => 'CONMEBOL'],
        ['name' => 'Ecuador', 'iso_code' => 'ECU', 'confederation' => 'CONMEBOL'],
        ['name' => 'Paraguay', 'iso_code' => 'PAR', 'confederation' => 'CONMEBOL'],
        ['name' => 'Perú', 'iso_code' => 'PER', 'confederation' => 'CONMEBOL'],
        ['name' => 'Uruguay', 'iso_code' => 'URU', 'confederation' => 'CONMEBOL'],
        ['name' => 'Venezuela', 'iso_code' => 'VEN', 'confederation' => 'CONMEBOL'],
        ['name' => 'Bolivia', 'iso_code' => 'BOL', 'confederation' => 'CONMEBOL'],
    ];

    private static $index = 0;

    public function definition()
    {
        $country = self::$southAmericanCountries[self::$index % count(self::$southAmericanCountries)];
        self::$index++;

        return [
            'name' => $country['name'],
            'iso_code' => $country['iso_code'],
            'flag_url' => "https://flagcdn.com/w320/{$country['iso_code']}.png",
            'confederation' => $country['confederation'],
        ];
    }

    public function resetIndex()
    {
        self::$index = 0;
        return $this;
    }
}
```

#### **TeamFactory**

```bash
php artisan make:factory TeamFactory
```

```php
// database/factories/TeamFactory.php
<?php

namespace Database\Factories;

use App\Models\Country;
use Illuminate\Database\Eloquent\Factories\Factory;

class TeamFactory extends Factory
{
    public function definition()
    {
        $teamNames = [
            'Las Leonas', 'Las Guerreras', 'Las Amazonas', 'Las Águilas',
            'Las Panteras', 'Las Tigres', 'Las Cóndores', 'Las Diosas',
            'Las Valientes', 'Las Campeonas'
        ];

        $coaches = [
            'María González', 'Ana Rodriguez', 'Carmen López', 'Isabel Torres',
            'Patricia Morales', 'Rosa Hernández', 'Elena Vargas', 'Lucia Castillo'
        ];

        return [
            'name' => $this->faker->randomElement($teamNames),
            'country_id' => Country::factory(),
            'logo_url' => $this->faker->imageUrl(200, 200, 'sports'),
            'coach' => $this->faker->randomElement($coaches),
            'founded_year' => $this->faker->numberBetween(1950, 2020),
        ];
    }
}
```

#### **PlayerFactory**

```bash
php artisan make:factory PlayerFactory
```

```php
// database/factories/PlayerFactory.php
<?php

namespace Database\Factories;

use App\Models\Team;
use Illuminate\Database\Eloquent\Factories\Factory;

class PlayerFactory extends Factory
{
    private static $positions = ['GK', 'DEF', 'MID', 'FWD'];
    private static $femaleNames = [
        'María', 'Ana', 'Carmen', 'Laura', 'Sofia', 'Isabella', 'Valentina',
        'Camila', 'Victoria', 'Alejandra', 'Natalia', 'Andrea', 'Paula',
        'Daniela', 'Gabriela', 'Carolina', 'Fernanda', 'Mariana', 'Catalina'
    ];

    public function definition()
    {
        return [
            'name' => $this->faker->randomElement(self::$femaleNames) . ' ' . $this->faker->lastName,
            'team_id' => Team::factory(),
            'position' => $this->faker->randomElement(self::$positions),
            'jersey_number' => $this->faker->unique()->numberBetween(1, 23),
            'birth_date' => $this->faker->dateTimeBetween('-35 years', '-16 years'),
            'height' => $this->faker->randomFloat(2, 1.50, 1.90),
            'foot' => $this->faker->randomElement(['right', 'left', 'both']),
        ];
    }

    public function goalkeeper()
    {
        return $this->state(['position' => 'GK']);
    }

    public function defender()
    {
        return $this->state(['position' => 'DEF']);
    }

    public function midfielder()
    {
        return $this->state(['position' => 'MID']);
    }

    public function forward()
    {
        return $this->state(['position' => 'FWD']);
    }
}
```

#### **UserFactory (actualizar)**

```php
// database/factories/UserFactory.php
<?php

namespace Database\Factories;

use Illuminate\Database\Eloquent\Factories\Factory;
use Illuminate\Support\Facades\Hash;
use Illuminate\Support\Str;

class UserFactory extends Factory
{
    protected static ?string $password;

    public function definition()
    {
        return [
            'name' => fake()->name(),
            'email' => fake()->unique()->safeEmail(),
            'email_verified_at' => now(),
            'password' => static::$password ??= Hash::make('password'),
            'remember_token' => Str::random(10),
            'role' => 'viewer',
        ];
    }

    public function unverified()
    {
        return $this->state(fn (array $attributes) => [
            'email_verified_at' => null,
        ]);
    }

    public function admin()
    {
        return $this->state(['role' => 'admin']);
    }

    public function organizer()
    {
        return $this->state(['role' => 'organizer']);
    }

    public function viewer()
    {
        return $this->state(['role' => 'viewer']);
    }
}
```

---

## ⏱️ **Ejercicio 3: Unit Tests (15 minutos)**

### **Unit Test para Country Model**

```bash
php artisan make:test Unit/CountryTest --unit
```

```php
// tests/Unit/CountryTest.php
<?php

namespace Tests\Unit;

use App\Models\Country;
use App\Models\Team;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class CountryTest extends TestCase
{
    use RefreshDatabase;

    public function test_country_has_team_relationship()
    {
        $country = Country::factory()->create();
        $team = Team::factory()->create(['country_id' => $country->id]);

        $this->assertTrue($country->team->is($team));
        $this->assertTrue($team->country->is($country));
    }

    public function test_country_has_teams_relationship()
    {
        $country = Country::factory()->create();
        $teams = Team::factory()->count(2)->create(['country_id' => $country->id]);

        $this->assertCount(2, $country->teams);
        $this->assertTrue($country->teams->contains($teams[0]));
    }

    public function test_conmebol_scope_filters_correctly()
    {
        Country::factory()->create(['confederation' => 'CONMEBOL']);
        Country::factory()->create(['confederation' => 'UEFA']);
        Country::factory()->create(['confederation' => 'CONMEBOL']);

        $conmebolCountries = Country::conmebol()->get();

        $this->assertCount(2, $conmebolCountries);
        $conmebolCountries->each(function ($country) {
            $this->assertEquals('CONMEBOL', $country->confederation);
        });
    }

    public function test_flag_accessor_returns_default_when_null()
    {
        $country = Country::factory()->create([
            'iso_code' => 'ARG',
            'flag_url' => null
        ]);

        $expectedUrl = "https://flagcdn.com/w320/ARG.png";
        $this->assertEquals($expectedUrl, $country->flag);
    }

    public function test_flag_accessor_returns_existing_url()
    {
        $customUrl = 'https://example.com/custom-flag.png';
        $country = Country::factory()->create(['flag_url' => $customUrl]);

        $this->assertEquals($customUrl, $country->flag);
    }
}
```

### **Unit Test para Player Model**

```bash
php artisan make:test Unit/PlayerTest --unit
```

```php
// tests/Unit/PlayerTest.php
<?php

namespace Tests\Unit;

use App\Models\Player;
use App\Models\Team;
use App\Models\Statistic;
use Carbon\Carbon;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class PlayerTest extends TestCase
{
    use RefreshDatabase;

    public function test_player_belongs_to_team()
    {
        $team = Team::factory()->create();
        $player = Player::factory()->create(['team_id' => $team->id]);

        $this->assertTrue($player->team->is($team));
    }

    public function test_player_has_statistics_relationship()
    {
        $player = Player::factory()->create();
        $statistic = Statistic::factory()->create([
            'statisticable_id' => $player->id,
            'statisticable_type' => Player::class,
            'stat_type' => 'goals',
            'value' => 5
        ]);

        $this->assertCount(1, $player->statistics);
        $this->assertTrue($player->statistics->first()->is($statistic));
    }

    public function test_age_accessor_calculates_correctly()
    {
        $birthDate = Carbon::now()->subYears(25);
        $player = Player::factory()->create(['birth_date' => $birthDate]);

        $this->assertEquals(25, $player->age);
    }

    public function test_full_name_accessor_includes_jersey_number()
    {
        $player = Player::factory()->create([
            'name' => 'Maria Rodriguez',
            'jersey_number' => 10
        ]);

        $this->assertEquals('Maria Rodriguez (#10)', $player->full_name);
    }

    public function test_name_mutator_capitalizes_correctly()
    {
        $player = Player::factory()->create(['name' => 'maria rodriguez']);

        $this->assertEquals('Maria Rodriguez', $player->name);
    }

    public function test_goals_attribute_sums_goal_statistics()
    {
        $player = Player::factory()->create();

        Statistic::factory()->create([
            'statisticable_id' => $player->id,
            'statisticable_type' => Player::class,
            'stat_type' => 'goals',
            'value' => 3
        ]);

        Statistic::factory()->create([
            'statisticable_id' => $player->id,
            'statisticable_type' => Player::class,
            'stat_type' => 'goals',
            'value' => 2
        ]);

        $this->assertEquals(5, $player->goals);
    }

    public function test_is_goalkeeper_method()
    {
        $goalkeeper = Player::factory()->goalkeeper()->create();
        $defender = Player::factory()->defender()->create();

        $this->assertTrue($goalkeeper->isGoalkeeper());
        $this->assertFalse($defender->isGoalkeeper());
    }

    public function test_by_position_scope()
    {
        Player::factory()->goalkeeper()->create();
        Player::factory()->defender()->count(2)->create();
        Player::factory()->midfielder()->create();

        $defenders = Player::byPosition('DEF')->get();
        $goalkeepers = Player::goalkeepers()->get();

        $this->assertCount(2, $defenders);
        $this->assertCount(1, $goalkeepers);

        $defenders->each(function ($player) {
            $this->assertEquals('DEF', $player->position);
        });
    }
}
```

---

## ⏱️ **Ejercicio 4: Feature Tests (20 minutos)**

### **Feature Test para Country API**

```bash
php artisan make:test Feature/CountryApiTest
```

```php
// tests/Feature/CountryApiTest.php
<?php

namespace Tests\Feature;

use App\Models\Country;
use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class CountryApiTest extends TestCase
{
    use RefreshDatabase;

    private function authenticatedUser($role = 'viewer')
    {
        $user = User::factory()->create(['role' => $role]);
        $this->actingAs($user, 'sanctum');
        return $user;
    }

    public function test_can_list_countries()
    {
        $this->authenticatedUser();

        Country::factory()->count(3)->create();

        $response = $this->getJson('/api/countries');

        $response->assertStatus(200)
                ->assertJsonStructure([
                    'data' => [
                        '*' => ['id', 'name', 'iso_code', 'flag_url', 'confederation']
                    ]
                ]);
    }

    public function test_can_show_country()
    {
        $this->authenticatedUser();

        $country = Country::factory()->create();

        $response = $this->getJson("/api/countries/{$country->id}");

        $response->assertStatus(200)
                ->assertJson([
                    'data' => [
                        'id' => $country->id,
                        'name' => $country->name,
                        'iso_code' => $country->iso_code
                    ]
                ]);
    }

    public function test_admin_can_create_country()
    {
        $this->authenticatedUser('admin');

        $countryData = [
            'name' => 'Test Country',
            'iso_code' => 'TST',
            'confederation' => 'CONMEBOL'
        ];

        $response = $this->postJson('/api/countries', $countryData);

        $response->assertStatus(201)
                ->assertJson([
                    'data' => $countryData
                ]);

        $this->assertDatabaseHas('countries', $countryData);
    }

    public function test_viewer_cannot_create_country()
    {
        $this->authenticatedUser('viewer');

        $countryData = [
            'name' => 'Test Country',
            'iso_code' => 'TST',
            'confederation' => 'CONMEBOL'
        ];

        $response = $this->postJson('/api/countries', $countryData);

        $response->assertStatus(403);
        $this->assertDatabaseMissing('countries', $countryData);
    }

    public function test_create_country_validation()
    {
        $this->authenticatedUser('admin');

        $response = $this->postJson('/api/countries', []);

        $response->assertStatus(422)
                ->assertJsonValidationErrors(['name', 'iso_code', 'confederation']);
    }

    public function test_admin_can_update_country()
    {
        $this->authenticatedUser('admin');

        $country = Country::factory()->create();
        $updateData = ['name' => 'Updated Name'];

        $response = $this->putJson("/api/countries/{$country->id}", $updateData);

        $response->assertStatus(200);
        $this->assertDatabaseHas('countries', [
            'id' => $country->id,
            'name' => 'Updated Name'
        ]);
    }

    public function test_admin_can_delete_country()
    {
        $this->authenticatedUser('admin');

        $country = Country::factory()->create();

        $response = $this->deleteJson("/api/countries/{$country->id}");

        $response->assertStatus(204);
        $this->assertDatabaseMissing('countries', ['id' => $country->id]);
    }

    public function test_unauthenticated_user_cannot_access_countries()
    {
        $response = $this->getJson('/api/countries');

        $response->assertStatus(401);
    }
}
```

### **Feature Test para Authentication**

```bash
php artisan make:test Feature/AuthenticationApiTest
```

```php
// tests/Feature/AuthenticationApiTest.php
<?php

namespace Tests\Feature;

use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Support\Facades\Hash;
use Tests\TestCase;

class AuthenticationApiTest extends TestCase
{
    use RefreshDatabase;

    public function test_user_can_register()
    {
        $userData = [
            'name' => 'Test User',
            'email' => 'test@example.com',
            'password' => 'password123',
            'password_confirmation' => 'password123',
            'role' => 'viewer'
        ];

        $response = $this->postJson('/api/auth/register', $userData);

        $response->assertStatus(201)
                ->assertJsonStructure([
                    'message',
                    'user' => ['id', 'name', 'email', 'role'],
                    'token',
                    'permissions'
                ]);

        $this->assertDatabaseHas('users', [
            'email' => 'test@example.com',
            'role' => 'viewer'
        ]);
    }

    public function test_registration_validation()
    {
        $response = $this->postJson('/api/auth/register', []);

        $response->assertStatus(422)
                ->assertJsonValidationErrors(['name', 'email', 'password']);
    }

    public function test_user_can_login()
    {
        $user = User::factory()->create([
            'email' => 'test@example.com',
            'password' => Hash::make('password123'),
            'role' => 'admin'
        ]);

        $response = $this->postJson('/api/auth/login', [
            'email' => 'test@example.com',
            'password' => 'password123'
        ]);

        $response->assertStatus(200)
                ->assertJsonStructure([
                    'message',
                    'user' => ['id', 'name', 'email', 'role'],
                    'token',
                    'permissions'
                ])
                ->assertJson([
                    'user' => [
                        'id' => $user->id,
                        'email' => 'test@example.com',
                        'role' => 'admin'
                    ]
                ]);
    }

    public function test_login_with_invalid_credentials()
    {
        $user = User::factory()->create([
            'email' => 'test@example.com',
            'password' => Hash::make('password123')
        ]);

        $response = $this->postJson('/api/auth/login', [
            'email' => 'test@example.com',
            'password' => 'wrongpassword'
        ]);

        $response->assertStatus(422)
                ->assertJsonValidationErrors(['email']);
    }

    public function test_authenticated_user_can_get_profile()
    {
        $user = User::factory()->create(['role' => 'organizer']);

        $response = $this->actingAs($user, 'sanctum')
                        ->getJson('/api/auth/me');

        $response->assertStatus(200)
                ->assertJson([
                    'user' => [
                        'id' => $user->id,
                        'email' => $user->email,
                        'role' => 'organizer'
                    ]
                ]);
    }

    public function test_user_can_logout()
    {
        $user = User::factory()->create();
        $token = $user->createToken('test-token')->plainTextToken;

        $response = $this->withHeaders([
            'Authorization' => 'Bearer ' . $token,
        ])->postJson('/api/auth/logout');

        $response->assertStatus(200)
                ->assertJson(['message' => 'Logged out successfully']);

        // Verificar que el token fue revocado
        $this->assertDatabaseMissing('personal_access_tokens', [
            'tokenable_id' => $user->id,
            'tokenable_type' => User::class
        ]);
    }

    public function test_user_can_change_password()
    {
        $user = User::factory()->create([
            'password' => Hash::make('oldpassword')
        ]);

        $response = $this->actingAs($user, 'sanctum')
                        ->postJson('/api/auth/change-password', [
                            'current_password' => 'oldpassword',
                            'new_password' => 'newpassword123',
                            'new_password_confirmation' => 'newpassword123'
                        ]);

        $response->assertStatus(200);

        // Verificar que la contraseña cambió
        $user->refresh();
        $this->assertTrue(Hash::check('newpassword123', $user->password));
    }

    public function test_role_middleware_protects_admin_routes()
    {
        $viewer = User::factory()->viewer()->create();
        $admin = User::factory()->admin()->create();

        // Viewer no puede acceder
        $response = $this->actingAs($viewer, 'sanctum')
                        ->getJson('/api/admin/users');
        $response->assertStatus(403);

        // Admin sí puede acceder
        $response = $this->actingAs($admin, 'sanctum')
                        ->getJson('/api/admin/users');
        $response->assertStatus(200);
    }
}
```

---

## ⏱️ **Ejercicio 5: Test Suite Completa (10 minutos)**

### **Ejecutar y Verificar Tests**

```bash
# Ejecutar todos los tests
php artisan test

# Ejecutar solo Unit tests
php artisan test tests/Unit

# Ejecutar solo Feature tests
php artisan test tests/Feature

# Ejecutar con coverage (requiere Xdebug)
php artisan test --coverage

# Ejecutar tests específicos
php artisan test --filter CountryTest
php artisan test --filter AuthenticationApiTest
```

### **Test de Integración Completa**

```bash
php artisan make:test Feature/TournamentIntegrationTest
```

```php
// tests/Feature/TournamentIntegrationTest.php
<?php

namespace Tests\Feature;

use App\Models\Country;
use App\Models\Team;
use App\Models\Player;
use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Tests\TestCase;

class TournamentIntegrationTest extends TestCase
{
    use RefreshDatabase;

    public function test_complete_tournament_setup_flow()
    {
        // 1. Admin se autentica
        $admin = User::factory()->admin()->create();
        $this->actingAs($admin, 'sanctum');

        // 2. Crear países
        $argentina = Country::factory()->create([
            'name' => 'Argentina',
            'iso_code' => 'ARG'
        ]);

        $brasil = Country::factory()->create([
            'name' => 'Brasil',
            'iso_code' => 'BRA'
        ]);

        // 3. Crear equipos
        $teamArg = Team::factory()->create([
            'name' => 'Argentina Women',
            'country_id' => $argentina->id
        ]);

        $teamBra = Team::factory()->create([
            'name' => 'Brasil Women',
            'country_id' => $brasil->id
        ]);

        // 4. Crear jugadoras
        Player::factory()->count(11)->create(['team_id' => $teamArg->id]);
        Player::factory()->count(11)->create(['team_id' => $teamBra->id]);

        // 5. Verificar estructura completa
        $response = $this->getJson('/api/tournament-overview');

        $response->assertStatus(200)
                ->assertJsonStructure([
                    'countries' => [
                        '*' => [
                            'id', 'name', 'iso_code',
                            'team' => [
                                'id', 'name',
                                'players' => [
                                    '*' => ['id', 'name', 'position']
                                ]
                            ]
                        ]
                    ]
                ]);

        // 6. Verificar datos en base de datos
        $this->assertDatabaseCount('countries', 2);
        $this->assertDatabaseCount('teams', 2);
        $this->assertDatabaseCount('players', 22);

        // 7. Verify relationships
        $this->assertEquals(11, $teamArg->players->count());
        $this->assertEquals(11, $teamBra->players->count());
        $this->assertTrue($teamArg->country->is($argentina));
        $this->assertTrue($teamBra->country->is($brasil));
    }

    public function test_viewer_can_only_read_public_data()
    {
        $viewer = User::factory()->viewer()->create();
        $this->actingAs($viewer, 'sanctum');

        // Setup data
        $country = Country::factory()->create();
        $team = Team::factory()->create(['country_id' => $country->id]);

        // Can read
        $this->getJson('/api/countries')->assertStatus(200);
        $this->getJson('/api/teams')->assertStatus(200);

        // Cannot create
        $this->postJson('/api/countries', [])->assertStatus(403);
        $this->postJson('/api/teams', [])->assertStatus(403);

        // Cannot update
        $this->putJson("/api/countries/{$country->id}", [])->assertStatus(403);

        // Cannot delete
        $this->deleteJson("/api/countries/{$country->id}")->assertStatus(403);
    }
}
```

---

## 🎯 **Rúbrica de Evaluación**

| Criterio          | Excelente (4)                        | Bueno (3)                     | Satisfactorio (2)       | Insuficiente (1)      |
| ----------------- | ------------------------------------ | ----------------------------- | ----------------------- | --------------------- |
| **Factories**     | Datos realistas, estados, relaciones | Factories básicas funcionando | Datos simples           | Factories incompletas |
| **Unit Tests**    | Models, scopes, métodos, relaciones  | Tests básicos de modelos      | Algunos tests unitarios | Tests mínimos         |
| **Feature Tests** | CRUD completo, auth, roles           | Endpoints principales         | Algunos endpoints       | Tests básicos         |
| **Coverage**      | > 80% cobertura crítica              | > 60% cobertura               | > 40% cobertura         | < 40% cobertura       |
| **Integration**   | Flujos completos end-to-end          | Integración básica            | Algunos flujos          | Sin integración       |
| **Tiempo**        | < 15 minutos                         | 15-20 min                     | 20-30 min               | > 30 min              |

---

## 🔧 **Comandos de Testing Útiles**

```bash
# Tests rápidos (sin coverage)
php artisan test --parallel

# Tests con output detallado
php artisan test --verbose

# Tests que fallan primero
php artisan test --stop-on-failure

# Tests específicos por nombre
php artisan test --filter "can_create_country"

# Re-ejecutar solo tests que fallaron
php artisan test --retry

# Coverage en HTML (requiere Xdebug)
php artisan test --coverage-html coverage-report
```

---

## 💡 **Tips de Competencia**

### **Estrategia Rápida:**

1. **Setup básico** (2 min): .env.testing + phpunit.xml
2. **Factories principales** (5 min): Country, Team, Player, User
3. **Feature tests críticos** (5 min): Auth + CRUD principal
4. **Unit tests importantes** (3 min): Modelos con lógica de negocio

### **Prioridades por Tiempo:**

- **< 10 min:** Auth tests + 1 CRUD completo
- **< 15 min:** + Unit tests principales + Factories
- **< 20 min:** + Integration tests + Coverage > 80%

---

**🔥 Con esta test suite, los competidores garantizan código confiable y obtienen puntos extra por calidad!**
