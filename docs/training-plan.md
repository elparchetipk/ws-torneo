# Plan de Entrenamiento WorldSkills 2025 - Habilidad 17 Tecnologías Web

## Información General del Plan

**Objetivo Principal:** Preparar competidores para desarrollar una API REST completa en 4 horas  
**Duración del entrenamiento:** 5 días (40 horas de práctica)  
**Stack Tecnológico:** PHP/Laravel + SQLite + React  
**Metodología:** Práctica incremental con simulacros cronometrados  
**Modalidad:** Presencial con seguimiento individualizado

---

## Filosofía Pedagógica del Entrenamiento

### Principios Metodológicos

**Aprendizaje Progresivo por Capas:** Cada día construye sobre el conocimiento del anterior, desde fundamentos hasta integración completa. Comenzamos con herramientas básicas y avanzamos hacia la orquestación de sistemas completos.

**Práctica Deliberada:** Cada ejercicio está diseñado para desarrollar habilidades específicas necesarias en la competencia. No hay actividades "de relleno", cada minuto tiene un propósito claro.

**Simulación de Condiciones Reales:** A partir del día 2, todos los ejercicios se realizan bajo presión de tiempo similar a la competencia, desarrollando tanto habilidades técnicas como resistencia mental.

**Retroalimentación Inmediata:** Cada actividad incluye rúbricas claras y momentos de autoevaluación para que los competidores identifiquen áreas de mejora en tiempo real.

---

## DÍA 1: Fundamentos y Configuración del Entorno
**Duración total:** 8 horas  
**Objetivo:** Crear automatismos para setup y desarrollo básico

### Sesión Matutina (4 horas)

#### 1.1 Configuración del Entorno de Desarrollo (1 hora)
**Objetivo de Aprendizaje:** Los competidores deben poder configurar un entorno completo Laravel + React en menos de 10 minutos, sin consultar documentación.

**Metodología:**
- Demostración paso a paso del instructor (15 minutos)
- Práctica guiada en grupo (20 minutos)  
- Práctica individual cronometrada (15 minutos)
- Troubleshooting de problemas comunes (10 minutos)

**Actividades Específicas:**
Los competidores practicarán la instalación desde cero múltiples veces hasta lograr automatismo. Se enfocará en identificar y resolver problemas típicos de configuración que pueden surgir durante la competencia.

**Rúbrica de Evaluación:**
- **Excelente (4 puntos):** Setup completo en menos de 8 minutos, sin errores
- **Bueno (3 puntos):** Setup en 8-10 minutos, errores menores autocorregidos
- **Satisfactorio (2 puntos):** Setup en 10-15 minutos, requiere ayuda mínima
- **Insuficiente (1 punto):** No logra completar setup funcional en 15 minutos

#### 1.2 Migraciones y Modelos Laravel (1.5 horas)
**Objetivo de Aprendizaje:** Dominar la creación rápida de estructuras de base de datos complejas con relaciones múltiples.

**Metodología:**
- Análisis de patrones de diseño de BD deportivas (20 minutos)
- Ejercicio práctico: sistema de blog con usuarios, posts y comentarios (40 minutos)
- Práctica cronometrada: recrear estructura desde cero (20 minutos)
- Revisión de errores comunes y mejores prácticas (10 minutos)

**Habilidades Desarrolladas:**
Los competidores aprenderán a identificar relaciones complejas rápidamente y traducirlas a migraciones eficientes. Se enfatiza la importancia de las restricciones de integridad y las validaciones a nivel de base de datos.

#### 1.3 APIs RESTful con Laravel (1.5 horas)
**Objetivo de Aprendizaje:** Crear endpoints REST completos siguiendo convenciones estándar, con manejo de errores robusto.

**Metodología:**
- Explicación teórica de principios REST (15 minutos)
- Demostración de Resource Controllers y API Resources (25 minutos)
- Ejercicio guiado: CRUD completo de entidad simple (30 minutos)
- Práctica individual: implementar validaciones y manejo de errores (20 minutos)

