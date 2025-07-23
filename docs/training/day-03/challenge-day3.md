# 🏆 DESAFÍO FINAL DÍA 3: APIs Avanzadas y Validaciones Complejas

**Tiempo límite:** 4 horas ⏱️  
**Nivel:** Avanzado (WorldSkills Competition Level)  
**Puntuación total:** 100 puntos

---

## 🎯 **ESCENARIO DEL DESAFÍO**

Eres el **Lead Developer** de la **WorldSkills Tournament Platform 2025**. El sistema debe soportar **10,000 usuarios concurrentes** durante el torneo final, procesando **transferencias de jugadores en tiempo real**, **estadísticas complejas**, y **documentación automática** para múltiples APIs externas.

### **CONTEXTO CRÍTICO**

- El torneo inicia en **48 horas**
- Se esperan **50 equipos** con **1,200 jugadores** registrados
- Las APIs deben manejar **transferencias de última hora**
- Sistema de **monitoreo en tiempo real** para árbitros y organizadores
- **Documentación automática** para partners internacionales

---

## 📋 **REQUERIMIENTOS TÉCNICOS**

### **PARTE 1: Validaciones Complejas y Transferencias (25 pts)**

#### **R1.1 - Sistema de Transferencias (10 pts)**

```php
// Implementar transferencias de jugadores con validaciones complejas

POST /api/transfers
{
    "player_id": 15,
    "from_team_id": 3,
    "to_team_id": 7,
    "transfer_type": "loan|permanent|emergency",
    "valid_from": "2024-02-01T00:00:00Z",
    "valid_until": "2024-06-30T23:59:59Z",
    "transfer_fee": 50000.00,
    "documents": [
        "contract.pdf",
        "medical_clearance.pdf"
    ],
    "emergency_reason": "injury_replacement", // Solo si tipo emergency
    "fifa_approval": {
        "approval_number": "FIFA-2024-0156",
        "approved_by": "John Smith",
        "approved_at": "2024-01-28T14:30:00Z"
    }
}
```

**Validaciones requeridas:**

- ✅ Jugador existe y pertenece al equipo origen
- ✅ Equipo destino no excede límite de 25 jugadores
- ✅ No puede transferir más de 3 jugadores por ventana
- ✅ Transferencias de emergencia requieren justificación médica
- ✅ Validar ventana de transferencias (solo enero y julio)
- ✅ Documentos PDF válidos < 5MB cada uno
- ✅ FIFA approval requerido para transferencias > €100,000

#### **R1.2 - Validaciones de Partidos Simultáneos (8 pts)**

```php
// Validar conflictos de horarios para múltiples partidos

POST /api/matches/bulk-schedule
{
    "matches": [
        {
            "home_team_id": 1,
            "away_team_id": 2,
            "match_date": "2024-02-15T15:00:00Z",
            "venue": "Stadium A",
            "referee_id": 10
        },
        {
            "home_team_id": 3,
            "away_team_id": 4,
            "match_date": "2024-02-15T15:30:00Z",
            "venue": "Stadium B",
            "referee_id": 11
        }
    ]
}
```

**Validaciones requeridas:**

- ✅ No hay conflictos de equipos (misma fecha ±3 horas)
- ✅ Venues disponibles (no overlapping + tiempo de limpieza)
- ✅ Árbitros sin conflictos de horario
- ✅ Equipos tienen mínimo 16 jugadores disponibles
- ✅ No exceder 4 partidos por equipo por semana

#### **R1.3 - Custom Validation Rules (7 pts)**

```php
// Crear reglas personalizadas reutilizables

- ValidTransferWindow: Verificar ventana de transferencias
- ValidTeamCapacity: Límites por posición y total
- ValidMedicalClearance: Documentos médicos válidos
- ValidFifaApproval: Aprobaciones FIFA requeridas
- ValidVenueAvailability: Disponibilidad de venues
```

---

### **PARTE 2: Filtros y Paginación Avanzada (20 pts)**

#### **R2.1 - Sistema de Filtros Dinámicos (10 pts)**

