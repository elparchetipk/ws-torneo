# 🎣 Custom Hooks Avanzados para APIs

**Duración:** 1.5 horas (1:00 PM - 2:30 PM)  
**Objetivo:** Crear hooks personalizados que simplifiquen y aceleren el consumo de APIs Laravel

---

## 🎯 **Objetivos de la Sesión**

Al finalizar esta sesión, el competidor será capaz de:

✅ **Crear useApi genérico** para cualquier endpoint REST  
✅ **Implementar hooks específicos** por entidad (useCountries, useTeams, etc.)  
✅ **Manejar estados complejos** (loading, error, data) de forma elegante  
✅ **Optimizar re-renders** con técnicas de memoización  
✅ **Implementar cache local** para mejorar performance

---

## 🛠️ **Paso 1: Servicio Base de API (15 minutos)**

Primero, creamos un servicio centralizado para manejar todas las comunicaciones con el backend Laravel.

**📄 `src/services/api.js`:**

```javascript
import axios from 'axios';

// Configuración base de Axios
const api = axios.create({
  baseURL: '/api',
  headers: {
    'Content-Type': 'application/json',
    Accept: 'application/json',
  },
});

// Interceptor para manejo de errores global
api.interceptors.response.use(
  (response) => response,
  (error) => {
    const errorMessage =
      error.response?.data?.message || error.message || 'Error de conexión';

    console.error('API Error:', errorMessage);
    return Promise.reject(new Error(errorMessage));
  }
);

// Métodos HTTP genéricos
export const apiService = {
  get: (url, config = {}) => api.get(url, config),
  post: (url, data, config = {}) => api.post(url, data, config),
  put: (url, data, config = {}) => api.put(url, data, config),
  delete: (url, config = {}) => api.delete(url, config),
};

// Servicio REST genérico para CRUDs
export const createRestService = (endpoint) => ({
  getAll: (params = {}) => apiService.get(endpoint, { params }),
  getById: (id) => apiService.get(`${endpoint}/${id}`),
  create: (data) => apiService.post(endpoint, data),
  update: (id, data) => apiService.put(`${endpoint}/${id}`, data),
  delete: (id) => apiService.delete(`${endpoint}/${id}`),
});

export default api;
```

---

## 🎣 **Paso 2: Hook useApi Genérico (20 minutos)**

Este hook manejará el ciclo completo de cualquier petición API con estados de loading, error y data.

**📄 `src/hooks/useApi.js`:**

```javascript
import { useState, useEffect, useCallback, useRef } from 'react';
import { useApp } from '../contexts/AppContext';

const useApi = (apiFunction, dependencies = [], options = {}) => {
  const { addNotification } = useApp();
  const [data, setData] = useState(options.initialData || null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  const mountedRef = useRef(true);

  // Configuración por defecto
  const config = {
    executeOnMount: true,
    showSuccessNotification: false,
    showErrorNotification: true,
    cacheTime: 5 * 60 * 1000, // 5 minutos
    ...options,
  };

  // Función para ejecutar la API
  const execute = useCallback(
    async (...args) => {
      if (!mountedRef.current) return;

      setLoading(true);
      setError(null);

      try {
        const response = await apiFunction(...args);

        if (!mountedRef.current) return;

        setData(response.data);

        if (config.showSuccessNotification) {
          addNotification({
            type: 'success',
            message: 'Operación completada exitosamente',
          });
        }

        return response.data;
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

  // Ejecutar al montar si está configurado
  useEffect(() => {
    if (config.executeOnMount) {
      execute();
    }

    // Cleanup function
    return () => {
      mountedRef.current = false;
    };
  }, dependencies);

  // Reset function
  const reset = useCallback(() => {
    setData(config.initialData || null);
    setError(null);
    setLoading(false);
  }, [config.initialData]);

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

## 🎣 **Paso 3: Hook useAsyncOperation (15 minutos)**

Para operaciones que no requieren datos persistentes (crear, actualizar, eliminar).

**📄 `src/hooks/useAsyncOperation.js`:**

```javascript
import { useState, useCallback } from 'react';
import { useApp } from '../contexts/AppContext';

