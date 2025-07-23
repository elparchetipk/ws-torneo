# 🔒 Autenticación con Laravel Sanctum - Día 2.2

**Objetivo:** Implementar autenticación API segura en menos de 20 minutos

---

## 🎯 **Objetivo de la Sesión**

Los competidores deben poder implementar un sistema completo de **autenticación API** con Laravel Sanctum que incluya registro, login, logout, middleware de protección y roles básicos, todo en **menos de 20 minutos**.

---

## 🛡️ **Arquitectura de Autenticación API**

### **Flujo de Autenticación Completo**

```
🔐 Sistema de Autenticación WorldSkills

Registration Flow:
Client → POST /api/register → Validation → User Creation → Token Generation → Response

Login Flow:
Client → POST /api/login → Credentials Check → Token Generation → User Data + Token

Protected Requests:
Client → GET /api/protected → Bearer Token → Middleware → User Verification → Resource

Logout Flow:
Client → POST /api/logout → Token Verification → Token Revocation → Success Response

Roles & Permissions:
- admin: CRUD completo en todos los recursos
- organizer: Gestión de partidos y resultados
- viewer: Solo lectura de datos públicos
```

---

## ⏱️ **Ejercicio 1: Setup Sanctum (10 minutos)**

### **Paso 1: Instalación y Configuración (5 minutos)**

```bash
# CRONÓMETRO: INICIAR ⏰

# 1. Instalar Sanctum
composer require laravel/sanctum

# 2. Publish the Sanctum configuration
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"

# 3. Ejecutar migraciones
php artisan migrate

# 4. Agregar Sanctum middleware
```

#### **Configurar Sanctum en app/Http/Kernel.php**

```php
// app/Http/Kernel.php
protected $middlewareGroups = [
    'api' => [
        \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
        'throttle:api',
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],
];
```

#### **Configurar config/sanctum.php**

```php
// config/sanctum.php
'stateful' => explode(',', env('SANCTUM_STATEFUL_DOMAINS', sprintf(
    '%s%s',
    'localhost,localhost:3000,localhost:8000,127.0.0.1,127.0.0.1:8000,::1',
    Sanctum::currentApplicationUrlWithPort()
))),

'expiration' => env('SANCTUM_TOKEN_EXPIRATION', 60 * 24), // 24 horas
```

### **Paso 2: Modelo User con Sanctum (5 minutos)**

#### **Actualizar User Model**

```php
// app/Models/User.php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use Laravel\Sanctum\HasApiTokens;

class User extends Authenticatable
{
    use HasApiTokens, HasFactory, Notifiable;

    protected $fillable = [
        'name',
        'email',
        'password',
        'role',
    ];

    protected $hidden = [
        'password',
        'remember_token',
    ];

    protected $casts = [
        'email_verified_at' => 'datetime',
        'password' => 'hashed',
    ];

    /**
     * Roles disponibles en el sistema
     */
    const ROLES = [
        'admin' => 'Administrator',
        'organizer' => 'Tournament Organizer',
        'viewer' => 'Viewer'
    ];

    /**
     * Verificar si el usuario tiene un rol específico
     */
    public function hasRole($role)
    {
        return $this->role === $role;
    }

    /**
     * Verificar si es administrador
     */
    public function isAdmin()
    {
        return $this->hasRole('admin');
    }

    /**
     * Verificar si es organizador
     */
    public function isOrganizer()
    {
        return $this->hasRole('organizer');
    }

    /**
     * Obtener permisos del usuario
     */
    public function getPermissions()
    {
        return match($this->role) {
            'admin' => ['*'], // Todos los permisos
            'organizer' => ['matches.*', 'results.*', 'statistics.*'],
            'viewer' => ['*.read'],
            default => []
        };
    }

    /**
     * Scope para filtrar por rol
     */
    public function scopeByRole($query, $role)
    {
        return $query->where('role', $role);
    }
}
```

#### **Migración para agregar role a users**

```bash
php artisan make:migration add_role_to_users_table
```

