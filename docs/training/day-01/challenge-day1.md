# 🏆 Desafío Final - Día 1

**Duración:** 2 horas cronometradas  
**Objetivo:** CRUD completo con frontend y backend funcionales

---

## 🎯 **Descripción del Desafío**

Crear un sistema completo de gestión de **Tareas** (Tasks) que incluya:

- **Backend Laravel:** API REST completa
- **Frontend React:** Interfaz funcional con Vite
- **Integración:** Comunicación fluida entre ambos

### **Entidad: Task**

```
Task (Tarea)
├─ id (PK)
├─ title (string, required, max:255)
├─ description (text, optional)
├─ status (enum: 'pending', 'in_progress', 'completed')
├─ priority (enum: 'low', 'medium', 'high')
├─ due_date (date, optional)
├─ created_at
└─ updated_at
```

---

## ⏱️ **Cronograma del Desafío**

### **Fase 1: Setup y Base de Datos (30 minutos)**

#### **Setup del Proyecto (10 minutos)**

```bash
# ⏰ CRONÓMETRO: INICIAR
composer create-project laravel/laravel task-manager-api
cd task-manager-api

# Configurar SQLite
touch database/database.sqlite
sed -i 's/DB_CONNECTION=mysql/DB_CONNECTION=sqlite/' .env

# Iniciar Laravel
php artisan serve &

# Frontend
cd ..
pnpm create vite task-manager-frontend --template react
cd task-manager-frontend
pnpm install
pnpm add axios react-router-dom
pnpm dev &
```

#### **Migración y Modelo (20 minutos)**

```bash
# Crear modelo completo
php artisan make:model Task -mcr
```

```php
// database/migrations/xxxx_create_tasks_table.php
public function up()
{
    Schema::create('tasks', function (Blueprint $table) {
        $table->id();
        $table->string('title');
        $table->text('description')->nullable();
        $table->enum('status', ['pending', 'in_progress', 'completed'])->default('pending');
        $table->enum('priority', ['low', 'medium', 'high'])->default('medium');
        $table->date('due_date')->nullable();
        $table->timestamps();

        // Índices para filtros
        $table->index(['status', 'priority']);
        $table->index('due_date');
    });
}
```

```php
// app/Models/Task.php
class Task extends Model
{
    protected $fillable = [
        'title', 'description', 'status', 'priority', 'due_date'
    ];

    protected $casts = [
        'due_date' => 'date',
    ];

    // Scopes útiles
    public function scopeByStatus($query, $status)
    {
        return $query->where('status', $status);
    }

    public function scopeByPriority($query, $priority)
    {
        return $query->where('priority', $priority);
    }

    public function scopeDueSoon($query, $days = 7)
    {
        return $query->where('due_date', '<=', now()->addDays($days));
    }
}
```

### **Fase 2: API REST Completa (45 minutos)**

#### **Controller con Validaciones (25 minutos)**

```bash
php artisan make:controller TaskController --resource --api
php artisan make:request StoreTaskRequest
php artisan make:request UpdateTaskRequest
php artisan make:resource TaskResource
```

```php
// app/Http/Requests/StoreTaskRequest.php
public function rules()
{
    return [
        'title' => 'required|string|max:255',
        'description' => 'nullable|string',
        'status' => 'in:pending,in_progress,completed',
        'priority' => 'in:low,medium,high',
        'due_date' => 'nullable|date|after_or_equal:today',
    ];
}

public function messages()
{
    return [
        'title.required' => 'El título es obligatorio',
        'title.max' => 'El título no puede superar 255 caracteres',
        'status.in' => 'Estado inválido',
        'priority.in' => 'Prioridad inválida',
        'due_date.after_or_equal' => 'La fecha debe ser hoy o posterior',
    ];
}
```

