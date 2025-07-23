# 🚨 Error Handling y Logging Profesional - Día 3.5

**Objetivo:** Implementar manejo de errores robusto con logging estructurado en menos de 15 minutos

---

## 🎯 **Objetivo de la Sesión**

Los competidores deben poder crear **sistemas de error handling** con logging estructurado, recuperación automática, y monitoreo de errores que cumplan estándares de producción en **menos de 15 minutos**.

---

## 🔥 **Arquitectura de Error Handling WorldSkills**

### **Niveles de Error Handling**

```text
🏆 ERROR HANDLING PARA SISTEMA DEPORTIVO

HTTP Layer:
├─ 400 Bad Request (Validation errors)
├─ 401 Unauthorized (Authentication failed)
├─ 403 Forbidden (Permission denied)
├─ 404 Not Found (Resource not found)
├─ 409 Conflict (Business rule violation)
├─ 422 Unprocessable Entity (Validation errors)
├─ 429 Too Many Requests (Rate limiting)
└─ 500 Internal Server Error (System errors)

Application Layer:
├─ Business Logic Exceptions
├─ Database Connection Errors
├─ External API Failures
├─ File System Errors
└─ Memory/Performance Issues

Logging Levels:
├─ EMERGENCY: System is unusable
├─ ALERT: Action must be taken immediately
├─ CRITICAL: Critical conditions
├─ ERROR: Error conditions
├─ WARNING: Warning conditions
├─ NOTICE: Normal but significant condition
├─ INFO: Informational messages
└─ DEBUG: Debug-level messages

Recovery Strategies:
├─ Automatic Retry (Network failures)
├─ Circuit Breaker (External services)
├─ Graceful Degradation (Feature fallbacks)
└─ Data Validation Cleanup (Corrupt data)
```

---

## ⏱️ **Ejercicio 1: Exception Hierarchy (18 minutos)**

### **Paso 1: Custom Exception Classes (10 minutos)**

```bash
# CRONÓMETRO: INICIAR ⏰

# Crear estructura de excepciones
mkdir -p app/Exceptions/Business
mkdir -p app/Exceptions/External
mkdir -p app/Exceptions/Validation

# Crear clases de excepción
touch app/Exceptions/BaseApiException.php
touch app/Exceptions/Business/TournamentException.php
touch app/Exceptions/Business/PlayerRegistrationException.php
touch app/Exceptions/Business/MatchSchedulingException.php
touch app/Exceptions/External/ExternalServiceException.php
touch app/Exceptions/Validation/BusinessRuleException.php
```

```php
// app/Exceptions/BaseApiException.php
<?php

namespace App\Exceptions;

use Exception;
use Illuminate\Http\JsonResponse;

abstract class BaseApiException extends Exception
{
    protected $statusCode = 500;
    protected $errorCode;
    protected $context = [];
    protected $userMessage;
    protected $debugInfo = [];

    public function __construct(
        $message = '',
        $errorCode = null,
        $context = [],
        $userMessage = null,
        Exception $previous = null
    ) {
        parent::__construct($message, 0, $previous);

        $this->errorCode = $errorCode ?? $this->getDefaultErrorCode();
        $this->context = $context;
        $this->userMessage = $userMessage ?? $this->getDefaultUserMessage();
        $this->debugInfo = $this->collectDebugInfo();
    }

    abstract protected function getDefaultErrorCode(): string;
    abstract protected function getDefaultUserMessage(): string;

    public function getStatusCode(): int
    {
        return $this->statusCode;
    }

    public function getErrorCode(): string
    {
        return $this->errorCode;
    }

    public function getContext(): array
    {
        return $this->context;
    }

    public function getUserMessage(): string
    {
        return $this->userMessage;
    }

    public function toArray(): array
    {
        return [
            'message' => $this->getUserMessage(),
            'error_code' => $this->getErrorCode(),
            'status_code' => $this->getStatusCode(),
            'timestamp' => now()->toISOString(),
            'trace_id' => $this->generateTraceId(),
            'context' => $this->getContext(),
            'debug' => app()->environment('local') ? $this->debugInfo : null,
        ];
    }

    public function render($request): JsonResponse
    {
        // Log del error
        $this->logError();

        // Respuesta JSON consistente
        return response()->json(
            array_filter($this->toArray()),
            $this->getStatusCode()
        );
    }

    protected function logError(): void
    {
        $logLevel = $this->getLogLevel();
        $logContext = [
            'error_code' => $this->getErrorCode(),
            'status_code' => $this->getStatusCode(),
            'context' => $this->getContext(),
            'trace_id' => $this->generateTraceId(),
            'user_id' => auth()->id(),
            'url' => request()->fullUrl(),
            'method' => request()->method(),
            'ip' => request()->ip(),
            'user_agent' => request()->userAgent(),
        ];

        logger()->log($logLevel, $this->getMessage(), $logContext);
    }

    protected function getLogLevel(): string
    {
        return match (true) {
            $this->statusCode >= 500 => 'error',
            $this->statusCode >= 400 => 'warning',
            default => 'info'
        };
    }

    protected function generateTraceId(): string
    {
        return request()->header('X-Trace-Id') ?? uniqid('trace_', true);
    }

    protected function collectDebugInfo(): array
    {
        return [
            'file' => $this->getFile(),
            'line' => $this->getLine(),
            'trace' => collect($this->getTrace())->take(5)->toArray(),
            'memory_usage' => memory_get_usage(true),
            'peak_memory' => memory_get_peak_usage(true),
        ];
    }
}
```

