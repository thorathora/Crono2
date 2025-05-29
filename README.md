
import React, { useState, useEffect, useRef, useCallback } from 'react';
import ReactDOM from 'react-dom/client';

// --- SVG Icons ---
const PlayIcon: React.FC = () => (
  <svg viewBox="0 0 24 24" fill="currentColor" width="24" height="24">
    <path d="M8 5v14l11-7z" />
  </svg>
);

const PauseIcon: React.FC = () => (
  <svg viewBox="0 0 24 24" fill="currentColor" width="24" height="24">
    <path d="M6 19h4V5H6v14zm8-14v14h4V5h-4z" />
  </svg>
);

const ResetIcon: React.FC = () => (
  <svg viewBox="0 0 24 24" fill="currentColor" width="24" height="24">
    <path d="M12 5V1L7 6l5 5V7c3.31 0 6 2.69 6 6s-2.69 6-6 6-6-2.69-6-6H4c0 4.42 3.58 8 8 8s8-3.58 8-8-3.58-8-8-8z" />
  </svg>
);

const DeleteIcon: React.FC = () => (
  <svg viewBox="0 0 24 24" fill="currentColor" width="24" height="24">
    <path d="M6 19c0 1.1.9 2 2 2h8c1.1 0 2-.9 2-2V7H6v12zM19 4h-3.5l-1-1h-5l-1 1H5v2h14V4z" />
  </svg>
);

const AddIcon: React.FC = () => (
  <svg viewBox="0 0 24 24" fill="currentColor" width="24" height="24">
    <path d="M19 13h-6v6h-2v-6H5v-2h6V5h2v6h6v2z" />
  </svg>
);

const SunIcon: React.FC = () => (
    <svg viewBox="0 0 24 24" width="24" height="24" fill="currentColor">
        <path d="M12 7c-2.76 0-5 2.24-5 5s2.24 5 5 5 5-2.24 5-5-2.24-5-5-5zM12 9c1.65 0 3 1.35 3 3s-1.35 3-3 3-3-1.35-3-3 1.35-3 3-3zM12 2C6.48 2 2 6.48 2 12s4.48 10 10 10 10-4.48 10-10S17.52 2 12 2zM12 20c-4.41 0-8-3.59-8-8s3.59-8 8-8 8 3.59 8 8-3.59 8-8 8zM4.93 4.93l1.41 1.41C7.32 7.32 8 8.66 8 10H6c0-1.5.47-2.89 1.26-4.07zM19.07 19.07l-1.41-1.41c-1.02-1.02-1.61-2.36-1.61-3.66h2c0 1.5-.47 2.89-1.26 4.07zM16 14c0 1.34-.68 2.55-1.76 3.24l1.41 1.41C17.53 17.11 18 15.6 18 14h-2zM7.34 16.07C6.26 15.38 6.24 15.37 6 14H4c0 1.6.47 3.11 1.34 4.34l1.41-1.41c.01-.01.01-.01 0 0zM12 4V2c1.5 0 2.89.47 4.07 1.26L14.66 4.67C13.98 4.24 13.05 4 12 4zM12 20v2c-1.5 0-2.89-.47-4.07-1.26l1.41-1.41C10.02 19.76 10.95 20 12 20zM9.33 4.66L7.93 3.26C8.74 2.47 10.09 2 11.59 2H12V4c-1.05 0-1.98.24-2.67.66zM14.67 19.34l1.41 1.41c-.78.79-2.13 1.25-3.63 1.25V20c1.05 0 1.98-.24 2.67-.66z"></path>
    </svg>
);

const MoonIcon: React.FC = () => (
    <svg viewBox="0 0 24 24" width="24" height="24" fill="currentColor">
        <path d="M10 2c-1.82 0-3.53.5-5 1.35C7.99 5.08 10 8.3 10 12s-2.01 6.92-5 8.65C6.47 21.5 8.18 22 10 22c5.52 0 10-4.48 10-10S15.52 2 10 2z"></path>
    </svg>
);

const FlagIcon: React.FC = () => (
  <svg viewBox="0 0 24 24" fill="currentColor" width="24" height="24">
    <path d="M14.4 6L14 4H5v17h2v-7h5.6l.4 2h7V6h-5.6zm3.6 8h-5l-.4-2H7V6h5l.4 2h5v6z"/>
  </svg>
);