**Enfoque Pedagógico:**
Se hace énfasis en la consistencia de respuestas JSON y el manejo elegante de errores, aspectos críticos que diferencia APIs profesionales de implementaciones básicas.

### Sesión Vespertina (4 horas)

#### 1.4 React Hooks y Consumo de APIs (2 horas)
**Objetivo de Aprendizaje:** Dominar el patrón moderno de React con hooks para consumo eficiente de APIs RESTful.

**Metodología:**
- Introducción a hooks con ejemplos comparativos (30 minutos)
- Ejercicio dirigido: componente de lista con fetch de datos (45 minutos)
- Práctica: implementar loading states y manejo de errores (30 minutos)
- Optimización: introducción a custom hooks (15 minutos)

**Desarrollo de Habilidades:**
Los competidores desarrollarán intuición para manejar estados asíncronos complejos y crear interfaces responsive que manejen elegantemente los tiempos de carga.

#### 1.5 Integración Frontend-Backend (2 horas)
**Objetivo de Aprendizaje:** Establecer comunicación fluida entre React y Laravel, resolviendo problemas comunes de CORS y configuración.

**Metodología:**
- Configuración de desarrollo con proxy (20 minutos)
- Resolución de problemas de CORS (25 minutos)
- Ejercicio integral: formulario con validación en ambos extremos (55 minutos)
- Testing manual con herramientas de desarrollo (20 minutos)

**Desafío del Día:**
Simulacro de 2 horas para crear un CRUD completo de entidad simple con frontend y backend funcionales. Este ejercicio establece la línea base de velocidad de desarrollo de cada competidor.

---

## DÍA 2: Modelado de Datos Deportivos
**Duración total:** 8 horas  
**Objetivo:** Dominar el modelado específico del dominio deportivo con todas sus complejidades

### Sesión Matutina (4 horas)

#### 2.1 Análisis del Dominio Deportivo (1 hora)
**Objetivo de Aprendizaje:** Desarrollar capacidad de análisis rápido de dominios complejos y identificación de reglas de negocio críticas.

**Metodología:**
- Análisis colaborativo del documento de requerimientos (20 minutos)
- Diagramado en pizarra de entidades y relaciones (25 minutos)
- Identificación grupal de reglas de negocio no obvias (15 minutos)

**Enfoque Pedagógico:**
Se entrena la habilidad de "leer entre líneas" en especificaciones técnicas, identificando reglas implícitas que pueden no estar explícitamente documentadas pero son críticas para el funcionamiento correcto del sistema.

#### 2.2 Diseño de Migraciones Complejas (1.5 horas)
**Objetivo de Aprendizaje:** Crear migraciones que capturen todas las restricciones del dominio deportivo, incluyendo validaciones a nivel de base de datos.

**Metodología:**
- Demostración de restricciones complejas (unique composite, foreign keys) (20 minutos)
- Ejercicio guiado: migración de equipos con restricciones (35 minutos)
- Práctica individual: migración de jugadoras con validaciones (25 minutos)
- Revisión de errores típicos y debugging (10 minutos)

**Desarrollo de Habilidades:**
Los competidores aprenderán a anticipar problemas de integridad de datos y diseñar esquemas robustos que prevengan estados inconsistentes desde el nivel de base de datos.

#### 2.3 Modelos Eloquent con Relaciones Avanzadas (1.5 horas)
**Objetivo de Aprendizaje:** Implementar relaciones complejas que soporten todas las consultas necesarias para el dominio deportivo.

**Metodología:**
- Explicación de relaciones polimórficas y complejas (20 minutos)
- Implementación guiada de modelo Team con múltiples relaciones (40 minutos)
- Ejercicio: crear queries complejas con relaciones anidadas (20 minutos)
- Optimización de consultas N+1 (10 minutos)

### Sesión Vespertina (4 horas)