```php
// app/Exceptions/Business/TournamentException.php
<?php

namespace App\Exceptions\Business;

use App\Exceptions\BaseApiException;

class TournamentException extends BaseApiException
{
    protected $statusCode = 409; // Conflict

    protected function getDefaultErrorCode(): string
    {
        return 'TOURNAMENT_BUSINESS_ERROR';
    }

    protected function getDefaultUserMessage(): string
    {
        return 'There was an issue with the tournament operation. Please check the tournament rules and try again.';
    }

    public static function tournamentAlreadyStarted($tournamentId): self
    {
        return new self(
            "Tournament {$tournamentId} has already started",
            'TOURNAMENT_ALREADY_STARTED',
            ['tournament_id' => $tournamentId],
            'This tournament has already started. No more teams can be registered.'
        );
    }

    public static function insufficientTeams($tournamentId, $currentCount, $minimumRequired): self
    {
        return new self(
            "Tournament {$tournamentId} has only {$currentCount} teams, minimum {$minimumRequired} required",
            'INSUFFICIENT_TEAMS',
            [
                'tournament_id' => $tournamentId,
                'current_count' => $currentCount,
                'minimum_required' => $minimumRequired
            ],
            "This tournament needs at least {$minimumRequired} teams to start. Currently has {$currentCount} teams."
        );
    }

    public static function maxTeamsReached($tournamentId, $maxTeams): self
    {
        return new self(
            "Tournament {$tournamentId} has reached maximum capacity of {$maxTeams} teams",
            'MAX_TEAMS_REACHED',
            ['tournament_id' => $tournamentId, 'max_teams' => $maxTeams],
            "This tournament has reached its maximum capacity of {$maxTeams} teams."
        );
    }

    public static function registrationClosed($tournamentId, $closedDate): self
    {
        return new self(
            "Registration for tournament {$tournamentId} closed on {$closedDate}",
            'REGISTRATION_CLOSED',
            ['tournament_id' => $tournamentId, 'closed_date' => $closedDate],
            'Registration for this tournament has closed.'
        );
    }
}
```

```php
// app/Exceptions/Business/PlayerRegistrationException.php
<?php

namespace App\Exceptions\Business;

use App\Exceptions\BaseApiException;

class PlayerRegistrationException extends BaseApiException
{
    protected $statusCode = 422;

    protected function getDefaultErrorCode(): string
    {
        return 'PLAYER_REGISTRATION_ERROR';
    }

    protected function getDefaultUserMessage(): string
    {
        return 'There was an issue with the player registration. Please check the requirements and try again.';
    }

    public static function teamAtCapacity($teamId, $maxPlayers): self
    {
        return new self(
            "Team {$teamId} has reached maximum capacity of {$maxPlayers} players",
            'TEAM_AT_CAPACITY',
            ['team_id' => $teamId, 'max_players' => $maxPlayers],
            "This team has reached its maximum capacity of {$maxPlayers} players."
        );
    }

    public static function jerseyNumberTaken($teamId, $jerseyNumber): self
    {
        return new self(
            "Jersey number {$jerseyNumber} is already taken in team {$teamId}",
            'JERSEY_NUMBER_TAKEN',
            ['team_id' => $teamId, 'jersey_number' => $jerseyNumber],
            "Jersey number {$jerseyNumber} is already taken by another player in this team."
        );
    }

    public static function insufficientGoalkeepers($teamId, $currentCount, $minimumRequired): self
    {
        return new self(
            "Team {$teamId} has only {$currentCount} goalkeepers, minimum {$minimumRequired} required",
            'INSUFFICIENT_GOALKEEPERS',
            [
                'team_id' => $teamId,
                'current_count' => $currentCount,
                'minimum_required' => $minimumRequired
            ],
            "Teams must have at least {$minimumRequired} goalkeepers. Currently has {$currentCount}."
        );
    }

    public static function playerTooYoung($playerId, $age, $minimumAge): self
    {
        return new self(
            "Player {$playerId} is {$age} years old, minimum age is {$minimumAge}",
            'PLAYER_TOO_YOUNG',
            ['player_id' => $playerId, 'age' => $age, 'minimum_age' => $minimumAge],
            "Player must be at least {$minimumAge} years old to participate in this category."
        );
    }

    public static function playerAlreadyRegistered($playerId, $tournamentId): self
    {
        return new self(
            "Player {$playerId} is already registered in tournament {$tournamentId}",
            'PLAYER_ALREADY_REGISTERED',
            ['player_id' => $playerId, 'tournament_id' => $tournamentId],
            'This player is already registered in the tournament with another team.'
        );
    }
}
```