// --- Interfaces ---
interface RegisteredTotal {
  id: string;
  registeredAt: number;
  totalTime: number;
}

interface Timer {
  id: string;
  name: string;
  startTime: number | null;
  elapsedTime: number;
  isRunning: boolean;
  registeredTotals: RegisteredTotal[];
  color: string; // New property for timer color
}

// --- Constants ---
const LOCAL_STORAGE_KEY_TIMERS = 'multiStopwatchApp.timers.v5'; 
const LOCAL_STORAGE_KEY_ACTIVE_ID = 'multiStopwatchApp.activeTimerId.v3';
const LOCAL_STORAGE_KEY_THEME = 'multiStopwatchApp.theme.v2';

const TIMER_COLORS = [
  '#C0392B', // Pomegranate
  '#2980B9', // Belize Hole (Blue)
  '#27AE60', // Nephritis (Green)
  '#8E44AD', // Wisteria (Purple)
  '#D35400', // Pumpkin (Orange)
  '#16A085', // Green Sea (Teal)
  '#2C3E50', // Midnight Blue
  '#7F8C8D', // Asbestos (Gray)
];


// --- Utility Functions ---
const generateId = (): string => Math.random().toString(36).substring(2, 11);

const formatTime = (milliseconds: number): string => {
  const totalSeconds = Math.floor(milliseconds / 1000);
  const ms = String(milliseconds % 1000).padStart(3, '0').slice(0,2);
  const seconds = String(totalSeconds % 60).padStart(2, '0');
  const totalMinutes = Math.floor(totalSeconds / 60);
  const minutes = String(totalMinutes % 60).padStart(2, '0');
  const hours = String(Math.floor(totalMinutes / 60)).padStart(2, '0');
  return `${hours}:${minutes}:${seconds}.${ms}`;
};

