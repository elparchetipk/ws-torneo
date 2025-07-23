# ⚡ Ejercicios de Configuración del Entorno - Día 1.1

**Objetivo:** Setup completo Laravel + React en menos de 10 minutos

---

## 🎯 **Objetivo de la Sesión**

Los competidores deben poder configurar un entorno completo **Laravel + React + Vite + pnpm** en **menos de 10 minutos**, sin consultar documentación.

---

## ⏱️ **Cronómetro de Práctica**

### **Ejercicio 1: Setup Básico (8 minutos)**

#### **Paso 1: Backend Laravel (4 minutos)**

```bash
# CRONÓMETRO: INICIAR ⏰
composer create-project laravel/laravel torneo-api --no-interaction
cd torneo-api

# Configurar SQLite
touch database/database.sqlite
sed -i 's/DB_CONNECTION=mysql/DB_CONNECTION=sqlite/' .env
sed -i '/DB_HOST/d' .env
sed -i '/DB_PORT/d' .env
sed -i '/DB_DATABASE/c\DB_DATABASE=/absolute/path/to/database.sqlite' .env
sed -i '/DB_USERNAME/d' .env
sed -i '/DB_PASSWORD/d' .env

# Verificar
php artisan key:generate
php artisan migrate
php artisan serve &
# ✅ Debe estar en http://localhost:8000 en 4 minutos
```

#### **Paso 2: Frontend React + Vite (4 minutos)**

```bash
# Continuar cronómetro ⏰
cd ..
pnpm create vite torneo-frontend --template react --yes
cd torneo-frontend

# Instalar dependencias críticas
pnpm install
pnpm add axios react-router-dom

# Configurar proxy (vite.config.js)
cat > vite.config.js << 'EOF'
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  server: {
    port: 3000,
    proxy: {
      '/api': {
        target: 'http://localhost:8000',
        changeOrigin: true,
        secure: false,
      }
    }
  }
})
EOF

pnpm dev &
# ✅ Debe estar en http://localhost:3000 en 8 minutos total
# CRONÓMETRO: PARAR ⏰
```

---

## 🏃‍♂️ **Práctica Cronometrada**

### **Ronda 1: Práctica Guiada (20 minutos)**

- **5 minutos:** Demostración del instructor
- **15 minutos:** Práctica grupal paso a paso

### **Ronda 2: Práctica Individual (15 minutos)**

- **Intentos ilimitados:** Repetir hasta lograr < 10 minutos
- **Registro de tiempos:** Anotar cada intento

### **Ronda 3: Competencia Interna (10 minutos)**

- **1 intento final:** Setup completo cronometrado
- **Ranking del grupo:** Quien logre el mejor tiempo

---

## 🎯 **Rúbrica de Evaluación**

| Criterio            | Excelente (4)      | Bueno (3)           | Satisfactorio (2) | Insuficiente (1) |
| ------------------- | ------------------ | ------------------- | ----------------- | ---------------- |
| **Tiempo**          | < 8 minutos        | 8-10 min            | 10-15 min         | > 15 min         |
| **Funcionalidad**   | Todo funciona      | Errores menores     | Ayuda mínima      | No funciona      |
| **Automatismo**     | Sin consultar docs | 1-2 consultas       | Varias consultas  | Dependiente      |
| **Troubleshooting** | Resuelve solo      | Resuelve con pistas | Necesita ayuda    | No resuelve      |

---

## 🔧 **Troubleshooting Común**

### **Problema 1: Composer muy lento**

```bash
# Solución: Usar cache global
composer global require hirak/prestissimo
composer config -g repo.packagist composer https://packagist.org
```

### **Problema 2: pnpm no instalado**

```bash
# Solución rápida
npm install -g pnpm
# Verificar
pnpm --version
```

### **Problema 3: Puerto ocupado**

```bash
# Laravel en puerto alternativo
php artisan serve --port=8001

# React en puerto alternativo
pnpm dev --port 3001
```

