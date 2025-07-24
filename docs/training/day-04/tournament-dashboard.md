# 🏆 Dashboard del Torneo - Componentes Avanzados

**Duración:** 1 hora (5:00 PM - 6:00 PM)  
**Objetivo:** Crear dashboard interactivo con tabla de posiciones y estadísticas en tiempo real

---

## 🎯 **Objetivos de la Sesión**

Al finalizar esta sesión, el competidor será capaz de:

✅ **Crear tabla de posiciones** interactiva con ordenamiento automático  
✅ **Implementar estadísticas** del torneo en tiempo real  
✅ **Desarrollar gráficos simples** con CSS y HTML  
✅ **Construir widgets informativos** del dashboard  
✅ **Optimizar performance** para actualizaciones frecuentes

---

## 🏆 **Paso 1: Componente Tabla de Posiciones (20 minutos)**

El componente más importante y complejo del dashboard deportivo.

### **📄 `src/components/tournament/StandingsTable.jsx`:**

```jsx
import { useMemo } from 'react';
import {
  TrophyIcon,
  ArrowUpIcon,
  ArrowDownIcon,
} from '@heroicons/react/24/solid';
import { useStandings } from '../../hooks/useStandings';
import clsx from 'clsx';

const StandingsTable = ({
  phase = 'all',
  showOnlyTop = null,
  compact = false,
}) => {
  const { standings, loading, error } = useStandings(phase);

  // Procesar datos para mostrar tendencias
  const processedStandings = useMemo(() => {
    if (!standings) return [];

    return standings.map((team, index) => ({
      ...team,
      position: index + 1,
      trend:
        index < 3 ? 'up' : index > standings.length - 4 ? 'down' : 'stable',
      isChampion: index === 0,
      isTopThree: index < 3,
      isRelegation: index >= standings.length - 3,
    }));
  }, [standings]);

  // Limitar resultados si es necesario
  const displayStandings = useMemo(() => {
    if (showOnlyTop && processedStandings.length > showOnlyTop) {
      return processedStandings.slice(0, showOnlyTop);
    }
    return processedStandings;
  }, [processedStandings, showOnlyTop]);

  const getTrendIcon = (trend) => {
    switch (trend) {
      case 'up':
        return <ArrowUpIcon className="w-4 h-4 text-green-500" />;
      case 'down':
        return <ArrowDownIcon className="w-4 h-4 text-red-500" />;
      default:
        return <div className="w-4 h-4" />; // Espacio vacío
    }
  };

  const getPositionStyle = (team) => {
    if (team.isChampion) {
      return 'bg-gradient-to-r from-yellow-50 to-yellow-100 border-l-4 border-yellow-400';
    }
    if (team.isTopThree) {
      return 'bg-gradient-to-r from-green-50 to-green-100 border-l-4 border-green-400';
    }
    if (team.isRelegation) {
      return 'bg-gradient-to-r from-red-50 to-red-100 border-l-4 border-red-400';
    }
    return 'hover:bg-gray-50';
  };

  if (loading) {
    return (
      <div className="animate-pulse">
        <div className="bg-white shadow rounded-lg overflow-hidden">
          <div className="h-12 bg-gray-200 mb-4"></div>
          {Array.from({ length: 8 }).map((_, i) => (
            <div
              key={i}
              className="h-16 bg-gray-100 mb-2 mx-4"></div>
          ))}
        </div>
      </div>
    );
  }

  if (error) {
    return (
      <div className="bg-red-50 border border-red-200 rounded-lg p-4">
        <p className="text-red-800">
          Error cargando tabla de posiciones: {error}
        </p>
      </div>
    );
  }

  return (
    <div className="bg-white shadow rounded-lg overflow-hidden">
      {/* Header */}
      <div className="bg-gradient-to-r from-primary-600 to-primary-700 px-6 py-4">
        <div className="flex items-center">
          <TrophyIcon className="w-6 h-6 text-yellow-300 mr-2" />
          <h2 className="text-xl font-bold text-white">Tabla de Posiciones</h2>
        </div>
      </div>

      {/* Tabla */}
      <div className="overflow-x-auto">
        <table className="min-w-full">
          <thead className="bg-gray-50">
            <tr>
              <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                Pos
              </th>
              <th className="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                Equipo
              </th>
              {!compact && (
                <>
                  <th className="px-6 py-3 text-center text-xs font-medium text-gray-500 uppercase tracking-wider">
                    PJ
                  </th>
                  <th className="px-6 py-3 text-center text-xs font-medium text-gray-500 uppercase tracking-wider">
                    PG
                  </th>
                  <th className="px-6 py-3 text-center text-xs font-medium text-gray-500 uppercase tracking-wider">
                    PE
                  </th>
                  <th className="px-6 py-3 text-center text-xs font-medium text-gray-500 uppercase tracking-wider">
                    PP
                  </th>
                  <th className="px-6 py-3 text-center text-xs font-medium text-gray-500 uppercase tracking-wider">
                    GF
                  </th>
                  <th className="px-6 py-3 text-center text-xs font-medium text-gray-500 uppercase tracking-wider">
                    GC
                  </th>
                  <th className="px-6 py-3 text-center text-xs font-medium text-gray-500 uppercase tracking-wider">
                    DG
                  </th>
                </>
              )}
              <th className="px-6 py-3 text-center text-xs font-medium text-gray-500 uppercase tracking-wider">
                Pts
              </th>
            </tr>
          </thead>
          <tbody className="bg-white divide-y divide-gray-200">
            {displayStandings.map((team) => (
              <tr
                key={team.id}
                className={clsx(
                  'transition-colors duration-150',
                  getPositionStyle(team)
                )}>
                <td className="px-6 py-4 whitespace-nowrap">
                  <div className="flex items-center">
                    <span className="text-lg font-bold text-gray-900 mr-2">
                      {team.position}
                    </span>
                    {getTrendIcon(team.trend)}
                  </div>
                </td>

                <td className="px-6 py-4 whitespace-nowrap">
                  <div className="flex items-center">
                    {team.country?.flag && (
                      <img
                        src={team.country.flag}
                        alt={team.country.name}
                        className="w-6 h-4 mr-3 object-cover rounded"
                      />
                    )}
                    <div>
                      <div className="text-sm font-medium text-gray-900">
                        {team.name}
                      </div>
                      <div className="text-sm text-gray-500">
                        {team.country?.name}
                      </div>
                    </div>
                  </div>
                </td>

                {!compact && (
                  <>
                    <td className="px-6 py-4 whitespace-nowrap text-center text-sm text-gray-900">
                      {team.matches_played}
                    </td>
                    <td className="px-6 py-4 whitespace-nowrap text-center text-sm text-green-600 font-medium">
                      {team.wins}
                    </td>
                    <td className="px-6 py-4 whitespace-nowrap text-center text-sm text-gray-600">
                      {team.draws}
                    </td>
                    <td className="px-6 py-4 whitespace-nowrap text-center text-sm text-red-600">
                      {team.losses}
                    </td>
                    <td className="px-6 py-4 whitespace-nowrap text-center text-sm text-gray-900">
                      {team.goals_for}
                    </td>
                    <td className="px-6 py-4 whitespace-nowrap text-center text-sm text-gray-900">
                      {team.goals_against}
                    </td>
                    <td className="px-6 py-4 whitespace-nowrap text-center text-sm font-medium">
                      <span
                        className={clsx(
                          team.goal_difference >= 0
                            ? 'text-green-600'
                            : 'text-red-600'
                        )}>
                        {team.goal_difference >= 0 ? '+' : ''}
                        {team.goal_difference}
                      </span>
                    </td>
                  </>
                )}

                <td className="px-6 py-4 whitespace-nowrap text-center">
                  <span className="text-lg font-bold text-gray-900">
                    {team.points}
                  </span>
                </td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>

      {/* Leyenda */}
      {!compact && (
        <div className="bg-gray-50 px-6 py-3 border-t">
          <div className="flex flex-wrap gap-4 text-xs">
            <div className="flex items-center">
              <div className="w-3 h-3 bg-yellow-400 rounded mr-2"></div>
              <span className="text-gray-600">Campeón</span>
            </div>
            <div className="flex items-center">
              <div className="w-3 h-3 bg-green-400 rounded mr-2"></div>
              <span className="text-gray-600">Clasificación</span>
            </div>
            <div className="flex items-center">
              <div className="w-3 h-3 bg-red-400 rounded mr-2"></div>
              <span className="text-gray-600">Descenso</span>
            </div>
          </div>
        </div>
      )}
    </div>
  );
};

export default StandingsTable;
```

