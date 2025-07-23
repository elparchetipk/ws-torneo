# 🛠️ Guía de Configuración del Entorno

## Configuración Rápida para Competencia WorldSkills 2025

### ⚡ Setup Objetivo: < 10 minutos

---

## 📋 Pre-requisitos

### Software Necesario:

- **PHP 8.1+** con extensiones: sqlite3, pdo_sqlite, openssl, json, tokenizer
- **Composer 2.0+** (gestor de dependencias PHP)
- **Node.js 18+** (para frontend)
- **pnpm** (gestor de paquetes, más rápido que npm)
- **Git** (control de versiones)
- **VSCode** (editor recomendado)
- **Postman** (testing de APIs)

### Verificación rápida:

```bash
# Verificar instalaciones
php --version && php -m | grep -E "(sqlite3|pdo_sqlite)"
composer --version
node --version && npm --version
pnpm --version || npm install -g pnpm
git --version
```

---

## 🚀 Setup de Proyecto Completo

### Paso 1: Backend Laravel (3-4 minutos)

```bash
# 1. Crear proyecto Laravel
composer create-project laravel/laravel ws-torneo-api
cd ws-torneo-api

# 2. Configurar base de datos SQLite
echo "DB_CONNECTION=sqlite" > .env.local
echo "DB_DATABASE=/absolute/path/to/database.sqlite" >> .env.local
touch database/database.sqlite

# 3. Generar modelos con migraciones, controladores y resources
php artisan make:model Country -mcr
php artisan make:model Team -mcr
php artisan make:model Player -mcr
php artisan make:model Match -mcr

# 4. Ejecutar migraciones y seeders (cuando estén listos)
php artisan migrate:fresh --seed

# 5. Iniciar servidor
php artisan serve
# ✅ Laravel disponible en http://localhost:8000
```

### Paso 2: Frontend React + Vite (2-3 minutos)

```bash
# 1. Crear proyecto React con Vite
pnpm create vite ws-torneo-frontend --template react
cd ws-torneo-frontend

# 2. Instalar dependencias base
pnpm install

# 3. Instalar dependencias adicionales
pnpm add axios react-router-dom

# 4. Opcional: Instalar Tailwind CSS
pnpm add -D tailwindcss postcss autoprefixer
npx tailwindcss init -p

# 5. Iniciar servidor de desarrollo
pnpm dev
# ✅ React disponible en http://localhost:3000
```

### Paso 3: Configuración de Comunicación (1-2 minutos)

#### Frontend - Configurar Proxy (vite.config.js):

```js
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
});
```

#### Backend - Configurar CORS (Laravel):

```bash
# Instalar Laravel Sanctum para CORS
php artisan install:api
```

---

## ⏱️ Cronómetro de Setup

### Tiempos Objetivo por Componente:

- **Verificación pre-requisitos:** 30 segundos
- **Creación proyecto Laravel:** 2 minutos
- **Creación proyecto React + Vite:** 1.5 minutos
- **Instalación dependencias:** 2 minutos
- **Configuración comunicación:** 1 minuto
- **Verificación funcionamiento:** 30 segundos
- **Buffer para problemas:** 2.5 minutos

**TOTAL: 10 minutos máximo**

---

## 🔧 Troubleshooting Rápido

### Problemas Comunes:

#### Laravel no inicia:

```bash
# Verificar permisos
chmod -R 775 storage bootstrap/cache
# Regenerar key
php artisan key:generate
```

#### Frontend no conecta con API:

```bash
# Verificar puertos
netstat -tulpn | grep :8000  # Laravel
netstat -tulpn | grep :3000  # React
```

#### pnpm no funciona:

```bash
# Reinstalar pnpm
npm uninstall -g pnpm
npm install -g pnpm@latest
```

#### Errores de CORS:

```php
// En config/cors.php
'paths' => ['api/*', 'sanctum/csrf-cookie'],
'allowed_origins' => ['http://localhost:3000'],
```

---

## ✅ Verificación Final

### Checklist de Setup Exitoso:

- [ ] Laravel responde en http://localhost:8000
- [ ] React responde en http://localhost:3000
- [ ] Base de datos SQLite creada y conectada
- [ ] Proxy Vite → Laravel configurado
- [ ] CORS habilitado en Laravel
- [ ] Postman listo para testing de APIs

### Test de Comunicación:

```bash
# Crear ruta de prueba en Laravel (routes/api.php)
Route::get('/test', function () {
    return response()->json(['message' => 'API funcionando!']);
});

# Probar desde React
fetch('/api/test').then(r => r.json()).then(console.log)
```

---

## 📝 Scripts de Automatización

### Script completo de setup (setup.sh):

```bash
#!/bin/bash
echo "🚀 Iniciando setup WorldSkills..."

# Backend
composer create-project laravel/laravel ws-torneo-api --no-interaction
cd ws-torneo-api
touch database/database.sqlite
sed -i 's/DB_CONNECTION=mysql/DB_CONNECTION=sqlite/' .env
php artisan key:generate
php artisan serve &

# Frontend
cd ..
pnpm create vite ws-torneo-frontend --template react
cd ws-torneo-frontend
pnpm install
pnpm add axios react-router-dom
pnpm dev &

echo "✅ Setup completado en $(date)"
```

---

**⚠️ Nota importante:** Practicar este setup múltiples veces hasta lograr automatismo. En competencia cada segundo cuenta!