```php
// app/Exceptions/Business/MatchSchedulingException.php
<?php

namespace App\Exceptions\Business;

use App\Exceptions\BaseApiException;

class MatchSchedulingException extends BaseApiException
{
    protected $statusCode = 409;

    protected function getDefaultErrorCode(): string
    {
        return 'MATCH_SCHEDULING_ERROR';
    }

    protected function getDefaultUserMessage(): string
    {
        return 'There was a conflict with the match scheduling. Please check the date, time, and teams.';
    }

    public static function venueNotAvailable($venue, $matchDate): self
    {
        return new self(
            "Venue '{$venue}' is not available on {$matchDate}",
            'VENUE_NOT_AVAILABLE',
            ['venue' => $venue, 'match_date' => $matchDate],
            "The venue '{$venue}' is already booked for the requested time."
        );
    }

    public static function teamHasConflict($teamId, $matchDate): self
    {
        return new self(
            "Team {$teamId} has another match scheduled close to {$matchDate}",
            'TEAM_SCHEDULE_CONFLICT',
            ['team_id' => $teamId, 'match_date' => $matchDate],
            'This team has another match scheduled too close to the requested time.'
        );
    }

    public static function refereeNotAvailable($refereeId, $matchDate): self
    {
        return new self(
            "Referee {$refereeId} is not available on {$matchDate}",
            'REFEREE_NOT_AVAILABLE',
            ['referee_id' => $refereeId, 'match_date' => $matchDate],
            'The assigned referee is not available at the requested time.'
        );
    }

    public static function matchAlreadyCompleted($matchId): self
    {
        return new self(
            "Match {$matchId} is already completed and cannot be modified",
            'MATCH_ALREADY_COMPLETED',
            ['match_id' => $matchId],
            'This match has already been completed and cannot be modified.'
        );
    }
}
```

### **Paso 2: External Service Exceptions (8 minutos)**

```php
// app/Exceptions/External/ExternalServiceException.php
<?php

namespace App\Exceptions\External;

use App\Exceptions\BaseApiException;

class ExternalServiceException extends BaseApiException
{
    protected $statusCode = 503; // Service Unavailable
    protected $serviceName;
    protected $retryAfter;

    public function __construct(
        $serviceName,
        $message = '',
        $errorCode = null,
        $context = [],
        $retryAfter = 60,
        $previous = null
    ) {
        $this->serviceName = $serviceName;
        $this->retryAfter = $retryAfter;

        parent::__construct($message, $errorCode, $context, null, $previous);
    }

    protected function getDefaultErrorCode(): string
    {
        return 'EXTERNAL_SERVICE_ERROR';
    }

    protected function getDefaultUserMessage(): string
    {
        return 'An external service is temporarily unavailable. Please try again later.';
    }

    public function render($request): \Illuminate\Http\JsonResponse
    {
        $response = parent::render($request);

        // Agregar header Retry-After
        $response->header('Retry-After', $this->retryAfter);

        return $response;
    }

    public static function serviceUnavailable($serviceName, $retryAfter = 60): self
    {
        return new self(
            $serviceName,
            "External service '{$serviceName}' is currently unavailable",
            'SERVICE_UNAVAILABLE',
            ['service' => $serviceName],
            $retryAfter
        );
    }

    public static function timeoutError($serviceName, $timeout): self
    {
        return new self(
            $serviceName,
            "Request to '{$serviceName}' timed out after {$timeout} seconds",
            'SERVICE_TIMEOUT',
            ['service' => $serviceName, 'timeout' => $timeout],
            30
        );
    }

    public static function authenticationFailed($serviceName): self
    {
        return new self(
            $serviceName,
            "Authentication failed for service '{$serviceName}'",
            'SERVICE_AUTH_FAILED',
            ['service' => $serviceName],
            0 // No retry for auth failures
        );
    }

    public static function rateLimitExceeded($serviceName, $resetTime): self
    {
        return new self(
            $serviceName,
            "Rate limit exceeded for service '{$serviceName}'",
            'RATE_LIMIT_EXCEEDED',
            ['service' => $serviceName, 'reset_time' => $resetTime],
            $resetTime
        );
    }
}
```

---

## ⏱️ **Ejercicio 2: Global Exception Handler (15 minutos)**

### **Paso 1: Enhanced Exception Handler (10 minutos)**

