# 🏗️ Arquitectura React para WorldSkills

**Duración:** 1 hora (12:00 PM - 1:00 PM)  
**Objetivo:** Establecer una arquitectura React escalable y optimizada para desarrollo rápido

---

## 🎯 **Objetivos de la Sesión**

Al finalizar esta sesión, el competidor será capaz de:

✅ **Configurar un proyecto React** optimizado en menos de 10 minutos  
✅ **Estructurar carpetas** de manera profesional y escalable  
✅ **Configurar herramientas esenciales** (Tailwind, Router, etc.)  
✅ **Crear componentes base** reutilizables para todo el proyecto  
✅ **Establecer convenciones** que aceleren el desarrollo futuro

---

## 📁 **Estructura de Proyecto Optimizada**

### **Paso 1: Inicialización del Proyecto (5 minutos)**

```bash
# Crear proyecto con Vite (más rápido que CRA)
cd ws-torneo-api/
npm create vite@latest frontend -- --template react
cd frontend

# Instalar dependencias con pnpm (más eficiente)
pnpm install

# Instalar dependencias esenciales para WorldSkills
pnpm add react-router-dom@6 tailwindcss@3 @tailwindcss/forms
pnpm add @heroicons/react axios date-fns clsx
pnpm add -D @types/node prettier eslint-config-prettier
```

### **Paso 2: Configuración de Tailwind CSS (5 minutos)**

```bash
# Inicializar Tailwind
npx tailwindcss init -p
```

**📄 `tailwind.config.js`:**

```javascript
/** @type {import('tailwindcss').Config} */
export default {
  content: ['./index.html', './src/**/*.{js,ts,jsx,tsx}'],
  theme: {
    extend: {
      colors: {
        primary: {
          50: '#eff6ff',
          500: '#3b82f6',
          600: '#2563eb',
          700: '#1d4ed8',
        },
        success: '#10b981',
        warning: '#f59e0b',
        error: '#ef4444',
      },
    },
  },
  plugins: [require('@tailwindcss/forms')],
};
```

**📄 `src/index.css`:**

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

/* Componentes base personalizados */
@layer components {
  .btn-primary {
    @apply bg-primary-500 hover:bg-primary-600 text-white font-medium py-2 px-4 rounded-lg transition-colors duration-200;
  }

  .btn-secondary {
    @apply bg-gray-200 hover:bg-gray-300 text-gray-800 font-medium py-2 px-4 rounded-lg transition-colors duration-200;
  }

  .card {
    @apply bg-white rounded-lg shadow-md border border-gray-200 p-6;
  }

  .input-field {
    @apply mt-1 block w-full rounded-md border-gray-300 shadow-sm focus:border-primary-500 focus:ring-primary-500;
  }

  .table-header {
    @apply px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider;
  }

  .table-cell {
    @apply px-6 py-4 whitespace-nowrap text-sm text-gray-900;
  }
}
```

### **Paso 3: Estructura de Carpetas Profesional (10 minutos)**

```
src/
├── components/           # Componentes reutilizables
│   ├── ui/              # Componentes base (Button, Modal, etc.)
│   ├── forms/           # Componentes de formularios
│   ├── tables/          # Componentes de tablas
│   └── layout/          # Layout components
├── pages/               # Páginas principales
│   ├── Countries/
│   ├── Teams/
│   ├── Players/
│   ├── Matches/
│   └── Dashboard/
├── hooks/               # Custom hooks
│   ├── useApi.js
│   ├── useCountries.js
│   └── useTeams.js
├── services/            # Servicios de API
│   ├── api.js
│   ├── countries.js
│   └── teams.js
├── utils/               # Utilidades
│   ├── constants.js
│   ├── helpers.js
│   └── validators.js
├── contexts/            # Context providers
│   └── AppContext.jsx
└── App.jsx             # Componente principal
```

### **Paso 4: Configuración del Router (10 minutos)**

**📄 `src/App.jsx`:**

```jsx
import { BrowserRouter as Router, Routes, Route } from 'react-router-dom';
import { AppProvider } from './contexts/AppContext';
import Layout from './components/layout/Layout';
import Dashboard from './pages/Dashboard/Dashboard';
import Countries from './pages/Countries/Countries';
import Teams from './pages/Teams/Teams';
import Players from './pages/Players/Players';
import Matches from './pages/Matches/Matches';
import './index.css';