### **Problema 4: Permisos SQLite**

```bash
# Dar permisos completos
chmod 777 database/
chmod 666 database/database.sqlite
```

### **Problema 5: CORS bloqueando**

```bash
# Instalar Sanctum para CORS
php artisan install:api
```

---

## 📝 **Hoja de Registro de Tiempos**

| Intento | Tiempo Total | Backend | Frontend | Problemas Encontrados | Notas |
| ------- | ------------ | ------- | -------- | --------------------- | ----- |
| 1       | **_:_**      | **_:_** | **_:_**  |                       |       |
| 2       | **_:_**      | **_:_** | **_:_**  |                       |       |
| 3       | **_:_**      | **_:_** | **_:_**  |                       |       |
| 4       | **_:_**      | **_:_** | **_:_**  |                       |       |
| 5       | **_:_**      | **_:_** | **_:_**  |                       |       |

**Objetivo:** Lograr consistentemente < 10 minutos

---

## 🎯 **Test de Verificación Final**

### **Checklist de Setup Exitoso:**

- [ ] Laravel responde en http://localhost:8000
- [ ] Página de bienvenida Laravel visible
- [ ] Base de datos SQLite creada y conectada
- [ ] React responde en http://localhost:3000
- [ ] Página React por defecto visible
- [ ] Proxy configurado (verificar con F12 → Network)
- [ ] Hot reload funcionando (cambiar algo en App.jsx)

### **Test de Comunicación:**

```bash
# En Laravel: routes/api.php
Route::get('/test', function () {
    return response()->json(['message' => 'API conectada correctamente!', 'timestamp' => now()]);
});

# En React: src/App.jsx - agregar este botón
<button onClick={() => fetch('/api/test').then(r => r.json()).then(console.log)}>
  Test API
</button>
```

---

## 🏆 **Logros a Desbloquear**

- 🥉 **Bronce:** Setup completo en < 15 minutos
- 🥈 **Plata:** Setup completo en < 10 minutos
- 🥇 **Oro:** Setup completo en < 8 minutos
- 💎 **Diamante:** Setup completo en < 6 minutos (¡velocidad ninja!)

---

## 📚 **Script de Automatización (Para Practicar)**

```bash
#!/bin/bash
# setup-worldskills.sh

echo "🚀 Iniciando setup WorldSkills..."
INICIO=$(date +%s)

# Backend
echo "📦 Creando proyecto Laravel..."
composer create-project laravel/laravel torneo-api --no-interaction --quiet
cd torneo-api
touch database/database.sqlite
sed -i 's/DB_CONNECTION=mysql/DB_CONNECTION=sqlite/' .env
php artisan key:generate --no-interaction
php artisan migrate --no-interaction
php artisan serve > /dev/null 2>&1 &
LARAVEL_PID=$!
echo "✅ Laravel listo en http://localhost:8000"

# Frontend
cd ..
echo "⚛️ Creando proyecto React..."
pnpm create vite torneo-frontend --template react --yes > /dev/null
cd torneo-frontend
pnpm install > /dev/null
pnpm add axios react-router-dom > /dev/null

# Configurar proxy
cat > vite.config.js << 'EOF'
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
export default defineConfig({
  plugins: [react()],
  server: {
    port: 3000,
    proxy: {'/api': {target: 'http://localhost:8000', changeOrigin: true}}
  }
})
EOF

pnpm dev > /dev/null 2>&1 &
VITE_PID=$!
echo "✅ React listo en http://localhost:3000"

FIN=$(date +%s)
TIEMPO=$((FIN-INICIO))
echo "🏁 Setup completado en ${TIEMPO} segundos"

echo "🔧 Para detener servidores:"
echo "kill $LARAVEL_PID $VITE_PID"
```

---

**💡 Tip:** ¡Practica este setup hasta que sea completamente automático! En la competencia cada segundo cuenta.
