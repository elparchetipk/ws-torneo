# ✅ Validaciones Complejas y Form Requests - Día 3.1

**Objetivo:** Implementar validaciones robustas con reglas personalizadas en menos de 15 minutos

---

## 🎯 **Objetivo de la Sesión**

Los competidores deben poder crear **Form Requests complejos** con validaciones condicionales, reglas personalizadas, y manejo de arrays anidados que cubran todos los casos edge de un sistema deportivo en **menos de 15 minutos**.

---

## 🛡️ **Arquitectura de Validaciones Deportivas**

### **Casos de Validación Complejos WorldSkills**

```
🏆 VALIDACIONES PARA SISTEMA DEPORTIVO

Match Registration:
├─ Teams: Existencia, no duplicados, mismo torneo
├─ Date/Time: Futuro, no conflictos de horario, ventana válida
├─ Venue: Disponibilidad, capacidad, tipo de superficie
└─ Rules: Formato específico por categoría

Player Registration:
├─ Age: Mínimo/máximo por categoría
├─ Team: Límite de jugadores, posiciones balanceadas
├─ Documents: Formatos válidos, tamaños apropiados
└─ Eligibility: Nacionalidad, suspensiones, transferencias

Results Submission:
├─ Match: Estado válido, participantes correctos
├─ Scores: Formato según deporte, lógica de sets/games
├─ Statistics: Rangos válidos, coherencia interna
└─ Authorization: Solo árbitros/organizadores autorizados
```

---

## ⏱️ **Ejercicio 1: Form Requests Básicos (20 minutos)**

### **Paso 1: Crear Form Requests (5 minutos)**

```bash
# CRONÓMETRO: INICIAR ⏰

# Crear Form Requests para entidades principales
php artisan make:request StorePlayerRequest
php artisan make:request UpdatePlayerRequest
php artisan make:request StoreMatchRequest
php artisan make:request UpdateMatchRequest
php artisan make:request StoreResultRequest
```

### **Paso 2: Player Validation (10 minutos)**

#### **StorePlayerRequest - Validaciones Complejas**

