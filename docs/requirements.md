# Requerimientos Funcionales - API REST Competencia Fútbol Femenino Suramericano

## Información General del Proyecto

**Competencia:** WorldSkills 2025 - Habilidad 17 Tecnologías Web  
**Tiempo límite:** 4 horas  
**Tecnologías:** PHP/Laravel + SQLite + React  
**Tipo de aplicación:** API REST para gestión de competencia deportiva  

---

## Contexto del Proyecto

Se debe desarrollar una aplicación web completa para gestionar un torneo de fútbol femenino suramericano que incluya los países participantes, equipos, jugadoras, partidos y resultados. La aplicación debe permitir tanto la gestión administrativa como la consulta pública de información del torneo.

---

## Requerimientos Funcionales Prioritarios

### RF001 - Gestión de Países
**Prioridad:** Alta  
**Tiempo estimado:** 30 minutos  

**Descripción:** El sistema debe permitir la gestión de países participantes en el torneo.

**Funcionalidades específicas:**
- Crear, leer, actualizar y eliminar países
- Cada país debe tener: código ISO (3 letras), nombre, bandera (URL), confederación
- Validación de códigos ISO únicos
- Al menos 8 países suramericanos predefinidos

**Endpoints requeridos:**
- `GET /api/countries` - Listar todos los países
- `GET /api/countries/{id}` - Obtener país específico
- `POST /api/countries` - Crear nuevo país
- `PUT /api/countries/{id}` - Actualizar país
- `DELETE /api/countries/{id}` - Eliminar país

### RF002 - Gestión de Equipos
**Prioridad:** Alta  
**Tiempo estimado:** 45 minutos  

**Descripción:** El sistema debe permitir la gestión de equipos nacionales femeninos.

**Funcionalidades específicas:**
- Crear, leer, actualizar y eliminar equipos
- Cada equipo debe tener: nombre, país_id, escudo (URL), entrenador, año_fundación
- Un país solo puede tener un equipo en el torneo
- Validación de integridad referencial con países

**Endpoints requeridos:**
- `GET /api/teams` - Listar todos los equipos
- `GET /api/teams/{id}` - Obtener equipo específico
- `POST /api/teams` - Crear nuevo equipo
- `PUT /api/teams/{id}` - Actualizar equipo
- `DELETE /api/teams/{id}` - Eliminar equipo
- `GET /api/teams/{id}/players` - Obtener jugadoras del equipo

### RF003 - Gestión de Jugadoras
**Prioridad:** Alta  
**Tiempo estimado:** 45 minutos  

**Descripción:** El sistema debe permitir la gestión de jugadoras de cada equipo.

**Funcionalidades específicas:**
- Crear, leer, actualizar y eliminar jugadoras
- Cada jugadora debe tener: nombre, apellido, equipo_id, número_camiseta, posición, edad, foto (URL)
- Número de camiseta único por equipo
- Máximo 23 jugadoras por equipo
- Posiciones válidas: Portera, Defensora, Mediocampista, Delantera

**Endpoints requeridos:**
- `GET /api/players` - Listar todas las jugadoras
- `GET /api/players/{id}` - Obtener jugadora específica
- `POST /api/players` - Crear nueva jugadora
- `PUT /api/players/{id}` - Actualizar jugadora
- `DELETE /api/players/{id}` - Eliminar jugadora
- `GET /api/players/team/{team_id}` - Jugadoras por equipo

### RF004 - Gestión de Partidos
**Prioridad:** Alta  
**Tiempo estimado:** 60 minutos  

**Descripción:** El sistema debe permitir la gestión del fixture y partidos del torneo.

**Funcionalidades específicas:**
- Crear, leer, actualizar partidos
- Cada partido debe tener: equipo_local_id, equipo_visitante_id, fecha_hora, estadio, fase, estado
- Estados válidos: Programado, En Curso, Finalizado, Suspendido
- Fases válidas: Fase de Grupos, Cuartos, Semifinal, Tercer Puesto, Final
- Validación: un equipo no puede jugar contra sí mismo
- Validación: no pueden haber partidos simultáneos del mismo equipo

**Endpoints requeridos:**
- `GET /api/matches` - Listar todos los partidos
- `GET /api/matches/{id}` - Obtener partido específico
- `POST /api/matches` - Crear nuevo partido
- `PUT /api/matches/{id}` - Actualizar partido
- `GET /api/matches/team/{team_id}` - Partidos de un equipo
- `GET /api/matches/date/{date}` - Partidos por fecha
- `GET /api/matches/phase/{phase}` - Partidos por fase

### RF005 - Gestión de Resultados
**Prioridad:** Alta  
**Tiempo estimado:** 45 minutos  

**Descripción:** El sistema debe permitir registrar y consultar resultados de partidos.

**Funcionalidades específicas:**
- Registrar resultado de partido (goles local, goles visitante)
- Solo se puede registrar resultado si el partido está "Finalizado"
- Calcular automáticamente ganador/perdedor/empate
- Historial de cambios en resultados

