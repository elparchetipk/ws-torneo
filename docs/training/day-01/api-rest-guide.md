# 🔗 Guía de APIs RESTful con Laravel - Día 1.3

**Objetivo:** Crear endpoints REST completos con manejo de errores robusto

---

## 🎯 **Fundamentos REST en 15 minutos**

### **Principios REST Esenciales**

#### **1. Recursos y URIs**

```
Recurso: Posts
GET    /api/posts      # Listar todos
GET    /api/posts/1    # Obtener específico
POST   /api/posts      # Crear nuevo
PUT    /api/posts/1    # Actualizar completo
PATCH  /api/posts/1    # Actualizar parcial
DELETE /api/posts/1    # Eliminar
```

#### **2. Códigos de Estado HTTP**

```
200 OK              # Éxito general
201 Created         # Recurso creado
204 No Content      # Éxito sin contenido
400 Bad Request     # Error del cliente
401 Unauthorized    # No autenticado
403 Forbidden       # No autorizado
404 Not Found       # Recurso no existe
422 Unprocessable   # Errores de validación
500 Internal Error  # Error del servidor
```

#### **3. Formato de Respuesta Consistente**

```json
{
  "data": { /* contenido */ },
  "message": "Operación exitosa",
  "status": 200
}

// Para errores
{
  "message": "Error de validación",
  "errors": {
    "title": ["El título es obligatorio"]
  },
  "status": 422
}
```

---

## ⚡ **Implementación Práctica: Resource Controller**

### **Paso 1: Crear Controller Base (5 minutos)**

```bash
# Crear controlador con todos los métodos REST
php artisan make:controller PostController --resource --api

# Crear Form Requests para validación
php artisan make:request StorePostRequest
php artisan make:request UpdatePostRequest

# Crear API Resource para respuestas
php artisan make:resource PostResource
```

### **Paso 2: Configurar Rutas API (2 minutos)**

```php
// routes/api.php
Route::apiResource('posts', PostController::class);

// Esto genera automáticamente:
// GET    /api/posts           → index()
// POST   /api/posts           → store()
// GET    /api/posts/{post}    → show()
// PUT    /api/posts/{post}    → update()
// DELETE /api/posts/{post}    → destroy()
```

### **Paso 3: Implementar Controller Completo (18 minutos)**

```php
// app/Http/Controllers/PostController.php
<?php

namespace App\Http\Controllers;

use App\Models\Post;
use App\Http\Requests\StorePostRequest;
use App\Http\Requests\UpdatePostRequest;
use App\Http\Resources\PostResource;
use Illuminate\Http\Request;

class PostController extends Controller
{
    /**
     * GET /api/posts
     * Listar todos los posts con paginación
     */
    public function index(Request $request)
    {
        $posts = Post::with(['user', 'comments'])
            ->when($request->published, function ($query) {
                $query->published();
            })
            ->when($request->user_id, function ($query, $userId) {
                $query->byUser($userId);
            })
            ->latest()
            ->paginate(15);

        return PostResource::collection($posts);
    }

    /**
     * POST /api/posts
     * Crear nuevo post
     */
    public function store(StorePostRequest $request)
    {
        $post = Post::create([
            'user_id' => $request->user_id,
            'title' => $request->title,
            'slug' => str()->slug($request->title),
            'content' => $request->content,
            'published' => $request->published ?? false,
            'published_at' => $request->published ? now() : null,
        ]);

        $post->load(['user', 'comments']);

        return response()->json([
            'data' => new PostResource($post),
            'message' => 'Post creado exitosamente',
            'status' => 201
        ], 201);
    }

    /**
     * GET /api/posts/{post}
     * Mostrar post específico
     */
    public function show(Post $post)
    {
        $post->load(['user', 'approvedComments.user']);

        return response()->json([
            'data' => new PostResource($post),
            'status' => 200
        ]);
    }

    /**
     * PUT /api/posts/{post}
     * Actualizar post completo
     */
    public function update(UpdatePostRequest $request, Post $post)
    {
        $post->update([
            'title' => $request->title,
            'slug' => str()->slug($request->title),
            'content' => $request->content,
            'published' => $request->published ?? $post->published,
            'published_at' => $request->published && !$post->published ? now() : $post->published_at,
        ]);

        $post->load(['user', 'comments']);

        return response()->json([
            'data' => new PostResource($post),
            'message' => 'Post actualizado exitosamente',
            'status' => 200
        ]);
    }

    /**
     * DELETE /api/posts/{post}
     * Eliminar post
     */
    public function destroy(Post $post)
    {
        $post->delete();

        return response()->json([
            'message' => 'Post eliminado exitosamente',
            'status' => 204
        ], 204);
    }
}
```

