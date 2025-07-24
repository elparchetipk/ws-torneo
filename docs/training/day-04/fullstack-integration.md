# 🔗 Integración Full-Stack React + Laravel

**Duración:** 1 hora (4:00 PM - 5:00 PM)  
**Objetivo:** Conectar el frontend React con el backend Laravel de manera seamless y robusta

---

## 🎯 **Objetivos de la Sesión**

Al finalizar esta sesión, el competidor será capaz de:

✅ **Configurar proxy de desarrollo** para evitar problemas de CORS  
✅ **Implementar autenticación** con Sanctum (si requerido)  
✅ **Manejar errores globalmente** en toda la aplicación  
✅ **Sincronizar validaciones** entre frontend y backend  
✅ **Optimizar requests** con técnicas de caching

---

## 🔧 **Paso 1: Configuración de Proxy (10 minutos)**

Configurar Vite para que funcione seamlessly con Laravel durante desarrollo.

### **📄 Actualizar `vite.config.js`:**

```javascript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  server: {
    port: 3000,
    host: true, // Permitir acceso desde red local
    proxy: {
      // Proxy todas las rutas API a Laravel
      '/api': {
        target: 'http://localhost:8000',
        changeOrigin: true,
        secure: false,
      },
      // Proxy para storage de Laravel (imágenes, archivos)
      '/storage': {
        target: 'http://localhost:8000',
        changeOrigin: true,
        secure: false,
      },
    },
  },
  build: {
    outDir: '../public/frontend',
    emptyOutDir: true,
    sourcemap: false, // Desactivar en producción para WorldSkills
  },
  // Configuración para desarrollo más rápido
  optimizeDeps: {
    include: ['react', 'react-dom', 'react-router-dom'],
  },
});
```

### **📄 Configurar CORS en Laravel `config/cors.php`:**

```php
<?php

return [
    'paths' => ['api/*', 'sanctum/csrf-cookie'],
    'allowed_methods' => ['*'],
    'allowed_origins' => [
        'http://localhost:3000',
        'http://127.0.0.1:3000',
    ],
    'allowed_origins_patterns' => [],
    'allowed_headers' => ['*'],
    'exposed_headers' => [],
    'max_age' => 0,
    'supports_credentials' => true,
];
```

---

## 🔒 **Paso 2: Autenticación con Sanctum (15 minutos)**

Implementar autenticación básica si es requerida en el proyecto.

### **📄 `src/services/auth.js`:**

```javascript
import { apiService } from './api';

export const authService = {
  // Obtener CSRF Cookie
  async getCsrfCookie() {
    return apiService.get('/sanctum/csrf-cookie');
  },

  // Login
  async login(credentials) {
    await this.getCsrfCookie();
    return apiService.post('/login', credentials);
  },

  // Logout
  async logout() {
    return apiService.post('/logout');
  },

  // Obtener usuario autenticado
  async me() {
    return apiService.get('/user');
  },

  // Verificar si hay token válido
  async checkAuth() {
    try {
      const response = await this.me();
      return response.data;
    } catch (error) {
      return null;
    }
  },
};
```

### **📄 `src/contexts/AuthContext.jsx`:**

```jsx
import { createContext, useContext, useReducer, useEffect } from 'react';
import { authService } from '../services/auth';

const AuthContext = createContext();

const authReducer = (state, action) => {
  switch (action.type) {
    case 'SET_LOADING':
      return { ...state, loading: action.payload };
    case 'SET_USER':
      return {
        ...state,
        user: action.payload,
        authenticated: !!action.payload,
      };
    case 'SET_ERROR':
      return { ...state, error: action.payload, loading: false };
    case 'LOGOUT':
      return { ...state, user: null, authenticated: false };
    default:
      return state;
  }
};

export const AuthProvider = ({ children }) => {
  const [state, dispatch] = useReducer(authReducer, {
    user: null,
    authenticated: false,
    loading: true,
    error: null,
  });

  // Verificar autenticación al cargar
  useEffect(() => {
    checkAuthStatus();
  }, []);

  const checkAuthStatus = async () => {
    try {
      const user = await authService.checkAuth();
      dispatch({ type: 'SET_USER', payload: user });
    } catch (error) {
      dispatch({ type: 'SET_USER', payload: null });
    } finally {
      dispatch({ type: 'SET_LOADING', payload: false });
    }
  };

  const login = async (credentials) => {
    dispatch({ type: 'SET_LOADING', payload: true });
    try {
      await authService.login(credentials);
      const user = await authService.me();
      dispatch({ type: 'SET_USER', payload: user.data });
      return user.data;
    } catch (error) {
      dispatch({ type: 'SET_ERROR', payload: error.message });
      throw error;
    }
  };

  const logout = async () => {
    try {
      await authService.logout();
    } catch (error) {
      console.error('Error durante logout:', error);
    } finally {
      dispatch({ type: 'LOGOUT' });
    }
  };

  return (
    <AuthContext.Provider
      value={{
        ...state,
        login,
        logout,
        checkAuthStatus,
      }}>
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth debe usarse dentro de AuthProvider');
  }
  return context;
};
```

