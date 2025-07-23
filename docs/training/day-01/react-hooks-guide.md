# ⚛️ Guía de React Hooks y Consumo de APIs - Día 1.4

**Objetivo:** Dominar patrones modernos de React para consumo eficiente de APIs

---

## 🎯 **Hooks Esenciales para APIs (30 minutos)**

### **useState para Estado Local**

```jsx
// Patrón básico para datos de API
const [data, setData] = useState(null);
const [loading, setLoading] = useState(true);
const [error, setError] = useState(null);

// Patrón para formularios
const [formData, setFormData] = useState({
  title: '',
  content: '',
  published: false,
});

// Patrón para listas con filtros
const [posts, setPosts] = useState([]);
const [filters, setFilters] = useState({
  published: null,
  user_id: null,
  search: '',
});
```

### **useEffect para Ciclo de Vida**

```jsx
// Cargar datos al montar componente
useEffect(() => {
  const fetchPosts = async () => {
    try {
      setLoading(true);
      const response = await fetch('/api/posts');
      const result = await response.json();
      setPosts(result.data);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  fetchPosts();
}, []); // Array vacío = solo al montar

// Recargar cuando cambien filtros
useEffect(() => {
  fetchPostsWithFilters();
}, [filters]); // Se ejecuta cuando filters cambia

// Cleanup para evitar memory leaks
useEffect(() => {
  const controller = new AbortController();

  fetch('/api/posts', { signal: controller.signal })
    .then((response) => response.json())
    .then((data) => setPosts(data));

  return () => controller.abort(); // Cleanup
}, []);
```

---

## 🔧 **Custom Hook para APIs**

### **useApi Hook Genérico**

```jsx
// hooks/useApi.js
import { useState, useEffect } from 'react';

export const useApi = (url, options = {}) => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  const fetchData = async () => {
    try {
      setLoading(true);
      setError(null);

      const response = await fetch(url, {
        headers: {
          Accept: 'application/json',
          'Content-Type': 'application/json',
          ...options.headers,
        },
        ...options,
      });

      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      const result = await response.json();
      setData(result.data || result);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  useEffect(() => {
    if (url) {
      fetchData();
    }
  }, [url]);

  const refetch = () => fetchData();

  return { data, loading, error, refetch };
};
```

### **usePosts Hook Específico**

```jsx
// hooks/usePosts.js
import { useState, useEffect } from 'react';

export const usePosts = (filters = {}) => {
  const [posts, setPosts] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const [pagination, setPagination] = useState(null);

  const fetchPosts = async () => {
    try {
      setLoading(true);
      setError(null);

      const params = new URLSearchParams();
      Object.entries(filters).forEach(([key, value]) => {
        if (value !== null && value !== '') {
          params.append(key, value);
        }
      });

      const response = await fetch(`/api/posts?${params}`);
      const result = await response.json();

      setPosts(result.data);
      setPagination(result.meta || null);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  const createPost = async (postData) => {
    try {
      const response = await fetch('/api/posts', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          Accept: 'application/json',
        },
        body: JSON.stringify(postData),
      });

      if (!response.ok) {
        const errorData = await response.json();
        throw new Error(errorData.message || 'Error al crear post');
      }

      const result = await response.json();
      setPosts((prev) => [result.data, ...prev]);
      return result.data;
    } catch (err) {
      setError(err.message);
      throw err;
    }
  };

  const updatePost = async (id, postData) => {
    try {
      const response = await fetch(`/api/posts/${id}`, {
        method: 'PUT',
        headers: {
          'Content-Type': 'application/json',
          Accept: 'application/json',
        },
        body: JSON.stringify(postData),
      });

      if (!response.ok) {
        const errorData = await response.json();
        throw new Error(errorData.message || 'Error al actualizar post');
      }

      const result = await response.json();
      setPosts((prev) =>
        prev.map((post) => (post.id === id ? result.data : post))
      );
      return result.data;
    } catch (err) {
      setError(err.message);
      throw err;
    }
  };

  const deletePost = async (id) => {
    try {
      const response = await fetch(`/api/posts/${id}`, {
        method: 'DELETE',
        headers: { Accept: 'application/json' },
      });

      if (!response.ok) {
        throw new Error('Error al eliminar post');
      }

      setPosts((prev) => prev.filter((post) => post.id !== id));
    } catch (err) {
      setError(err.message);
      throw err;
    }
  };

  useEffect(() => {
    fetchPosts();
  }, [JSON.stringify(filters)]);

  return {
    posts,
    loading,
    error,
    pagination,
    refetch: fetchPosts,
    createPost,
    updatePost,
    deletePost,
  };
};
```

---

## 📋 **Componentes con Hooks**

### **Lista de Posts con Estados**

