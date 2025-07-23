# 🔗 Guía de Integración Frontend-Backend - Día 1.5

**Objetivo:** Establecer comunicación fluida entre React y Laravel

---

## 🎯 **Configuración de Comunicación (20 minutos)**

### **Parte 1: Configurar Proxy en Vite (5 minutos)**

```js
// vite.config.js - Configuración completa
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  server: {
    port: 3000,
    host: true, // Permite acceso desde la red
    proxy: {
      '/api': {
        target: 'http://localhost:8000',
        changeOrigin: true,
        secure: false,
        configure: (proxy, _options) => {
          proxy.on('error', (err, _req, _res) => {
            console.log('proxy error', err);
          });
          proxy.on('proxyReq', (proxyReq, req, _res) => {
            console.log('Sending Request to the Target:', req.method, req.url);
          });
          proxy.on('proxyRes', (proxyRes, req, _res) => {
            console.log(
              'Received Response from the Target:',
              proxyRes.statusCode,
              req.url
            );
          });
        },
      },
    },
  },
  build: {
    outDir: 'dist',
    sourcemap: true,
  },
});
```

### **Parte 2: Configurar CORS en Laravel (10 minutos)**

```php
// config/cors.php - Configuración para desarrollo
<?php

return [
    'paths' => ['api/*', 'sanctum/csrf-cookie'],

    'allowed_methods' => ['*'],

    'allowed_origins' => [
        'http://localhost:3000',
        'http://127.0.0.1:3000',
        'http://localhost:3001', // Puerto alternativo
    ],

    'allowed_origins_patterns' => [],

    'allowed_headers' => ['*'],

    'exposed_headers' => [],

    'max_age' => 0,

    'supports_credentials' => false,
];
```

```php
// bootstrap/app.php - Registrar middleware CORS
<?php

use Illuminate\Foundation\Application;
use Illuminate\Foundation\Configuration\Exceptions;
use Illuminate\Foundation\Configuration\Middleware;

return Application::configure(basePath: dirname(__DIR__))
    ->withRouting(
        web: __DIR__.'/../routes/web.php',
        api: __DIR__.'/../routes/api.php',
        commands: __DIR__.'/../routes/console.php',
        health: '/up',
    )
    ->withMiddleware(function (Middleware $middleware) {
        $middleware->api(prepend: [
            \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
        ]);

        // Alias para simplificar
        $middleware->alias([
            'verified' => \App\Http\Middleware\EnsureEmailIsVerified::class,
        ]);
    })
    ->withExceptions(function (Exceptions $exceptions) {
        //
    })->create();
```

### **Parte 3: Test de Conectividad (5 minutos)**

```php
// routes/api.php - Rutas de prueba
<?php

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Route;

// Ruta de test simple
Route::get('/test', function () {
    return response()->json([
        'message' => 'API conectada correctamente!',
        'timestamp' => now()->toISOString(),
        'server' => 'Laravel ' . app()->version()
    ]);
});

// Test con datos de prueba
Route::get('/test-data', function () {
    return response()->json([
        'data' => [
            ['id' => 1, 'name' => 'Usuario Test 1'],
            ['id' => 2, 'name' => 'Usuario Test 2'],
            ['id' => 3, 'name' => 'Usuario Test 3'],
        ],
        'message' => 'Datos de prueba',
        'status' => 200
    ]);
});

// Test de POST
Route::post('/test-post', function (Request $request) {
    return response()->json([
        'received' => $request->all(),
        'message' => 'POST recibido correctamente',
        'status' => 201
    ], 201);
});
```

---

## 🧪 **Servicio de API Centralizado**

### **API Service Base**