```php
// app/Http/Requests/StorePlayerRequest.php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rule;
use Carbon\Carbon;

class StorePlayerRequest extends FormRequest
{
    public function authorize()
    {
        // Solo admin y organizadores pueden crear jugadores
        return $this->user()->hasRole(['admin', 'organizer']);
    }

    public function rules()
    {
        return [
            // Datos básicos
            'name' => [
                'required',
                'string',
                'min:2',
                'max:100',
                'regex:/^[a-zA-ZÀ-ÿ\s]+$/', // Solo letras y espacios
            ],

            // Team assignment con validaciones cruzadas
            'team_id' => [
                'required',
                'exists:teams,id',
                function ($attribute, $value, $fail) {
                    // Verificar que el equipo no tenga más de 23 jugadores
                    $playerCount = \App\Models\Player::where('team_id', $value)->count();
                    if ($playerCount >= 23) {
                        $fail('El equipo ya tiene el máximo de 23 jugadores.');
                    }
                },
                // Verificar que el torneo no haya iniciado
                function ($attribute, $value, $fail) {
                    $team = \App\Models\Team::find($value);
                    if ($team && $team->tournament && $team->tournament->hasStarted()) {
                        $fail('No se pueden agregar jugadores después del inicio del torneo.');
                    }
                }
            ],

            // Position con reglas específicas
            'position' => [
                'required',
                Rule::in(['GK', 'DEF', 'MID', 'FWD']),
                // Validar que no haya más de 3 porteros por equipo
                function ($attribute, $value, $fail) {
                    if ($value === 'GK' && $this->team_id) {
                        $gkCount = \App\Models\Player::where('team_id', $this->team_id)
                                                   ->where('position', 'GK')
                                                   ->count();
                        if ($gkCount >= 3) {
                            $fail('Un equipo no puede tener más de 3 porteros.');
                        }
                    }
                }
            ],

            // Jersey number con unicidad por equipo
            'jersey_number' => [
                'required',
                'integer',
                'between:1,99',
                Rule::unique('players')->where(function ($query) {
                    return $query->where('team_id', $this->team_id);
                })
            ],

            // Birth date con validaciones de edad
            'birth_date' => [
                'required',
                'date',
                'before_or_equal:' . Carbon::now()->subYears(16)->format('Y-m-d'),
                'after_or_equal:' . Carbon::now()->subYears(45)->format('Y-m-d'),
                // Validación por categoría
                function ($attribute, $value, $fail) {
                    $birthDate = Carbon::parse($value);
                    $age = $birthDate->age;

                    $category = $this->input('category', 'senior');

                    switch ($category) {
                        case 'junior':
                            if ($age > 21) {
                                $fail('Los jugadores junior no pueden tener más de 21 años.');
                            }
                            break;
                        case 'senior':
                            if ($age < 18 || $age > 40) {
                                $fail('Los jugadores senior deben tener entre 18 y 40 años.');
                            }
                            break;
                        case 'veteran':
                            if ($age < 35) {
                                $fail('Los jugadores veteranos deben tener al menos 35 años.');
                            }
                            break;
                    }
                }
            ],

            // Height y weight con rangos realistas
            'height' => [
                'nullable',
                'numeric',
                'between:1.50,2.20'
            ],

            'weight' => [
                'nullable',
                'numeric',
                'between:45,150'
            ],

            // Foot preference
            'foot' => [
                'required',
                Rule::in(['right', 'left', 'both'])
            ],

            // Documents array validation
            'documents' => [
                'sometimes',
                'array',
                'max:5'
            ],

            'documents.*' => [
                'file',
                'mimes:pdf,jpg,jpeg,png',
                'max:2048' // 2MB max
            ],

            // Emergency contact
            'emergency_contact' => [
                'required',
                'array'
            ],

            'emergency_contact.name' => [
                'required',
                'string',
                'max:100'
            ],

            'emergency_contact.phone' => [
                'required',
                'regex:/^\+?[1-9]\d{1,14}$/' // E.164 format
            ],

            'emergency_contact.relationship' => [
                'required',
                Rule::in(['parent', 'spouse', 'sibling', 'guardian', 'other'])
            ],
        ];
    }

    public function messages()
    {
        return [
            'name.required' => 'El nombre del jugador es obligatorio.',
            'name.regex' => 'El nombre solo puede contener letras y espacios.',
            'team_id.exists' => 'El equipo seleccionado no existe.',
            'jersey_number.unique' => 'Este número de camiseta ya está ocupado en el equipo.',
            'birth_date.before_or_equal' => 'El jugador debe tener al menos 16 años.',
            'birth_date.after_or_equal' => 'El jugador no puede tener más de 45 años.',
            'height.between' => 'La altura debe estar entre 1.50m y 2.20m.',
            'weight.between' => 'El peso debe estar entre 45kg y 150kg.',
            'documents.*.mimes' => 'Los documentos deben ser archivos PDF o imágenes (JPG, PNG).',
            'documents.*.max' => 'Cada documento no puede superar los 2MB.',
            'emergency_contact.phone.regex' => 'El teléfono de emergencia debe tener un formato válido.',
        ];
    }

    public function attributes()
    {
        return [
            'name' => 'nombre',
            'team_id' => 'equipo',
            'position' => 'posición',
            'jersey_number' => 'número de camiseta',
            'birth_date' => 'fecha de nacimiento',
            'height' => 'altura',
            'weight' => 'peso',
            'emergency_contact.name' => 'nombre del contacto de emergencia',
            'emergency_contact.phone' => 'teléfono de emergencia',
        ];
    }

    public function withValidator($validator)
    {
        $validator->after(function ($validator) {
            // Validación global: verificar coherencia de datos
            if ($this->position === 'GK' && $this->height && $this->height < 1.70) {
                $validator->errors()->add('height', 'Los porteros generalmente deben medir al menos 1.70m.');
            }

            // Verificar que el peso sea coherente con la altura
            if ($this->height && $this->weight) {
                $bmi = $this->weight / ($this->height * $this->height);
                if ($bmi < 16 || $bmi > 35) {
                    $validator->errors()->add('weight', 'El IMC calculado está fuera del rango saludable para un atleta.');
                }
            }
        });
    }
}
```

### **Paso 3: Match Validation con Lógica Compleja (5 minutos)**