```php
// database/migrations/xxxx_add_role_to_users_table.php
<?php

use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration
{
    public function up()
    {
        Schema::table('users', function (Blueprint $table) {
            $table->enum('role', ['admin', 'organizer', 'viewer'])
                  ->default('viewer')
                  ->after('email');
        });
    }

    public function down()
    {
        Schema::table('users', function (Blueprint $table) {
            $table->dropColumn('role');
        });
    }
};
```

---

## ⏱️ **Ejercicio 2: Controllers de Autenticación (15 minutos)**

### **Paso 1: AuthController (10 minutos)**

```bash
php artisan make:controller AuthController
```

```php
// app/Http/Controllers/AuthController.php
<?php

namespace App\Http\Controllers;

use App\Models\User;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;
use Illuminate\Support\Facades\Hash;
use Illuminate\Validation\ValidationException;

class AuthController extends Controller
{
    /**
     * Registro de nuevo usuario
     */
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
            'password' => Hash::make($request->password),
            'role' => $request->role ?? 'viewer',
        ]);

        // Generar token con permisos específicos
        $token = $user->createToken('auth-token', $user->getPermissions())->plainTextToken;

        return response()->json([
            'message' => 'User registered successfully',
            'user' => $user->only(['id', 'name', 'email', 'role']),
            'token' => $token,
            'permissions' => $user->getPermissions(),
        ], 201);
    }

    /**
     * Login de usuario
     */
    public function login(Request $request)
    {
        $request->validate([
            'email' => 'required|email',
            'password' => 'required',
        ]);

        $user = User::where('email', $request->email)->first();

        if (!$user || !Hash::check($request->password, $user->password)) {
            throw ValidationException::withMessages([
                'email' => ['The provided credentials are incorrect.'],
            ]);
        }

        // Revocar tokens anteriores (opcional)
        $user->tokens()->delete();

        // Generar nuevo token
        $token = $user->createToken('auth-token', $user->getPermissions())->plainTextToken;

        return response()->json([
            'message' => 'Login successful',
            'user' => $user->only(['id', 'name', 'email', 'role']),
            'token' => $token,
            'permissions' => $user->getPermissions(),
        ]);
    }

    /**
     * Obtener usuario autenticado
     */
    public function me(Request $request)
    {
        $user = $request->user();

        return response()->json([
            'user' => $user->only(['id', 'name', 'email', 'role']),
            'permissions' => $user->getPermissions(),
            'token_name' => $request->user()->currentAccessToken()->name,
        ]);
    }

    /**
     * Logout de usuario
     */
    public function logout(Request $request)
    {
        // Revocar el token actual
        $request->user()->currentAccessToken()->delete();

        return response()->json([
            'message' => 'Logged out successfully'
        ]);
    }

    /**
     * Logout de todos los dispositivos
     */
    public function logoutAll(Request $request)
    {
        // Revocar todos los tokens del usuario
        $request->user()->tokens()->delete();

        return response()->json([
            'message' => 'Logged out from all devices successfully'
        ]);
    }

    /**
     * Cambiar contraseña
     */
    public function changePassword(Request $request)
    {
        $request->validate([
            'current_password' => 'required',
            'new_password' => 'required|string|min:8|confirmed',
        ]);

        $user = $request->user();

        if (!Hash::check($request->current_password, $user->password)) {
            throw ValidationException::withMessages([
                'current_password' => ['Current password is incorrect.'],
            ]);
        }

        $user->update([
            'password' => Hash::make($request->new_password)
        ]);

        // Revocar todos los tokens para forzar re-login
        $user->tokens()->delete();

        return response()->json([
            'message' => 'Password changed successfully. Please login again.'
        ]);
    }
}
```

### **Paso 2: Middleware de Roles (5 minutos)**

```bash
php artisan make:middleware CheckRole
```

```php
// app/Http/Middleware/CheckRole.php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;

class CheckRole
{
    public function handle(Request $request, Closure $next, ...$roles)
    {
        $user = $request->user();

        if (!$user) {
            return response()->json(['message' => 'Unauthenticated'], 401);
        }

        // Si el usuario es admin, tiene acceso a todo
        if ($user->isAdmin()) {
            return $next($request);
        }

        // Verificar si el usuario tiene alguno de los roles requeridos
        if (!empty($roles) && !in_array($user->role, $roles)) {
            return response()->json([
                'message' => 'Access denied. Required roles: ' . implode(', ', $roles),
                'user_role' => $user->role
            ], 403);
        }

        return $next($request);
    }
}
```

