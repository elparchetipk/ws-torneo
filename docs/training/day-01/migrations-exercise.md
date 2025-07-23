# 🗄️ Ejercicio de Migraciones y Modelos Laravel - Día 1.2

**Objetivo:** Dominar BD complejas con relaciones múltiples en tiempo récord

---

## 🎯 **Objetivo de la Sesión**

Crear estructuras de base de datos complejas con relaciones múltiples, traduciendo rápidamente de diagramas conceptuales a migraciones Laravel eficientes.

---

## 📚 **Ejercicio Principal: Sistema de Blog**

### **Contexto del Ejercicio**

Implementaremos un sistema de blog con **usuarios**, **posts** y **comentarios** que servirá como práctica para el dominio deportivo más complejo.

### **Diagrama de Entidades**

```
User (usuarios)
├─ id (PK)
├─ name
├─ email (unique)
├─ password
└─ created_at, updated_at

Post (artículos)
├─ id (PK)
├─ user_id (FK → users.id)
├─ title
├─ slug (unique)
├─ content (text)
├─ published (boolean)
├─ published_at
└─ created_at, updated_at

Comment (comentarios)
├─ id (PK)
├─ post_id (FK → posts.id)
├─ user_id (FK → users.id)
├─ content (text)
├─ approved (boolean)
└─ created_at, updated_at
```

---

## ⏱️ **Ejercicio Cronometrado (40 minutos)**

### **Fase 1: Creación de Modelos y Migraciones (15 minutos)**

```bash
# ⏰ CRONÓMETRO: INICIAR
cd torneo-api

# Crear modelos con migraciones, controladores y resources
php artisan make:model User -mcr
php artisan make:model Post -mcr
php artisan make:model Comment -mcr

# ✅ Debe completarse en 2 minutos
```

### **Fase 2: Diseño de Migraciones (15 minutos)**

#### **Migración Users (ya existe, solo modificar si necesario)**

```php
// database/migrations/2024_01_01_000000_create_users_table.php
public function up()
{
    Schema::create('users', function (Blueprint $table) {
        $table->id();
        $table->string('name');
        $table->string('email')->unique();
        $table->timestamp('email_verified_at')->nullable();
        $table->string('password');
        $table->rememberToken();
        $table->timestamps();
    });
}
```

#### **Migración Posts**

```php
// database/migrations/2024_01_02_000000_create_posts_table.php
public function up()
{
    Schema::create('posts', function (Blueprint $table) {
        $table->id();
        $table->foreignId('user_id')->constrained()->onDelete('cascade');
        $table->string('title');
        $table->string('slug')->unique();
        $table->text('content');
        $table->boolean('published')->default(false);
        $table->timestamp('published_at')->nullable();
        $table->timestamps();

        // Índices para performance
        $table->index(['published', 'published_at']);
        $table->index('user_id');
    });
}
```

#### **Migración Comments**

```php
// database/migrations/2024_01_03_000000_create_comments_table.php
public function up()
{
    Schema::create('comments', function (Blueprint $table) {
        $table->id();
        $table->foreignId('post_id')->constrained()->onDelete('cascade');
        $table->foreignId('user_id')->constrained()->onDelete('cascade');
        $table->text('content');
        $table->boolean('approved')->default(false);
        $table->timestamps();

        // Índices compuestos
        $table->index(['post_id', 'approved']);
        $table->index('user_id');
    });
}
```

### **Fase 3: Configuración de Modelos (10 minutos)**

#### **Modelo User**

```php
// app/Models/User.php
class User extends Authenticatable
{
    protected $fillable = [
        'name', 'email', 'password',
    ];

    protected $hidden = [
        'password', 'remember_token',
    ];

    // Relaciones
    public function posts()
    {
        return $this->hasMany(Post::class);
    }

    public function comments()
    {
        return $this->hasMany(Comment::class);
    }

    public function publishedPosts()
    {
        return $this->hasMany(Post::class)->where('published', true);
    }
}
```

#### **Modelo Post**

```php
// app/Models/Post.php
class Post extends Model
{
    protected $fillable = [
        'user_id', 'title', 'slug', 'content', 'published', 'published_at'
    ];

    protected $casts = [
        'published' => 'boolean',
        'published_at' => 'datetime',
    ];

    // Relaciones
    public function user()
    {
        return $this->belongsTo(User::class);
    }

    public function comments()
    {
        return $this->hasMany(Comment::class);
    }

    public function approvedComments()
    {
        return $this->hasMany(Comment::class)->where('approved', true);
    }

    // Scopes útiles
    public function scopePublished($query)
    {
        return $query->where('published', true);
    }

    public function scopeByUser($query, $userId)
    {
        return $query->where('user_id', $userId);
    }
}
```

#### **Modelo Comment**

```php
// app/Models/Comment.php
class Comment extends Model
{
    protected $fillable = [
        'post_id', 'user_id', 'content', 'approved'
    ];

    protected $casts = [
        'approved' => 'boolean',
    ];

    // Relaciones
    public function post()
    {
        return $this->belongsTo(Post::class);
    }

    public function user()
    {
        return $this->belongsTo(User::class);
    }

    // Scopes
    public function scopeApproved($query)
    {
        return $query->where('approved', true);
    }
}
```

---

## 🧪 **Testing de Relaciones**

### **Seeder para Pruebas**