```php
// app/Http/Controllers/TaskController.php
class TaskController extends Controller
{
    public function index(Request $request)
    {
        $tasks = Task::query()
            ->when($request->status, fn($q, $status) => $q->byStatus($status))
            ->when($request->priority, fn($q, $priority) => $q->byPriority($priority))
            ->when($request->due_soon, fn($q) => $q->dueSoon())
            ->orderBy('created_at', 'desc')
            ->paginate(10);

        return TaskResource::collection($tasks);
    }

    public function store(StoreTaskRequest $request)
    {
        $task = Task::create($request->validated());

        return response()->json([
            'data' => new TaskResource($task),
            'message' => 'Tarea creada exitosamente'
        ], 201);
    }

    public function show(Task $task)
    {
        return new TaskResource($task);
    }

    public function update(UpdateTaskRequest $request, Task $task)
    {
        $task->update($request->validated());

        return response()->json([
            'data' => new TaskResource($task),
            'message' => 'Tarea actualizada exitosamente'
        ]);
    }

    public function destroy(Task $task)
    {
        $task->delete();

        return response()->json([
            'message' => 'Tarea eliminada exitosamente'
        ], 204);
    }
}
```

#### **Resource y Rutas (10 minutos)**

```php
// app/Http/Resources/TaskResource.php
public function toArray($request)
{
    return [
        'id' => $this->id,
        'title' => $this->title,
        'description' => $this->description,
        'status' => $this->status,
        'status_label' => [
            'pending' => 'Pendiente',
            'in_progress' => 'En Progreso',
            'completed' => 'Completada'
        ][$this->status],
        'priority' => $this->priority,
        'priority_label' => [
            'low' => 'Baja',
            'medium' => 'Media',
            'high' => 'Alta'
        ][$this->priority],
        'due_date' => $this->due_date?->format('Y-m-d'),
        'due_date_formatted' => $this->due_date?->format('d/m/Y'),
        'is_overdue' => $this->due_date && $this->due_date->isPast(),
        'created_at' => $this->created_at->format('Y-m-d H:i:s'),
        'updated_at' => $this->updated_at->format('Y-m-d H:i:s'),
    ];
}
```

```php
// routes/api.php
Route::apiResource('tasks', TaskController::class);

// Rutas adicionales útiles
Route::get('/tasks-stats', function () {
    return response()->json([
        'total' => Task::count(),
        'pending' => Task::byStatus('pending')->count(),
        'in_progress' => Task::byStatus('in_progress')->count(),
        'completed' => Task::byStatus('completed')->count(),
        'overdue' => Task::where('due_date', '<', now())->count(),
    ]);
});
```

#### **Seeder para Testing (10 minutos)**

```php
// database/seeders/TaskSeeder.php
public function run()
{
    $tasks = [
        [
            'title' => 'Completar documentación del proyecto',
            'description' => 'Escribir la documentación técnica completa',
            'status' => 'in_progress',
            'priority' => 'high',
            'due_date' => now()->addDays(3),
        ],
        [
            'title' => 'Revisar código del equipo',
            'description' => 'Code review de las últimas features',
            'status' => 'pending',
            'priority' => 'medium',
            'due_date' => now()->addDays(1),
        ],
        [
            'title' => 'Configurar CI/CD',
            'description' => null,
            'status' => 'completed',
            'priority' => 'low',
            'due_date' => now()->subDays(2),
        ]
    ];

    foreach ($tasks as $task) {
        Task::create($task);
    }
}
```

### **Fase 3: Frontend React (40 minutos)**

#### **Configuración y Servicios (15 minutos)**

```js
// vite.config.js
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

```js
// src/services/taskService.js
import apiService from './api';

class TaskService {
  async getTasks(filters = {}) {
    return apiService.get('/tasks', filters);
  }

  async getTask(id) {
    return apiService.get(`/tasks/${id}`);
  }

  async createTask(taskData) {
    return apiService.post('/tasks', taskData);
  }

  async updateTask(id, taskData) {
    return apiService.put(`/tasks/${id}`, taskData);
  }

  async deleteTask(id) {
    return apiService.delete(`/tasks/${id}`);
  }

  async getStats() {
    return apiService.get('/tasks-stats');
  }
}

export default new TaskService();
```

#### **Hooks Personalizados (10 minutos)**

```jsx
// src/hooks/useTasks.js
import { useState, useEffect } from 'react';
import taskService from '../services/taskService';