```php
// Endpoint con filtros complejos y búsqueda full-text

GET /api/players/advanced-search?
    search=Ronaldo Messi&
    positions[]=MID&positions[]=FWD&
    age_range[min]=22&age_range[max]=35&
    height_range[min]=1.75&height_range[max]=1.90&
    transfer_status=available&
    injury_status=fit&
    contract_expires_before=2024-12-31&
    market_value[min]=1000000&market_value[max]=50000000&
    nationality[]=Brazilian&nationality[]=Argentine&
    team_ids[]=1,2,3,4&
    tournaments[]=champions_league&tournaments[]=world_cup&
    statistics[goals][min]=10&
    statistics[assists][min]=5&
    sort_by=market_value&
    sort_direction=desc&
    include=team.tournament,statistics,transfers,contracts&
    page=2&
    per_page=25
```

**Filtros requeridos:**

- ✅ Búsqueda full-text en múltiples campos
- ✅ Filtros por rangos (edad, altura, valor mercado)
- ✅ Filtros por arrays (posiciones, nacionalidades, equipos)
- ✅ Filtros por relaciones (torneos, estadísticas)
- ✅ Filtros por fechas (contratos, transferencias)
- ✅ Ordenamiento por campos calculados

#### **R2.2 - Paginación Inteligente (5 pts)**

```php
// Respuesta con metadata avanzada

{
    "data": [...],
    "meta": {
        "current_page": 2,
        "from": 26,
        "to": 50,
        "per_page": 25,
        "total": 1247,
        "last_page": 50,
        "has_more_pages": true,
        "total_teams": 45,
        "position_distribution": {
            "GK": 124,
            "DEF": 398,
            "MID": 456,
            "FWD": 269
        },
        "nationality_stats": {
            "Brazilian": 156,
            "Argentine": 98,
            "Colombian": 87
        },
        "age_stats": {
            "average": 26.4,
            "median": 25,
            "min": 18,
            "max": 39
        },
        "market_value_stats": {
            "total": 2500000000,
            "average": 2004816,
            "top_10_average": 45000000
        }
    },
    "links": {...},
    "filters": {...},
    "performance": {
        "query_time_ms": 156,
        "cache_hit": false,
        "queries_executed": 8
    }
}
```

#### **R2.3 - Búsqueda con Scoring (5 pts)**

```php
// Sistema de relevancia en búsquedas

GET /api/search?q=Real Madrid Benzema&type=scored

// Algoritmo de scoring:
// - Coincidencia exacta: 10 puntos
// - Coincidencia al inicio: 8 puntos
// - Coincidencia de palabra: 6 puntos
// - Coincidencia parcial: 3 puntos
// - Relación (equipo/torneo): 2 puntos
```

---

### **PARTE 3: Caching y Performance (20 pts)**

#### **R3.1 - Sistema de Cache Multi-Nivel (12 pts)**

```php
// Implementar cache strategy para diferentes tipos de datos

- Tournament standings: Cache 5 min, invalidar en updates
- Player statistics: Cache 15 min, warm-up automático
- Match results: Cache 1 hora, ETag support
- Live scores: Cache 30 seg, WebSocket updates
- User preferences: Cache 24 horas, per-user
- API responses: HTTP cache + conditional requests
```

**Cache levels requeridos:**

- ✅ Application Cache (Redis)
- ✅ Database Query Cache
- ✅ HTTP Response Cache (ETag + Last-Modified)
- ✅ CDN Cache para assets estáticos

#### **R3.2 - Query Optimization (5 pts)**

```php
// Optimizar queries complejas con índices

- Eager loading para evitar N+1
- Subqueries para counts y aggregations
- Database indexes para filtros comunes
- Raw queries para estadísticas complejas
```

#### **R3.3 - Performance Monitoring (3 pts)**

```php
// Middleware de monitoreo automático

Headers requeridos:
- X-Response-Time: tiempo en ms
- X-Memory-Usage: memoria en MB
- X-Query-Count: número de queries
- X-Cache-Status: HIT|MISS|BYPASS
```