const useAsyncOperation = (options = {}) => {
  const { addNotification } = useApp();
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const config = {
    showSuccessNotification: true,
    showErrorNotification: true,
    successMessage: 'Operación completada exitosamente',
    ...options,
  };

  const execute = useCallback(
    async (operation) => {
      setLoading(true);
      setError(null);

      try {
        const result = await operation();

        if (config.showSuccessNotification) {
          addNotification({
            type: 'success',
            message: config.successMessage,
          });
        }

        return result;
      } catch (err) {
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
        setLoading(false);
      }
    },
    [config, addNotification]
  );

  const reset = useCallback(() => {
    setError(null);
    setLoading(false);
  }, []);

  return {
    loading,
    error,
    execute,
    reset,
  };
};

export default useAsyncOperation;
```

---

## 🎣 **Paso 4: Hooks Específicos por Entidad (30 minutos)**

Ahora creamos hooks especializados para cada entidad del torneo.

### **📄 `src/hooks/useCountries.js`:**

```javascript
import { useMemo } from 'react';
import useApi from './useApi';
import useAsyncOperation from './useAsyncOperation';
import { createRestService } from '../services/api';

const countriesService = createRestService('/countries');

export const useCountries = (options = {}) => {
  const {
    data: countries,
    loading,
    error,
    execute: refetch,
  } = useApi(() => countriesService.getAll(), [], {
    executeOnMount: true,
    ...options,
  });

  // Memoizar países ordenados alfabéticamente
  const sortedCountries = useMemo(() => {
    if (!countries) return [];
    return [...countries].sort((a, b) => a.name.localeCompare(b.name));
  }, [countries]);

  return {
    countries: sortedCountries,
    loading,
    error,
    refetch,
  };
};

export const useCountry = (id) => {
  return useApi(() => countriesService.getById(id), [id], {
    executeOnMount: !!id,
  });
};

export const useCountryOperations = () => {
  const createOperation = useAsyncOperation({
    successMessage: 'País creado exitosamente',
  });

  const updateOperation = useAsyncOperation({
    successMessage: 'País actualizado exitosamente',
  });

  const deleteOperation = useAsyncOperation({
    successMessage: 'País eliminado exitosamente',
  });

  return {
    create: {
      execute: (data) =>
        createOperation.execute(() => countriesService.create(data)),
      loading: createOperation.loading,
      error: createOperation.error,
    },
    update: {
      execute: (id, data) =>
        updateOperation.execute(() => countriesService.update(id, data)),
      loading: updateOperation.loading,
      error: updateOperation.error,
    },
    delete: {
      execute: (id) =>
        deleteOperation.execute(() => countriesService.delete(id)),
      loading: deleteOperation.loading,
      error: deleteOperation.error,
    },
  };
};
```

### **📄 `src/hooks/useTeams.js`:**

```javascript
import { useMemo } from 'react';
import useApi from './useApi';
import useAsyncOperation from './useAsyncOperation';
import { createRestService } from '../services/api';

const teamsService = createRestService('/teams');

export const useTeams = (filters = {}) => {
  const {
    data: teams,
    loading,
    error,
    execute: refetch,
  } = useApi(() => teamsService.getAll(filters), [JSON.stringify(filters)], {
    executeOnMount: true,
  });

  // Equipos agrupados por país
  const teamsByCountry = useMemo(() => {
    if (!teams) return {};

    return teams.reduce((acc, team) => {
      const countryName = team.country?.name || 'Sin país';
      if (!acc[countryName]) {
        acc[countryName] = [];
      }
      acc[countryName].push(team);
      return acc;
    }, {});
  }, [teams]);

  // Opciones para selects
  const teamOptions = useMemo(() => {
    if (!teams) return [];
    return teams.map((team) => ({
      value: team.id,
      label: `${team.name} (${team.country?.name})`,
    }));
  }, [teams]);

  return {
    teams: teams || [],
    teamsByCountry,
    teamOptions,
    loading,
    error,
    refetch,
  };
};

export const useTeam = (id) => {
  return useApi(() => teamsService.getById(id), [id], { executeOnMount: !!id });
};

export const useTeamOperations = () => {
  const createOperation = useAsyncOperation({
    successMessage: 'Equipo creado exitosamente',
  });

  const updateOperation = useAsyncOperation({
    successMessage: 'Equipo actualizado exitosamente',
  });

  const deleteOperation = useAsyncOperation({
    successMessage: 'Equipo eliminado exitosamente',
  });

  return {
    create: {
      execute: (data) =>
        createOperation.execute(() => teamsService.create(data)),
      loading: createOperation.loading,
      error: createOperation.error,
    },
    update: {
      execute: (id, data) =>
        updateOperation.execute(() => teamsService.update(id, data)),
      loading: updateOperation.loading,
      error: updateOperation.error,
    },
    delete: {
      execute: (id) => deleteOperation.execute(() => teamsService.delete(id)),
      loading: deleteOperation.loading,
      error: deleteOperation.error,
    },
  };
};
```

### **📄 `src/hooks/useStandings.js`:**

```javascript
import { useMemo } from 'react';
import useApi from './useApi';
import { apiService } from '../services/api';