function App() {
  return (
    <AppProvider>
      <Router>
        <Layout>
          <Routes>
            <Route
              path="/"
              element={<Dashboard />}
            />
            <Route
              path="/countries"
              element={<Countries />}
            />
            <Route
              path="/teams"
              element={<Teams />}
            />
            <Route
              path="/players"
              element={<Players />}
            />
            <Route
              path="/matches"
              element={<Matches />}
            />
          </Routes>
        </Layout>
      </Router>
    </AppProvider>
  );
}

export default App;
```

### **Paso 5: Layout Base y Navegación (15 minutos)**

**📄 `src/components/layout/Layout.jsx`:**

```jsx
import { useState } from 'react';
import Sidebar from './Sidebar';
import Header from './Header';

const Layout = ({ children }) => {
  const [sidebarOpen, setSidebarOpen] = useState(false);

  return (
    <div className="min-h-screen bg-gray-50">
      <Sidebar
        isOpen={sidebarOpen}
        onClose={() => setSidebarOpen(false)}
      />

      <div className="lg:pl-64">
        <Header onMenuClick={() => setSidebarOpen(true)} />

        <main className="py-6">
          <div className="mx-auto max-w-7xl px-4 sm:px-6 lg:px-8">
            {children}
          </div>
        </main>
      </div>
    </div>
  );
};

export default Layout;
```

**📄 `src/components/layout/Sidebar.jsx`:**

```jsx
import { Fragment } from 'react';
import { Dialog, Transition } from '@headlessui/react';
import { Link, useLocation } from 'react-router-dom';
import {
  XMarkIcon,
  HomeIcon,
  GlobeAmericasIcon,
  UserGroupIcon,
  UsersIcon,
  TrophyIcon,
} from '@heroicons/react/24/outline';

const navigation = [
  { name: 'Dashboard', href: '/', icon: HomeIcon },
  { name: 'Países', href: '/countries', icon: GlobeAmericasIcon },
  { name: 'Equipos', href: '/teams', icon: UserGroupIcon },
  { name: 'Jugadoras', href: '/players', icon: UsersIcon },
  { name: 'Partidos', href: '/matches', icon: TrophyIcon },
];

function classNames(...classes) {
  return classes.filter(Boolean).join(' ');
}