```php
// database/seeders/BlogSeeder.php
class BlogSeeder extends Seeder
{
    public function run()
    {
        // Crear usuarios
        $user1 = User::create([
            'name' => 'Juan Pérez',
            'email' => 'juan@example.com',
            'password' => bcrypt('password123')
        ]);

        $user2 = User::create([
            'name' => 'María García',
            'email' => 'maria@example.com',
            'password' => bcrypt('password123')
        ]);

        // Crear posts
        $post1 = Post::create([
            'user_id' => $user1->id,
            'title' => 'Mi primer artículo',
            'slug' => 'mi-primer-articulo',
            'content' => 'Contenido del primer artículo...',
            'published' => true,
            'published_at' => now()
        ]);

        $post2 = Post::create([
            'user_id' => $user2->id,
            'title' => 'Laravel y bases de datos',
            'slug' => 'laravel-y-bases-de-datos',
            'content' => 'Tutorial sobre migraciones...',
            'published' => true,
            'published_at' => now()
        ]);

        // Crear comentarios
        Comment::create([
            'post_id' => $post1->id,
            'user_id' => $user2->id,
            'content' => '¡Excelente artículo!',
            'approved' => true
        ]);

        Comment::create([
            'post_id' => $post2->id,
            'user_id' => $user1->id,
            'content' => 'Muy útil, gracias por compartir',
            'approved' => true
        ]);
    }
}
```

### **Comandos de Testing**

```bash
# Ejecutar migraciones y seeder
php artisan migrate:fresh --seed --seeder=BlogSeeder

# Probar relaciones en Tinker
php artisan tinker

# En Tinker:
$user = User::first();
$user->posts; // Posts del usuario
$user->publishedPosts; // Solo posts publicados

$post = Post::first();
$post->user; // Autor del post
$post->comments; // Comentarios del post
$post->approvedComments; // Solo comentarios aprobados

$comment = Comment::first();
$comment->post; // Post del comentario
$comment->user; // Autor del comentario
```

---

## 🔍 **Ejercicio Práctico: Consultas Complejas**

### **Queries a Implementar (10 minutos)**

```php
// 1. Posts con sus autores y cantidad de comentarios
Post::with(['user', 'comments'])
    ->withCount('comments')
    ->published()
    ->get();

// 2. Usuarios con más de 2 posts publicados
User::has('publishedPosts', '>', 2)
    ->withCount('publishedPosts')
    ->get();

// 3. Posts más comentados (top 5)
Post::withCount('approvedComments')
    ->orderBy('approved_comments_count', 'desc')
    ->take(5)
    ->get();

// 4. Comentarios recientes de un post específico
Comment::where('post_id', 1)
    ->approved()
    ->with('user')
    ->latest()
    ->get();

// 5. Posts de usuario con comentarios aprobados
Post::where('user_id', 1)
    ->with(['approvedComments.user'])
    ->published()
    ->get();
```

---

## 🎯 **Rúbrica de Evaluación**

| Aspecto         | Excelente (4)          | Bueno (3)  | Satisfactorio (2) | Insuficiente (1) |
| --------------- | ---------------------- | ---------- | ----------------- | ---------------- |
| **Velocidad**   | < 35 min               | 35-40 min  | 40-45 min         | > 45 min         |
| **Migraciones** | Perfectas, con índices | Correctas  | Funcionales       | Con errores      |
| **Relaciones**  | Todas configuradas     | La mayoría | Básicas           | Incorrectas      |
| **Consultas**   | Optimizadas            | Correctas  | Funcionales       | No funcionan     |

---

## 🚀 **Práctica Cronometrada: Recrear desde Cero**

### **Desafío (20 minutos):**

1. **Borrar todo:** `rm -rf app/Models/* database/migrations/*`
2. **Recrear completo:** Modelos, migraciones, relaciones, seeder
3. **Probar funcionamiento:** Todas las consultas deben funcionar

### **Meta de Tiempo:**

- **< 15 minutos:** ⭐⭐⭐⭐ Velocidad ninja
- **15-20 minutos:** ⭐⭐⭐ Muy bueno
- **20-25 minutos:** ⭐⭐ Satisfactorio
- **> 25 minutos:** ⭐ Necesita más práctica

---

## 🔧 **Errores Comunes y Soluciones**

### **Error 1: Foreign key constraint**

```bash
# Problema: Orden incorrecto de migraciones
# Solución: Renombrar archivos con timestamps correctos
```

### **Error 2: Class not found**

```bash
# Problema: Modelo no existe o mal namespace
# Solución:
composer dump-autoload
php artisan optimize:clear
```

### **Error 3: Column not found**

```bash
# Problema: Campo no existe en migración
# Solución: Verificar nombres exactos en Schema
```

---

## 📈 **Progresión hacia Dominio Deportivo**

### **Paralelismo con Sistema Deportivo:**

- **User → Country:** Entidad base
- **Post → Team:** Entidad principal con FK
- **Comment → Player:** Entidad dependiente con doble FK
- **published → active:** Estados booleanos
- **Consultas complejas → Estadísticas deportivas**

### **Próximo Paso:**

Con esta base, en el Día 2 implementaremos:

- **Countries, Teams, Players** (mismo patrón)
- **Matches con relaciones complejas**
- **Estadísticas automáticas**

---

**🎯 Meta del ejercicio:** Al final, crear migraciones complejas debe ser tan automático como escribir código básico.