#### 2.4 Seeders con Datos Realistas (2 horas)
**Objetivo de Aprendizaje:** Crear conjuntos de datos de prueba que permitan testing realista de todas las funcionalidades.

**Metodología:**
- Análisis de requisitos de datos para testing completo (15 minutos)
- Implementación de seeders con datos de países suramericanos reales (45 minutos)
- Creación de fixture de partidos balanceado (45 minutos)
- Verificación de integridad de datos generados (15 minutos)

**Enfoque Pedagógico:**
Se enfatiza la importancia de datos de prueba realistas para identificar edge cases y problemas que solo aparecen con volúmenes y variedad de datos reales.

#### 2.5 Validaciones del Dominio (2 horas)
**Objetivo de Aprendizaje:** Implementar todas las reglas de negocio mediante validaciones robustas que manejen casos edge graciosamente.

**Metodología:**
- Identificación sistemática de todas las reglas de validación (20 minutos)
- Implementación de Form Requests con validaciones complejas (60 minutos)
- Testing de casos edge y manejo de errores (30 minutos)
- Creación de mensajes de error informativos (10 minutos)

**Desafío del Día:**
Simulacro cronometrado de 2 horas: implementar modelo completo de equipos y jugadoras con todas las validaciones, partiendo desde migraciones hasta seeders funcionales.

---

## DÍA 3: APIs y Controladores Avanzados
**Duración total:** 8 horas  
**Objetivo:** Implementar todos los endpoints requeridos con eficiencia y robustez

### Sesión Matutina (4 horas)

#### 3.1 Arquitectura de Controladores Resource (1.5 horas)
**Objetivo de Aprendizaje:** Dominar el patrón Resource Controller para crear APIs consistentes y mantenibles rápidamente.

**Metodología:**
- Análisis de estructura óptima de controladores (20 minutos)
- Implementación paso a paso de CountryController completo (40 minutos)
- Práctica: TeamController con relaciones (25 minutos)
- Revisión de convenciones y consistencia (5 minutos)

**Desarrollo de Habilidades:**
Los competidores desarrollarán velocidad para crear controladores siguiendo patrones establecidos, reduciendo tiempo de decisión y aumentando consistencia.

#### 3.2 API Resources para Respuestas Estructuradas (1 hora)
**Objetivo de Aprendizaje:** Crear respuestas JSON consistentes y bien estructuradas que faciliten el consumo desde el frontend.

**Metodología:**
- Demostración de API Resources vs respuestas directas (15 minutos)
- Implementación de Resources con relaciones condicionales (30 minutos)
- Práctica: Resources anidados para entidades complejas (15 minutos)

#### 3.3 Endpoints Especializados del Dominio (1.5 horas)
**Objetivo de Aprendizaje:** Implementar endpoints específicos que no siguen el patrón REST estándar pero son necesarios para el dominio.

**Metodología:**
- Análisis de endpoints no-CRUD necesarios (15 minutos)
- Implementación de filtros por fecha, equipo, fase (45 minutos)
- Endpoint para actualización de resultados de partidos (25 minutos)
- Testing manual con casos complejos (5 minutos)

### Sesión Vespertina (4 horas)

#### 3.4 Tabla de Posiciones y Cálculos Automáticos (2.5 horas)
**Objetivo de Aprendizaje:** Implementar lógica de negocio compleja para cálculos deportivos automáticos.

**Metodología:**
- Análisis de requerimientos de tabla de posiciones (20 minutos)
- Implementación step-by-step de cálculos estadísticos (80 minutos)
- Algoritmo de ordenamiento según criterios FIFA (40 minutos)
- Optimización de performance para consultas complejas (30 minutos)

**Enfoque Pedagógico:**
Este es el componente más complejo del sistema. Se enfatiza la importancia de dividir problemas complejos en partes manejables y testear cada parte individualmente.

#### 3.5 Testing de APIs (1.5 horas)
**Objetivo de Aprendizaje:** Crear estrategias de testing eficientes que validen funcionalidad sin consumir tiempo excesivo.