```js
// src/services/api.js
class ApiService {
  constructor(baseURL = '/api') {
    this.baseURL = baseURL;
    this.defaultHeaders = {
      'Content-Type': 'application/json',
      Accept: 'application/json',
    };
  }

  async request(endpoint, options = {}) {
    const url = `${this.baseURL}${endpoint}`;

    const config = {
      headers: {
        ...this.defaultHeaders,
        ...options.headers,
      },
      ...options,
    };

    try {
      const response = await fetch(url, config);

      // Manejar respuestas no-JSON (como 204 No Content)
      if (response.status === 204) {
        return { success: true };
      }

      const data = await response.json();

      if (!response.ok) {
        throw new ApiError(
          data.message || 'Error en la API',
          response.status,
          data
        );
      }

      return data;
    } catch (error) {
      if (error instanceof ApiError) {
        throw error;
      }

      // Error de red o parsing
      throw new ApiError('Error de conexión con el servidor', 0, null);
    }
  }

  // Métodos HTTP
  async get(endpoint, params = {}) {
    const queryString = new URLSearchParams(params).toString();
    const url = queryString ? `${endpoint}?${queryString}` : endpoint;

    return this.request(url, {
      method: 'GET',
    });
  }

  async post(endpoint, data = {}) {
    return this.request(endpoint, {
      method: 'POST',
      body: JSON.stringify(data),
    });
  }

  async put(endpoint, data = {}) {
    return this.request(endpoint, {
      method: 'PUT',
      body: JSON.stringify(data),
    });
  }

  async patch(endpoint, data = {}) {
    return this.request(endpoint, {
      method: 'PATCH',
      body: JSON.stringify(data),
    });
  }

  async delete(endpoint) {
    return this.request(endpoint, {
      method: 'DELETE',
    });
  }
}

// Clase para errores de API
class ApiError extends Error {
  constructor(message, status, data) {
    super(message);
    this.name = 'ApiError';
    this.status = status;
    this.data = data;
  }
}

// Instancia global
const apiService = new ApiService();

export default apiService;
export { ApiError };
```

### **Servicio Específico para Posts**

```js
// src/services/postService.js
import apiService from './api';

class PostService {
  // Obtener todos los posts
  async getPosts(filters = {}) {
    return apiService.get('/posts', filters);
  }

  // Obtener post específico
  async getPost(id) {
    return apiService.get(`/posts/${id}`);
  }

  // Crear nuevo post
  async createPost(postData) {
    return apiService.post('/posts', postData);
  }

  // Actualizar post
  async updatePost(id, postData) {
    return apiService.put(`/posts/${id}`, postData);
  }

  // Eliminar post
  async deletePost(id) {
    return apiService.delete(`/posts/${id}`);
  }

  // Buscar posts
  async searchPosts(query) {
    return apiService.get('/posts', { search: query });
  }

  // Posts de un usuario
  async getPostsByUser(userId) {
    return apiService.get('/posts', { user_id: userId });
  }
}

const postService = new PostService();
export default postService;
```

---

## 📝 **Formulario Completo con Validación**

### **Componente de Formulario Avanzado**