```php
// app/Exceptions/Handler.php - Mejorado
<?php

namespace App\Exceptions;

use Illuminate\Foundation\Exceptions\Handler as ExceptionHandler;
use Illuminate\Http\Request;
use Illuminate\Database\Eloquent\ModelNotFoundException;
use Illuminate\Validation\ValidationException;
use Illuminate\Auth\AuthenticationException;
use Illuminate\Auth\Access\AuthorizationException;
use Symfony\Component\HttpKernel\Exception\NotFoundHttpException;
use Symfony\Component\HttpKernel\Exception\MethodNotAllowedHttpException;
use Symfony\Component\HttpKernel\Exception\TooManyRequestsHttpException;
use Illuminate\Database\QueryException;
use Throwable;

class Handler extends ExceptionHandler
{
    protected $dontReport = [
        //
    ];

    protected $dontFlash = [
        'current_password',
        'password',
        'password_confirmation',
    ];

    public function register()
    {
        $this->reportable(function (Throwable $e) {
            // Log crítico para errores de sistema
            if ($this->shouldReportAsCritical($e)) {
                $this->logCriticalError($e);
            }
        });

        // Renderizar respuestas JSON para APIs
        $this->renderable(function (Throwable $e, Request $request) {
            if ($request->is('api/*') || $request->expectsJson()) {
                return $this->renderApiException($e, $request);
            }
        });
    }

    protected function renderApiException(Throwable $e, Request $request)
    {
        // Si ya es una BaseApiException, dejar que se renderice sola
        if ($e instanceof BaseApiException) {
            return $e->render($request);
        }

        // Convertir excepciones comunes a formato API
        return match (true) {
            $e instanceof ValidationException => $this->handleValidationException($e, $request),
            $e instanceof ModelNotFoundException => $this->handleModelNotFoundException($e, $request),
            $e instanceof NotFoundHttpException => $this->handleNotFoundHttpException($e, $request),
            $e instanceof AuthenticationException => $this->handleAuthenticationException($e, $request),
            $e instanceof AuthorizationException => $this->handleAuthorizationException($e, $request),
            $e instanceof MethodNotAllowedHttpException => $this->handleMethodNotAllowedException($e, $request),
            $e instanceof TooManyRequestsHttpException => $this->handleTooManyRequestsException($e, $request),
            $e instanceof QueryException => $this->handleQueryException($e, $request),
            default => $this->handleGenericException($e, $request)
        };
    }

    protected function handleValidationException(ValidationException $e, Request $request)
    {
        return response()->json([
            'message' => 'The given data was invalid.',
            'error_code' => 'VALIDATION_FAILED',
            'status_code' => 422,
            'errors' => $e->errors(),
            'timestamp' => now()->toISOString(),
            'trace_id' => $this->generateTraceId($request),
        ], 422);
    }

    protected function handleModelNotFoundException(ModelNotFoundException $e, Request $request)
    {
        $model = class_basename($e->getModel());

        return response()->json([
            'message' => "The requested {$model} was not found.",
            'error_code' => 'RESOURCE_NOT_FOUND',
            'status_code' => 404,
            'timestamp' => now()->toISOString(),
            'trace_id' => $this->generateTraceId($request),
            'context' => [
                'resource_type' => $model,
                'ids' => $e->getIds()
            ]
        ], 404);
    }

    protected function handleNotFoundHttpException(NotFoundHttpException $e, Request $request)
    {
        return response()->json([
            'message' => 'The requested resource was not found.',
            'error_code' => 'ENDPOINT_NOT_FOUND',
            'status_code' => 404,
            'timestamp' => now()->toISOString(),
            'trace_id' => $this->generateTraceId($request),
            'context' => [
                'path' => $request->getPathInfo(),
                'method' => $request->getMethod()
            ]
        ], 404);
    }

    protected function handleAuthenticationException(AuthenticationException $e, Request $request)
    {
        return response()->json([
            'message' => 'Authentication required.',
            'error_code' => 'AUTHENTICATION_REQUIRED',
            'status_code' => 401,
            'timestamp' => now()->toISOString(),
            'trace_id' => $this->generateTraceId($request),
        ], 401);
    }

    protected function handleAuthorizationException(AuthorizationException $e, Request $request)
    {
        return response()->json([
            'message' => 'You do not have permission to perform this action.',
            'error_code' => 'INSUFFICIENT_PERMISSIONS',
            'status_code' => 403,
            'timestamp' => now()->toISOString(),
            'trace_id' => $this->generateTraceId($request),
        ], 403);
    }

    protected function handleMethodNotAllowedException(MethodNotAllowedHttpException $e, Request $request)
    {
        return response()->json([
            'message' => "The {$request->getMethod()} method is not allowed for this endpoint.",
            'error_code' => 'METHOD_NOT_ALLOWED',
            'status_code' => 405,
            'timestamp' => now()->toISOString(),
            'trace_id' => $this->generateTraceId($request),
            'context' => [
                'method' => $request->getMethod(),
                'allowed_methods' => $e->getHeaders()['Allow'] ?? []
            ]
        ], 405);
    }

    protected function handleTooManyRequestsException(TooManyRequestsHttpException $e, Request $request)
    {
        $retryAfter = $e->getHeaders()['Retry-After'] ?? 60;

        $response = response()->json([
            'message' => 'Too many requests. Please slow down.',
            'error_code' => 'RATE_LIMIT_EXCEEDED',
            'status_code' => 429,
            'timestamp' => now()->toISOString(),
            'trace_id' => $this->generateTraceId($request),
            'context' => [
                'retry_after' => $retryAfter,
                'limit_type' => 'requests_per_minute'
            ]
        ], 429);

        return $response->header('Retry-After', $retryAfter);
    }

    protected function handleQueryException(QueryException $e, Request $request)
    {
        // Log del error de base de datos
        logger()->error('Database error occurred', [
            'message' => $e->getMessage(),
            'sql' => $e->getSql(),
            'bindings' => $e->getBindings(),
            'trace_id' => $this->generateTraceId($request),
            'url' => $request->fullUrl(),
        ]);

        // No exponer detalles de BD en producción
        $message = app()->environment('production')
            ? 'A database error occurred. Please try again.'
            : $e->getMessage();

        return response()->json([
            'message' => 'Internal server error.',
            'error_code' => 'DATABASE_ERROR',
            'status_code' => 500,
            'timestamp' => now()->toISOString(),
            'trace_id' => $this->generateTraceId($request),
            'debug' => app()->environment('local') ? [
                'sql' => $e->getSql(),
                'bindings' => $e->getBindings(),
            ] : null
        ], 500);
    }

    protected function handleGenericException(Throwable $e, Request $request)
    {
        // Log del error genérico
        logger()->error('Unhandled exception occurred', [
            'exception' => get_class($e),
            'message' => $e->getMessage(),
            'file' => $e->getFile(),
            'line' => $e->getLine(),
            'trace_id' => $this->generateTraceId($request),
            'url' => $request->fullUrl(),
        ]);

        $statusCode = method_exists($e, 'getStatusCode') ? $e->getStatusCode() : 500;

        return response()->json([
            'message' => app()->environment('production')
                ? 'An unexpected error occurred. Please try again.'
                : $e->getMessage(),
            'error_code' => 'INTERNAL_SERVER_ERROR',
            'status_code' => $statusCode,
            'timestamp' => now()->toISOString(),
            'trace_id' => $this->generateTraceId($request),
            'debug' => app()->environment('local') ? [
                'exception' => get_class($e),
                'file' => $e->getFile(),
                'line' => $e->getLine(),
                'trace' => collect($e->getTrace())->take(3)->toArray()
            ] : null
        ], $statusCode);
    }

    protected function shouldReportAsCritical(Throwable $e): bool
    {
        return $e instanceof QueryException ||
               $e instanceof \Error ||
               ($e instanceof \Exception && $e->getCode() >= 500);
    }

    protected function logCriticalError(Throwable $e): void
    {
        logger()->critical('Critical system error', [
            'exception' => get_class($e),
            'message' => $e->getMessage(),
            'file' => $e->getFile(),
            'line' => $e->getLine(),
            'memory_usage' => memory_get_usage(true),
            'peak_memory' => memory_get_peak_usage(true),
            'url' => request()->fullUrl(),
            'user_id' => auth()->id(),
            'trace_id' => $this->generateTraceId(request()),
        ]);

        // Notificar a administradores en producción
        if (app()->environment('production')) {
            // Aquí iría notificación a Slack, email, etc.
            // $this->notifyAdministrators($e);
        }
    }

    protected function generateTraceId(Request $request): string
    {
        return $request->header('X-Trace-Id') ?? uniqid('trace_', true);
    }
}
```

