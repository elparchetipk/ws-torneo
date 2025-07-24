# 🔧 Preparación y Warmup Session

**Duración:** 30 minutos (12:00 PM - 12:30 PM)  
**Objetivo:** Verificar que todo esté listo para los simulacros de competencia

---

## 🎯 **Objetivos de la Sesión**

Al finalizar esta sesión, el competidor tendrá:

✅ **Entorno verificado** y funcionando perfectamente  
✅ **Mindset competitivo** activado y enfocado  
✅ **Herramientas calibradas** para máximo rendimiento  
✅ **Estrategia clara** para abordar los simulacros  
✅ **Confianza técnica** validada con ejercicios rápidos

---

## ⚙️ **Verificación Técnica (15 minutos)**

### **Checklist de Entorno**

**Backend Laravel (5 minutos):**

```bash
# 1. Verificar versión de PHP
php --version
# Debe ser PHP 8.1+

# 2. Verificar Laravel
cd ws-torneo-api
php artisan --version
# Debe ser Laravel 11.x

# 3. Test rápido de base de datos
php artisan migrate:status
php artisan tinker
>>> User::count()
# Debe ejecutar sin errores

# 4. Verificar servidor
php artisan serve
# Debe arrancar en http://localhost:8000
```

**Frontend React (5 minutos):**

```bash
# 1. Verificar Node.js
node --version
# Debe ser Node 18+

# 2. Verificar pnpm
pnpm --version
# Debe estar instalado

# 3. Test del proyecto React (desde ws-torneo-api/frontend)
cd frontend
pnpm install
pnpm dev
# Debe arrancar en http://localhost:3000

# 4. Verificar proxy funcionando
# Abrir http://localhost:3000 y verificar que no hay errores CORS
```

**Herramientas Adicionales (5 minutos):**

```bash
# 1. Verificar Git
git --version
git status

# 2. Verificar VS Code
code --version

# 3. Verificar Postman o Thunder Client
# Abrir y verificar que funciona

# 4. Verificar navegador con DevTools
# Abrir Chrome/Firefox DevTools
```

### **📋 Test Rápido de Funcionalidad**

**Ejercicio de Warmup (10 minutos):**

1. **Crear modelo simple (3 min):**

   ```bash
   php artisan make:model TestModel -m
   # Agregar un campo 'name' en la migración
   php artisan migrate
   ```

2. **Crear API básica (4 min):**

   ```bash
   php artisan make:controller TestController --api
   # Implementar método index que retorne JSON
   # Agregar ruta en api.php
   ```

3. **Test desde frontend (3 min):**
   ```bash
   # Hacer fetch desde React console:
   fetch('/api/test').then(r => r.json()).then(console.log)
   # Debe funcionar sin errores
   ```

---

## 🧠 **Preparación Mental (10 minutos)**

### **Técnicas de Focus**

**Respiración de Competencia (2 min):**

```
1. Inhalar 4 segundos
2. Retener 4 segundos
3. Exhalar 4 segundos
4. Repetir 5 veces
```

**Visualización de Éxito (3 min):**

- 🎯 Imagínate completando el simulacro exitosamente
- 🏆 Visualiza cada funcionalidad working perfectly
- ⚡ Ve tu código limpio y eficiente
- 😎 Siéntete confiado y en control

**Afirmaciones Positivas (2 min):**

- _"Tengo las habilidades necesarias para destacar"_
- _"Mi preparación me respalda completamente"_
- _"Manejo la presión con calma y focus"_
- _"Cada línea de código me acerca al éxito"_

**Estrategia de Tiempo (3 min):**

Revisar mentalmente la distribución óptima:

**Para Simulacro de 3 horas:**

- 0-30 min: Setup + países CRUD
- 30-60 min: Equipos CRUD + validaciones
- 60-90 min: Jugadoras + relaciones
- 90-120 min: Partidos + resultados
- 120-150 min: Tabla de posiciones
- 150-180 min: Frontend básico + integración

**Para Simulacro de 2 horas:**

- 0-20 min: Setup + países API
- 20-40 min: Equipos API
- 40-60 min: Partidos básicos
- 60-80 min: Tabla de posiciones
- 80-120 min: Frontend esencial

---

## 🎯 **Estrategias de Competencia (5 minutos)**

### **Principios Fundamentales**

**1. Funcionalidad Antes que Perfección**

```
✅ Algo que funciona > Algo perfecto incompleto
✅ CRUD básico > UI perfecta sin backend
✅ APIs funcionales > Validaciones exhaustivas
```

**2. Gestión de Riesgo**