```php
// app/Http/Requests/StoreMatchRequest.php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use Illuminate\Validation\Rule;
use Carbon\Carbon;

class StoreMatchRequest extends FormRequest
{
    public function rules()
    {
        return [
            // Teams validation
            'home_team_id' => [
                'required',
                'exists:teams,id',
                'different:away_team_id'
            ],

            'away_team_id' => [
                'required',
                'exists:teams,id',
                // Validar que ambos equipos estén en el mismo torneo
                function ($attribute, $value, $fail) {
                    $homeTeam = \App\Models\Team::find($this->home_team_id);
                    $awayTeam = \App\Models\Team::find($value);

                    if ($homeTeam && $awayTeam) {
                        if ($homeTeam->tournament_id !== $awayTeam->tournament_id) {
                            $fail('Los equipos deben pertenecer al mismo torneo.');
                        }
                    }
                }
            ],

            // Match date con múltiples validaciones
            'match_date' => [
                'required',
                'date',
                'after:now',
                // Validar que esté dentro de las fechas del torneo
                function ($attribute, $value, $fail) {
                    if ($this->home_team_id) {
                        $team = \App\Models\Team::find($this->home_team_id);
                        $tournament = $team?->tournament;

                        if ($tournament) {
                            $matchDate = Carbon::parse($value);

                            if ($matchDate->lt($tournament->start_date) ||
                                $matchDate->gt($tournament->end_date)) {
                                $fail('La fecha del partido debe estar dentro del período del torneo.');
                            }
                        }
                    }
                },
                // Validar conflictos de horario
                function ($attribute, $value, $fail) {
                    $matchDate = Carbon::parse($value);

                    // Verificar si alguno de los equipos ya tiene un partido ±2 horas
                    $conflictingMatches = \App\Models\Match::where(function ($query) {
                        $query->where('home_team_id', $this->home_team_id)
                              ->orWhere('away_team_id', $this->home_team_id)
                              ->orWhere('home_team_id', $this->away_team_id)
                              ->orWhere('away_team_id', $this->away_team_id);
                    })
                    ->whereBetween('match_date', [
                        $matchDate->copy()->subHours(2),
                        $matchDate->copy()->addHours(2)
                    ])
                    ->exists();

                    if ($conflictingMatches) {
                        $fail('Uno de los equipos tiene otro partido muy cerca de esta fecha/hora.');
                    }
                }
            ],

            // Venue validation
            'venue' => [
                'required',
                'string',
                'max:100',
                // Validar disponibilidad del venue
                function ($attribute, $value, $fail) {
                    if ($this->match_date) {
                        $matchDate = Carbon::parse($this->match_date);

                        $venueConflict = \App\Models\Match::where('venue', $value)
                            ->whereBetween('match_date', [
                                $matchDate->copy()->subHours(1),
                                $matchDate->copy()->addHours(3) // Considerar tiempo de partido + limpieza
                            ])
                            ->exists();

                        if ($venueConflict) {
                            $fail('El venue ya está ocupado en ese horario.');
                        }
                    }
                }
            ],

            // Round validation based on tournament format
            'round' => [
                'required',
                'string',
                Rule::in(['Group Stage', 'Round of 16', 'Quarterfinal', 'Semifinal', 'Third Place', 'Final']),
                // Validar orden lógico de rondas
                function ($attribute, $value, $fail) {
                    if ($this->home_team_id) {
                        $team = \App\Models\Team::find($this->home_team_id);
                        $tournament = $team?->tournament;

                        if ($tournament) {
                            $currentRound = $tournament->current_round ?? 'Group Stage';

                            $roundOrder = [
                                'Group Stage' => 1,
                                'Round of 16' => 2,
                                'Quarterfinal' => 3,
                                'Semifinal' => 4,
                                'Third Place' => 5,
                                'Final' => 5
                            ];

                            if ($roundOrder[$value] < $roundOrder[$currentRound]) {
                                $fail('No se puede programar un partido de una ronda anterior.');
                            }
                        }
                    }
                }
            ],

            // Referee assignment
            'referee_id' => [
                'nullable',
                'exists:users,id',
                // Validar que el referee no tenga conflictos
                function ($attribute, $value, $fail) {
                    if ($value && $this->match_date) {
                        $matchDate = Carbon::parse($this->match_date);

                        $refereeConflict = \App\Models\Match::where('referee_id', $value)
                            ->whereBetween('match_date', [
                                $matchDate->copy()->subHours(1),
                                $matchDate->copy()->addHours(3)
                            ])
                            ->exists();

                        if ($refereeConflict) {
                            $fail('El árbitro ya tiene asignado otro partido en ese horario.');
                        }
                    }
                }
            ],
        ];
    }

    public function messages()
    {
        return [
            'home_team_id.different' => 'Un equipo no puede jugar contra sí mismo.',
            'match_date.after' => 'La fecha del partido debe ser futura.',
            'round.in' => 'La ronda seleccionada no es válida.',
            'referee_id.exists' => 'El árbitro seleccionado no existe.',
        ];
    }
}
```