---

## ⚠️ **Paso 3: Manejo Global de Errores (15 minutos)**

Sistema robusto para manejar errores de API y mostrar notificaciones apropiadas.

### **📄 `src/components/ui/Notification.jsx`:**

```jsx
import { Fragment } from 'react';
import { Transition } from '@headlessui/react';
import {
  CheckCircleIcon,
  ExclamationTriangleIcon,
  InformationCircleIcon,
  XCircleIcon,
  XMarkIcon,
} from '@heroicons/react/24/outline';
import { useApp } from '../../contexts/AppContext';

const iconMap = {
  success: CheckCircleIcon,
  error: XCircleIcon,
  warning: ExclamationTriangleIcon,
  info: InformationCircleIcon,
};

const colorMap = {
  success: 'text-green-400',
  error: 'text-red-400',
  warning: 'text-yellow-400',
  info: 'text-blue-400',
};

const bgColorMap = {
  success: 'bg-green-50',
  error: 'bg-red-50',
  warning: 'bg-yellow-50',
  info: 'bg-blue-50',
};

const NotificationContainer = () => {
  const { notifications, removeNotification } = useApp();

  return (
    <div className="fixed top-4 right-4 z-50 space-y-2">
      {notifications.map((notification) => {
        const Icon = iconMap[notification.type] || InformationCircleIcon;

        return (
          <Transition
            key={notification.id}
            show={true}
            as={Fragment}
            enter="transform ease-out duration-300 transition"
            enterFrom="translate-y-2 opacity-0 sm:translate-y-0 sm:translate-x-2"
            enterTo="translate-y-0 opacity-100 sm:translate-x-0"
            leave="transition ease-in duration-100"
            leaveFrom="opacity-100"
            leaveTo="opacity-0">
            <div
              className={`max-w-sm w-full ${
                bgColorMap[notification.type]
              } shadow-lg rounded-lg pointer-events-auto ring-1 ring-black ring-opacity-5 overflow-hidden`}>
              <div className="p-4">
                <div className="flex items-start">
                  <div className="flex-shrink-0">
                    <Icon
                      className={`h-6 w-6 ${colorMap[notification.type]}`}
                    />
                  </div>
                  <div className="ml-3 w-0 flex-1 pt-0.5">
                    {notification.title && (
                      <p className="text-sm font-medium text-gray-900">
                        {notification.title}
                      </p>
                    )}
                    <p className="text-sm text-gray-500">
                      {notification.message}
                    </p>
                  </div>
                  <div className="ml-4 flex-shrink-0 flex">
                    <button
                      className="bg-white rounded-md inline-flex text-gray-400 hover:text-gray-500 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-primary-500"
                      onClick={() => removeNotification(notification.id)}>
                      <XMarkIcon className="h-5 w-5" />
                    </button>
                  </div>
                </div>
              </div>
            </div>
          </Transition>
        );
      })}
    </div>
  );
};

export default NotificationContainer;
```

### **📄 Actualizar `src/services/api.js` con manejo mejorado:**