```
🚫 No experimentes con tecnologías nuevas
🚫 No optimizes prematuramente
🚫 No te quedes atascado en un problema >15 min
```

**3. Testing Continuo**

```
✅ Testea cada endpoint inmediatamente
✅ Verifica CRUD completo antes de continuar
✅ Valida integración frontend-backend frecuentemente
```

**4. Backup y Recovery**

```
💾 Commit frecuente con mensajes claros
💾 Push a repositorio cada 30 minutos
💾 Ten plan B para funcionalidades complejas
```

### **Técnicas Anti-Pánico**

**Si algo no funciona:**

1. **Stop, Breathe (30 seg)** - No entres en pánico
2. **Read the Error (1 min)** - Lee el error completo
3. **Google Quick (2 min)** - Búsqueda rápida y específica
4. **Simplify (5 min)** - Implementa versión más simple
5. **Move On (10 min)** - Si no se resuelve, continúa

**Si vas atrasado:**

1. **Assess (1 min)** - Evalúa qué es realmente crítico
2. **Drop Features (2 min)** - Elimina funcionalidades no esenciales
3. **Focus Core (resto)** - Todo el foco en funcionalidades críticas

---

## 📝 **Quick Reference Cards**

### **Laravel Speed Commands**

```bash
# Generar recursos completos
php artisan make:model Country -mrc
php artisan make:request StoreCountryRequest

# Migraciones rápidas
php artisan make:migration create_countries_table
php artisan migrate

# Seeders y datos
php artisan make:seeder CountrySeeder
php artisan db:seed

# APIs y rutas
php artisan route:list --name=api
php artisan make:resource CountryResource
```

### **React Speed Snippets**

```jsx
// Hook básico para API
const { data, loading, error } = useApi(() =>
  fetch('/api/countries').then((r) => r.json())
);

// Componente de tabla básica
const DataTable = ({ data, columns }) => (
  <table className="min-w-full">
    <thead>
      <tr>
        {columns.map((col) => (
          <th key={col.key}>{col.title}</th>
        ))}
      </tr>
    </thead>
    <tbody>
      {data.map((row) => (
        <tr key={row.id}>
          {columns.map((col) => (
            <td key={col.key}>{row[col.key]}</td>
          ))}
        </tr>
      ))}
    </tbody>
  </table>
);

// Formulario básico
const [formData, setFormData] = useState({});
const handleSubmit = async (e) => {
  e.preventDefault();
  await fetch('/api/countries', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(formData),
  });
};
```

### **CSS Speed Classes**

```css
/* Layout rápido */
.container {
  @apply max-w-7xl mx-auto px-4;
}
.card {
  @apply bg-white rounded-lg shadow p-6;
}
.btn {
  @apply px-4 py-2 rounded font-medium;
}
.btn-primary {
  @apply bg-blue-500 text-white hover:bg-blue-600;
}

/* Grid responsivo */
.grid-responsive {
  @apply grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6;
}

/* Estados */
.loading {
  @apply animate-pulse bg-gray-200;
}
.error {
  @apply text-red-600 bg-red-50 p-2 rounded;
}
```

---

## ✅ **Checklist Final**

Antes de comenzar los simulacros, confirma:

### **Técnico:**

- [ ] Laravel corriendo en puerto 8000
- [ ] React corriendo en puerto 3000
- [ ] Base de datos SQLite funcionando
- [ ] CORS configurado correctamente
- [ ] Postman/Thunder Client listo
- [ ] Git repositorio inicializado

### **Mental:**

- [ ] Estrategia de tiempo clara
- [ ] Mindset competitivo activado
- [ ] Técnicas anti-pánico memorizadas
- [ ] Prioridades bien definidas
- [ ] Confianza en las habilidades

### **Físico:**

- [ ] Entorno de trabajo cómodo
- [ ] Hidratación adecuada
- [ ] Energía y focus óptimos
- [ ] Distracciones eliminadas
- [ ] Timer/cronómetro listo

---

## 🚀 **¡Listos para la Batalla!**

Has completado 4 días intensivos de preparación. Tienes el conocimiento, las habilidades y las herramientas. Ahora es momento de **demostrar tu excelencia** bajo presión.

### **Recordatorios Finales:**

🎯 **Confía en tu preparación** - Tienes todo lo necesario  
⚡ **Mantén el ritmo** - Velocidad constante beats perfectionism  
🧠 **Piensa como competidor** - Soluciones prácticas over teorías  
💪 **Demuestra tu valor** - Este es tu momento de brillar

---

**¡El warmup está completo. Es hora de competir! 🏆**