---

## 📊 **Paso 2: Estadísticas del Torneo (15 minutos)**

Widget para mostrar estadísticas generales del torneo.

### **📄 `src/components/tournament/TournamentStats.jsx`:**

```jsx
import { useMemo } from 'react';
import {
  TrophyIcon,
  UsersIcon,
  ClockIcon,
  ChartBarIcon,
} from '@heroicons/react/24/outline';
import { useStandings } from '../../hooks/useStandings';
import { useTeams } from '../../hooks/useTeams';

const StatCard = ({
  title,
  value,
  subtitle,
  icon: Icon,
  color = 'primary',
}) => {
  const colorClasses = {
    primary: 'bg-primary-500 text-primary-600',
    green: 'bg-green-500 text-green-600',
    blue: 'bg-blue-500 text-blue-600',
    orange: 'bg-orange-500 text-orange-600',
  };

  return (
    <div className="bg-white overflow-hidden shadow rounded-lg">
      <div className="p-5">
        <div className="flex items-center">
          <div className="flex-shrink-0">
            <div
              className={`p-3 rounded-md ${colorClasses[color]} bg-opacity-10`}>
              <Icon className={`w-6 h-6 ${colorClasses[color]}`} />
            </div>
          </div>
          <div className="ml-5 w-0 flex-1">
            <dl>
              <dt className="text-sm font-medium text-gray-500 truncate">
                {title}
              </dt>
              <dd className="flex items-baseline">
                <div className="text-2xl font-semibold text-gray-900">
                  {value}
                </div>
                {subtitle && (
                  <div className="ml-2 text-sm text-gray-500">{subtitle}</div>
                )}
              </dd>
            </dl>
          </div>
        </div>
      </div>
    </div>
  );
};

const TournamentStats = () => {
  const { standings, loading: standingsLoading } = useStandings();
  const { teams, loading: teamsLoading } = useTeams();

  const stats = useMemo(() => {
    if (!standings || !teams) return {};

    const totalTeams = teams.length;
    const totalMatches =
      standings.reduce((sum, team) => sum + team.matches_played, 0) / 2; // Dividir por 2 porque cada partido cuenta para 2 equipos
    const totalGoals = standings.reduce((sum, team) => sum + team.goals_for, 0);
    const averageGoalsPerMatch =
      totalMatches > 0 ? (totalGoals / totalMatches).toFixed(1) : 0;

    // Buscar el equipo líder
    const leader = standings[0];

    // Calcular partidos restantes (asumiendo torneo todos contra todos)
    const totalPossibleMatches = (totalTeams * (totalTeams - 1)) / 2;
    const remainingMatches = totalPossibleMatches - totalMatches;

    return {
      totalTeams,
      totalMatches,
      totalGoals,
      averageGoalsPerMatch,
      leader,
      remainingMatches,
    };
  }, [standings, teams]);

  const loading = standingsLoading || teamsLoading;

  if (loading) {
    return (
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
        {Array.from({ length: 4 }).map((_, i) => (
          <div
            key={i}
            className="bg-white shadow rounded-lg p-5 animate-pulse">
            <div className="h-6 bg-gray-200 rounded mb-2"></div>
            <div className="h-8 bg-gray-200 rounded"></div>
          </div>
        ))}
      </div>
    );
  }

  return (
    <div className="space-y-6">
      <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
        <StatCard
          title="Equipos Participantes"
          value={stats.totalTeams}
          icon={UsersIcon}
          color="primary"
        />

        <StatCard
          title="Partidos Jugados"
          value={stats.totalMatches}
          subtitle={`${stats.remainingMatches} restantes`}
          icon={ClockIcon}
          color="green"
        />

        <StatCard
          title="Goles Totales"
          value={stats.totalGoals}
          subtitle={`${stats.averageGoalsPerMatch} por partido`}
          icon={ChartBarIcon}
          color="orange"
        />

        <StatCard
          title="Líder"
          value={stats.leader?.name || 'N/A'}
          subtitle={`${stats.leader?.points || 0} puntos`}
          icon={TrophyIcon}
          color="blue"
        />
      </div>

      {/* Gráfico de rendimiento simple */}
      <div className="bg-white shadow rounded-lg p-6">
        <h3 className="text-lg font-medium text-gray-900 mb-4">
          Rendimiento por Equipos (Top 6)
        </h3>
        <div className="space-y-3">
          {standings?.slice(0, 6).map((team, index) => {
            const maxPoints = Math.max(...standings.map((t) => t.points));
            const percentage =
              maxPoints > 0 ? (team.points / maxPoints) * 100 : 0;

            return (
              <div
                key={team.id}
                className="flex items-center">
                <div className="w-24 text-sm text-gray-600 truncate">
                  {team.name}
                </div>
                <div className="flex-1 mx-4">
                  <div className="bg-gray-200 rounded-full h-4">
                    <div
                      className={`h-4 rounded-full transition-all duration-500 ${
                        index === 0
                          ? 'bg-yellow-400'
                          : index < 3
                          ? 'bg-green-400'
                          : 'bg-blue-400'
                      }`}
                      style={{ width: `${percentage}%` }}></div>
                  </div>
                </div>
                <div className="w-16 text-sm font-medium text-gray-900 text-right">
                  {team.points} pts
                </div>
              </div>
            );
          })}
        </div>
      </div>
    </div>
  );
};

export default TournamentStats;
```