#### **Registrar Middleware en Kernel.php**

```php
// app/Http/Kernel.php
protected $middlewareAliases = [
    // ... otros middlewares
    'role' => \App\Http\Middleware\CheckRole::class,
];
```

---

## ⏱️ **Ejercicio 3: Rutas Protegidas (10 minutos)**

### **Configurar rutas de autenticación**

```php
// routes/api.php
<?php

use App\Http\Controllers\AuthController;
use Illuminate\Support\Facades\Route;

/*
|--------------------------------------------------------------------------
| Authentication Routes (Public)
|--------------------------------------------------------------------------
*/

Route::prefix('auth')->group(function () {
    Route::post('/register', [AuthController::class, 'register']);
    Route::post('/login', [AuthController::class, 'login']);
});

/*
|--------------------------------------------------------------------------
| Protected Routes (Require Authentication)
|--------------------------------------------------------------------------
*/

Route::middleware('auth:sanctum')->group(function () {

    // Auth management routes
    Route::prefix('auth')->group(function () {
        Route::get('/me', [AuthController::class, 'me']);
        Route::post('/logout', [AuthController::class, 'logout']);
        Route::post('/logout-all', [AuthController::class, 'logoutAll']);
        Route::post('/change-password', [AuthController::class, 'changePassword']);
    });

    // Admin-only routes
    Route::middleware('role:admin')->group(function () {
        Route::get('/admin/users', function () {
            return response()->json(User::all());
        });

        Route::delete('/admin/users/{user}', function (User $user) {
            $user->delete();
            return response()->json(['message' => 'User deleted successfully']);
        });
    });

    // Organizer routes (can manage matches and results)
    Route::middleware('role:admin,organizer')->group(function () {
        Route::apiResource('matches', MatchController::class);
        Route::apiResource('results', ResultController::class);
        Route::post('/matches/{match}/result', 'MatchController@storeResult');
    });

    // Viewer routes (read-only access)
    Route::middleware('role:admin,organizer,viewer')->group(function () {
        Route::get('/countries', 'CountryController@index');
        Route::get('/teams', 'TeamController@index');
        Route::get('/players', 'PlayerController@index');
        Route::get('/matches', 'MatchController@index');
        Route::get('/standings', 'TournamentController@standings');
        Route::get('/statistics', 'StatisticController@index');
    });
});

/*
|--------------------------------------------------------------------------
| Public Routes (No Authentication Required)
|--------------------------------------------------------------------------
*/

Route::get('/public/standings', 'TournamentController@publicStandings');
Route::get('/public/matches/today', 'MatchController@todayMatches');
Route::get('/public/top-scorers', 'PlayerController@topScorers');
```

---

## ⏱️ **Ejercicio 4: Testing de Autenticación (15 minutos)**

### **Crear Tests Básicos**

```bash
php artisan make:test AuthenticationTest
```

```php
// tests/Feature/AuthenticationTest.php
<?php

namespace Tests\Feature;

use App\Models\User;
use Illuminate\Foundation\Testing\RefreshDatabase;
use Illuminate\Support\Facades\Hash;
use Tests\TestCase;

class AuthenticationTest extends TestCase
{
    use RefreshDatabase;

    public function test_user_can_register()
    {
        $response = $this->postJson('/api/auth/register', [
            'name' => 'Test User',
            'email' => 'test@example.com',
            'password' => 'password123',
            'password_confirmation' => 'password123',
            'role' => 'viewer'
        ]);

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
                ]);
    }

    public function test_user_cannot_login_with_invalid_credentials()
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

    public function test_authenticated_user_can_access_protected_route()
    {
        $user = User::factory()->create(['role' => 'viewer']);

        $response = $this->actingAs($user, 'sanctum')
                        ->getJson('/api/auth/me');

        $response->assertStatus(200)
                ->assertJson([
                    'user' => [
                        'id' => $user->id,
                        'email' => $user->email,
                        'role' => 'viewer'
                    ]
                ]);
    }

    public function test_unauthenticated_user_cannot_access_protected_route()
    {
        $response = $this->getJson('/api/auth/me');

        $response->assertStatus(401);
    }

    public function test_admin_can_access_admin_routes()
    {
        $admin = User::factory()->create(['role' => 'admin']);

        $response = $this->actingAs($admin, 'sanctum')
                        ->getJson('/api/admin/users');

        $response->assertStatus(200);
    }

    public function test_non_admin_cannot_access_admin_routes()
    {
        $user = User::factory()->create(['role' => 'viewer']);

        $response = $this->actingAs($user, 'sanctum')
                        ->getJson('/api/admin/users');

        $response->assertStatus(403);
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
}
```