**Metodología:**
- Introducción a testing de APIs con herramientas manuales (20 minutos)
- Creación de colección Postman sistemática (40 minutos)
- Testing de casos edge y validaciones (20 minutos)
- Documentación básica de endpoints (10 minutos)

**Desafío del Día:**
Simulacro intensivo de 3 horas: implementar todos los endpoints de RF001-RF005 partiendo de modelos existentes. El objetivo es validar que pueden completar el backend completo en tiempo récord.

---

## DÍA 4: Frontend React Profesional
**Duración total:** 8 horas  
**Objetivo:** Crear interfaz completa, funcional y visualmente atractiva

### Sesión Matutina (4 horas)

#### 4.1 Arquitectura de Proyecto React (1 hora)
**Objetivo de Aprendizaje:** Establecer estructura de proyecto que facilite desarrollo rápido y mantenible.

**Metodología:**
- Análisis de estructuras de carpetas óptimas (15 minutos)
- Setup de herramientas de desarrollo (Tailwind, React Router) (25 minutos)
- Creación de plantillas de componentes reutilizables (20 minutos)

**Enfoque Pedagógico:**
Se enfatiza que una buena organización inicial ahorra tiempo significativo durante el desarrollo intensivo.

#### 4.2 Sistema de Hooks Personalizados (1.5 horas)
**Objetivo de Aprendizaje:** Crear abstraciones que simplifiquen el consumo de APIs y manejo de estado.

**Metodología:**
- Demostración de custom hooks para APIs (25 minutos)
- Implementación guiada de useApi hook genérico (35 minutos)
- Práctica: hooks específicos para entidades del dominio (25 minutos)
- Patterns para manejo de loading y errores (5 minutos)

#### 4.3 Componentes de Lista y Formularios (1.5 horas)
**Objetivo de Aprendizaje:** Crear componentes reutilizables que aceleren el desarrollo de CRUDs.

**Metodología:**
- Diseño de componentes de lista con funcionalidad completa (45 minutos)
- Implementación de formularios con validación (35 minutos)
- Patterns para reutilización y composición (10 minutos)

### Sesión Vespertina (4 horas)

#### 4.4 Integración Frontend-Backend Completa (2 horas)
**Objetivo de Aprendizaje:** Conectar todos los componentes React con APIs Laravel de manera eficiente.

**Metodología:**
- Configuración de servicios de API centralizados (30 minutos)
- Implementación de manejo de errores global (30 minutos)
- Testing de flujos completos usuario-API-base de datos (45 minutos)
- Optimización de performance y UX (15 minutos)

#### 4.5 Componentes Avanzados Específicos del Dominio (2 horas)
**Objetivo de Aprendizaje:** Crear componentes especializados para funcionalidades deportivas únicas.

**Metodología:**
- Tabla de posiciones con ordenamiento dinámico (60 minutos)
- Dashboard de estadísticas con visualizaciones básicas (45 minutos)
- Componentes de partidos con estados visuales (15 minutos)

**Desafío del Día:**
Simulacro de 3 horas: crear frontend completo para gestión de países y equipos, incluyendo listados, formularios y validaciones, conectado a API real.

---

## DÍA 5: Integración Total y Simulacros de Competencia
**Duración total:** 8 horas  
**Objetivo:** Validar preparación completa mediante simulacros cronometrados

### Sesión Matutina (4 horas)

#### 5.1 Simulacro Completo I (4 horas)
**Objetivo de Aprendizaje:** Ejecutar el reto completo en condiciones idénticas a la competencia.

**Metodología del Simulacro:**
- **Preparación (15 minutos):** Lectura de requerimientos, planificación de tiempo
- **Implementación (3h 30min):** Desarrollo sin interrupciones ni ayuda externa
- **Presentación (15 minutos):** Demostración de funcionalidades implementadas