```jsx
// src/components/PostFormIntegrated.jsx
import React, { useState, useEffect } from 'react';
import postService from '../services/postService';
import { ApiError } from '../services/api';

const PostFormIntegrated = ({ postId = null, onSuccess, onCancel }) => {
  const [formData, setFormData] = useState({
    title: '',
    content: '',
    published: false,
    user_id: 1,
  });

  const [errors, setErrors] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [isLoading, setIsLoading] = useState(false);

  // Cargar datos si estamos editando
  useEffect(() => {
    if (postId) {
      loadPost();
    }
  }, [postId]);

  const loadPost = async () => {
    try {
      setIsLoading(true);
      const response = await postService.getPost(postId);
      const post = response.data;

      setFormData({
        title: post.title,
        content: post.content,
        published: post.published,
        user_id: post.user_id,
      });
    } catch (error) {
      console.error('Error cargando post:', error);
      alert('Error al cargar el post: ' + error.message);
    } finally {
      setIsLoading(false);
    }
  };

  const handleChange = (e) => {
    const { name, value, type, checked } = e.target;

    setFormData((prev) => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value,
    }));

    // Limpiar error del campo
    if (errors[name]) {
      setErrors((prev) => ({
        ...prev,
        [name]: null,
      }));
    }
  };

  const validateForm = () => {
    const newErrors = {};

    if (!formData.title.trim()) {
      newErrors.title = 'El título es obligatorio';
    } else if (formData.title.length > 255) {
      newErrors.title = 'El título no puede superar 255 caracteres';
    }

    if (!formData.content.trim()) {
      newErrors.content = 'El contenido es obligatorio';
    } else if (formData.content.length < 10) {
      newErrors.content = 'El contenido debe tener al menos 10 caracteres';
    }

    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  const handleSubmit = async (e) => {
    e.preventDefault();

    // Validación del lado del cliente
    if (!validateForm()) {
      return;
    }

    try {
      setIsSubmitting(true);
      setErrors({});

      let response;
      if (postId) {
        response = await postService.updatePost(postId, formData);
      } else {
        response = await postService.createPost(formData);
      }

      // Notificar éxito al componente padre
      if (onSuccess) {
        onSuccess(response.data);
      }

      // Limpiar formulario si es creación
      if (!postId) {
        setFormData({
          title: '',
          content: '',
          published: false,
          user_id: 1,
        });
      }

      alert(
        postId ? 'Post actualizado exitosamente' : 'Post creado exitosamente'
      );
    } catch (error) {
      console.error('Error:', error);

      if (error instanceof ApiError && error.status === 422) {
        // Errores de validación del servidor
        setErrors(error.data.errors);
      } else {
        alert('Error: ' + error.message);
      }
    } finally {
      setIsSubmitting(false);
    }
  };

  if (isLoading) {
    return (
      <div className="flex justify-center items-center py-8">
        <div className="animate-spin rounded-full h-8 w-8 border-b-2 border-blue-500"></div>
        <span className="ml-2">Cargando post...</span>
      </div>
    );
  }

  return (
    <div className="max-w-2xl mx-auto bg-white p-6 rounded-lg shadow">
      <h2 className="text-2xl font-bold mb-6">
        {postId ? 'Editar Post' : 'Crear Nuevo Post'}
      </h2>

      <form
        onSubmit={handleSubmit}
        className="space-y-6">
        {/* Título */}
        <div>
          <label
            htmlFor="title"
            className="block text-sm font-medium text-gray-700 mb-2">
            Título *
          </label>
          <input
            type="text"
            id="title"
            name="title"
            value={formData.title}
            onChange={handleChange}
            className={`w-full px-3 py-2 border rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500 ${
              errors.title ? 'border-red-500' : 'border-gray-300'
            }`}
            placeholder="Ingresa el título del post"
            disabled={isSubmitting}
          />
          {errors.title && (
            <p className="mt-1 text-sm text-red-600">{errors.title}</p>
          )}
        </div>

        {/* Contenido */}
        <div>
          <label
            htmlFor="content"
            className="block text-sm font-medium text-gray-700 mb-2">
            Contenido *
          </label>
          <textarea
            id="content"
            name="content"
            value={formData.content}
            onChange={handleChange}
            rows={8}
            className={`w-full px-3 py-2 border rounded-md shadow-sm focus:outline-none focus:ring-2 focus:ring-blue-500 ${
              errors.content ? 'border-red-500' : 'border-gray-300'
            }`}
            placeholder="Escribe el contenido del post..."
            disabled={isSubmitting}
          />
          {errors.content && (
            <p className="mt-1 text-sm text-red-600">{errors.content}</p>
          )}
          <p className="mt-1 text-sm text-gray-500">
            {formData.content.length} caracteres (mínimo 10)
          </p>
        </div>

        {/* Publicado */}
        <div>
          <label className="flex items-center">
            <input
              type="checkbox"
              name="published"
              checked={formData.published}
              onChange={handleChange}
              className="rounded border-gray-300 text-blue-600 shadow-sm focus:border-blue-300 focus:ring focus:ring-blue-200 focus:ring-opacity-50"
              disabled={isSubmitting}
            />
            <span className="ml-2 text-sm text-gray-700">
              Publicar inmediatamente
            </span>
          </label>
        </div>

        {/* Botones */}
        <div className="flex justify-end space-x-4">
          <button
            type="button"
            onClick={onCancel}
            className="px-4 py-2 text-sm font-medium text-gray-700 bg-white border border-gray-300 rounded-md hover:bg-gray-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500"
            disabled={isSubmitting}>
            Cancelar
          </button>

          <button
            type="submit"
            disabled={isSubmitting}
            className="px-4 py-2 text-sm font-medium text-white bg-blue-600 border border-transparent rounded-md hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500 disabled:opacity-50 disabled:cursor-not-allowed">
            {isSubmitting ? (
              <>
                <svg
                  className="animate-spin -ml-1 mr-2 h-4 w-4 text-white inline"
                  fill="none"
                  viewBox="0 0 24 24">
                  <circle
                    className="opacity-25"
                    cx="12"
                    cy="12"
                    r="10"
                    stroke="currentColor"
                    strokeWidth="4"></circle>
                  <path
                    className="opacity-75"
                    fill="currentColor"
                    d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
                </svg>
                {postId ? 'Actualizando...' : 'Creando...'}
              </>
            ) : postId ? (
              'Actualizar Post'
            ) : (
              'Crear Post'
            )}
          </button>
        </div>
      </form>
    </div>
  );
};

export default PostFormIntegrated;
```