### **Paso 2: Circuit Breaker Pattern (5 minutos)**

```php
// app/Services/CircuitBreakerService.php
<?php

namespace App\Services;

use Illuminate\Support\Facades\Cache;
use Illuminate\Support\Facades\Log;
use App\Exceptions\External\ExternalServiceException;

class CircuitBreakerService
{
    protected $failureThreshold = 5;
    protected $recoveryTimeout = 60; // segundos
    protected $monitoringPeriod = 300; // 5 minutos

    public function call(string $serviceName, callable $callback, callable $fallback = null)
    {
        $circuitKey = "circuit_breaker:{$serviceName}";
        $failureCountKey = "circuit_failures:{$serviceName}";

        // Verificar estado del circuit breaker
        $circuitState = Cache::get($circuitKey, 'closed');

        if ($circuitState === 'open') {
            // Circuit breaker abierto - verificar si es hora de intentar
            $lastFailure = Cache::get("{$circuitKey}:last_failure");

            if ($lastFailure && (time() - $lastFailure) > $this->recoveryTimeout) {
                // Cambiar a half-open para probar
                Cache::put($circuitKey, 'half-open', $this->monitoringPeriod);
                Log::info("Circuit breaker for {$serviceName} changed to half-open");
            } else {
                // Aún abierto - usar fallback
                return $this->executeFallback($serviceName, $fallback);
            }
        }

        try {
            // Intentar la operación
            $result = $callback();

            // Éxito - resetear contador y cerrar circuit
            Cache::forget($failureCountKey);
            Cache::put($circuitKey, 'closed', $this->monitoringPeriod);

            return $result;

        } catch (\Exception $e) {
            // Fallo - incrementar contador
            $failures = Cache::increment($failureCountKey);
            Cache::put("{$circuitKey}:last_failure", time(), $this->monitoringPeriod);

            Log::warning("Service {$serviceName} failed", [
                'failures' => $failures,
                'error' => $e->getMessage()
            ]);

            // Verificar si se debe abrir el circuit breaker
            if ($failures >= $this->failureThreshold) {
                Cache::put($circuitKey, 'open', $this->monitoringPeriod);

                Log::error("Circuit breaker opened for {$serviceName}", [
                    'failure_count' => $failures,
                    'threshold' => $this->failureThreshold
                ]);
            }

            // Si hay fallback, usarlo; si no, propagar la excepción
            if ($fallback) {
                return $this->executeFallback($serviceName, $fallback);
            }

            throw ExternalServiceException::serviceUnavailable($serviceName);
        }
    }

    protected function executeFallback(string $serviceName, callable $fallback = null)
    {
        if ($fallback) {
            Log::info("Using fallback for service {$serviceName}");
            return $fallback();
        }

        throw ExternalServiceException::serviceUnavailable($serviceName);
    }

    public function getCircuitState(string $serviceName): array
    {
        $circuitKey = "circuit_breaker:{$serviceName}";
        $failureCountKey = "circuit_failures:{$serviceName}";

        return [
            'service' => $serviceName,
            'state' => Cache::get($circuitKey, 'closed'),
            'failure_count' => Cache::get($failureCountKey, 0),
            'last_failure' => Cache::get("{$circuitKey}:last_failure"),
            'threshold' => $this->failureThreshold,
            'recovery_timeout' => $this->recoveryTimeout,
        ];
    }
}
```