### **Ejecutar Tests**

```bash
# Ejecutar tests de autenticación
php artisan test --filter AuthenticationTest

# Ejecutar todos los tests
php artisan test
```

---

## ⏱️ **Ejercicio 5: Frontend Integration (15 minutos)**

### **React Hook para Autenticación**

```jsx
// src/hooks/useAuth.js
import { useState, useEffect, createContext, useContext } from 'react';
import authService from '../services/authService';

const AuthContext = createContext();

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
};

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [token, setToken] = useState(localStorage.getItem('token'));
  const [loading, setLoading] = useState(true);

  // Configurar token en axios
  useEffect(() => {
    if (token) {
      authService.setAuthToken(token);
      // Verificar si el token es válido
      loadUser();
    } else {
      setLoading(false);
    }
  }, [token]);

  const loadUser = async () => {
    try {
      const response = await authService.getMe();
      setUser(response.user);
    } catch (error) {
      // Token inválido
      logout();
    } finally {
      setLoading(false);
    }
  };

  const login = async (credentials) => {
    try {
      const response = await authService.login(credentials);
      setUser(response.user);
      setToken(response.token);
      localStorage.setItem('token', response.token);
      return response;
    } catch (error) {
      throw error;
    }
  };

  const register = async (userData) => {
    try {
      const response = await authService.register(userData);
      setUser(response.user);
      setToken(response.token);
      localStorage.setItem('token', response.token);
      return response;
    } catch (error) {
      throw error;
    }
  };

  const logout = async () => {
    try {
      if (token) {
        await authService.logout();
      }
    } catch (error) {
      console.error('Logout error:', error);
    } finally {
      setUser(null);
      setToken(null);
      localStorage.removeItem('token');
      authService.removeAuthToken();
    }
  };

  const hasRole = (role) => {
    return user?.role === role;
  };

  const isAdmin = () => hasRole('admin');
  const isOrganizer = () => hasRole('organizer');
  const canManageMatches = () => isAdmin() || isOrganizer();

  const value = {
    user,
    token,
    loading,
    login,
    register,
    logout,
    hasRole,
    isAdmin,
    isOrganizer,
    canManageMatches,
    isAuthenticated: !!user,
  };

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
};
```

### **Servicio de Autenticación**

```js
// src/services/authService.js
import apiService from './api';

class AuthService {
  setAuthToken(token) {
    if (token) {
      apiService.defaults.headers.common['Authorization'] = `Bearer ${token}`;
    }
  }

  removeAuthToken() {
    delete apiService.defaults.headers.common['Authorization'];
  }

  async register(userData) {
    const response = await apiService.post('/auth/register', userData);
    return response.data;
  }

  async login(credentials) {
    const response = await apiService.post('/auth/login', credentials);
    return response.data;
  }

  async logout() {
    const response = await apiService.post('/auth/logout');
    return response.data;
  }

  async getMe() {
    const response = await apiService.get('/auth/me');
    return response.data;
  }

  async changePassword(passwordData) {
    const response = await apiService.post(
      '/auth/change-password',
      passwordData
    );
    return response.data;
  }
}

export default new AuthService();
```

### **Componente de Login**