---

### **PARTE 4: Documentación Automática (15 pts)**

#### **R4.1 - OpenAPI 3.0 Completa (8 pts)**

```php
// Documentación con ejemplos interactivos

- Todos los endpoints documentados
- Schemas completos con validaciones
- Ejemplos de request/response
- Códigos de error con descripciones
- Authentication schemes (Sanctum)
- Múltiples servers (dev/prod)
```

#### **R4.2 - Swagger UI Personalizado (4 pts)**

```php
// Interface personalizada con branding WorldSkills

- Logo y colores corporativos
- Quick start guide integrado
- Token authentication helper
- Download de Postman collection
- Performance metrics display
```

#### **R4.3 - Contract Testing (3 pts)**

```php
// Tests automáticos de contratos API

- Validación de response schemas
- Testing de error responses
- Validación de headers
- Performance benchmarks
```

---

### **PARTE 5: Error Handling Profesional (20 pts)**

#### **R5.1 - Jerarquía de Excepciones (8 pts)**

```php
// Sistema completo de excepciones personalizadas

- BaseApiException con context y logging
- BusinessLogicException para reglas de negocio
- ExternalServiceException con circuit breaker
- ValidationException con mensajes personalizados
- DatabaseException con retry logic
```

#### **R5.2 - Global Exception Handler (6 pts)**

```php
// Handler unificado con logging estructurado

- Formato consistente de errores JSON
- Logging por niveles (business/system/audit)
- Trace IDs para debugging
- Notificaciones automáticas para errores críticos
- Debug info solo en desarrollo
```

#### **R5.3 - Error Recovery (6 pts)**

```php
// Estrategias de recuperación automática

- Circuit breaker para servicios externos
- Retry logic con exponential backoff
- Graceful degradation con fallbacks
- Cache como último recurso
- Health checks automáticos
```

---

## 🛠️ **ENTREGABLES**

### **ESTRUCTURA DE ARCHIVOS REQUERIDA**

```text
app/
├── Http/
│   ├── Controllers/Api/
│   │   ├── TransferController.php
│   │   ├── AdvancedPlayerController.php
│   │   └── BulkOperationsController.php
│   ├── Requests/
│   │   ├── TransferPlayerRequest.php
│   │   ├── BulkScheduleMatchesRequest.php
│   │   └── AdvancedSearchRequest.php
│   ├── Filters/
│   │   ├── AdvancedPlayerFilter.php
│   │   └── TransferFilter.php
│   └── Middleware/
│       ├── PerformanceMonitor.php
│       └── CacheResponse.php
├── Rules/
│   ├── ValidTransferWindow.php
│   ├── ValidTeamCapacity.php
│   ├── ValidMedicalClearance.php
│   └── ValidVenueAvailability.php
├── Services/
│   ├── TransferService.php
│   ├── CacheService.php
│   ├── SearchService.php
│   └── PerformanceService.php
├── Exceptions/
│   ├── BaseApiException.php
│   ├── Business/
│   │   ├── TransferException.php
│   │   └── CapacityException.php
│   └── External/
│       └── FifaApiException.php
└── Models/
    ├── Transfer.php
    └── PlayerContract.php

database/migrations/
├── create_transfers_table.php
└── add_performance_indexes.php

tests/Feature/
├── TransferApiTest.php
├── AdvancedSearchTest.php
├── CachingTest.php
└── ErrorHandlingTest.php

docs/
└── api-documentation/
    ├── openapi.yaml
    └── postman-collection.json
```

---

## 🎯 **CRITERIOS DE EVALUACIÓN**