const Sidebar = ({ isOpen, onClose }) => {
  const location = useLocation();

  const SidebarContent = () => (
    <div className="flex grow flex-col gap-y-5 overflow-y-auto bg-primary-600 px-6 pb-2">
      <div className="flex h-16 shrink-0 items-center">
        <h1 className="text-xl font-bold text-white">Torneo Femenino SA</h1>
      </div>
      <nav className="flex flex-1 flex-col">
        <ul
          role="list"
          className="flex flex-1 flex-col gap-y-7">
          <li>
            <ul
              role="list"
              className="-mx-2 space-y-1">
              {navigation.map((item) => (
                <li key={item.name}>
                  <Link
                    to={item.href}
                    className={classNames(
                      location.pathname === item.href
                        ? 'bg-primary-700 text-white'
                        : 'text-primary-200 hover:text-white hover:bg-primary-700',
                      'group flex gap-x-3 rounded-md p-2 text-sm leading-6 font-semibold'
                    )}
                    onClick={onClose}>
                    <item.icon
                      className="h-6 w-6 shrink-0"
                      aria-hidden="true"
                    />
                    {item.name}
                  </Link>
                </li>
              ))}
            </ul>
          </li>
        </ul>
      </nav>
    </div>
  );

  return (
    <>
      {/* Mobile sidebar */}
      <Transition.Root
        show={isOpen}
        as={Fragment}>
        <Dialog
          as="div"
          className="relative z-50 lg:hidden"
          onClose={onClose}>
          <Transition.Child
            as={Fragment}
            enter="transition-opacity ease-linear duration-300"
            enterFrom="opacity-0"
            enterTo="opacity-100"
            leave="transition-opacity ease-linear duration-300"
            leaveFrom="opacity-100"
            leaveTo="opacity-0">
            <div className="fixed inset-0 bg-gray-900/80" />
          </Transition.Child>

          <div className="fixed inset-0 flex">
            <Transition.Child
              as={Fragment}
              enter="transition ease-in-out duration-300 transform"
              enterFrom="-translate-x-full"
              enterTo="translate-x-0"
              leave="transition ease-in-out duration-300 transform"
              leaveFrom="translate-x-0"
              leaveTo="-translate-x-full">
              <Dialog.Panel className="relative mr-16 flex w-full max-w-xs flex-1">
                <Transition.Child
                  as={Fragment}
                  enter="ease-in-out duration-300"
                  enterFrom="opacity-0"
                  enterTo="opacity-100"
                  leave="ease-in-out duration-300"
                  leaveFrom="opacity-100"
                  leaveTo="opacity-0">
                  <div className="absolute left-full top-0 flex w-16 justify-center pt-5">
                    <button
                      type="button"
                      className="-m-2.5 p-2.5"
                      onClick={onClose}>
                      <XMarkIcon className="h-6 w-6 text-white" />
                    </button>
                  </div>
                </Transition.Child>
                <SidebarContent />
              </Dialog.Panel>
            </Transition.Child>
          </div>
        </Dialog>
      </Transition.Root>

      {/* Static sidebar for desktop */}
      <div className="hidden lg:fixed lg:inset-y-0 lg:z-50 lg:flex lg:w-64 lg:flex-col">
        <SidebarContent />
      </div>
    </>
  );
};

export default Sidebar;
```

**📄 `src/components/layout/Header.jsx`:**

```jsx
import { Bars3Icon } from '@heroicons/react/24/outline';

const Header = ({ onMenuClick }) => {
  return (
    <div className="sticky top-0 z-40 flex h-16 shrink-0 items-center gap-x-4 border-b border-gray-200 bg-white px-4 shadow-sm sm:gap-x-6 sm:px-6 lg:px-8">
      <button
        type="button"
        className="-m-2.5 p-2.5 text-gray-700 lg:hidden"
        onClick={onMenuClick}>
        <Bars3Icon className="h-6 w-6" />
      </button>

      {/* Separator */}
      <div className="h-6 w-px bg-gray-200 lg:hidden" />

      <div className="flex flex-1 gap-x-4 self-stretch lg:gap-x-6">
        <div className="flex items-center gap-x-4 lg:gap-x-6">
          <h2 className="text-lg font-semibold text-gray-900">
            Sistema de Gestión del Torneo
          </h2>
        </div>
      </div>
    </div>
  );
};

export default Header;
```

### **Paso 6: Context Provider Global (10 minutos)**

**📄 `src/contexts/AppContext.jsx`:**

```jsx
import { createContext, useContext, useReducer } from 'react';

// Estado inicial
const initialState = {
  user: null,
  loading: false,
  error: null,
  notifications: [],
};

// Tipos de acciones
const actionTypes = {
  SET_LOADING: 'SET_LOADING',
  SET_ERROR: 'SET_ERROR',
  CLEAR_ERROR: 'CLEAR_ERROR',
  ADD_NOTIFICATION: 'ADD_NOTIFICATION',
  REMOVE_NOTIFICATION: 'REMOVE_NOTIFICATION',
};