---

## ✅ **Validaciones con Form Requests**

### **StorePostRequest**

```php
// app/Http/Requests/StorePostRequest.php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class StorePostRequest extends FormRequest
{
    public function authorize()
    {
        return true; // En competencia, siempre true
    }

    public function rules()
    {
        return [
            'user_id' => 'required|exists:users,id',
            'title' => 'required|string|max:255|unique:posts,title',
            'content' => 'required|string|min:10',
            'published' => 'boolean',
        ];
    }

    public function messages()
    {
        return [
            'user_id.required' => 'El usuario es obligatorio',
            'user_id.exists' => 'El usuario no existe',
            'title.required' => 'El título es obligatorio',
            'title.unique' => 'Ya existe un post con este título',
            'content.required' => 'El contenido es obligatorio',
            'content.min' => 'El contenido debe tener al menos 10 caracteres',
        ];
    }
}
```

### **UpdatePostRequest**

```php
// app/Http/Requests/UpdatePostRequest.php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rule;

class UpdatePostRequest extends FormRequest
{
    public function authorize()
    {
        return true;
    }

    public function rules()
    {
        return [
            'title' => [
                'required',
                'string',
                'max:255',
                Rule::unique('posts')->ignore($this->post)
            ],
            'content' => 'required|string|min:10',
            'published' => 'boolean',
        ];
    }

    public function messages()
    {
        return [
            'title.required' => 'El título es obligatorio',
            'title.unique' => 'Ya existe otro post con este título',
            'content.required' => 'El contenido es obligatorio',
            'content.min' => 'El contenido debe tener al menos 10 caracteres',
        ];
    }
}
```

---

## 📦 **API Resources para Respuestas Estructuradas**

### **PostResource**

```php
// app/Http/Resources/PostResource.php
<?php

namespace App\Http\Resources;

use Illuminate\Http\Resources\Json\JsonResource;

class PostResource extends JsonResource
{
    public function toArray($request)
    {
        return [
            'id' => $this->id,
            'title' => $this->title,
            'slug' => $this->slug,
            'content' => $this->content,
            'published' => $this->published,
            'published_at' => $this->published_at?->format('Y-m-d H:i:s'),
            'created_at' => $this->created_at->format('Y-m-d H:i:s'),
            'updated_at' => $this->updated_at->format('Y-m-d H:i:s'),

            // Relaciones condicionales
            'user' => $this->whenLoaded('user', function () {
                return [
                    'id' => $this->user->id,
                    'name' => $this->user->name,
                    'email' => $this->user->email,
                ];
            }),

            'comments' => $this->whenLoaded('comments', function () {
                return $this->comments->map(function ($comment) {
                    return [
                        'id' => $comment->id,
                        'content' => $comment->content,
                        'approved' => $comment->approved,
                        'created_at' => $comment->created_at->format('Y-m-d H:i:s'),
                        'user' => $comment->whenLoaded('user', [
                            'id' => $comment->user->id,
                            'name' => $comment->user->name,
                        ]),
                    ];
                });
            }),

            'comments_count' => $this->whenCounted('comments'),
        ];
    }
}
```

---

## 🧪 **Testing Manual con Postman**

### **Collection: Blog API**

#### **1. GET - Listar Posts**

```
GET http://localhost:8000/api/posts
Accept: application/json

Query Params (opcionales):
- published=1
- user_id=1
- page=1
```

#### **2. POST - Crear Post**

```
POST http://localhost:8000/api/posts
Content-Type: application/json
Accept: application/json

{
    "user_id": 1,
    "title": "Mi nuevo post desde API",
    "content": "Contenido del post creado via API REST",
    "published": true
}
```