// --- Main App Component ---
const App: React.FC = () => {
  const [timers, setTimers] = useState<Timer[]>(() => {
    const storedTimers = localStorage.getItem(LOCAL_STORAGE_KEY_TIMERS);
    if (storedTimers) {
      try {
        return JSON.parse(storedTimers).map((timer: any, index: number) => {
          // eslint-disable-next-line @typescript-eslint/no-unused-vars
          const { loggedTimes, ...restOfTimer } = timer; 
          return {
            ...restOfTimer,
            registeredTotals: Array.isArray(timer.registeredTotals) ? timer.registeredTotals : [],
            color: timer.color || TIMER_COLORS[index % TIMER_COLORS.length] // Add color if missing
          };
        });
      } catch (error) {
        console.error("Error al analizar temporizadores desde localStorage:", error);
        return [];
      }
    }
    return [];
  });
  const [activeTimerId, setActiveTimerId] = useState<string | null>(() => {
    return localStorage.getItem(LOCAL_STORAGE_KEY_ACTIVE_ID) || null;
  });
  const [newTimerName, setNewTimerName] = useState<string>('');
  const [currentTimeDisplay, setCurrentTimeDisplay] = useState<number>(0);
  const [theme, setTheme] = useState<'light' | 'dark'>(() => {
    const storedTheme = localStorage.getItem(LOCAL_STORAGE_KEY_THEME) as 'light' | 'dark';
    return storedTheme || 'light';
  });

  const requestRef = useRef<number>();
  const activeTimer = timers.find(t => t.id === activeTimerId);

  // Persist timers and activeTimerId to localStorage
  useEffect(() => {
    localStorage.setItem(LOCAL_STORAGE_KEY_TIMERS, JSON.stringify(timers));
    if (timers.length > 0 && !activeTimerId && timers[0]) {
      setActiveTimerId(timers[0].id);
    } else if (timers.length === 0 && activeTimerId) {
        setActiveTimerId(null); 
    }
  }, [timers, activeTimerId]);

  useEffect(() => {
    if (activeTimerId) {
      localStorage.setItem(LOCAL_STORAGE_KEY_ACTIVE_ID, activeTimerId);
    } else {
      localStorage.removeItem(LOCAL_STORAGE_KEY_ACTIVE_ID);
    }
  }, [activeTimerId]);

  // Theme management
  useEffect(() => {
    document.body.setAttribute('data-theme', theme);
    localStorage.setItem(LOCAL_STORAGE_KEY_THEME, theme);
  }, [theme]);

  const toggleTheme = () => {
    setTheme(prevTheme => (prevTheme === 'light' ? 'dark' : 'light'));
  };

  const animate = useCallback((_timestamp: DOMHighResTimeStamp) => {
    if (activeTimer && activeTimer.isRunning && activeTimer.startTime !== null) {
      setCurrentTimeDisplay(Date.now() - activeTimer.startTime + activeTimer.elapsedTime);
    }
    requestRef.current = requestAnimationFrame((newTimestamp) => animate(newTimestamp));
  }, [activeTimer]);

  useEffect(() => {
    if (activeTimer?.isRunning) {
      if(!requestRef.current) {
        requestRef.current = requestAnimationFrame((timestamp) => animate(timestamp));
      }
    } else if (requestRef.current) {
      cancelAnimationFrame(requestRef.current);
      requestRef.current = undefined;
    }

    if (activeTimer) {
      setCurrentTimeDisplay(activeTimer.isRunning && activeTimer.startTime !== null 
        ? (Date.now() - activeTimer.startTime + activeTimer.elapsedTime) 
        : activeTimer.elapsedTime);
    } else {
      setCurrentTimeDisplay(0);
    }
    
    return () => {
      if (requestRef.current) {
        cancelAnimationFrame(requestRef.current);
        requestRef.current = undefined;
      }
    };
  }, [activeTimer, animate]);


  const handleAddTimer = () => {
    if (newTimerName.trim() === '') return;
    const newId = generateId();
    const newTimer: Timer = {
      id: newId,
      name: newTimerName.trim(),
      startTime: null,
      elapsedTime: 0,
      isRunning: false,
      registeredTotals: [],
      color: TIMER_COLORS[timers.length % TIMER_COLORS.length], // Assign color
    };
    setTimers(prevTimers => [...prevTimers, newTimer]);
    setActiveTimerId(newId);
    setNewTimerName('');
  };

  const updateTimer = (id: string, updates: Partial<Timer>) => {
    setTimers(prevTimers =>
      prevTimers.map(timer => (timer.id === id ? { ...timer, ...updates } : timer))
    );
  };

  const handleStartPauseResume = () => {
    if (!activeTimer) return;

    if (activeTimer.isRunning) { // Pausar
      updateTimer(activeTimer.id, {
        isRunning: false,
        elapsedTime: (Date.now() - (activeTimer.startTime || Date.now())) + activeTimer.elapsedTime,
        startTime: null,
      });
    } else { // Iniciar o Reanudar
      updateTimer(activeTimer.id, {
        isRunning: true,
        startTime: Date.now(),
      });
    }
  };

  const handleReset = () => {
    if (!activeTimer) return;
    updateTimer(activeTimer.id, {
      isRunning: false,
      startTime: null,
      elapsedTime: 0,
    });
    setCurrentTimeDisplay(0);
  };

  const handleRegisterTotal = () => {
    if (!activeTimer) return;
    const currentTotal = activeTimer.isRunning && activeTimer.startTime !== null
      ? (Date.now() - activeTimer.startTime) + activeTimer.elapsedTime
      : activeTimer.elapsedTime;

    if (currentTotal > 0) { 
      const newRegisteredTotal: RegisteredTotal = {
        id: generateId(),
        registeredAt: Date.now(),
        totalTime: currentTotal,
      };
      updateTimer(activeTimer.id, {
        registeredTotals: [...activeTimer.registeredTotals, newRegisteredTotal].sort((a,b) => b.registeredAt - a.registeredAt),
      });
    }
  };


  const handleDeleteTimer = () => {
    if (!activeTimerId) return;
    setTimers(prevTimers => {
      const remainingTimers = prevTimers.filter(timer => timer.id !== activeTimerId);
      if (remainingTimers.length > 0 && remainingTimers[0]) {
        setActiveTimerId(remainingTimers[0].id);
      } else {
        setActiveTimerId(null);
      }
      return remainingTimers;
    });
  };
  
  const getControlButton = () => {
    if (!activeTimer) return null;
    if (activeTimer.isRunning) {
      return <button onClick={handleStartPauseResume} className="btn-icon btn-pause" aria-label="Pausar temporizador"><PauseIcon /></button>;
    }
    const label = activeTimer.elapsedTime > 0 ? "Reanudar temporizador" : "Iniciar temporizador";
    return <button onClick={handleStartPauseResume} className={`btn-icon ${activeTimer.elapsedTime > 0 ? 'btn-resume' : 'btn-start'}`} aria-label={label}><PlayIcon /></button>;
  };

  return (
    <>
      <div className="app-header">
        <h1>CRONO</h1>
        <button onClick={toggleTheme} className="theme-toggle-button" aria-label={`Cambiar a modo ${theme === 'light' ? 'oscuro' : 'claro'}`}>
          {theme === 'light' ? <MoonIcon /> : <SunIcon />}
        </button>
      </div>
      <div className="app-container" role="application" aria-label="Widget de Múltiples Temporizadores">
        <div className="sidebar" role="navigation" aria-label="Lista de temporizadores">
          <div className="timer-creation-area">
            <input
              type="text"
              value={newTimerName}
              onChange={(e) => setNewTimerName(e.target.value)}
              onKeyPress={(e) => e.key === 'Enter' && handleAddTimer()}
              placeholder="Nombre del nuevo temporizador"
              aria-label="Nombre del nuevo temporizador"
            />
            <button onClick={handleAddTimer} className="btn-icon btn-add-timer" aria-label="Añadir nuevo temporizador"><AddIcon /></button>
          </div>
          <h2>Temporizadores</h2>
          <div className="timer-list" role="radiogroup" aria-label="Seleccionar un temporizador">
            {timers.length === 0 && <p style={{fontSize: '0.9em', color: 'var(--text-secondary-color)'}}>Aún no hay temporizadores. ¡Añade uno!</p>}
            {timers.map(timer => (
              <button
                key={timer.id}
                onClick={() => setActiveTimerId(timer.id)}
                className={timer.id === activeTimerId ? 'active' : ''}
                style={{ backgroundColor: timer.color }}
                role="radio"
                aria-checked={timer.id === activeTimerId}
                aria-label={`Seleccionar temporizador ${timer.name}`}
              >
                {timer.name}
              </button>
            ))}
          </div>
        </div>

        <div className="main-content" role="main" aria-live="polite">
          {activeTimer ? (
            <div className="active-timer-area">
              <h2>{activeTimer.name}</h2>
              <div className="time-display" aria-label={`Tiempo actual: ${formatTime(currentTimeDisplay)}`}>
                {formatTime(currentTimeDisplay)}
              </div>
              <div className="controls">
                {getControlButton()}
                <button onClick={handleReset} className="btn-icon btn-reset" disabled={activeTimer.elapsedTime === 0 && !activeTimer.isRunning} aria-label="Reiniciar temporizador">
                  <ResetIcon />
                </button>
                <button onClick={handleRegisterTotal} className="btn-icon btn-register-total" disabled={activeTimer.elapsedTime === 0 && !activeTimer.isRunning} aria-label="Registrar tiempo total">
                  <FlagIcon />
                </button>
                <button onClick={handleDeleteTimer} className="btn-icon btn-delete" aria-label="Eliminar temporizador actual">
                  <DeleteIcon />
                </button>
              </div>
              {activeTimer.registeredTotals.length > 0 && (
                <div className="registered-totals-area">
                  <h3>Totales Registrados</h3>
                  <ul className="data-list" aria-label="Lista de tiempos totales registrados">
                    {activeTimer.registeredTotals.map((total, index) => (
                      <li key={total.id}>
                        <span className="log-index">#{activeTimer.registeredTotals.length - index}</span>
                        <span>{formatTime(total.totalTime)}</span>
                      </li>
                    ))}
                  </ul>
                </div>
              )}
            </div>
          ) : (
            <p>Ningún temporizador seleccionado. Añade un nuevo temporizador o selecciona uno existente.</p>
          )}
        </div>
      </div>
    </>
  );
};

const container = document.getElementById('root');
if (container) {
  const root = ReactDOM.createRoot(container);
  root.render(
    <React.StrictMode>
      <App />
    </React.StrictMode>
  );
}
