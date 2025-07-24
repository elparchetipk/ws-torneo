# 🏆 Simulacro Completo I - Validación Total

**Duración:** 3 horas (12:30 PM - 3:30 PM)  
**Objetivo:** Implementar sistema completo de gestión del torneo en condiciones de competencia real

---

## 📋 **Especificaciones del Proyecto**

### **Contexto**

Desarrollar un sistema web para gestionar el **Torneo Femenino Suramericano de Fútbol 2025**. El sistema debe permitir la gestión completa de países, equipos, jugadoras, partidos y generar automáticamente la tabla de posiciones.

### **Stack Tecnológico Obligatorio**

- **Backend:** PHP/Laravel 11 + SQLite
- **Frontend:** React 18+ + Vite + Tailwind CSS
- **Tiempo Límite:** 3 horas exactas

---

## 🎯 **Requerimientos Funcionales**

### **RF001: Gestión de Países (30 minutos)**

**Descripción:** Sistema CRUD para países participantes del torneo.

**Criterios de Aceptación:**

- ✅ **Modelo Country** con campos: id, name, code, created_at, updated_at
- ✅ **Migración** correcta con validaciones de BD
- ✅ **API REST** completa: GET, POST, PUT, DELETE `/api/countries`
- ✅ **Validaciones:** name requerido (min 2 chars), code único (2 chars)
- ✅ **Seeder** con países suramericanos: Argentina, Brasil, Colombia, Chile, Uruguay, Paraguay, Perú, Venezuela, Ecuador, Bolivia

**Puntuación:** 15 puntos

- Modelo y migración (5 pts)
- API funcionando (7 pts)
- Validaciones y seeder (3 pts)

### **RF002: Gestión de Equipos (45 minutos)**

**Descripción:** Sistema CRUD para equipos de fútbol femenino.

**Criterios de Aceptación:**

- ✅ **Modelo Team** con campos: id, name, country_id, founded_year, created_at, updated_at
- ✅ **Relación** belongsTo con Country
- ✅ **API REST** completa con información del país
- ✅ **Validaciones:** name requerido, country_id existe, founded_year opcional (1800-2025)
- ✅ **Seeder** con al menos 12 equipos de diferentes países

**Puntuación:** 20 puntos

- Modelo con relaciones (7 pts)
- API con países incluidos (8 pts)
- Validaciones y seeder (5 pts)

### **RF003: Gestión de Jugadoras (45 minutos)**

**Descripción:** Sistema CRUD para jugadoras de los equipos.

**Criterios de Aceptación:**

- ✅ **Modelo Player** con campos: id, first_name, last_name, birth_date, position, jersey_number, team_id
- ✅ **Relación** belongsTo con Team
- ✅ **API REST** completa con información del equipo y país
- ✅ **Validaciones:** nombres requeridos, edad 16-45 años, jersey_number único por equipo (1-99)
- ✅ **Seeder** con al menos 50 jugadoras distribuidas en los equipos

**Puntuación:** 25 puntos

- Modelo con relaciones (8 pts)
- API con relaciones anidadas (10 pts)
- Validaciones complejas (7 pts)

### **RF004: Gestión de Partidos (45 minutos)**

**Descripción:** Sistema para registrar partidos y resultados.

**Criterios de Aceptación:**

- ✅ **Modelo GameMatch** con campos: id, home_team_id, away_team_id, home_goals, away_goals, match_date, phase, played
- ✅ **Relaciones** belongsTo con Team (home y away)
- ✅ **API REST** para partidos con equipos incluidos
- ✅ **Endpoint especial** PUT `/api/matches/{id}/result` para actualizar resultado
- ✅ **Validaciones:** equipos diferentes, fecha futura, goles >= 0
- ✅ **Seeder** con fixture de al menos 20 partidos

**Puntuación:** 25 puntos

- Modelo con relaciones (8 pts)
- API completa (10 pts)
- Endpoint de resultados (7 pts)

### **RF005: Tabla de Posiciones (30 minutos)**

**Descripción:** Cálculo automático de tabla de posiciones.

**Criterios de Aceptación:**

- ✅ **Endpoint** GET `/api/standings` que calcule posiciones
- ✅ **Cálculos:** PJ, PG, PE, PP, GF, GC, DG, Puntos (victoria=3, empate=1)
- ✅ **Ordenamiento:** por puntos desc, diferencia de gol desc, goles a favor desc
- ✅ **Datos incluidos:** información del equipo y país
- ✅ **Performance:** consulta optimizada sin N+1

**Puntuación:** 30 puntos

- Cálculos correctos (15 pts)
- Ordenamiento FIFA (10 pts)
- Performance optimizada (5 pts)

### **RF006: Frontend Funcional (45 minutos)**

**Descripción:** Interfaz React para gestión del torneo.

**Criterios de Aceptación:**

- ✅ **Arquitectura:** Proyecto React con routing funcional
- ✅ **Páginas:** Dashboard, Países, Equipos, Jugadoras, Partidos
- ✅ **CRUD Países:** Listado, crear, editar, eliminar con formularios
- ✅ **CRUD Equipos:** Con select de países funcionando
- ✅ **Dashboard:** Tabla de posiciones básica visible
- ✅ **Integración:** APIs conectadas sin errores
- ✅ **UI:** Tailwind CSS aplicado, responsive básico

**Puntuación:** 35 puntos