---

## 📅 **Paso 3: Próximos Partidos (10 minutos)**

Widget para mostrar los próximos partidos programados.

### **📄 `src/components/tournament/UpcomingMatches.jsx`:**

```jsx
import { useState, useMemo } from 'react';
import { CalendarIcon, ClockIcon } from '@heroicons/react/24/outline';
import { format, isToday, isTomorrow, addDays } from 'date-fns';
import { es } from 'date-fns/locale';

const MatchCard = ({ match }) => {
  const matchDate = new Date(match.match_date);
  const isMatchToday = isToday(matchDate);
  const isMatchTomorrow = isTomorrow(matchDate);

  const getDateLabel = () => {
    if (isMatchToday) return 'Hoy';
    if (isMatchTomorrow) return 'Mañana';
    return format(matchDate, 'dd MMM', { locale: es });
  };

  const getTimeLabel = () => {
    return format(matchDate, 'HH:mm');
  };

  return (
    <div
      className={`border rounded-lg p-4 transition-colors duration-200 ${
        isMatchToday
          ? 'border-green-300 bg-green-50'
          : isMatchTomorrow
          ? 'border-blue-300 bg-blue-50'
          : 'border-gray-200 hover:border-gray-300'
      }`}>
      <div className="flex justify-between items-start mb-3">
        <div className="flex items-center text-sm text-gray-500">
          <CalendarIcon className="w-4 h-4 mr-1" />
          {getDateLabel()}
        </div>
        <div className="flex items-center text-sm text-gray-500">
          <ClockIcon className="w-4 h-4 mr-1" />
          {getTimeLabel()}
        </div>
      </div>

      <div className="flex items-center justify-between">
        <div className="flex-1 text-center">
          <div className="flex items-center justify-center mb-1">
            {match.home_team.country?.flag && (
              <img
                src={match.home_team.country.flag}
                alt={match.home_team.country.name}
                className="w-6 h-4 mr-2 object-cover rounded"
              />
            )}
            <span className="font-medium text-gray-900">
              {match.home_team.name}
            </span>
          </div>
          <span className="text-xs text-gray-500">Local</span>
        </div>

        <div className="px-4">
          <div className="text-lg font-bold text-gray-400">VS</div>
        </div>

        <div className="flex-1 text-center">
          <div className="flex items-center justify-center mb-1">
            <span className="font-medium text-gray-900">
              {match.away_team.name}
            </span>
            {match.away_team.country?.flag && (
              <img
                src={match.away_team.country.flag}
                alt={match.away_team.country.name}
                className="w-6 h-4 ml-2 object-cover rounded"
              />
            )}
          </div>
          <span className="text-xs text-gray-500">Visitante</span>
        </div>
      </div>

      {match.phase && (
        <div className="mt-3 text-center">
          <span className="inline-block bg-gray-100 text-gray-800 text-xs px-2 py-1 rounded">
            {match.phase}
          </span>
        </div>
      )}
    </div>
  );
};

const UpcomingMatches = ({ limit = 6 }) => {
  const [matches] = useState([]); // Esto vendría de un hook useMatches
  const [loading] = useState(false);

  const upcomingMatches = useMemo(() => {
    const now = new Date();
    const nextWeek = addDays(now, 7);

    return matches
      .filter((match) => {
        const matchDate = new Date(match.match_date);
        return matchDate >= now && matchDate <= nextWeek && !match.played;
      })
      .sort((a, b) => new Date(a.match_date) - new Date(b.match_date))
      .slice(0, limit);
  }, [matches, limit]);

  if (loading) {
    return (
      <div className="bg-white shadow rounded-lg p-6">
        <h3 className="text-lg font-medium text-gray-900 mb-4">
          Próximos Partidos
        </h3>
        <div className="space-y-4">
          {Array.from({ length: 3 }).map((_, i) => (
            <div
              key={i}
              className="animate-pulse">
              <div className="h-20 bg-gray-200 rounded-lg"></div>
            </div>
          ))}
        </div>
      </div>
    );
  }

  return (
    <div className="bg-white shadow rounded-lg p-6">
      <h3 className="text-lg font-medium text-gray-900 mb-4">
        Próximos Partidos
      </h3>

      {upcomingMatches.length === 0 ? (
        <div className="text-center py-8">
          <CalendarIcon className="w-12 h-12 text-gray-400 mx-auto mb-3" />
          <p className="text-gray-500">
            No hay partidos programados para los próximos días
          </p>
        </div>
      ) : (
        <div className="space-y-4">
          {upcomingMatches.map((match) => (
            <MatchCard
              key={match.id}
              match={match}
            />
          ))}
        </div>
      )}
    </div>
  );
};

export default UpcomingMatches;
```