```jsx
// components/PostsList.jsx
import React, { useState } from 'react';
import { usePosts } from '../hooks/usePosts';
import LoadingSpinner from './LoadingSpinner';
import ErrorMessage from './ErrorMessage';

const PostsList = () => {
  const [filters, setFilters] = useState({
    published: '',
    user_id: '',
    search: '',
  });

  const { posts, loading, error, refetch } = usePosts(filters);

  const handleFilterChange = (key, value) => {
    setFilters((prev) => ({
      ...prev,
      [key]: value,
    }));
  };

  if (loading) return <LoadingSpinner />;
  if (error)
    return (
      <ErrorMessage
        message={error}
        onRetry={refetch}
      />
    );

  return (
    <div className="posts-list">
      {/* Filtros */}
      <div className="filters mb-6">
        <div className="flex gap-4">
          <select
            value={filters.published}
            onChange={(e) => handleFilterChange('published', e.target.value)}
            className="border rounded px-3 py-2">
            <option value="">Todos</option>
            <option value="1">Publicados</option>
            <option value="0">Borradores</option>
          </select>

          <input
            type="text"
            placeholder="Buscar..."
            value={filters.search}
            onChange={(e) => handleFilterChange('search', e.target.value)}
            className="border rounded px-3 py-2 flex-1"
          />

          <button
            onClick={refetch}
            className="bg-blue-500 text-white px-4 py-2 rounded">
            Recargar
          </button>
        </div>
      </div>

      {/* Lista */}
      <div className="posts-grid">
        {posts.length === 0 ? (
          <p className="text-gray-500">No se encontraron posts</p>
        ) : (
          posts.map((post) => (
            <PostCard
              key={post.id}
              post={post}
            />
          ))
        )}
      </div>
    </div>
  );
};

export default PostsList;
```

### **Componente de Post Individual**

```jsx
// components/PostCard.jsx
import React, { useState } from 'react';

const PostCard = ({ post, onDelete, onEdit }) => {
  const [isDeleting, setIsDeleting] = useState(false);

  const handleDelete = async () => {
    if (window.confirm('¿Estás seguro de eliminar este post?')) {
      try {
        setIsDeleting(true);
        await onDelete(post.id);
      } catch (error) {
        alert('Error al eliminar: ' + error.message);
      } finally {
        setIsDeleting(false);
      }
    }
  };

  return (
    <div className="post-card border rounded-lg p-6 shadow-sm">
      <div className="flex justify-between items-start mb-4">
        <h3 className="text-xl font-semibold">{post.title}</h3>
        <span
          className={`px-2 py-1 rounded text-sm ${
            post.published
              ? 'bg-green-100 text-green-800'
              : 'bg-yellow-100 text-yellow-800'
          }`}>
          {post.published ? 'Publicado' : 'Borrador'}
        </span>
      </div>

      <p className="text-gray-600 mb-4">{post.content.substring(0, 150)}...</p>

      <div className="flex justify-between items-center text-sm text-gray-500">
        <div>
          <p>Por: {post.user?.name}</p>
          <p>{new Date(post.created_at).toLocaleDateString()}</p>
        </div>

        <div className="flex gap-2">
          <button
            onClick={() => onEdit(post)}
            className="text-blue-600 hover:underline">
            Editar
          </button>
          <button
            onClick={handleDelete}
            disabled={isDeleting}
            className="text-red-600 hover:underline disabled:opacity-50">
            {isDeleting ? 'Eliminando...' : 'Eliminar'}
          </button>
        </div>
      </div>
    </div>
  );
};

export default PostCard;
```

---

## 📝 **Formularios con Hooks**

### **Formulario de Creación/Edición**

```jsx
// components/PostForm.jsx
import React, { useState, useEffect } from 'react';

const PostForm = ({ initialData = null, onSubmit, onCancel }) => {
  const [formData, setFormData] = useState({
    title: '',
    content: '',
    published: false,
    user_id: 1, // En competencia, valor fijo
  });

  const [errors, setErrors] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  // Cargar datos iniciales si estamos editando
  useEffect(() => {
    if (initialData) {
      setFormData({
        title: initialData.title || '',
        content: initialData.content || '',
        published: initialData.published || false,
        user_id: initialData.user_id || 1,
      });
    }
  }, [initialData]);

  const handleChange = (e) => {
    const { name, value, type, checked } = e.target;
    setFormData((prev) => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value,
    }));

    // Limpiar error del campo al escribir
    if (errors[name]) {
      setErrors((prev) => ({
        ...prev,
        [name]: null,
      }));
    }
  };

  const handleSubmit = async (e) => {
    e.preventDefault();

    try {
      setIsSubmitting(true);
      setErrors({});

      await onSubmit(formData);

      // Limpiar formulario si es creación
      if (!initialData) {
        setFormData({
          title: '',
          content: '',
          published: false,
          user_id: 1,
        });
      }
    } catch (error) {
      // Manejar errores de validación
      if (error.response?.status === 422) {
        setErrors(error.response.data.errors);
      } else {
        alert('Error: ' + error.message);
      }
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <form
      onSubmit={handleSubmit}
      className="space-y-4">
      <div>
        <label className="block text-sm font-medium mb-1">Título</label>
        <input
          type="text"
          name="title"
          value={formData.title}
          onChange={handleChange}
          className={`w-full border rounded px-3 py-2 ${
            errors.title ? 'border-red-500' : 'border-gray-300'
          }`}
          required
        />
        {errors.title && (
          <p className="text-red-500 text-sm mt-1">{errors.title[0]}</p>
        )}
      </div>

      <div>
        <label className="block text-sm font-medium mb-1">Contenido</label>
        <textarea
          name="content"
          value={formData.content}
          onChange={handleChange}
          rows={6}
          className={`w-full border rounded px-3 py-2 ${
            errors.content ? 'border-red-500' : 'border-gray-300'
          }`}
          required
        />
        {errors.content && (
          <p className="text-red-500 text-sm mt-1">{errors.content[0]}</p>
        )}
      </div>

      <div>
        <label className="flex items-center">
          <input
            type="checkbox"
            name="published"
            checked={formData.published}
            onChange={handleChange}
            className="mr-2"
          />
          Publicar inmediatamente
        </label>
      </div>

      <div className="flex gap-4">
        <button
          type="submit"
          disabled={isSubmitting}
          className="bg-blue-500 text-white px-6 py-2 rounded disabled:opacity-50">
          {isSubmitting ? 'Guardando...' : initialData ? 'Actualizar' : 'Crear'}
        </button>

        <button
          type="button"
          onClick={onCancel}
          className="bg-gray-500 text-white px-6 py-2 rounded">
          Cancelar
        </button>
      </div>
    </form>
  );
};

export default PostForm;
```