```jsx
// src/components/LoginForm.jsx
import React, { useState } from 'react';
import { useAuth } from '../hooks/useAuth';

const LoginForm = () => {
  const [credentials, setCredentials] = useState({
    email: '',
    password: '',
  });
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState('');

  const { login } = useAuth();

  const handleSubmit = async (e) => {
    e.preventDefault();
    setLoading(true);
    setError('');

    try {
      await login(credentials);
      // Redirect será manejado por el componente padre
    } catch (error) {
      setError(error.response?.data?.message || 'Login failed');
    } finally {
      setLoading(false);
    }
  };

  return (
    <div className="max-w-md mx-auto bg-white rounded-lg shadow-md p-6">
      <h2 className="text-2xl font-bold text-center mb-6">Iniciar Sesión</h2>

      {error && (
        <div className="bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded mb-4">
          {error}
        </div>
      )}

      <form
        onSubmit={handleSubmit}
        className="space-y-4">
        <div>
          <label className="block text-sm font-medium text-gray-700 mb-2">
            Email
          </label>
          <input
            type="email"
            value={credentials.email}
            onChange={(e) =>
              setCredentials((prev) => ({
                ...prev,
                email: e.target.value,
              }))
            }
            className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
            required
          />
        </div>

        <div>
          <label className="block text-sm font-medium text-gray-700 mb-2">
            Contraseña
          </label>
          <input
            type="password"
            value={credentials.password}
            onChange={(e) =>
              setCredentials((prev) => ({
                ...prev,
                password: e.target.value,
              }))
            }
            className="w-full px-3 py-2 border border-gray-300 rounded-md focus:outline-none focus:ring-2 focus:ring-blue-500"
            required
          />
        </div>

        <button
          type="submit"
          disabled={loading}
          className="w-full bg-blue-500 text-white py-2 px-4 rounded-md hover:bg-blue-600 disabled:opacity-50 disabled:cursor-not-allowed">
          {loading ? 'Iniciando sesión...' : 'Iniciar Sesión'}
        </button>
      </form>
    </div>
  );
};

export default LoginForm;
```

---

## 🎯 **Rúbrica de Evaluación**

| Criterio                 | Excelente (4)                              | Bueno (3)                     | Satisfactorio (2)     | Insuficiente (1)         |
| ------------------------ | ------------------------------------------ | ----------------------------- | --------------------- | ------------------------ |
| **Setup Sanctum**        | Configuración completa y optimizada        | Setup correcto básico         | Setup funcional       | Errores en configuración |
| **Auth Controllers**     | Todos los endpoints + validaciones + roles | Endpoints básicos funcionando | Login/register básico | Solo estructura básica   |
| **Middleware & Roles**   | Sistema de roles completo con permisos     | Middleware básico funcionando | Protección básica     | No implementado          |
| **Testing**              | Tests completos con múltiples casos        | Tests básicos funcionando     | Algunos tests         | Sin tests                |
| **Frontend Integration** | Hook completo + contexto + componentes     | Integración básica            | Conexión simple       | No integrado             |
| **Tiempo**               | < 20 minutos                               | 20-25 min                     | 25-35 min             | > 35 min                 |

---

## 🔧 **Troubleshooting Común**

### **Error: Unauthenticated**

```php
// Verificar que el token se esté enviando correctamente
// Headers: Authorization: Bearer {token}

// En Postman/Insomnia:
Authorization: Bearer 1|laravel_sanctum_token_here
```

### **Error: Token Mismatch**

```php
// Verificar configuración de CORS y dominios
// config/sanctum.php
'stateful' => explode(',', env('SANCTUM_STATEFUL_DOMAINS', 'localhost:3000')),
```

### **Error: 419 CSRF Token Mismatch**

```php
// Para APIs, asegurar que la ruta esté en routes/api.php
// Y que se use 'auth:sanctum' no 'auth:web'
```

---

## 💡 **Tips de Competencia**

### **Velocidad de Implementación:**

1. **Templates preparados** para AuthController
2. **Snippets** para middleware y rutas
3. **Tests básicos** como checklist de funcionalidad

### **Puntos Críticos:**

- ✅ **Validaciones** en registro y login
- ✅ **Roles y permisos** bien definidos
- ✅ **Tokens** con expiración apropiada
- ✅ **Middleware** en rutas correctas

---

**🔥 Con este sistema de autenticación, los competidores tendrán APIs seguras y profesionales en tiempo récord!**