---

## ⏱️ **Ejercicio 3: Structured Logging (12 minutos)**

### **Paso 1: Custom Log Channels (7 minutos)**

```php
// config/logging.php - Canales personalizados
'channels' => [
    'stack' => [
        'driver' => 'stack',
        'channels' => ['single', 'daily'],
        'ignore_exceptions' => false,
    ],

    'single' => [
        'driver' => 'single',
        'path' => storage_path('logs/laravel.log'),
        'level' => env('LOG_LEVEL', 'debug'),
    ],

    'daily' => [
        'driver' => 'daily',
        'path' => storage_path('logs/laravel.log'),
        'level' => env('LOG_LEVEL', 'debug'),
        'days' => 14,
    ],

    // Canal específico para errores de negocio
    'business' => [
        'driver' => 'daily',
        'path' => storage_path('logs/business.log'),
        'level' => 'warning',
        'days' => 30,
        'tap' => [App\Logging\BusinessLogFormatter::class],
    ],

    // Canal para errores de sistema
    'system' => [
        'driver' => 'daily',
        'path' => storage_path('logs/system.log'),
        'level' => 'error',
        'days' => 60,
        'tap' => [App\Logging\SystemLogFormatter::class],
    ],

    // Canal para auditoría
    'audit' => [
        'driver' => 'daily',
        'path' => storage_path('logs/audit.log'),
        'level' => 'info',
        'days' => 365,
        'tap' => [App\Logging\AuditLogFormatter::class],
    ],

    // Canal para performance
    'performance' => [
        'driver' => 'daily',
        'path' => storage_path('logs/performance.log'),
        'level' => 'info',
        'days' => 7,
        'tap' => [App\Logging\PerformanceLogFormatter::class],
    ],

    'slack' => [
        'driver' => 'slack',
        'url' => env('LOG_SLACK_WEBHOOK_URL'),
        'username' => 'Tournament API',
        'emoji' => ':warning:',
        'level' => 'critical',
    ],
],
```

```php
// app/Logging/BusinessLogFormatter.php
<?php

namespace App\Logging;

use Monolog\Formatter\JsonFormatter;
use Monolog\LogRecord;

class BusinessLogFormatter
{
    public function __invoke($logger)
    {
        foreach ($logger->getHandlers() as $handler) {
            $handler->setFormatter(new class extends JsonFormatter {
                public function format(LogRecord $record): string
                {
                    $formatted = [
                        'timestamp' => $record->datetime->toISOString(),
                        'level' => $record->level->name,
                        'message' => $record->message,
                        'type' => 'business_error',
                        'context' => $record->context,
                        'extra' => $record->extra,
                    ];

                    // Agregar información específica de negocio
                    if (isset($record->context['error_code'])) {
                        $formatted['error_code'] = $record->context['error_code'];
                    }

                    if (isset($record->context['user_id'])) {
                        $formatted['user_id'] = $record->context['user_id'];
                    }

                    return json_encode($formatted, JSON_UNESCAPED_SLASHES | JSON_UNESCAPED_UNICODE) . "\n";
                }
            });
        }
    }
}
```

### **Paso 2: Structured Logging Service (5 minutos)**

