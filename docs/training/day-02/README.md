# 📅 Día 2: Relaciones Avanzadas y Autenticación

**Duración:** 8 horas  
**Objetivo:** Dominar relaciones complejas, autenticación segura y testing automatizado

---

## 🎯 **Agenda del Día**

### **Mañana (4 horas)**

1. **[Relaciones Eloquent Avanzadas](eloquent-relations.md)** (2.5 horas)
2. **[Autenticación con Sanctum](sanctum-auth.md)** (1.5 horas)

### **Tarde (4 horas)**

3. **[Testing Automatizado](testing-guide.md)** (2 horas)
4. **[Seeders y Factories](seeders-factories.md)** (1 hora)
5. **[Optimización de Queries](query-optimization.md)** (1 hora)

### **🏆 Desafío Final**

6. **[Challenge Día 2](challenge-day2.md)** (2 horas cronometradas)

---

## 📚 **Recursos del Día**

| Archivo                 | Descripción                          | Duración | Tipo              |
| ----------------------- | ------------------------------------ | -------- | ----------------- |
| `eloquent-relations.md` | Relaciones Many-to-Many, Polymorphic | 150 min  | �️ Base de Datos  |
| `sanctum-auth.md`       | API Authentication segura            | 90 min   | � Seguridad       |
| `testing-guide.md`      | PHPUnit y Feature Tests              | 120 min  | 🧪 Testing        |
| `seeders-factories.md`  | Datos de prueba realistas            | 60 min   | 🌱 Seeding        |
| `query-optimization.md` | Performance y N+1 queries            | 60 min   | ⚡ Optimización   |
| `challenge-day2.md`     | **DESAFÍO FINAL**                    | 120 min  | 🏆 **Evaluación** |

---

## ⚡ **Competencias Clave Desarrolladas**

### **Modelado de Datos Deportivos**

✅ **Relaciones One-to-Many** (País → Equipos → Jugadoras)  
✅ **Relaciones Many-to-Many** (Partidos ↔ Equipos con pivot)  
✅ **Relaciones Polymorphic** (Estadísticas para jugadoras/equipos)  
✅ **Constraints de integridad** referencial y validaciones  
✅ **Índices optimizados** para consultas rápidas

### **Autenticación API**

✅ **Laravel Sanctum** para tokens API seguros  
✅ **Middleware de autenticación** personalizado  
✅ **Roles y permisos** básicos (admin/viewer)  
✅ **Rate limiting** para protección de endpoints  
✅ **Password hashing** con bcrypt/argon2

### **Testing y Calidad**

✅ **Feature Tests** para endpoints completos  
✅ **Unit Tests** para modelos y servicios  
✅ **Database Factories** con datos realistas  
✅ **Seeders organizados** por entorno  
✅ **Assertions avanzadas** para validaciones  
✅ **Mocking** de servicios externos

### **Performance y Optimización**

✅ **N+1 Query Problem** resuelto  
✅ **Database indexing** estratégico  
✅ **API Response caching**  
✅ **Frontend code splitting**  
✅ **Bundle optimization** con Vite

---

## 🎯 **Objetivos de Aprendizaje**

Al completar el Día 2, cada competidor debe dominar:

### **Arquitectura Avanzada**

- 🏗️ **Modelos relacionados** con integridad referencial
- 🔐 **Sistema de autenticación** completo y seguro
- 🧪 **Test coverage** > 80% en código crítico
- ⚡ **Queries optimizadas** sin N+1 problems

### **Velocidad de Desarrollo**

- ⏱️ **Relaciones Many-to-Many** en < 15 minutos
- ⏱️ **Auth completo** (login/register/middleware) en < 20 minutos
- ⏱️ **Test suite básico** en < 10 minutos
- ⏱️ **Deploy y optimización** en < 15 minutos

### **Calidad Profesional**

- 🛡️ **Seguridad** siguiendo OWASP guidelines
- 📊 **Performance monitoring** básico
- 🔍 **Error handling** robusto
- 📝 **Documentación** de APIs automática

---

## 📊 **Métricas de Éxito del Día**