---

## ⏱️ **Ejercicio 2: Custom Validation Rules (25 minutos)**

### **Paso 1: Crear Custom Rules (10 minutos)**

```bash
# Crear reglas de validación personalizadas
php artisan make:rule ValidTournamentTeam
php artisan make:rule ValidMatchScore
php artisan make:rule ValidPlayerTransfer
php artisan make:rule ValidSeasonPeriod
```

### **Paso 2: ValidTournamentTeam Rule (8 minutos)**

```php
// app/Rules/ValidTournamentTeam.php
<?php

namespace App\Rules;

use Illuminate\Contracts\Validation\Rule;
use App\Models\Team;
use App\Models\Tournament;

class ValidTournamentTeam implements Rule
{
    private $tournamentId;
    private $message;

    public function __construct($tournamentId = null)
    {
        $this->tournamentId = $tournamentId;
    }

    public function passes($attribute, $value)
    {
        $team = Team::find($value);

        if (!$team) {
            $this->message = 'El equipo seleccionado no existe.';
            return false;
        }

        // Si se especifica torneo, validar que el equipo esté registrado
        if ($this->tournamentId) {
            $tournament = Tournament::find($this->tournamentId);

            if (!$tournament) {
                $this->message = 'El torneo especificado no existe.';
                return false;
            }

            // Verificar que el equipo esté registrado en el torneo
            if (!$tournament->teams()->where('teams.id', $value)->exists()) {
                $this->message = "El equipo {$team->name} no está registrado en este torneo.";
                return false;
            }

            // Verificar que el equipo tenga al menos 16 jugadores
            if ($team->players()->count() < 16) {
                $this->message = "El equipo {$team->name} debe tener al menos 16 jugadores registrados.";
                return false;
            }

            // Verificar que el equipo tenga al menos 2 porteros
            if ($team->players()->where('position', 'GK')->count() < 2) {
                $this->message = "El equipo {$team->name} debe tener al menos 2 porteros.";
                return false;
            }
        }

        return true;
    }

    public function message()
    {
        return $this->message ?? 'El equipo seleccionado no es válido para este torneo.';
    }
}
```

### **Paso 3: ValidMatchScore Rule (7 minutos)**