```javascript
import axios from 'axios';

// Configuración base de Axios
const api = axios.create({
  baseURL: '/api',
  headers: {
    'Content-Type': 'application/json',
    Accept: 'application/json',
  },
  withCredentials: true, // Para Sanctum
});

// Interceptor para requests (agregar tokens, etc.)
api.interceptors.request.use(
  (config) => {
    // Agregar token si existe
    const token = localStorage.getItem('auth_token');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }

    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);

// Interceptor para responses con manejo mejorado de errores
api.interceptors.response.use(
  (response) => response,
  (error) => {
    let errorMessage = 'Error de conexión';
    let errorDetails = {};

    if (error.response) {
      const { status, data } = error.response;

      switch (status) {
        case 400:
          errorMessage = 'Solicitud inválida';
          break;
        case 401:
          errorMessage = 'No autorizado';
          // Redirigir a login si es necesario
          break;
        case 403:
          errorMessage = 'Sin permisos para esta acción';
          break;
        case 404:
          errorMessage = 'Recurso no encontrado';
          break;
        case 422:
          errorMessage = 'Datos de validación incorrectos';
          errorDetails = data.errors || {};
          break;
        case 429:
          errorMessage = 'Demasiadas solicitudes. Intenta más tarde.';
          break;
        case 500:
          errorMessage = 'Error interno del servidor';
          break;
        default:
          errorMessage = data.message || `Error ${status}`;
      }

      if (data.message) {
        errorMessage = data.message;
      }
    } else if (error.request) {
      errorMessage = 'Sin respuesta del servidor';
    }

    // Crear error personalizado con más información
    const customError = new Error(errorMessage);
    customError.status = error.response?.status;
    customError.details = errorDetails;
    customError.original = error;

    return Promise.reject(customError);
  }
);

// Métodos con mejor manejo de errores
export const apiService = {
  get: (url, config = {}) => api.get(url, config),
  post: (url, data, config = {}) => api.post(url, data, config),
  put: (url, data, config = {}) => api.put(url, data, config),
  patch: (url, data, config = {}) => api.patch(url, data, config),
  delete: (url, config = {}) => api.delete(url, config),

  // Método para upload de archivos
  upload: (url, formData, onProgress) => {
    return api.post(url, formData, {
      headers: {
        'Content-Type': 'multipart/form-data',
      },
      onUploadProgress: (progressEvent) => {
        if (onProgress) {
          const percentCompleted = Math.round(
            (progressEvent.loaded * 100) / progressEvent.total
          );
          onProgress(percentCompleted);
        }
      },
    });
  },
};

export default api;
```

---

## 🔄 **Paso 4: Sincronización de Validaciones (10 minutos)**

Crear sistema para reutilizar reglas de validación entre frontend y backend.

### **📄 `src/utils/validators.js`:**

```javascript
// Reglas de validación que coinciden con Laravel
export const validationRules = {
  // Países
  country: {
    name: [
      { required: true, message: 'El nombre del país es requerido' },
      { minLength: 2, message: 'El nombre debe tener al menos 2 caracteres' },
      { maxLength: 100, message: 'El nombre no puede superar 100 caracteres' },
    ],
    code: [
      { required: true, message: 'El código del país es requerido' },
      {
        pattern: /^[A-Z]{2}$/,
        message: 'El código debe ser de 2 letras mayúsculas',
      },
    ],
  },

  // Equipos
  team: {
    name: [
      { required: true, message: 'El nombre del equipo es requerido' },
      { minLength: 2, message: 'El nombre debe tener al menos 2 caracteres' },
      { maxLength: 100, message: 'El nombre no puede superar 100 caracteres' },
    ],
    country_id: [{ required: true, message: 'Debe seleccionar un país' }],
    founded_year: [
      {
        validate: (value) => {
          if (!value) return true; // Opcional
          const year = parseInt(value);
          const currentYear = new Date().getFullYear();
          return year >= 1800 && year <= currentYear;
        },
        message: 'El año debe estar entre 1800 y el año actual',
      },
    ],
  },

  // Jugadoras
  player: {
    first_name: [
      { required: true, message: 'El nombre es requerido' },
      { minLength: 2, message: 'El nombre debe tener al menos 2 caracteres' },
      { maxLength: 50, message: 'El nombre no puede superar 50 caracteres' },
    ],
    last_name: [
      { required: true, message: 'El apellido es requerido' },
      { minLength: 2, message: 'El apellido debe tener al menos 2 caracteres' },
      { maxLength: 50, message: 'El apellido no puede superar 50 caracteres' },
    ],
    birth_date: [
      { required: true, message: 'La fecha de nacimiento es requerida' },
      {
        validate: (value) => {
          if (!value) return false;
          const birthDate = new Date(value);
          const today = new Date();
          const age = today.getFullYear() - birthDate.getFullYear();
          return age >= 16 && age <= 45;
        },
        message: 'La jugadora debe tener entre 16 y 45 años',
      },
    ],
    position: [{ required: true, message: 'La posición es requerida' }],
    jersey_number: [
      { required: true, message: 'El número de camiseta es requerido' },
      {
        validate: (value) => {
          const num = parseInt(value);
          return num >= 1 && num <= 99;
        },
        message: 'El número debe estar entre 1 y 99',
      },
    ],
    team_id: [{ required: true, message: 'Debe seleccionar un equipo' }],
  },

  // Partidos
  match: {
    home_team_id: [
      { required: true, message: 'Debe seleccionar el equipo local' },
    ],
    away_team_id: [
      { required: true, message: 'Debe seleccionar el equipo visitante' },
      {
        validate: (value, values) => value !== values.home_team_id,
        message: 'El equipo visitante debe ser diferente al local',
      },
    ],
    match_date: [
      { required: true, message: 'La fecha del partido es requerida' },
      {
        validate: (value) => {
          if (!value) return false;
          const matchDate = new Date(value);
          const today = new Date();
          return matchDate >= today;
        },
        message: 'La fecha del partido debe ser futura',
      },
    ],
    phase: [{ required: true, message: 'La fase del torneo es requerida' }],
  },
};

// Helper para obtener reglas específicas
export const getValidationRules = (entity) => {
  return validationRules[entity] || {};
};

// Validator personalizado para fechas
export const validateDate = (dateString, minAge = null, maxAge = null) => {
  if (!dateString) return false;

  const date = new Date(dateString);
  if (isNaN(date.getTime())) return false;

  if (minAge || maxAge) {
    const today = new Date();
    const age = today.getFullYear() - date.getFullYear();

    if (minAge && age < minAge) return false;
    if (maxAge && age > maxAge) return false;
  }

  return true;
};
```