- Setup y routing (10 pts)
- CRUDs funcionando (15 pts)
- Dashboard con tabla (10 pts)

---

## ⏰ **Distribución de Tiempo Recomendada**

```
🕐 12:30-13:00 (30min) → RF001: Países
🕐 13:00-13:45 (45min) → RF002: Equipos
🕐 13:45-14:30 (45min) → RF003: Jugadoras
🕐 14:30-15:15 (45min) → RF004: Partidos
🕐 15:15-15:45 (30min) → RF005: Tabla de Posiciones
🕐 15:45-16:30 (45min) → RF006: Frontend
```

**⚠️ IMPORTANTE:** Si vas atrasado, prioriza en este orden:

1. RF001 + RF002 (países y equipos)
2. RF005 (tabla de posiciones)
3. RF006 (frontend básico)
4. RF004 (partidos)
5. RF003 (jugadoras)

---

## 📊 **Rúbrica de Evaluación**

### **Puntuación Total: 150 puntos**

| Rango       | Calificación     | Descripción                            |
| ----------- | ---------------- | -------------------------------------- |
| 135-150 pts | **Excelente**    | Listo para competencia internacional   |
| 120-134 pts | **Muy Bueno**    | Listo con práctica adicional mínima    |
| 105-119 pts | **Bueno**        | Necesita refuerzo en áreas específicas |
| 90-104 pts  | **Suficiente**   | Requiere práctica intensiva adicional  |
| <90 pts     | **Insuficiente** | Necesita refuerzo fundamental          |

### **Criterios de Calidad Adicionales**

**Bonificaciones (+5 pts cada una):**

- ✅ **Código limpio:** Nombres descriptivos, estructura clara
- ✅ **Error handling:** Manejo elegante de errores
- ✅ **UX superior:** Loading states, notificaciones
- ✅ **Performance:** Consultas optimizadas, frontend fluido
- ✅ **Responsive:** UI perfecta en móvil y desktop

**Penalizaciones (-5 pts cada una):**

- ❌ **Errores en consola:** JavaScript o PHP errors
- ❌ **APIs rotas:** Endpoints que no funcionan
- ❌ **UI rota:** Componentes que no renderizan
- ❌ **Validaciones faltantes:** Datos inválidos permitidos
- ❌ **Integración fallida:** Frontend-backend desconectado

---

## 🛠️ **Instrucciones de Setup**

### **Paso 1: Inicialización (5 minutos)**

```bash
# Crear proyecto Laravel
composer create-project laravel/laravel ws-torneo-simulacro1
cd ws-torneo-simulacro1

# Configurar base de datos SQLite
touch database/database.sqlite

# Configurar .env
DB_CONNECTION=sqlite
DB_DATABASE=/absolute/path/to/database/database.sqlite

# Instalar dependencias
composer install
php artisan key:generate
```

### **Paso 2: Frontend Setup (5 minutos)**

```bash
# Crear proyecto React
npm create vite@latest frontend -- --template react
cd frontend

# Instalar dependencias
npm install
npm install react-router-dom tailwindcss @tailwindcss/forms axios
npm install @heroicons/react @headlessui/react

# Configurar Tailwind
npx tailwindcss init -p
```

### **Paso 3: Configuración Inicial (5 minutos)**

```bash
# Configurar CORS en Laravel
php artisan install:api

# Configurar proxy en Vite
# Editar vite.config.js para proxy a Laravel

# Arrancar servidores
php artisan serve &
cd frontend && npm run dev
```

---

## 📝 **Entregables**

Al final del simulacro, debes tener:

### **Backend:**

- ✅ Proyecto Laravel funcional con todas las APIs
- ✅ Base de datos SQLite con datos de prueba
- ✅ Postman collection o endpoints documentados

### **Frontend:**

- ✅ Aplicación React funcionando
- ✅ Navegación entre páginas
- ✅ CRUDs básicos funcionando
- ✅ Dashboard con tabla de posiciones

### **Integración:**

- ✅ Frontend consumiendo APIs Laravel
- ✅ CORS configurado correctamente
- ✅ Error handling básico

---

## 🎯 **Estrategias de Éxito**

### **Para Backend:**

1. **Usa Resource Controllers** para APIs estándar
2. **Implementa API Resources** para respuestas consistentes
3. **Valida con Form Requests** desde el inicio
4. **Testea cada endpoint** inmediatamente con Postman

### **Para Frontend:**

1. **Crea estructura básica** primero
2. **Implementa un CRUD completo** como template
3. **Copia y adapta** para otras entidades
4. **Conecta APIs progresivamente**

### **Para Debugging:**

1. **Lee los errores completos** antes de googlear
2. **Usa console.log** estratégicamente
3. **Verifica Network tab** para requests fallidos
4. **Mantén Laravel logs abiertos**

---

## 🏁 **Finalización**

### **Últimos 15 minutos:**

1. **Testea flujos completos** usuario final
2. **Verifica que no hay errores** en consola
3. **Documenta problemas conocidos**
4. **Prepara demostración** de funcionalidades

### **Demostración (5 minutos):**

1. **Tour rápido** de funcionalidades implementadas
2. **Mostrar tabla de posiciones** calculada
3. **Demonstrar CRUDs** funcionando
4. **Explicar decisiones técnicas** principales

---

**¡Es tu momento de brillar! Demuestra todo lo que has aprendido. 🌟**