**Distribución de Tiempo Objetivo:**
- Hora 1: Setup, migraciones, modelos y seeders
- Hora 2: APIs de países, equipos y jugadoras
- Hora 3: APIs de partidos, resultados y tabla de posiciones  
- Hora 4: Frontend básico con CRUDs principales

**Rúbrica de Evaluación Simulacro:**
- **RF001-RF003 completados (30%):** CRUDs básicos funcionando
- **RF004-RF005 completados (25%):** Gestión de partidos y resultados
- **RF006 completado (20%):** Tabla de posiciones funcional
- **Frontend funcional (15%):** Interfaz usable para operaciones principales
- **Calidad de código (10%):** Buenas prácticas y organización

### Sesión Vespertina (4 horas)

#### 5.2 Análisis de Resultados y Mejoras (1 hora)
**Objetivo de Aprendizaje:** Identificar áreas de mejora específicas basadas en performance del simulacro.

**Metodología:**
- Autoevaluación estructurada con rúbricas (20 minutos)
- Revisión de código y identificación de problemas (25 minutos)
- Plan personalizado de mejoras (15 minutos)

#### 5.3 Simulacro Completo II (3 horas)
**Objetivo de Aprendizaje:** Validar mejoras y confirmar readiness para competencia.

**Enfoque del Segundo Simulacro:**
Se reduce el tiempo a 3 horas para simular presión adicional y forzar priorización efectiva. Los competidores deben demostrar capacity de completar funcionalidades core bajo presión extrema.

**Criterios de Aprobación:**
- Completar RF001-RF005 en tiempo asignado
- Código funcional sin errores críticos
- Interface básica pero funcional
- Demostración fluida de funcionalidades

---

## Sistema de Evaluación Continua

### Rúbricas Diarias de Progreso

#### Día 1-2: Fundamentos Técnicos
- **Velocidad de Setup (25%):** Tiempo para configurar entorno funcional
- **Modelado de Datos (35%):** Correctitud de migraciones y relaciones
- **APIs Básicas (40%):** Funcionalidad de endpoints CRUD

#### Día 3-4: Funcionalidad Avanzada
- **Completitud de APIs (40%):** Todos los endpoints requeridos funcionando
- **Manejo de Errores (25%):** Validaciones y respuestas de error apropiadas
- **Frontend Funcional (35%):** Interfaces usables para todas las operaciones

#### Día 5: Integración Total
- **Performance bajo Presión (50%):** Capacidad de entregar bajo tiempo límite
- **Priorización (25%):** Decisiones correctas sobre qué implementar primero
- **Calidad Final (25%):** Estado del código y funcionalidad al final del tiempo

### Indicadores de Readiness para Competencia

**Competidor Listo:**
- Completa simulacros en tiempo asignado con 80%+ de funcionalidad
- Demuestra automatismo en tareas básicas (setup, CRUDs simples)
- Mantiene calma bajo presión y prioriza efectivamente
- Código organizado y funcional sin debugging extensivo

**Competidor Necesita Más Práctica:**
- Requiere más tiempo del asignado para funcionalidad básica
- Errores frecuentes en configuración o sintaxis básica
- Dificultad para priorizar bajo presión
- Código que requiere debugging significativo para funcionar

---

## Recursos de Apoyo Continuado

### Materiales de Referencia Rápida
- Cheat sheets de comandos Laravel más utilizados
- Snippets de código para patrones comunes
- Lista de validaciones típicas del dominio deportivo
- Estructura de respuestas JSON estándar

### Práctica Adicional Recomendada
- Ejercicios diarios de speed coding (30 minutos)
- Implementación de variaciones del dominio (otros deportes)
- Optimización de tiempo en tareas repetitivas
- Simulacros adicionales con diferentes requerimientos

Este plan de entrenamiento está diseñado para llevar a competidores desde conocimientos básicos hasta readiness competitiva en 5 días intensivos, combinando fundamentos técnicos sólidos con práctica deliberada bajo presión.