#### **3. GET - Mostrar Post Específico**

```
GET http://localhost:8000/api/posts/1
Accept: application/json
```

#### **4. PUT - Actualizar Post**

```
PUT http://localhost:8000/api/posts/1
Content-Type: application/json
Accept: application/json

{
    "title": "Título actualizado",
    "content": "Contenido actualizado del post",
    "published": false
}
```

#### **5. DELETE - Eliminar Post**

```
DELETE http://localhost:8000/api/posts/1
Accept: application/json
```

---

## 🔧 **Manejo de Errores Avanzado**

### **Handler Global de Errores**

```php
// app/Exceptions/Handler.php
public function render($request, Throwable $exception)
{
    if ($request->expectsJson()) {

        // Errores de validación (422)
        if ($exception instanceof ValidationException) {
            return response()->json([
                'message' => 'Error de validación',
                'errors' => $exception->errors(),
                'status' => 422
            ], 422);
        }

        // Modelo no encontrado (404)
        if ($exception instanceof ModelNotFoundException) {
            return response()->json([
                'message' => 'Recurso no encontrado',
                'status' => 404
            ], 404);
        }

        // Error de autorización (403)
        if ($exception instanceof AuthorizationException) {
            return response()->json([
                'message' => 'No autorizado para esta acción',
                'status' => 403
            ], 403);
        }

        // Error genérico del servidor (500)
        return response()->json([
            'message' => 'Error interno del servidor',
            'status' => 500
        ], 500);
    }

    return parent::render($request, $exception);
}
```

---

## 🎯 **Ejercicio Práctico: CRUD Completo (30 minutos)**

### **Objetivo:** Implementar CRUD de Comments

```bash
# ⏰ CRONÓMETRO: 30 MINUTOS
php artisan make:controller CommentController --resource --api
php artisan make:request StoreCommentRequest
php artisan make:request UpdateCommentRequest
php artisan make:resource CommentResource

# Implementar:
# 1. Todas las rutas API
# 2. Métodos del controlador
# 3. Validaciones específicas
# 4. Resource con relaciones
# 5. Testing en Postman
```

### **Retos Adicionales:**

- **Filtros:** Comentarios por post, por usuario, solo aprobados
- **Paginación:** 10 comentarios por página
- **Ordenamiento:** Por fecha, por aprobación
- **Búsqueda:** Por contenido del comentario

---

## 📊 **Rúbrica de Evaluación**

| Aspecto            | Excelente (4)           | Bueno (3)     | Satisfactorio (2) | Insuficiente (1) |
| ------------------ | ----------------------- | ------------- | ----------------- | ---------------- |
| **Endpoints**      | Todos funcionando       | La mayoría    | Básicos           | Con errores      |
| **Validaciones**   | Completas + mensajes    | Completas     | Básicas           | Faltantes        |
| **Responses**      | API Resources + códigos | Estructuradas | JSON básico       | Inconsistentes   |
| **Manejo Errores** | Global + específico     | Básico        | Mínimo            | Sin manejo       |

---

## 🚀 **Patrones para Competencia**

### **Template de Controller**

```php
class EntityController extends Controller
{
    public function index(Request $request)
    {
        $entities = Entity::with($request->include ?? [])
            ->when($request->filter, function($q, $filter) {
                // Aplicar filtros dinámicos
            })
            ->paginate($request->per_page ?? 15);

        return EntityResource::collection($entities);
    }

    public function store(StoreEntityRequest $request)
    {
        $entity = Entity::create($request->validated());
        return new EntityResource($entity);
    }

    // ... resto de métodos siguiendo el mismo patrón
}
```

### **Template de Request**

```php
class StoreEntityRequest extends FormRequest
{
    public function authorize() { return true; }

    public function rules()
    {
        return [
            'required_field' => 'required|string|max:255',
            'unique_field' => 'required|unique:table,field',
            'foreign_key' => 'required|exists:other_table,id',
        ];
    }

    public function messages()
    {
        return [
            'required_field.required' => 'El campo es obligatorio',
            // Mensajes en español siempre
        ];
    }
}
```

---

**🎯 Meta:** Al final de esta sesión, crear una API REST completa debe ser un proceso automático de < 20 minutos por entidad.