export const useStandings = (phase = 'all') => {
  const {
    data: standings,
    loading,
    error,
    execute: refetch,
  } = useApi(
    () => apiService.get('/standings', { params: { phase } }),
    [phase],
    { executeOnMount: true }
  );

  // Calcular estadísticas adicionales
  const stats = useMemo(() => {
    if (!standings) return {};

    const totalTeams = standings.length;
    const totalMatches = standings.reduce(
      (sum, team) => sum + team.matches_played,
      0
    );
    const totalGoals = standings.reduce((sum, team) => sum + team.goals_for, 0);

    return {
      totalTeams,
      totalMatches,
      totalGoals,
      averageGoalsPerMatch:
        totalMatches > 0 ? (totalGoals / totalMatches).toFixed(2) : 0,
    };
  }, [standings]);

  // Top 3 equipos
  const topThree = useMemo(() => {
    if (!standings) return [];
    return standings.slice(0, 3);
  }, [standings]);

  return {
    standings: standings || [],
    stats,
    topThree,
    loading,
    error,
    refetch,
  };
};
```

---

## 🎣 **Paso 5: Hook para Manejo de Formularios (15 minutos)**

Hook especializado para manejar estados de formularios con validación.

**📄 `src/hooks/useForm.js`:**

```javascript
import { useState, useCallback, useMemo } from 'react';

const useForm = (initialValues = {}, validationRules = {}) => {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});

  // Validar un campo específico
  const validateField = useCallback(
    (name, value) => {
      const rules = validationRules[name];
      if (!rules) return '';

      for (const rule of rules) {
        if (rule.required && (!value || value.toString().trim() === '')) {
          return rule.message || `${name} es requerido`;
        }

        if (rule.minLength && value && value.length < rule.minLength) {
          return (
            rule.message ||
            `${name} debe tener al menos ${rule.minLength} caracteres`
          );
        }

        if (rule.maxLength && value && value.length > rule.maxLength) {
          return (
            rule.message ||
            `${name} no puede tener más de ${rule.maxLength} caracteres`
          );
        }

        if (rule.pattern && value && !rule.pattern.test(value)) {
          return rule.message || `${name} tiene un formato inválido`;
        }

        if (rule.validate && !rule.validate(value, values)) {
          return rule.message || `${name} es inválido`;
        }
      }

      return '';
    },
    [validationRules, values]
  );

  // Validar todos los campos
  const validate = useCallback(() => {
    const newErrors = {};
    let isValid = true;

    Object.keys(validationRules).forEach((fieldName) => {
      const error = validateField(fieldName, values[fieldName]);
      if (error) {
        newErrors[fieldName] = error;
        isValid = false;
      }
    });

    setErrors(newErrors);
    return isValid;
  }, [values, validateField, validationRules]);

  // Cambiar valor de un campo
  const setValue = useCallback(
    (name, value) => {
      setValues((prev) => ({ ...prev, [name]: value }));

      // Validar el campo si ya fue tocado
      if (touched[name]) {
        const error = validateField(name, value);
        setErrors((prev) => ({ ...prev, [name]: error }));
      }
    },
    [touched, validateField]
  );

  // Marcar campo como tocado
  const setFieldTouched = useCallback(
    (name) => {
      setTouched((prev) => ({ ...prev, [name]: true }));

      // Validar cuando se marca como tocado
      const error = validateField(name, values[name]);
      setErrors((prev) => ({ ...prev, [name]: error }));
    },
    [values, validateField]
  );

  // Reset del formulario
  const reset = useCallback(() => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
  }, [initialValues]);

  // Handler para inputs
  const getFieldProps = useCallback(
    (name) => ({
      value: values[name] || '',
      onChange: (e) => setValue(name, e.target.value),
      onBlur: () => setFieldTouched(name),
      error: touched[name] ? errors[name] : '',
    }),
    [values, errors, touched, setValue, setFieldTouched]
  );

  // Verificar si el formulario es válido
  const isValid = useMemo(() => {
    return Object.keys(errors).length === 0 && Object.keys(touched).length > 0;
  }, [errors, touched]);

  return {
    values,
    errors,
    touched,
    isValid,
    setValue,
    setFieldTouched,
    validate,
    reset,
    getFieldProps,
  };
};