```php
// app/Services/LoggingService.php
<?php

namespace App\Services;

use Illuminate\Support\Facades\Log;

class LoggingService
{
    public function logBusinessError(string $errorCode, string $message, array $context = []): void
    {
        $logContext = array_merge([
            'error_code' => $errorCode,
            'user_id' => auth()->id(),
            'session_id' => session()->getId(),
            'ip_address' => request()->ip(),
            'user_agent' => request()->userAgent(),
            'url' => request()->fullUrl(),
            'method' => request()->method(),
            'timestamp' => now()->toISOString(),
        ], $context);

        Log::channel('business')->error($message, $logContext);
    }

    public function logSystemError(\Throwable $exception, array $context = []): void
    {
        $logContext = array_merge([
            'exception_class' => get_class($exception),
            'file' => $exception->getFile(),
            'line' => $exception->getLine(),
            'trace' => $exception->getTraceAsString(),
            'memory_usage' => memory_get_usage(true),
            'peak_memory' => memory_get_peak_usage(true),
            'user_id' => auth()->id(),
            'url' => request()->fullUrl(),
            'timestamp' => now()->toISOString(),
        ], $context);

        Log::channel('system')->error($exception->getMessage(), $logContext);

        // Si es crítico, también notificar por Slack
        if ($this->isCriticalError($exception)) {
            Log::channel('slack')->critical('Critical system error occurred', [
                'exception' => get_class($exception),
                'message' => $exception->getMessage(),
                'url' => request()->fullUrl(),
            ]);
        }
    }

    public function logUserAction(string $action, array $context = []): void
    {
        $logContext = array_merge([
            'action' => $action,
            'user_id' => auth()->id(),
            'user_email' => auth()->user()?->email,
            'ip_address' => request()->ip(),
            'user_agent' => request()->userAgent(),
            'url' => request()->fullUrl(),
            'method' => request()->method(),
            'timestamp' => now()->toISOString(),
        ], $context);

        Log::channel('audit')->info("User action: {$action}", $logContext);
    }

    public function logPerformanceMetric(string $operation, float $duration, array $context = []): void
    {
        $logContext = array_merge([
            'operation' => $operation,
            'duration_ms' => round($duration * 1000, 2),
            'memory_usage' => memory_get_usage(true),
            'peak_memory' => memory_get_peak_usage(true),
            'url' => request()->fullUrl(),
            'timestamp' => now()->toISOString(),
        ], $context);

        Log::channel('performance')->info("Performance metric: {$operation}", $logContext);
    }

    public function logApiRequest(array $context = []): void
    {
        $request = request();

        $logContext = array_merge([
            'method' => $request->method(),
            'url' => $request->fullUrl(),
            'ip' => $request->ip(),
            'user_agent' => $request->userAgent(),
            'user_id' => auth()->id(),
            'request_id' => $request->header('X-Request-ID', uniqid()),
            'content_type' => $request->header('Content-Type'),
            'accept' => $request->header('Accept'),
            'timestamp' => now()->toISOString(),
        ], $context);

        // No logear datos sensibles
        $inputData = $request->except(['password', 'password_confirmation', 'token']);
        if (!empty($inputData)) {
            $logContext['input'] = $inputData;
        }

        Log::channel('audit')->info('API request', $logContext);
    }

    public function logApiResponse(int $statusCode, float $duration, array $context = []): void
    {
        $logContext = array_merge([
            'status_code' => $statusCode,
            'duration_ms' => round($duration * 1000, 2),
            'memory_usage' => memory_get_usage(true),
            'url' => request()->fullUrl(),
            'timestamp' => now()->toISOString(),
        ], $context);

        $level = $statusCode >= 500 ? 'error' : ($statusCode >= 400 ? 'warning' : 'info');

        Log::channel('audit')->{$level}('API response', $logContext);
    }

    protected function isCriticalError(\Throwable $exception): bool
    {
        return $exception instanceof \Error ||
               $exception instanceof \ParseError ||
               $exception instanceof \TypeError ||
               ($exception instanceof \Exception && str_contains($exception->getMessage(), 'database'));
    }
}
```

---

## ⏱️ **Ejercicio 4: Error Recovery (10 minutos)**

### **Paso 1: Retry Middleware (5 minutos)**

```php
// app/Http/Middleware/RetryMiddleware.php
<?php

namespace App\Http\Middleware;

use Closure;
use Illuminate\Http\Request;
use App\Services\LoggingService;

class RetryMiddleware
{
    protected $loggingService;
    protected $maxRetries = 3;
    protected $retryDelay = 1000; // milliseconds

    public function __construct(LoggingService $loggingService)
    {
        $this->loggingService = $loggingService;
    }

    public function handle(Request $request, Closure $next, $maxRetries = 3)
    {
        $this->maxRetries = (int) $maxRetries;
        $attempt = 0;

        while ($attempt < $this->maxRetries) {
            try {
                return $next($request);
            } catch (\Exception $e) {
                $attempt++;

                if (!$this->shouldRetry($e) || $attempt >= $this->maxRetries) {
                    throw $e;
                }

                $this->loggingService->logBusinessError(
                    'REQUEST_RETRY',
                    "Retrying request (attempt {$attempt}/{$this->maxRetries})",
                    [
                        'attempt' => $attempt,
                        'max_retries' => $this->maxRetries,
                        'exception' => get_class($e),
                        'message' => $e->getMessage(),
                    ]
                );

                // Esperar antes del siguiente intento
                usleep($this->retryDelay * 1000 * $attempt); // Backoff exponencial
            }
        }

        throw $e; // No debería llegar aquí, pero por seguridad
    }

    protected function shouldRetry(\Exception $e): bool
    {
        // Solo reintentar para errores temporales
        return $e instanceof \Illuminate\Database\QueryException ||
               $e instanceof \GuzzleHttp\Exception\ConnectException ||
               $e instanceof \GuzzleHttp\Exception\RequestException ||
               (method_exists($e, 'getStatusCode') && $e->getStatusCode() >= 500);
    }
}
```