| Competencia              | Tiempo Meta | Nivel Requerido                 |
| ------------------------ | ----------- | ------------------------------- |
| **Relaciones Complejas** | < 20 min    | Many-to-Many + Polymorphic      |
| **Auth + Middleware**    | < 25 min    | Login/Register/Protected routes |
| **Test Suite Completo**  | < 15 min    | Feature + Unit tests            |
| **Optimización**         | < 10 min    | Eager loading + indexing        |
| **Desafío Final**        | 120 min     | **≥ 85/100 puntos**             |

---

## 🚀 **Conexión con Día 1**

**Builds upon:**

- ✅ Setup automático Laravel + React (Día 1)
- ✅ CRUD básico con validaciones (Día 1)
- ✅ Hooks y integración frontend (Día 1)

**Adds complexity:**

- 🔗 **Relaciones** entre múltiples entidades
- 🔐 **Autenticación** y autorización
- 🧪 **Testing** automatizado
- ⚡ **Performance** optimizado

---

## 🛠️ **Stack Tecnológico del Día**

### **Backend Avanzado**

- **Laravel Sanctum** para API authentication
- **PHPUnit** para testing automatizado
- **Eloquent Relations** avanzadas
- **Database Seeders** con Factory pattern

### **Frontend Testing**

- **Jest** para unit testing
- **React Testing Library** para componentes
- **MSW** (Mock Service Worker) para API mocking
- **Vitest** integrado con Vite

### **DevOps y Optimización**

- **Laravel Debugbar** para profiling
- **Database indexes** estratégicos
- **API response caching**
- **Vite bundle analyzer**

---

## 💡 **Tips para el Instructor**

### **Gestión de Complejidad**

- 🧩 **Introducir conceptos** progresivamente
- 🔗 **Conectar** cada concepto con casos de uso reales
- 🎯 **Enfocarse** en patrones más utilizados en competencias

### **Hands-on Practice**

- 👨‍💻 **Live coding** de relaciones complejas
- 🧪 **TDD approach** para enseñar testing
- ⚡ **Performance profiling** en vivo

### **Preparación para WorldSkills**

- ⏱️ **Time pressure simulation** en cada ejercicio
- 🏆 **Competition scenarios** realistas
- 🔧 **Troubleshooting** de errores comunes

---

## 🚀 **Preparación para Día 3**

Con las bases del Día 2, estaremos listos para:

- **APIs avanzadas** con filtros, paginación y búsquedas
- **Validaciones complejas** con reglas de negocio
- **Caching estratégico** para performance
- **Documentación automática** con Swagger/OpenAPI

---

## 💡 **Tips para el Instructor**

### **Gestión de Complejidad**

- 🎯 **Incrementar gradualmente** la dificultad
- 🔄 **Conexión constante** con casos de uso deportivos
- 🚨 **Identificar** problemas de comprensión temprano

### **Metodología**

- 📊 **Diagramas ER** para visualizar relaciones
- 🧑‍🤝‍🧑 **Code review** colaborativo del diseño
- 🏃‍♂️ **Mini-challenges** cada 45 minutos

### **Evaluación Continua**

- ✅ **Checkpoints** de integridad referencial
- 🔒 **Verificación** de seguridad implementada
- 🧪 **Ejecución** de tests como validación

---

## 📋 **Prerrequisitos Técnicos**

### **Herramientas Adicionales del Día:**

- **SQLite Browser** para visualizar relaciones
- **Postman/Insomnia** con colecciones de auth
- **PHPUnit** configurado para testing
- **Laravel Telescope** (opcional) para debugging

### **Conocimientos Base:**

- Sintaxis SQL básica (JOINs, constraints)
- Conceptos de REST API y HTTP status codes
- Patrones de testing (Arrange, Act, Assert)
- Fundamentos de seguridad web

---

**🔥 Día 2: ¡Donde la aplicación evoluciona de simple a profesional!**

- **API documentation** automatizada con Swagger

---

**🔥 Día 2: ¡Elevando la complejidad hacia nivel profesional WorldSkills!**