---

## 📊 **Paso 5: Cache y Optimización (10 minutos)**

Implementar cache básico para mejorar performance.

### **📄 `src/utils/cache.js`:**

```javascript
class SimpleCache {
  constructor(defaultTTL = 5 * 60 * 1000) {
    // 5 minutos por defecto
    this.cache = new Map();
    this.defaultTTL = defaultTTL;
  }

  set(key, value, ttl = this.defaultTTL) {
    const expiry = Date.now() + ttl;
    this.cache.set(key, { value, expiry });
  }

  get(key) {
    const item = this.cache.get(key);
    if (!item) return null;

    if (Date.now() > item.expiry) {
      this.cache.delete(key);
      return null;
    }

    return item.value;
  }

  has(key) {
    return this.get(key) !== null;
  }

  delete(key) {
    return this.cache.delete(key);
  }

  clear() {
    this.cache.clear();
  }

  // Limpiar entradas expiradas
  cleanup() {
    const now = Date.now();
    for (const [key, item] of this.cache.entries()) {
      if (now > item.expiry) {
        this.cache.delete(key);
      }
    }
  }
}

export const apiCache = new SimpleCache();

// Limpiar cache cada 10 minutos
setInterval(() => {
  apiCache.cleanup();
}, 10 * 60 * 1000);
```

### **📄 Actualizar `src/hooks/useApi.js` con cache:**

```javascript
import { useState, useEffect, useCallback, useRef } from 'react';
import { useApp } from '../contexts/AppContext';
import { apiCache } from '../utils/cache';

const useApi = (apiFunction, dependencies = [], options = {}) => {
  const { addNotification } = useApp();
  const [data, setData] = useState(options.initialData || null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const mountedRef = useRef(true);

  const config = {
    executeOnMount: true,
    showSuccessNotification: false,
    showErrorNotification: true,
    cacheKey: null,
    cacheTime: 5 * 60 * 1000, // 5 minutos
    ...options,
  };

  const execute = useCallback(
    async (...args) => {
      if (!mountedRef.current) return;

      // Verificar cache si hay cacheKey
      if (config.cacheKey && args.length === 0) {
        const cachedData = apiCache.get(config.cacheKey);
        if (cachedData) {
          setData(cachedData);
          return cachedData;
        }
      }

      setLoading(true);
      setError(null);

      try {
        const response = await apiFunction(...args);

        if (!mountedRef.current) return;

        const responseData = response.data;
        setData(responseData);

        // Guardar en cache si hay cacheKey
        if (config.cacheKey && args.length === 0) {
          apiCache.set(config.cacheKey, responseData, config.cacheTime);
        }

        if (config.showSuccessNotification) {
          addNotification({
            type: 'success',
            message: 'Operación completada exitosamente',
          });
        }

        return responseData;
      } catch (err) {
        if (!mountedRef.current) return;

        const errorMessage = err.message || 'Error en la operación';
        setError(errorMessage);

        if (config.showErrorNotification) {
          addNotification({
            type: 'error',
            message: errorMessage,
          });
        }

        throw err;
      } finally {
        if (mountedRef.current) {
          setLoading(false);
        }
      }
    },
    [apiFunction, config, addNotification]
  );

  useEffect(() => {
    if (config.executeOnMount) {
      execute();
    }

    return () => {
      mountedRef.current = false;
    };
  }, dependencies);

  const reset = useCallback(() => {
    setData(config.initialData || null);
    setError(null);
    setLoading(false);

    // Limpiar cache si existe
    if (config.cacheKey) {
      apiCache.delete(config.cacheKey);
    }
  }, [config.initialData, config.cacheKey]);

  return {
    data,
    loading,
    error,
    execute,
    reset,
  };
};

export default useApi;
```