```php
// app/Rules/ValidMatchScore.php
<?php

namespace App\Rules;

use Illuminate\Contracts\Validation\Rule;

class ValidMatchScore implements Rule
{
    private $sportType;
    private $message;

    public function __construct($sportType = 'football')
    {
        $this->sportType = $sportType;
    }

    public function passes($attribute, $value)
    {
        // Validar que sea un array con la estructura correcta
        if (!is_array($value)) {
            $this->message = 'El resultado debe ser un array.';
            return false;
        }

        switch ($this->sportType) {
            case 'football':
                return $this->validateFootballScore($value);
            case 'tennis':
                return $this->validateTennisScore($value);
            case 'basketball':
                return $this->validateBasketballScore($value);
            default:
                $this->message = 'Tipo de deporte no soportado.';
                return false;
        }
    }

    private function validateFootballScore($score)
    {
        // Estructura esperada: ['home' => 2, 'away' => 1]
        if (!isset($score['home']) || !isset($score['away'])) {
            $this->message = 'El resultado debe incluir goles de local y visitante.';
            return false;
        }

        if (!is_numeric($score['home']) || !is_numeric($score['away'])) {
            $this->message = 'Los goles deben ser números.';
            return false;
        }

        if ($score['home'] < 0 || $score['away'] < 0) {
            $this->message = 'Los goles no pueden ser negativos.';
            return false;
        }

        if ($score['home'] > 20 || $score['away'] > 20) {
            $this->message = 'Resultado poco realista (más de 20 goles).';
            return false;
        }

        return true;
    }

    private function validateTennisScore($score)
    {
        // Estructura: ['sets' => [['home' => 6, 'away' => 4], ['home' => 6, 'away' => 2]]]
        if (!isset($score['sets']) || !is_array($score['sets'])) {
            $this->message = 'Debe especificar los sets jugados.';
            return false;
        }

        $sets = $score['sets'];
        $setsCount = count($sets);

        // Validar número de sets (2-5 en tenis)
        if ($setsCount < 2 || $setsCount > 5) {
            $this->message = 'Un partido de tenis debe tener entre 2 y 5 sets.';
            return false;
        }

        $homeSets = 0;
        $awaySets = 0;

        foreach ($sets as $setIndex => $set) {
            if (!isset($set['home']) || !isset($set['away'])) {
                $this->message = "El set " . ($setIndex + 1) . " debe tener resultado para ambos jugadores.";
                return false;
            }

            $homeGames = $set['home'];
            $awayGames = $set['away'];

            // Validar que los games sean válidos
            if (!$this->isValidTennisSet($homeGames, $awayGames)) {
                $this->message = "El resultado del set " . ($setIndex + 1) . " no es válido: {$homeGames}-{$awayGames}";
                return false;
            }

            // Contar sets ganados
            if ($homeGames > $awayGames) {
                $homeSets++;
            } else {
                $awaySets++;
            }
        }

        // Validar que haya un ganador claro
        if ($homeSets == $awaySets) {
            $this->message = 'Debe haber un ganador del partido.';
            return false;
        }

        return true;
    }

    private function isValidTennisSet($home, $away)
    {
        // Casos válidos en tenis:
        // 6-0, 6-1, 6-2, 6-3, 6-4 (ganador por 2+ games con mínimo 6)
        // 7-5 (ganador por 2 games)
        // 7-6 (tiebreak)

        $max = max($home, $away);
        $min = min($home, $away);

        // Set normal (6 games mínimo, diferencia de 2)
        if ($max >= 6 && $max - $min >= 2 && $max <= 6) {
            return true;
        }

        // Set 7-5 (diferencia de 2)
        if ($max == 7 && $min == 5) {
            return true;
        }

        // Tiebreak 7-6
        if ($max == 7 && $min == 6) {
            return true;
        }

        return false;
    }

    public function message()
    {
        return $this->message;
    }
}
```

---

## ⏱️ **Ejercicio 3: Array y File Validation (20 minutos)**

### **Paso 1: Bulk Operations Validation (12 minutos)**