// Reducer
const appReducer = (state, action) => {
  switch (action.type) {
    case actionTypes.SET_LOADING:
      return { ...state, loading: action.payload };

    case actionTypes.SET_ERROR:
      return { ...state, error: action.payload, loading: false };

    case actionTypes.CLEAR_ERROR:
      return { ...state, error: null };

    case actionTypes.ADD_NOTIFICATION:
      return {
        ...state,
        notifications: [
          ...state.notifications,
          {
            id: Date.now(),
            ...action.payload,
          },
        ],
      };

    case actionTypes.REMOVE_NOTIFICATION:
      return {
        ...state,
        notifications: state.notifications.filter(
          (notification) => notification.id !== action.payload
        ),
      };

    default:
      return state;
  }
};

// Context
const AppContext = createContext();

// Provider
export const AppProvider = ({ children }) => {
  const [state, dispatch] = useReducer(appReducer, initialState);

  // Actions
  const actions = {
    setLoading: (loading) =>
      dispatch({ type: actionTypes.SET_LOADING, payload: loading }),

    setError: (error) =>
      dispatch({ type: actionTypes.SET_ERROR, payload: error }),

    clearError: () => dispatch({ type: actionTypes.CLEAR_ERROR }),

    addNotification: (notification) =>
      dispatch({ type: actionTypes.ADD_NOTIFICATION, payload: notification }),

    removeNotification: (id) =>
      dispatch({ type: actionTypes.REMOVE_NOTIFICATION, payload: id }),
  };

  return (
    <AppContext.Provider value={{ ...state, ...actions }}>
      {children}
    </AppContext.Provider>
  );
};

// Hook personalizado
export const useApp = () => {
  const context = useContext(AppContext);
  if (!context) {
    throw new Error('useApp debe usarse dentro de AppProvider');
  }
  return context;
};
```

### **Paso 7: Configuración de Desarrollo (5 minutos)**

**📄 `vite.config.js`:**

```javascript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  server: {
    port: 3000,
    proxy: {
      '/api': {
        target: 'http://localhost:8000',
        changeOrigin: true,
        secure: false,
      },
    },
  },
  build: {
    outDir: '../public/frontend',
    emptyOutDir: true,
  },
});
```

---

## ⚡ **Scripts de Desarrollo Útiles**

**📄 `package.json` (agregar a scripts):**

```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview",
    "lint": "eslint . --ext js,jsx --report-unused-disable-directives --max-warnings 0",
    "format": "prettier --write \"src/**/*.{js,jsx,css,md}\""
  }
}
```

---

## 🧪 **Testing de la Arquitectura**

### **Verificación Rápida (5 minutos):**

```bash
# Levantar el servidor de desarrollo
pnpm run dev

# En otra terminal, verificar que Laravel esté corriendo
cd ws-torneo-api
php artisan serve
```

**Checklist de Verificación:**

- ✅ React se carga en http://localhost:3000
- ✅ Navegación entre páginas funciona
- ✅ Sidebar responsive en mobile
- ✅ Tailwind CSS aplicado correctamente
- ✅ No errores en consola del navegador

---

## 🎯 **Próximos Pasos**

Con esta arquitectura establecida, estás listo para:

1. **Crear custom hooks** para consumir APIs Laravel
2. **Desarrollar componentes** reutilizables de formularios y tablas
3. **Implementar lógica de negocio** específica del torneo
4. **Optimizar performance** con técnicas avanzadas

---

## 💡 **Tips para WorldSkills**

### **Velocidad de Setup:**

- 🚀 **Usa snippets** para configuraciones comunes
- 🚀 **Automatiza** con scripts de inicialización
- 🚀 **Practica** hasta lograr setup en menos de 10 minutos

### **Arquitectura Escalable:**

- 📁 **Estructura consistente** facilita navegación rápida
- 📁 **Separación de responsabilidades** reduce debugging
- 📁 **Convenciones claras** aceleran desarrollo

### **Herramientas de Productividad:**

- ⚡ **Vite HMR** para feedback inmediato
- ⚡ **Tailwind** para styling rápido
- ⚡ **React DevTools** para debugging eficiente

---

**¡Arquitectura lista! Ahora construyamos funcionalidad sobre esta base sólida. 🏗️**