| Componente                 | Puntos | Criterios de Excelencia                                                                                                        |
| -------------------------- | ------ | ------------------------------------------------------------------------------------------------------------------------------ |
| **Validaciones Complejas** | 25     | ✅ Form Requests completos<br>✅ Custom Rules reutilizables<br>✅ Validaciones condicionales<br>✅ Manejo de arrays y archivos |
| **Filtros Avanzados**      | 20     | ✅ Query Builder dinámico<br>✅ Búsqueda full-text con scoring<br>✅ Paginación con metadata<br>✅ Performance < 200ms         |
| **Caching Multi-Nivel**    | 20     | ✅ Redis + HTTP cache<br>✅ Invalidación inteligente<br>✅ ETag + Last-Modified<br>✅ Performance monitoring                   |
| **Documentación**          | 15     | ✅ OpenAPI 3.0 completa<br>✅ Swagger UI personalizado<br>✅ Contract testing<br>✅ Ejemplos funcionales                       |
| **Error Handling**         | 20     | ✅ Jerarquía de excepciones<br>✅ Logging estructurado<br>✅ Recovery automático<br>✅ Circuit breaker                         |

---

## 🏁 **BONUS CHALLENGES (+20 pts extra)**

### **B1. WebSocket Real-Time Updates (+8 pts)**

```php
// Implementar actualizaciones en tiempo real
- Live scores durante partidos
- Notificaciones de transferencias
- Updates de standings automáticos
```

### **B2. API Rate Limiting Inteligente (+6 pts)**

```php
// Rate limiting por tipo de usuario
- Admin: 1000 req/min
- Coach: 500 req/min
- Fan: 100 req/min
- Burst allowance con token bucket
```

### **B3. Multi-Language Support (+6 pts)**

```php
// Soporte multi-idioma para errores
- Headers Accept-Language
- Mensajes de error traducidos
- Documentación en inglés/español
```

---

## 📊 **MÉTRICAS DE RENDIMIENTO ESPERADAS**

| Métrica          | Target  | Excelente |
| ---------------- | ------- | --------- |
| Response Time    | < 500ms | < 200ms   |
| Concurrent Users | 1,000   | 10,000    |
| Cache Hit Rate   | > 80%   | > 95%     |
| Error Rate       | < 1%    | < 0.1%    |
| Uptime           | 99.9%   | 99.99%    |
| Memory Usage     | < 128MB | < 64MB    |

---

## 🚀 **COMMANDS DE INICIO**

```bash
# Setup inicial
php artisan migrate --seed
php artisan l5-swagger:generate
php artisan cache:clear

# Tests de performance
ab -n 1000 -c 10 http://localhost:8000/api/players
wrk -t12 -c400 -d30s http://localhost:8000/api/matches

# Monitoring
php artisan queue:work
php artisan schedule:run

# Verificación final
php artisan test --group=integration
curl http://localhost:8000/api/documentation
```

---

## 🏆 **CRITERIOS DE APROBACIÓN**

### **MÍNIMO PARA APROBAR (70/100 pts):**

- ✅ Validaciones básicas funcionando
- ✅ Filtros y paginación implementados
- ✅ Cache básico configurado
- ✅ Documentación OpenAPI básica
- ✅ Error handling global

### **NIVEL WORLDSKILLS (90+/100 pts):**

- ✅ Todos los requerimientos implementados
- ✅ Performance < 200ms promedio
- ✅ Cache hit rate > 90%
- ✅ Documentación interactiva completa
- ✅ Error recovery automático

### **MEDALLA DE ORO (100+/120 pts):**

- ✅ Todos los bonus completados
- ✅ Performance excepcional
- ✅ Código limpio y bien documentado
- ✅ Tests comprensivos
- ✅ Innovación en implementación

---

## ⏰ **TIMELINE SUGERIDO**

| Tiempo          | Actividad                      | Puntos |
| --------------- | ------------------------------ | ------ |
| **0-60 min**    | Setup + Validaciones complejas | 25 pts |
| **60-120 min**  | Filtros avanzados + Paginación | 20 pts |
| **120-180 min** | Sistema de cache multi-nivel   | 20 pts |
| **180-210 min** | Documentación automática       | 15 pts |
| **210-240 min** | Error handling + Testing       | 20 pts |

---

**🏆 ¡Este es el desafío final del Día 3! Demuestra que puedes crear APIs de nivel WorldSkills que soporten miles de usuarios con performance excepcional y documentación profesional!**

**🔥 ¡Que comience la competencia!** ⏱️