export const useTasks = (filters = {}) => {
  const [tasks, setTasks] = useState([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);

  const fetchTasks = async () => {
    try {
      setLoading(true);
      setError(null);
      const response = await taskService.getTasks(filters);
      setTasks(response.data);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  };

  const createTask = async (taskData) => {
    const response = await taskService.createTask(taskData);
    setTasks((prev) => [response.data, ...prev]);
    return response.data;
  };

  const updateTask = async (id, taskData) => {
    const response = await taskService.updateTask(id, taskData);
    setTasks((prev) =>
      prev.map((task) => (task.id === id ? response.data : task))
    );
    return response.data;
  };

  const deleteTask = async (id) => {
    await taskService.deleteTask(id);
    setTasks((prev) => prev.filter((task) => task.id !== id));
  };

  useEffect(() => {
    fetchTasks();
  }, [JSON.stringify(filters)]);

  return {
    tasks,
    loading,
    error,
    refetch: fetchTasks,
    createTask,
    updateTask,
    deleteTask,
  };
};
```

#### **Componentes UI (15 minutos)**

```jsx
// src/components/TaskList.jsx
import React, { useState } from 'react';
import { useTasks } from '../hooks/useTasks';

const TaskList = () => {
  const [filters, setFilters] = useState({
    status: '',
    priority: '',
  });

  const { tasks, loading, error, deleteTask } = useTasks(filters);

  const getStatusColor = (status) => {
    const colors = {
      pending: 'bg-yellow-100 text-yellow-800',
      in_progress: 'bg-blue-100 text-blue-800',
      completed: 'bg-green-100 text-green-800',
    };
    return colors[status] || '';
  };

  const getPriorityColor = (priority) => {
    const colors = {
      low: 'bg-gray-100 text-gray-800',
      medium: 'bg-orange-100 text-orange-800',
      high: 'bg-red-100 text-red-800',
    };
    return colors[priority] || '';
  };

  if (loading) return <div>Cargando tareas...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <div className="space-y-6">
      {/* Filtros */}
      <div className="flex gap-4">
        <select
          value={filters.status}
          onChange={(e) =>
            setFilters((prev) => ({ ...prev, status: e.target.value }))
          }
          className="border rounded px-3 py-2">
          <option value="">Todos los estados</option>
          <option value="pending">Pendiente</option>
          <option value="in_progress">En Progreso</option>
          <option value="completed">Completada</option>
        </select>

        <select
          value={filters.priority}
          onChange={(e) =>
            setFilters((prev) => ({ ...prev, priority: e.target.value }))
          }
          className="border rounded px-3 py-2">
          <option value="">Todas las prioridades</option>
          <option value="low">Baja</option>
          <option value="medium">Media</option>
          <option value="high">Alta</option>
        </select>
      </div>

      {/* Lista de tareas */}
      <div className="grid gap-4">
        {tasks.map((task) => (
          <div
            key={task.id}
            className="border rounded-lg p-4 bg-white shadow">
            <div className="flex justify-between items-start mb-2">
              <h3 className="text-lg font-semibold">{task.title}</h3>
              <div className="flex gap-2">
                <span
                  className={`px-2 py-1 rounded text-xs ${getStatusColor(
                    task.status
                  )}`}>
                  {task.status_label}
                </span>
                <span
                  className={`px-2 py-1 rounded text-xs ${getPriorityColor(
                    task.priority
                  )}`}>
                  {task.priority_label}
                </span>
              </div>
            </div>

            {task.description && (
              <p className="text-gray-600 mb-2">{task.description}</p>
            )}

            {task.due_date && (
              <p
                className={`text-sm mb-2 ${
                  task.is_overdue ? 'text-red-600' : 'text-gray-500'
                }`}>
                Vence: {task.due_date_formatted}
                {task.is_overdue && ' (Vencida)'}
              </p>
            )}

            <div className="flex justify-end gap-2">
              <button className="text-blue-600 hover:underline text-sm">
                Editar
              </button>
              <button
                onClick={() => {
                  if (confirm('¿Eliminar esta tarea?')) {
                    deleteTask(task.id);
                  }
                }}
                className="text-red-600 hover:underline text-sm">
                Eliminar
              </button>
            </div>
          </div>
        ))}
      </div>

      {tasks.length === 0 && (
        <p className="text-center text-gray-500 py-8">
          No se encontraron tareas
        </p>
      )}
    </div>
  );
};

export default TaskList;
```

### **Fase 4: Integración y Testing (25 minutos)**

#### **App Principal (10 minutos)**

```jsx
// src/App.jsx
import React, { useState } from 'react';
import TaskList from './components/TaskList';
import TaskForm from './components/TaskForm';

function App() {
  const [showForm, setShowForm] = useState(false);
  const [editingTask, setEditingTask] = useState(null);

  return (
    <div className="min-h-screen bg-gray-100">
      <div className="container mx-auto px-4 py-8">
        <div className="flex justify-between items-center mb-8">
          <h1 className="text-3xl font-bold text-gray-900">Gestor de Tareas</h1>
          <button
            onClick={() => setShowForm(true)}
            className="bg-blue-500 text-white px-4 py-2 rounded hover:bg-blue-600">
            Nueva Tarea
          </button>
        </div>

        {showForm && (
          <div className="mb-8">
            <TaskForm
              initialData={editingTask}
              onSuccess={() => {
                setShowForm(false);
                setEditingTask(null);
              }}
              onCancel={() => {
                setShowForm(false);
                setEditingTask(null);
              }}
            />
          </div>
        )}

        <TaskList
          onEdit={(task) => {
            setEditingTask(task);
            setShowForm(true);
          }}
        />
      </div>
    </div>
  );
}

export default App;
```

#### **Testing Manual (15 minutos)**

```bash
# Ejecutar migraciones y seeders
php artisan migrate:fresh --seed

# Probar endpoints en browser:
# http://localhost:8000/api/tasks
# http://localhost:8000/api/tasks-stats

# Probar frontend:
# http://localhost:3000
```

---

## 🏆 **Criterios de Evaluación**

### **Funcionalidad Mínima Requerida (70%)**

- [ ] ✅ **Setup completo** en < 15 minutos
- [ ] ✅ **Migración** con todos los campos y restricciones
- [ ] ✅ **API REST** con los 5 endpoints básicos funcionando
- [ ] ✅ **Validaciones** en Laravel con mensajes en español
- [ ] ✅ **Frontend** mostrando lista de tareas
- [ ] ✅ **Integración** React ↔ Laravel funcionando

### **Funcionalidad Avanzada (20%)**

- [ ] ✅ **Filtros** por estado y prioridad
- [ ] ✅ **Formulario** para crear/editar tareas
- [ ] ✅ **Estados de loading** y manejo de errores
- [ ] ✅ **Validación** en frontend y backend

### **Calidad y Mejores Prácticas (10%)**

- [ ] ✅ **Código organizado** en servicios y hooks
- [ ] ✅ **UI responsiva** y atractiva
- [ ] ✅ **Console limpia** sin errores
- [ ] ✅ **Seeders** con datos de prueba

---

## 🎯 **Rúbrica de Puntuación**

| Nivel             | Puntos | Descripción                                          |
| ----------------- | ------ | ---------------------------------------------------- |
| **Excelente**     | 90-100 | Todo funcionando + extras (stats, filtros avanzados) |
| **Bueno**         | 80-89  | CRUD completo funcionando sin errores                |
| **Satisfactorio** | 70-79  | CRUD básico, algunos errores menores                 |
| **Insuficiente**  | < 70   | No completa funcionalidad mínima                     |

---

## 📝 **Entrega**

Al finalizar las 2 horas, demostrar:

1. **Backend funcionando:** Mostrar Postman/Browser con endpoints
2. **Frontend funcionando:** Demostrar todas las operaciones CRUD
3. **Integración completa:** Crear, editar y eliminar tareas desde la UI
4. **Código limpio:** Estructura organizada y sin errores en console

---

**🔥 ¡Este desafío establece la línea base de velocidad de desarrollo de cada competidor para WorldSkills!**