---

## 🎯 **Componentes Utilitarios**

### **LoadingSpinner**

```jsx
// components/LoadingSpinner.jsx
import React from 'react';

const LoadingSpinner = ({ size = 'medium', message = 'Cargando...' }) => {
  const sizeClasses = {
    small: 'w-4 h-4',
    medium: 'w-8 h-8',
    large: 'w-12 h-12',
  };

  return (
    <div className="flex flex-col items-center justify-center p-8">
      <div
        className={`animate-spin rounded-full border-b-2 border-blue-500 ${sizeClasses[size]}`}></div>
      {message && <p className="mt-2 text-gray-600">{message}</p>}
    </div>
  );
};

export default LoadingSpinner;
```

### **ErrorMessage**

```jsx
// components/ErrorMessage.jsx
import React from 'react';

const ErrorMessage = ({ message, onRetry }) => {
  return (
    <div className="bg-red-50 border border-red-200 rounded-lg p-4">
      <div className="flex items-center">
        <div className="flex-shrink-0">
          <svg
            className="h-5 w-5 text-red-400"
            viewBox="0 0 20 20"
            fill="currentColor">
            <path
              fillRule="evenodd"
              d="M10 18a8 8 0 100-16 8 8 0 000 16zM8.707 7.293a1 1 0 00-1.414 1.414L8.586 10l-1.293 1.293a1 1 0 101.414 1.414L10 11.414l1.293 1.293a1 1 0 001.414-1.414L11.414 10l1.293-1.293a1 1 0 00-1.414-1.414L10 8.586 8.707 7.293z"
              clipRule="evenodd"
            />
          </svg>
        </div>
        <div className="ml-3 flex-1">
          <h3 className="text-sm font-medium text-red-800">
            Error al cargar los datos
          </h3>
          <p className="mt-1 text-sm text-red-700">{message}</p>
        </div>
        {onRetry && (
          <div className="ml-4">
            <button
              onClick={onRetry}
              className="bg-red-100 hover:bg-red-200 text-red-800 px-3 py-1 rounded text-sm">
              Reintentar
            </button>
          </div>
        )}
      </div>
    </div>
  );
};

export default ErrorMessage;
```

---

## 🧪 **Ejercicio Práctico (45 minutos)**

### **Objetivo:** Crear interfaz completa de Blog

**Tareas a completar:**

1. ✅ **Hook personalizado** - `usePosts` funcional
2. ✅ **Lista con filtros** - Posts filtrados en tiempo real
3. ✅ **Formulario CRUD** - Crear/editar posts
4. ✅ **Estados de carga** - Loading, error, success
5. ✅ **Manejo de errores** - Validaciones del servidor

### **Estructura objetivo:**

```
src/
├── hooks/
│   ├── useApi.js
│   └── usePosts.js
├── components/
│   ├── PostsList.jsx
│   ├── PostCard.jsx
│   ├── PostForm.jsx
│   ├── LoadingSpinner.jsx
│   └── ErrorMessage.jsx
└── App.jsx
```

---

## 📊 **Rúbrica de Evaluación**

| Aspecto            | Excelente (4)               | Bueno (3)         | Satisfactorio (2) | Insuficiente (1) |
| ------------------ | --------------------------- | ----------------- | ----------------- | ---------------- |
| **Custom Hooks**   | Reutilizables + optimizados | Funcionales       | Básicos           | Con errores      |
| **Estados**        | Loading + error + success   | Loading + error   | Solo loading      | Sin estados      |
| **Formularios**    | Validación + UX             | Validación básica | Solo envío        | Sin validación   |
| **Manejo Errores** | Global + específico         | Básico            | Mínimo            | Sin manejo       |

---

**🎯 Meta:** Desarrollar intuición para manejar estados asíncronos complejos y crear interfaces que manejen elegantemente los tiempos de carga.