---

## 🧪 **Testing Manual con Herramientas**

### **Test en Browser DevTools**

```js
// Abrir Consola del navegador y probar:

// 1. Test básico de conectividad
fetch('/api/test')
  .then((r) => r.json())
  .then(console.log);

// 2. Test de datos
fetch('/api/test-data')
  .then((r) => r.json())
  .then(console.log);

// 3. Test POST
fetch('/api/test-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    Accept: 'application/json',
  },
  body: JSON.stringify({
    name: 'Test desde React',
    message: 'Hola Laravel!',
  }),
})
  .then((r) => r.json())
  .then(console.log);

// 4. Test del servicio de API
import apiService from './src/services/api.js';
apiService.get('/test').then(console.log);
```

### **Test con React Developer Tools**

1. **Instalar extensión:** React Developer Tools
2. **Inspeccionar componentes:** Ver props y state en tiempo real
3. **Profiler:** Medir performance de renders
4. **Hooks:** Inspeccionar custom hooks

---

## 🔧 **Troubleshooting de Integración**

### **Problema 1: CORS Error**

```
Error: Access to fetch at 'http://localhost:8000/api/posts'
from origin 'http://localhost:3000' has been blocked by CORS policy
```

**Solución:**

```bash
# Verificar que CORS esté configurado
php artisan config:cache
php artisan serve --host=0.0.0.0
```

### **Problema 2: Proxy no funciona**

```js
// Verificar vite.config.js
console.log('Proxy configurado para:', import.meta.env.DEV);

// Test directo sin proxy
fetch('http://localhost:8000/api/test')
  .then((r) => r.json())
  .then(console.log);
```

### **Problema 3: 500 Internal Server Error**

```bash
# Ver logs de Laravel
tail -f storage/logs/laravel.log

# Verificar rutas
php artisan route:list
```

### **Problema 4: Network Tab Debug**

1. Abrir DevTools → Network
2. Filtrar por Fetch/XHR
3. Inspeccionar Request/Response headers
4. Verificar códigos de estado

---

## 🎯 **Ejercicio Integral (55 minutos)**

### **Objetivo:** Blog completo funcionando

**Checklist de implementación:**

- [ ] ✅ Proxy Vite configurado
- [ ] ✅ CORS Laravel configurado
- [ ] ✅ API Service centralizado
- [ ] ✅ Post Service específico
- [ ] ✅ Formulario con validación dual
- [ ] ✅ Lista con operaciones CRUD
- [ ] ✅ Manejo de errores completo
- [ ] ✅ States de loading en toda la app

### **Demo final debe mostrar:**

1. **Crear post** desde React → Laravel
2. **Listar posts** con filtros en tiempo real
3. **Editar post** cargando datos del servidor
4. **Eliminar post** con confirmación
5. **Manejo de errores** de validación y red

---

## 📊 **Rúbrica de Evaluación**

| Aspecto          | Excelente (4)                 | Bueno (3)           | Satisfactorio (2) | Insuficiente (1) |
| ---------------- | ----------------------------- | ------------------- | ----------------- | ---------------- |
| **Comunicación** | Proxy + CORS perfectos        | Comunicación fluida | Conexión básica   | Con errores      |
| **Servicios**    | Centralizados + reutilizables | Organizados         | Funcionales       | Mezclados        |
| **Validación**   | Cliente + servidor            | Una validación      | Básica            | Sin validar      |
| **UX**           | Loading + errores + success   | Loading + errores   | Solo loading      | Sin feedback     |

---

**🎯 Meta:** Al final de esta sesión, la comunicación React ↔ Laravel debe ser tan fluida como trabajar con una API real en producción.