### **Paso 2: Graceful Degradation (5 minutos)**

```php
// app/Services/GracefulDegradationService.php
<?php

namespace App\Services;

use Illuminate\Support\Facades\Cache;
use App\Services\LoggingService;

class GracefulDegradationService
{
    protected $loggingService;

    public function __construct(LoggingService $loggingService)
    {
        $this->loggingService = $loggingService;
    }

    public function withFallback(string $operation, callable $primary, callable $fallback = null, int $cacheTtl = 300)
    {
        try {
            $result = $primary();

            // Cache del resultado exitoso
            $cacheKey = "fallback_cache:" . md5($operation);
            Cache::put($cacheKey, $result, $cacheTtl);

            return $result;

        } catch (\Exception $e) {
            $this->loggingService->logBusinessError(
                'GRACEFUL_DEGRADATION',
                "Primary operation failed, attempting fallback: {$operation}",
                [
                    'operation' => $operation,
                    'exception' => get_class($e),
                    'message' => $e->getMessage(),
                ]
            );

            // Intentar fallback si está disponible
            if ($fallback) {
                try {
                    return $fallback();
                } catch (\Exception $fallbackException) {
                    $this->loggingService->logSystemError($fallbackException, [
                        'operation' => $operation,
                        'stage' => 'fallback_failed'
                    ]);
                }
            }

            // Intentar usar cache como último recurso
            $cacheKey = "fallback_cache:" . md5($operation);
            $cachedResult = Cache::get($cacheKey);

            if ($cachedResult !== null) {
                $this->loggingService->logBusinessError(
                    'CACHE_FALLBACK_USED',
                    "Using cached data as fallback for: {$operation}",
                    ['operation' => $operation]
                );

                return $cachedResult;
            }

            // Si todo falla, propagar la excepción original
            throw $e;
        }
    }

    public function getDefaultPlayerStats(): array
    {
        return [
            'goals' => 0,
            'assists' => 0,
            'yellow_cards' => 0,
            'red_cards' => 0,
            'matches_played' => 0,
            'average_rating' => 0.0,
            'note' => 'Statistics temporarily unavailable'
        ];
    }

    public function getDefaultTournamentStandings(): array
    {
        return [
            'standings' => [],
            'last_updated' => null,
            'note' => 'Standings temporarily unavailable - using cached data'
        ];
    }

    public function getEmptyPaginatedResponse(): array
    {
        return [
            'data' => [],
            'meta' => [
                'current_page' => 1,
                'from' => null,
                'to' => null,
                'per_page' => 15,
                'total' => 0,
                'last_page' => 1,
                'has_more_pages' => false,
            ],
            'links' => [
                'first' => null,
                'last' => null,
                'prev' => null,
                'next' => null,
            ],
            'note' => 'Data temporarily unavailable'
        ];
    }
}
```

---

## 🎯 **Testing Error Handling**

```bash
# Test de excepciones personalizadas
php artisan make:test ExceptionHandlingTest

# Simular errores de base de datos
php artisan tinker
>>> throw new \Illuminate\Database\QueryException('connection', 'select * from users', [], new \Exception('Connection failed'));

# Test de circuit breaker
>>> app(\App\Services\CircuitBreakerService::class)->call('test_service', function() { throw new \Exception('Service failed'); });

# Ver logs estructurados
tail -f storage/logs/business.log | jq .
tail -f storage/logs/system.log | jq .
```

---

## 🎯 **Rúbrica de Evaluación**

| Criterio                | Excelente (4)                  | Bueno (3)           | Satisfactorio (2)         | Insuficiente (1)          |
| ----------------------- | ------------------------------ | ------------------- | ------------------------- | ------------------------- |
| **Exception Hierarchy** | Jerarquía completa + contexto  | Excepciones básicas | Algunas custom exceptions | Solo excepciones built-in |
| **Global Handler**      | Handler completo + logging     | Handler básico      | Manejo parcial            | Sin manejo global         |
| **Structured Logging**  | Logs estructurados + canales   | Logging básico      | Logs simples              | Sin logging               |
| **Error Recovery**      | Circuit breaker + fallbacks    | Algunas estrategias | Recovery básico           | Sin recovery              |
| **Response Format**     | Formato consistente + metadata | Respuestas JSON     | Formato básico            | Respuestas inconsistentes |
| **Tiempo**              | < 15 minutos                   | 15-20 min           | 20-30 min                 | > 30 min                  |

---

## 🔧 **Commands de Debug**

```bash
# Ver logs en tiempo real
tail -f storage/logs/laravel.log
tail -f storage/logs/business.log
tail -f storage/logs/system.log

# Test de errores
curl -X POST http://localhost:8000/api/players \
  -H "Content-Type: application/json" \
  -d '{"invalid": "data"}'

# Verificar estado de circuit breakers
php artisan tinker
>>> app(\App\Services\CircuitBreakerService::class)->getCircuitState('external_api');
```

---

**🚨 Con este sistema de error handling, tu API será robusta y recuperable ante cualquier fallo del sistema!**