```php
// app/Http/Requests/BulkPlayerImportRequest.php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;
use App\Rules\ValidTournamentTeam;

class BulkPlayerImportRequest extends FormRequest
{
    public function rules()
    {
        return [
            // File validation
            'import_file' => [
                'required',
                'file',
                'mimes:csv,xlsx',
                'max:5120', // 5MB
                function ($attribute, $value, $fail) {
                    // Validar estructura del archivo CSV
                    if ($value->getClientOriginalExtension() === 'csv') {
                        $handle = fopen($value->getPathname(), 'r');
                        $header = fgetcsv($handle);
                        fclose($handle);

                        $requiredColumns = ['name', 'position', 'jersey_number', 'birth_date', 'team_id'];
                        $missingColumns = array_diff($requiredColumns, $header ?: []);

                        if (!empty($missingColumns)) {
                            $fail('El archivo CSV debe contener las columnas: ' . implode(', ', $missingColumns));
                        }
                    }
                }
            ],

            // Tournament context
            'tournament_id' => [
                'required',
                'exists:tournaments,id'
            ],

            // Validation options
            'validate_only' => [
                'sometimes',
                'boolean'
            ],

            'skip_duplicates' => [
                'sometimes',
                'boolean'
            ],

            // Array de jugadores (alternativa al archivo)
            'players' => [
                'sometimes',
                'array',
                'min:1',
                'max:50' // Máximo 50 jugadores por importación
            ],

            'players.*.name' => [
                'required_with:players',
                'string',
                'min:2',
                'max:100'
            ],

            'players.*.team_id' => [
                'required_with:players',
                new ValidTournamentTeam(request('tournament_id'))
            ],

            'players.*.position' => [
                'required_with:players',
                'in:GK,DEF,MID,FWD'
            ],

            'players.*.jersey_number' => [
                'required_with:players',
                'integer',
                'between:1,99',
                // Validar unicidad dentro del array
                function ($attribute, $value, $fail) {
                    $teamId = data_get($this->input('players'), str_replace('.jersey_number', '.team_id', $attribute));

                    // Buscar duplicados en el array actual
                    $numbers = collect($this->input('players', []))
                        ->where('team_id', $teamId)
                        ->pluck('jersey_number')
                        ->filter();

                    if ($numbers->duplicates()->contains($value)) {
                        $fail('El número de camiseta está duplicado en el equipo.');
                    }

                    // Verificar en base de datos
                    $exists = \App\Models\Player::where('team_id', $teamId)
                        ->where('jersey_number', $value)
                        ->exists();

                    if ($exists) {
                        $fail('El número de camiseta ya existe en el equipo.');
                    }
                }
            ],

            'players.*.birth_date' => [
                'required_with:players',
                'date',
                'before_or_equal:' . now()->subYears(16)->format('Y-m-d')
            ],

            // Metadata para importación
            'import_options' => [
                'sometimes',
                'array'
            ],

            'import_options.update_existing' => [
                'sometimes',
                'boolean'
            ],

            'import_options.notify_conflicts' => [
                'sometimes',
                'boolean'
            ],
        ];
    }

    public function withValidator($validator)
    {
        $validator->after(function ($validator) {
            // Validación global del array de jugadores
            $players = $this->input('players', []);

            if (!empty($players)) {
                // Verificar distribución de posiciones por equipo
                $teamPositions = collect($players)->groupBy('team_id');

                foreach ($teamPositions as $teamId => $teamPlayers) {
                    $positions = $teamPlayers->countBy('position');

                    // Validar mínimos por posición
                    if (($positions['GK'] ?? 0) < 2) {
                        $validator->errors()->add('players', "El equipo ID {$teamId} debe tener al menos 2 porteros.");
                    }

                    if (($positions['DEF'] ?? 0) < 4) {
                        $validator->errors()->add('players', "El equipo ID {$teamId} debe tener al menos 4 defensores.");
                    }
                }
            }
        });
    }

    public function messages()
    {
        return [
            'import_file.mimes' => 'El archivo debe ser CSV o Excel (.xlsx).',
            'import_file.max' => 'El archivo no puede superar los 5MB.',
            'players.max' => 'No se pueden importar más de 50 jugadores a la vez.',
            'players.*.name.required_with' => 'El nombre es obligatorio para cada jugador.',
            'players.*.jersey_number.between' => 'El número de camiseta debe estar entre 1 y 99.',
        ];
    }
}
```

### **Paso 2: File Upload Validation (8 minutos)**