---

## 🎯 **Paso 4: Dashboard Principal (10 minutos)**

Componente que integra todos los widgets del dashboard.

### **📄 `src/pages/Dashboard/Dashboard.jsx`:**

```jsx
import { useState } from 'react';
import StandingsTable from '../../components/tournament/StandingsTable';
import TournamentStats from '../../components/tournament/TournamentStats';
import UpcomingMatches from '../../components/tournament/UpcomingMatches';
import { RefreshIcon } from '@heroicons/react/24/outline';
import Button from '../../components/ui/Button';

const Dashboard = () => {
  const [refreshing, setRefreshing] = useState(false);

  const handleRefresh = async () => {
    setRefreshing(true);
    // Simular refresh de datos
    await new Promise((resolve) => setTimeout(resolve, 1000));

    // Aquí normalmente refrescarías los datos de los hooks
    // refetchStandings();
    // refetchMatches();

    setRefreshing(false);
  };

  return (
    <div className="space-y-6">
      {/* Header */}
      <div className="flex justify-between items-center">
        <div>
          <h1 className="text-2xl font-bold text-gray-900">
            Dashboard del Torneo
          </h1>
          <p className="mt-1 text-sm text-gray-500">
            Torneo Femenino Suramericano 2025
          </p>
        </div>

        <Button
          onClick={handleRefresh}
          loading={refreshing}
          variant="outline"
          className="flex items-center">
          <RefreshIcon className="w-4 h-4 mr-2" />
          Actualizar
        </Button>
      </div>

      {/* Estadísticas generales */}
      <TournamentStats />

      {/* Contenido principal */}
      <div className="grid grid-cols-1 lg:grid-cols-3 gap-6">
        {/* Tabla de posiciones - ocupa 2 columnas */}
        <div className="lg:col-span-2">
          <StandingsTable />
        </div>

        {/* Sidebar con información adicional */}
        <div className="space-y-6">
          <UpcomingMatches limit={4} />

          {/* Top 3 Resumido */}
          <div className="bg-white shadow rounded-lg p-6">
            <h3 className="text-lg font-medium text-gray-900 mb-4">Top 3</h3>
            <StandingsTable
              showOnlyTop={3}
              compact={true}
            />
          </div>
        </div>
      </div>

      {/* Información adicional */}
      <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
        <div className="bg-white shadow rounded-lg p-6">
          <h3 className="text-lg font-medium text-gray-900 mb-4">
            Información del Torneo
          </h3>
          <dl className="space-y-2">
            <div className="flex justify-between">
              <dt className="text-sm text-gray-500">Formato:</dt>
              <dd className="text-sm text-gray-900">Todos contra todos</dd>
            </div>
            <div className="flex justify-between">
              <dt className="text-sm text-gray-500">Duración:</dt>
              <dd className="text-sm text-gray-900">3 semanas</dd>
            </div>
            <div className="flex justify-between">
              <dt className="text-sm text-gray-500">Clasifican:</dt>
              <dd className="text-sm text-gray-900">Top 4 equipos</dd>
            </div>
          </dl>
        </div>

        <div className="bg-white shadow rounded-lg p-6">
          <h3 className="text-lg font-medium text-gray-900 mb-4">
            Récords del Torneo
          </h3>
          <dl className="space-y-2">
            <div className="flex justify-between">
              <dt className="text-sm text-gray-500">Mayor goleada:</dt>
              <dd className="text-sm text-gray-900">5-0</dd>
            </div>
            <div className="flex justify-between">
              <dt className="text-sm text-gray-500">
                Más goles en un partido:
              </dt>
              <dd className="text-sm text-gray-900">7 goles</dd>
            </div>
            <div className="flex justify-between">
              <dt className="text-sm text-gray-500">Racha ganadora:</dt>
              <dd className="text-sm text-gray-900">5 partidos</dd>
            </div>
          </dl>
        </div>
      </div>
    </div>
  );
};

export default Dashboard;
```