**Endpoints requeridos:**
- `POST /api/matches/{id}/result` - Registrar resultado
- `PUT /api/matches/{id}/result` - Actualizar resultado
- `GET /api/matches/{id}/result` - Obtener resultado

### RF006 - Tabla de Posiciones
**Prioridad:** Media  
**Tiempo estimado:** 30 minutos  

**Descripción:** El sistema debe generar automáticamente la tabla de posiciones.

**Funcionalidades específicas:**
- Calcular puntos (3 por victoria, 1 por empate, 0 por derrota)
- Calcular partidos jugados, ganados, empatados, perdidos
- Calcular goles a favor, goles en contra, diferencia de goles
- Ordenar por: puntos, diferencia de goles, goles a favor

**Endpoints requeridos:**
- `GET /api/standings` - Tabla de posiciones general
- `GET /api/standings/team/{team_id}` - Posición de un equipo específico

### RF007 - Dashboard de Estadísticas
**Prioridad:** Baja  
**Tiempo estimado:** 25 minutos  

**Descripción:** El sistema debe mostrar estadísticas generales del torneo.

**Funcionalidades específicas:**
- Total de partidos jugados/por jugar
- Total de goles marcados
- Promedio de goles por partido
- Equipo con más goles a favor
- Equipo con menos goles en contra

**Endpoints requeridos:**
- `GET /api/statistics/general` - Estadísticas generales
- `GET /api/statistics/teams` - Estadísticas por equipos

---

## Requerimientos Funcionales Opcionales (Si queda tiempo)

### RF008 - Gestión de Goles por Jugadora
**Tiempo estimado:** 20 minutos  
**Descripción:** Registrar qué jugadora marcó cada gol en cada partido.

### RF009 - Búsqueda y Filtros
**Tiempo estimado:** 15 minutos  
**Descripción:** Implementar búsqueda por nombre de jugadora, equipo o país.

### RF010 - Validaciones Avanzadas
**Tiempo estimado:** 15 minutos  
**Descripción:** Validaciones adicionales de negocio y mejores mensajes de error.

---

## Estructura de Base de Datos Sugerida

### Tabla: countries
- id (PK)
- iso_code (VARCHAR(3), UNIQUE)
- name (VARCHAR(100))
- flag_url (VARCHAR(255))
- confederation (VARCHAR(50))
- created_at, updated_at

### Tabla: teams
- id (PK)
- name (VARCHAR(100))
- country_id (FK)
- logo_url (VARCHAR(255))
- coach (VARCHAR(100))
- founded_year (INTEGER)
- created_at, updated_at

### Tabla: players
- id (PK)
- first_name (VARCHAR(50))
- last_name (VARCHAR(50))
- team_id (FK)
- jersey_number (INTEGER)
- position (ENUM)
- age (INTEGER)
- photo_url (VARCHAR(255))
- created_at, updated_at

### Tabla: matches
- id (PK)
- home_team_id (FK)
- away_team_id (FK)
- match_date (DATETIME)
- stadium (VARCHAR(100))
- phase (ENUM)
- status (ENUM)
- home_goals (INTEGER, DEFAULT 0)
- away_goals (INTEGER, DEFAULT 0)
- created_at, updated_at

---

## Consideraciones de Implementación

### Backend (PHP/Laravel)
- Usar migraciones para crear la estructura de BD
- Implementar seeders con datos de prueba
- Usar validación de formularios (Form Requests)
- Implementar relaciones Eloquent adecuadas
- Usar Resource Controllers para APIs RESTful
- Implementar middleware para CORS
- Usar JSON responses consistentes

### Frontend (React)
- Crear componentes reutilizables
- Usar hooks para manejo de estado
- Implementar routing con React Router
- Usar Axios para consumir API
- Diseño responsive con CSS/Bootstrap
- Manejo básico de errores

### Gestión de Tiempo
- **Hora 1:** Setup, modelos, migraciones, seeders
- **Hora 2:** APIs de países, equipos y jugadoras
- **Hora 3:** APIs de partidos y resultados
- **Hora 4:** Frontend básico y tabla de posiciones

---

## Criterios de Evaluación Esperados

1. **Funcionalidad (40%):** APIs funcionando correctamente
2. **Código limpio (25%):** Buenas prácticas, comentarios, estructura
3. **Base de datos (20%):** Diseño correcto, relaciones, validaciones
4. **Frontend (15%):** Interfaz funcional y usable

---

## Datos de Prueba Sugeridos

### Países Participantes:
- Argentina (ARG)
- Brasil (BRA)
- Chile (CHI)
- Colombia (COL)
- Ecuador (ECU)
- Paraguay (PAR)
- Perú (PER)
- Uruguay (URU)

### Fases del Torneo:
1. Fase de Grupos (2 grupos de 4 equipos)
2. Semifinales
3. Tercer y Cuarto Puesto
4. Final

---

*Este documento está diseñado para ser completado en el tiempo límite de 4 horas de la competencia WorldSkills 2025.*