```php
// app/Http/Requests/UploadPlayerDocumentsRequest.php
<?php

namespace App\Http\Requests;

use Illuminate\Foundation\Http\FormRequest;

class UploadPlayerDocumentsRequest extends FormRequest
{
    public function rules()
    {
        return [
            'player_id' => [
                'required',
                'exists:players,id'
            ],

            'documents' => [
                'required',
                'array',
                'min:1',
                'max:10'
            ],

            'documents.*' => [
                'file',
                'max:5120', // 5MB
                // Validación por tipo de documento
                function ($attribute, $value, $fail) {
                    $index = explode('.', $attribute)[1];
                    $documentType = $this->input("document_types.{$index}");

                    switch ($documentType) {
                        case 'passport':
                        case 'id_card':
                            if (!in_array($value->getClientOriginalExtension(), ['pdf', 'jpg', 'jpeg', 'png'])) {
                                $fail('Los documentos de identidad deben ser PDF o imágenes.');
                            }
                            break;

                        case 'medical_certificate':
                            if ($value->getClientOriginalExtension() !== 'pdf') {
                                $fail('Los certificados médicos deben ser archivos PDF.');
                            }
                            break;

                        case 'photo':
                            if (!in_array($value->getClientOriginalExtension(), ['jpg', 'jpeg', 'png'])) {
                                $fail('Las fotos deben ser imágenes JPG o PNG.');
                            }

                            // Validar dimensiones de imagen
                            $imageInfo = getimagesize($value->getPathname());
                            if ($imageInfo) {
                                $width = $imageInfo[0];
                                $height = $imageInfo[1];

                                if ($width < 300 || $height < 400) {
                                    $fail('Las fotos deben tener al menos 300x400 píxeles.');
                                }

                                if ($width > 2000 || $height > 3000) {
                                    $fail('Las fotos no pueden superar 2000x3000 píxeles.');
                                }
                            }
                            break;
                    }
                }
            ],

            'document_types' => [
                'required',
                'array',
                'size:' . count($this->file('documents', []))
            ],

            'document_types.*' => [
                'required',
                'in:passport,id_card,medical_certificate,insurance_card,photo,other'
            ],

            'descriptions' => [
                'sometimes',
                'array'
            ],

            'descriptions.*' => [
                'nullable',
                'string',
                'max:255'
            ]
        ];
    }

    public function withValidator($validator)
    {
        $validator->after(function ($validator) {
            $documentTypes = $this->input('document_types', []);

            // Validar que haya al menos un documento de identidad
            $hasIdentity = collect($documentTypes)->intersect(['passport', 'id_card'])->isNotEmpty();
            if (!$hasIdentity) {
                $validator->errors()->add('document_types', 'Debe incluir al menos un documento de identidad (pasaporte o cédula).');
            }

            // Validar que no haya tipos duplicados
            $duplicates = collect($documentTypes)->duplicates();
            if ($duplicates->isNotEmpty()) {
                $validator->errors()->add('document_types', 'No puede subir múltiples documentos del mismo tipo.');
            }
        });
    }
}
```

---

## 🎯 **Rúbrica de Evaluación**

| Criterio                | Excelente (4)                               | Bueno (3)                          | Satisfactorio (2)             | Insuficiente (1)          |
| ----------------------- | ------------------------------------------- | ---------------------------------- | ----------------------------- | ------------------------- |
| **Form Requests**       | Todas las validaciones + lógica condicional | Validaciones principales correctas | Validaciones básicas          | Form Requests incompletos |
| **Custom Rules**        | Reglas reutilizables y complejas            | Reglas básicas funcionando         | Algunas reglas personalizadas | Sin custom rules          |
| **Array Validation**    | Validación anidada completa                 | Arrays simples validados           | Validación básica             | Sin validación de arrays  |
| **File Validation**     | Tipos, tamaños, dimensiones                 | Validación básica de archivos      | Solo tipos permitidos         | Sin validación archivos   |
| **Messages/Attributes** | Mensajes personalizados claros              | Algunos mensajes custom            | Mensajes por defecto          | Sin personalización       |
| **Tiempo**              | < 15 minutos                                | 15-20 min                          | 20-30 min                     | > 30 min                  |

---

## 🔧 **Debugging de Validaciones**

### **Comandos Útiles**

```bash
# Ver validaciones que fallan
php artisan tinker
>>> app(App\Http\Requests\StorePlayerRequest::class)->validate();

# Test de Custom Rules
>>> $rule = new App\Rules\ValidMatchScore('tennis');
>>> $rule->passes('score', ['sets' => [['home' => 6, 'away' => 4]]]);

# Validar archivos en terminal
>>> $request = request();
>>> $request->files->set('document', new \Illuminate\Http\UploadedFile('/path/to/test.pdf', 'test.pdf'));
```

### **Tips de Performance**

```php
// Evitar N+1 en validaciones
function ($attribute, $value, $fail) {
    // ❌ Malo - hace query por cada iteración
    $team = Team::find($value);

    // ✅ Bueno - cargar datos una vez
    static $teams = null;
    if ($teams === null) {
        $teams = Team::all()->keyBy('id');
    }
    $team = $teams->get($value);
}
```

---

**🔥 Con estas validaciones robustas, los competidores crearán APIs que manejan todos los casos edge de WorldSkills!**