---

## 📱 **Paso 5: Responsividad y Optimización (5 minutos)**

Mejoras para que el dashboard funcione perfectamente en móviles.

### **📄 CSS adicional para responsividad:**

```css
/* Agregar a src/index.css */

/* Tabla responsiva */
@media (max-width: 640px) {
  .standings-table {
    font-size: 12px;
  }

  .standings-table th,
  .standings-table td {
    padding: 8px 4px;
  }

  .standings-table .hidden-mobile {
    display: none;
  }
}

/* Cards responsivas */
@media (max-width: 768px) {
  .stats-grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

@media (max-width: 480px) {
  .stats-grid {
    grid-template-columns: 1fr;
  }
}

/* Animaciones suaves */
.standings-row {
  transition: all 0.3s ease;
}

.standings-row:hover {
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

/* Loading skeleton específico */
.skeleton-standings {
  background: linear-gradient(90deg, #f0f0f0 25%, #e0e0e0 50%, #f0f0f0 75%);
  background-size: 200% 100%;
  animation: loading 1.5s infinite;
}

@keyframes loading {
  0% {
    background-position: 200% 0;
  }
  100% {
    background-position: -200% 0;
  }
}
```

---

## 🎯 **Testing del Dashboard**

### **Checklist de funcionalidad:**

- ✅ **Tabla de posiciones** se carga y ordena correctamente
- ✅ **Estadísticas** calculan valores apropiados
- ✅ **Responsive design** funciona en móvil y desktop
- ✅ **Loading states** se muestran apropiadamente
- ✅ **Error handling** maneja fallos de API
- ✅ **Performance** es fluida con datos reales

---

## 💡 **Tips para WorldSkills**

### **Impresionar a los Jueces:**

1. **Animaciones sutiles** en transiciones
2. **Colores consistentes** con el tema del torneo
3. **Información clara** y fácil de entender
4. **Responsive perfecto** en todos los dispositivos
5. **Performance óptima** sin lags visibles

### **Debugging Rápido:**

```jsx
// Componente de debug para desarrollo
const DebugPanel = ({ data }) => {
  if (process.env.NODE_ENV !== 'development') return null;

  return (
    <div className="fixed bottom-4 left-4 bg-black text-white p-2 rounded text-xs max-w-xs">
      <pre>{JSON.stringify(data, null, 2)}</pre>
    </div>
  );
};
```

---

¡Con este dashboard interactivo, tendrás la interfaz más impresionante del torneo! 🏆✨