---

## 🧪 **Testing de la Integración (5 minutos)**

Script para verificar que todo funciona correctamente.

### **📄 `src/components/IntegrationTest.jsx`:**

```jsx
import { useState } from 'react';
import { useCountries } from '../hooks/useCountries';
import { apiService } from '../services/api';
import Button from './ui/Button';

const IntegrationTest = () => {
  const { countries, loading } = useCountries();
  const [testResults, setTestResults] = useState([]);

  const runTests = async () => {
    const results = [];

    try {
      // Test 1: API básica
      const response = await apiService.get('/countries');
      results.push({
        test: 'API Countries',
        status: 'success',
        data: `${response.data?.length || 0} países obtenidos`,
      });
    } catch (error) {
      results.push({
        test: 'API Countries',
        status: 'error',
        data: error.message,
      });
    }

    try {
      // Test 2: Validación
      await apiService.post('/countries', { name: '', code: '' });
    } catch (error) {
      if (error.status === 422) {
        results.push({
          test: 'Validación Backend',
          status: 'success',
          data: 'Validaciones funcionando correctamente',
        });
      } else {
        results.push({
          test: 'Validación Backend',
          status: 'warning',
          data: `Error inesperado: ${error.message}`,
        });
      }
    }

    setTestResults(results);
  };

  return (
    <div className="p-6 bg-white rounded-lg shadow">
      <h2 className="text-xl font-bold mb-4">Test de Integración</h2>

      <div className="space-y-4">
        <div>
          <p>
            Hook useCountries:{' '}
            {loading ? 'Cargando...' : `${countries.length} países`}
          </p>
        </div>

        <Button onClick={runTests}>Ejecutar Tests de API</Button>

        {testResults.length > 0 && (
          <div className="space-y-2">
            <h3 className="font-semibold">Resultados:</h3>
            {testResults.map((result, index) => (
              <div
                key={index}
                className={`p-2 rounded ${
                  result.status === 'success'
                    ? 'bg-green-100 text-green-800'
                    : result.status === 'error'
                    ? 'bg-red-100 text-red-800'
                    : 'bg-yellow-100 text-yellow-800'
                }`}>
                <strong>{result.test}:</strong> {result.data}
              </div>
            ))}
          </div>
        )}
      </div>
    </div>
  );
};

export default IntegrationTest;
```

---

## 🎯 **Checklist de Integración Exitosa**

- ✅ **Proxy configurado** y funcionando sin errores CORS
- ✅ **APIs respondiendo** correctamente desde React
- ✅ **Validaciones sincronizadas** entre frontend y backend
- ✅ **Errores manejados** de forma elegante
- ✅ **Loading states** funcionando en todas las operaciones
- ✅ **Notificaciones** mostrándose apropiadamente
- ✅ **Cache funcionando** para optimizar performance

---

## 💡 **Tips para WorldSkills**

### **Debugging Rápido:**

```javascript
// En development, agregar logs útiles
if (import.meta.env.DEV) {
  console.log('API Response:', response.data);
  console.log('Error:', error);
}
```

### **Validación de Conectividad:**

```javascript
// Test rápido de conectividad
const testConnection = async () => {
  try {
    await apiService.get('/health'); // Endpoint de health check
    return true;
  } catch {
    return false;
  }
};
```

---

¡Con esta integración robusta, tu aplicación React estará perfectamente conectada con el backend Laravel! 🚀