export default useForm;
```

---

## 🎣 **Paso 6: Hook para Debouncing (10 minutos)**

Útil para búsquedas y filtros en tiempo real.

**📄 `src/hooks/useDebounce.js`:**

```javascript
import { useState, useEffect } from 'react';

const useDebounce = (value, delay) => {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const handler = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(handler);
    };
  }, [value, delay]);

  return debouncedValue;
};

export default useDebounce;
```

**Uso en componente de búsqueda:**

```javascript
import { useState } from 'react';
import useDebounce from '../hooks/useDebounce';
import { useTeams } from '../hooks/useTeams';

const TeamSearch = () => {
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearchTerm = useDebounce(searchTerm, 300);

  const { teams, loading } = useTeams({
    search: debouncedSearchTerm,
  });

  return (
    <div>
      <input
        type="text"
        placeholder="Buscar equipos..."
        value={searchTerm}
        onChange={(e) => setSearchTerm(e.target.value)}
        className="input-field"
      />

      {loading && <p>Buscando...</p>}

      <div className="grid grid-cols-1 gap-4">
        {teams.map((team) => (
          <div
            key={team.id}
            className="card">
            <h3>{team.name}</h3>
            <p>{team.country?.name}</p>
          </div>
        ))}
      </div>
    </div>
  );
};
```

---

## 🧪 **Testing de los Hooks (5 minutos)**

Crear un componente simple para probar los hooks:

**📄 `src/components/HookTester.jsx`:**

```javascript
import { useCountries, useCountryOperations } from '../hooks/useCountries';
import { useTeams } from '../hooks/useTeams';

const HookTester = () => {
  const { countries, loading: countriesLoading } = useCountries();
  const { teams, loading: teamsLoading } = useTeams();
  const { create } = useCountryOperations();

  const handleCreateCountry = async () => {
    try {
      await create.execute({
        name: 'País de Prueba',
        code: 'TP',
      });
    } catch (error) {
      console.error('Error:', error);
    }
  };

  return (
    <div className="p-4">
      <h2 className="text-xl font-bold mb-4">Test de Hooks</h2>

      <div className="mb-4">
        <h3 className="font-semibold">Países ({countries?.length || 0})</h3>
        {countriesLoading ? (
          <p>Cargando países...</p>
        ) : (
          <ul>
            {countries?.slice(0, 3).map((country) => (
              <li key={country.id}>{country.name}</li>
            ))}
          </ul>
        )}
      </div>

      <div className="mb-4">
        <h3 className="font-semibold">Equipos ({teams?.length || 0})</h3>
        {teamsLoading ? (
          <p>Cargando equipos...</p>
        ) : (
          <ul>
            {teams?.slice(0, 3).map((team) => (
              <li key={team.id}>
                {team.name} - {team.country?.name}
              </li>
            ))}
          </ul>
        )}
      </div>

      <button
        onClick={handleCreateCountry}
        disabled={create.loading}
        className="btn-primary">
        {create.loading ? 'Creando...' : 'Crear País de Prueba'}
      </button>
    </div>
  );
};

export default HookTester;
```

---

## 💡 **Tips para Optimización**

### **Memoización Inteligente:**

```javascript
// ✅ Bueno: memoizar objetos complejos
const teamOptions = useMemo(() => {
  return teams.map((team) => ({
    value: team.id,
    label: team.name,
  }));
}, [teams]);

// ❌ Malo: memoizar valores primitivos
const teamCount = useMemo(() => teams.length, [teams]);
```

### **Manejo de Dependencias:**

```javascript
// ✅ Bueno: incluir todas las dependencias
const { data } = useApi(
  () => getTeams({ country: countryId, status }),
  [countryId, status]
);

// ❌ Malo: dependencias faltantes
const { data } = useApi(
  () => getTeams({ country: countryId, status }),
  [countryId] // Falta 'status'
);
```

---

## 🎯 **Siguientes Pasos**

Con estos hooks implementados, estarás listo para:

1. **Crear componentes** que consuman estas APIs eficientemente
2. **Implementar formularios** con validación integrada
3. **Desarrollar listas** con búsqueda y filtros en tiempo real
4. **Optimizar renders** con técnicas avanzadas

Los hooks personalizados son la **clave para desarrollo React rápido y consistente** en WorldSkills. ¡Practícalos hasta dominarlos completamente! 🚀
