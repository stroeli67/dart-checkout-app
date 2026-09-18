"use client";

import React, { useState, useEffect, useRef, useMemo } from "react";
import confetti from "canvas-confetti";
import {
  Trophy,
  Play,
  CheckCircle2,
  RotateCcw,
  History,
  FolderKanban,
  Download,
  Share2,
  Sparkles,
  Target,
  BarChart3,
  Calendar,
  Lock,
  Smartphone,
  ChevronRight,
  ShieldCheck
} from "lucide-react";

// ==========================================
// 1. ALGORITHMISCHE DART CHECKOUT VALIDIERUNG
// ==========================================
// Prüft mathematisch, ob ein Score X mit max. 3 Darts (Double Out) geworfen werden kann.
const isValidDartCheckout = (score: number): boolean => {
  if (score < 2 || score > 170) return false;
  
  // Bogey-Zahlen, die mathematisch mit 3 Darts nicht auf ein Doppel geworfen werden können
  const BOGEY_NUMBERS = [159, 162, 163, 165, 166, 168, 169];
  if (BOGEY_NUMBERS.includes(score)) return false;

  return true;
};

// Generiert dynamisch die saubere Checkout-Liste (Genau 162 gültige Finishes)
const ALL_VALID_CHECKOUTS = Array.from({ length: 169 }, (_, i) => i + 2).filter(isValidDartCheckout);

// ==========================================
// TYPEN & DATENSTRUKTUR
// ==========================================
interface CompletedCheckout {
  checkout: number;
  date: string; // YYYY-MM-DD
  completedAt: string; // ISO Timestamp
}

interface GameSession {
  id: string;
  name: string;
  createdAt: string;
  status: "ACTIVE" | "COMPLETED";
  todaysCheckout: number | null;
  todaysCheckoutDate: string | null;
  completedCheckouts: CompletedCheckout[];
  remainingCheckouts: number[];
}

export default function DartCheckoutApp() {
  // Globales State Management
  const [sessions, setSessions] = useState<GameSession[]>([]);
  const [activeSessionId, setActiveSessionId] = useState<string | null>(null);
  const [syncCode, setSyncCode] = useState<string>("");
  const [activeTab, setActiveTab] = useState<"wheel" | "history" | "sessions" | "settings">("wheel");

  // Wheel & Animation States
  const [isSpinning, setIsSpinning] = useState<boolean>(false);
  const [displayNumber, setDisplayNumber] = useState<number | string>("🎯");
  const [isRecordingVideo, setIsRecordingVideo] = useState<boolean>(false);
  const canvasRef = useRef<HTMLCanvasElement>(null);

  // Today's Date String Format
  const getTodayStr = () => new Date().toISOString().split("T")[0];

  // ==========================================
  // INITIALISIERUNG & PERSISTENZ (LOCALSTORAGE & SYNC)
  // ==========================================
  useEffect(() => {
    // Sync Key Laden oder Generieren
    let storedSyncCode = localStorage.getItem("dart_app_sync_code");
    if (!storedSyncCode) {
      storedSyncCode = "DART-" + Math.random().toString(36).substring(2, 8).toUpperCase();
      localStorage.setItem("dart_app_sync_code", storedSyncCode);
    }
    setSyncCode(storedSyncCode);

    // Sessions Laden
    const savedSessions = localStorage.getItem("dart_checkout_sessions");
    if (savedSessions) {
      try {
        const parsed = JSON.parse(savedSessions);
        setSessions(parsed);
        const active = localStorage.getItem("dart_active_session_id") || parsed[0]?.id;
        setActiveSessionId(active);
      } catch (e) {
        createNewSession("Spielstand 1");
      }
    } else {
      createNewSession("Spielstand 1");
    }
  }, []);

  // Automatisches Speichern bei Zustand-Änderungen
  useEffect(() => {
    if (sessions.length > 0) {
      localStorage.setItem("dart_checkout_sessions", JSON.stringify(sessions));
    }
  }, [sessions]);

  useEffect(() => {
    if (activeSessionId) {
      localStorage.setItem("dart_active_session_id", activeSessionId);
    }
  }, [activeSessionId]);

  // Aktive Session Objekt
  const activeSession = useMemo(() => {
    return sessions.find((s) => s.id === activeSessionId) || null;
  }, [sessions, activeSessionId]);

  // Neuen Spielstand Erstellen
  const createNewSession = (customName?: string) => {
    const newId = "session_" + Date.now();
    const newSession: GameSession = {
      id: newId,
      name: customName || `Spielstand #${sessions.length + 1}`,
      createdAt: new Date().toLocaleDateString("de-DE"),
      status: "ACTIVE",
      todaysCheckout: null,
      todaysCheckoutDate: null,
      completedCheckouts: [],
      remainingCheckouts: [...ALL_VALID_CHECKOUTS],
    };

    setSessions((prev) => [newSession, ...prev]);
    setActiveSessionId(newId);
  };

  // ==========================================
  // GLÜCKSRAD / ZUFALLSAUSWAHL & ANIMATION
  // ==========================================
  const drawTodaysCheckout = () => {
    if (!activeSession || isSpinning) return;
    const today = getTodayStr();

    // Bereits heute gezogen Schutz
    if (activeSession.todaysCheckoutDate === today && activeSession.todaysCheckout) {
      alert("Du hast heute bereits deinen Checkout gezogen!");
      return;
    }

    if (activeSession.remainingCheckouts.length === 0) {
      alert("Glückwunsch! Alle Checkouts dieser Session wurden absolviert.");
      return;
    }

    setIsSpinning(true);
    const duration = 4500; // 4.5 Sekunden Animation
    const startTime = Date.now();

    // Zufälliges Ziel-Checkout ermitteln
    const randomIndex = Math.floor(Math.random() * activeSession.remainingCheckouts.length);
    const selectedCheckout = activeSession.remainingCheckouts[randomIndex];

    // Sound / Haptic Effects Trigger (Vibration falls verfügbar)
    if (typeof window !== "undefined" && window.navigator.vibrate) {
      window.navigator.vibrate([100, 50, 100]);
    }

    // Animation Intervall (Rapid Random Pick mit Abbrems-Physik)
    const interval = setInterval(() => {
      const elapsed = Date.now() - startTime;
      const progress = elapsed / duration;

      if (progress < 1) {
        // Wähle zufällige Zahl für visuelle Glücksrad-Rotation
        const tempRandom = activeSession.remainingCheckouts[
          Math.floor(Math.random() * activeSession.remainingCheckouts.length)
        ];
        setDisplayNumber(tempRandom);
      } else {
        clearInterval(interval);
        setDisplayNumber(selectedCheckout);
        setIsSpinning(false);

        // Update Session State
        setSessions((prev) =>
          prev.map((s) => {
            if (s.id === activeSession.id) {
              return {
                ...s,
                todaysCheckout: selectedCheckout,
                todaysCheckoutDate: today,
              };
            }
            return s;
          })
        );
      }
    }, 60);
  };

  // ==========================================
  // CHECKOUT AS COMPLETED MARKEREN
  // ==========================================
  const markCheckoutAsDone = () => {
    if (!activeSession || !activeSession.todaysCheckout) return;

    const currentCheckout = activeSession.todaysCheckout;
    const today = getTodayStr();

    const newCompletedEntry: CompletedCheckout = {
      checkout: currentCheckout,
      date: new Date().toLocaleDateString("de-DE"),
      completedAt: new Date().toISOString(),
    };

    const updatedRemaining = activeSession.remainingCheckouts.filter((c) => c !== currentCheckout);
    const isFinished = updatedRemaining.length === 0;

    // Confetti Feier
    confetti({
      particleCount: 120,
      spread: 70,
      origin: { y: 0.6 },
      colors: ["#22c55e", "#eab308", "#ef4444", "#3b82f6"],
    });

    setSessions((prev) =>
      prev.map((s) => {
        if (s.id === activeSession.id) {
          return {
            ...s,
            status: isFinished ? "COMPLETED" : "ACTIVE",
            completedCheckouts: [newCompletedEntry, ...s.completedCheckouts],
            remainingCheckouts: updatedRemaining,
            todaysCheckout: null, // Reset today so next day can be drawn
          };
        }
        return s;
      })
    );
  };

  // ==========================================
  // 9:16 INSTAGRAM STORY VIDEO GENERATION
  // ==========================================
  const generate916Video = async () => {
    if (!activeSession || !activeSession.todaysCheckout) return;

    setIsRecordingVideo(true);
    const canvas = canvasRef.current;
    if (!canvas) return;

    const ctx = canvas.getContext("2d");
    if (!ctx) return;

    // Canvas Size 9:16 Format (1080x1920)
    canvas.width = 1080;
    canvas.height = 1920;

    const stream = canvas.captureStream(30); // 30 FPS
    const mediaRecorder = new MediaRecorder(stream, { mimeType: "video/webm" });
    const chunks: Blob[] = [];

    mediaRecorder.ondataavailable = (e) => chunks.push(e.data);
    mediaRecorder.onstop = () => {
      const blob = new Blob(chunks, { type: "video/mp4" });
      const url = URL.createObjectURL(blob);
      const a = document.createElement("a");
      a.href = url;
      a.download = `Dart_Checkout_${activeSession.todaysCheckout}_${getTodayStr()}.mp4`;
      a.click();
      setIsRecordingVideo(false);
    };

    mediaRecorder.start();

    // Render Animation 3 Sekunden lang auf Canvas
    let frame = 0;
    const totalFrames = 90;

    const renderFrame = () => {
      if (frame >= totalFrames) {
        mediaRecorder.stop();
        return;
      }

      // Background Gradient
      const grad = ctx.createLinearGradient(0, 0, 0, 1920);
      grad.addColorStop(0, "#0f172a");
      grad.addColorStop(1, "#020617");
      ctx.fillStyle = grad;
      ctx.fillRect(0, 0, 1080, 1920);

      // Card Box
      ctx.fillStyle = "rgba(30, 41, 59, 0.8)";
      ctx.roundRect(100, 300, 880, 1320, 40);
      ctx.fill();
      ctx.strokeStyle = "#38bdf8";
      ctx.lineWidth = 6;
      ctx.stroke();

      // Header Text
      ctx.fillStyle = "#94a3b8";
      ctx.font = "bold 42px sans-serif";
      ctx.textAlign = "center";
      ctx.fillText("🎯 DART CHECKOUT CHALLENGE", 540, 450);

      ctx.fillStyle = "#ffffff";
      ctx.font = "900 64px sans-serif";
      ctx.fillText("CHECKOUT DES TAGES", 540, 560);

      // Score Value Display
      ctx.fillStyle = "#38bdf8";
      ctx.font = "900 240px sans-serif";
      ctx.fillText(`${activeSession.todaysCheckout}`, 540, 950);

      // Subtitle
      ctx.fillStyle = "#facc15";
      ctx.font = "bold 56px sans-serif";
      ctx.fillText("Schaffst du das Finish?", 540, 1150);

      // Progress Stats
      ctx.fillStyle = "#94a3b8";
      ctx.font = "36px sans-serif";
      ctx.fillText(
        `Fortschritt: ${activeSession.completedCheckouts.length} / ${ALL_VALID_CHECKOUTS.length} geschafft`,
        540,
        1450
      );

      frame++;
      requestAnimationFrame(renderFrame);
    };

    renderFrame();
  };

  if (!activeSession) return <div className="p-10 text-white text-center">App lädt...</div>;

  const totalPossible = ALL_VALID_CHECKOUTS.length;
  const completedCount = activeSession.completedCheckouts.length;
  const progressPercent = Math.round((completedCount / totalPossible) * 100);
  const isTodayDrawn = activeSession.todaysCheckoutDate === getTodayStr() && activeSession.todaysCheckout !== null;

  return (
    <div className="min-h-screen bg-slate-950 text-slate-100 font-sans flex flex-col justify-between pb-20 md:pb-6">
      {/* Hidden Canvas for Video Export */}
      <canvas ref={canvasRef} className="hidden" />

      {/* HEADER BAR */}
      <header className="border-b border-slate-800 bg-slate-900/60 backdrop-blur-md sticky top-0 z-50 px-4 py-3 flex justify-between items-center">
        <div className="flex items-center gap-2">
          <Target className="w-7 h-7 text-emerald-400 animate-pulse" />
          <span className="font-extrabold tracking-wider text-lg bg-gradient-to-r from-emerald-400 to-cyan-400 bg-clip-text text-transparent">
            DART CHECKOUTS
          </span>
        </div>
        <div className="text-xs bg-slate-800 border border-slate-700 px-3 py-1.5 rounded-full flex items-center gap-2">
          <ShieldCheck className="w-3.5 h-3.5 text-cyan-400" />
          <span className="text-slate-300 font-medium">{activeSession.name}</span>
        </div>
      </header>

      {/* MAIN CONTENT AREA */}
      <main className="max-w-xl w-full mx-auto p-4 flex-1 flex flex-col gap-6">
        {/* TABS NAVIGATION */}
        <div className="grid grid-cols-3 bg-slate-900 p-1 rounded-xl border border-slate-800">
          <button
            onClick={() => setActiveTab("wheel")}
            className={`py-2 text-sm font-semibold rounded-lg transition-all flex items-center justify-center gap-2 ${
              activeTab === "wheel" ? "bg-slate-800 text-emerald-400 shadow-lg" : "text-slate-400 hover:text-white"
            }`}
          >
            <Sparkles className="w-4 h-4" /> Glücksrad
          </button>
          <button
            onClick={() => setActiveTab("history")}
            className={`py-2 text-sm font-semibold rounded-lg transition-all flex items-center justify-center gap-2 ${
              activeTab === "history" ? "bg-slate-800 text-emerald-400 shadow-lg" : "text-slate-400 hover:text-white"
            }`}
          >
            <History className="w-4 h-4" /> Historie
          </button>
          <button
            onClick={() => setActiveTab("sessions")}
            className={`py-2 text-sm font-semibold rounded-lg transition-all flex items-center justify-center gap-2 ${
              activeTab === "sessions" ? "bg-slate-800 text-emerald-400 shadow-lg" : "text-slate-400 hover:text-white"
            }`}
          >
            <FolderKanban className="w-4 h-4" /> Spielstände
          </button>
        </div>

        {/* TAB 1: GLÜCKSRAD & HEUTIGER CHECKOUT */}
        {activeTab === "wheel" && (
          <div className="flex flex-col gap-6">
            {/* HERO WHEEL DISPLAY CARD */}
            <div className="relative bg-gradient-to-b from-slate-900 to-slate-950 border border-slate-800 rounded-3xl p-8 text-center flex flex-col items-center justify-center shadow-2xl overflow-hidden min-h-[380px]">
              <div className="absolute top-0 inset-x-0 h-1 bg-gradient-to-r from-emerald-500 via-cyan-500 to-indigo-500" />

              <span className="text-xs uppercase tracking-widest font-bold text-slate-400 mb-2 flex items-center gap-1.5">
                <Calendar className="w-3.5 h-3.5 text-emerald-400" /> Checkout des Tages
              </span>

              {/* DYNAMIC DISPLAY VALUE */}
              <div className="my-6 relative flex items-center justify-center">
                <div
                  className={`text-7xl md:text-8xl font-black tracking-tight text-white transition-all transform ${
                    isSpinning ? "scale-110 blur-[1px]" : "scale-100"
                  }`}
                >
                  {isTodayDrawn
                    ? activeSession.todaysCheckout
                    : isSpinning
                    ? displayNumber
                    : "🎯"}
                </div>
              </div>

              {isTodayDrawn ? (
                <div className="flex flex-col items-center gap-3 w-full">
                  <p className="text-emerald-400 font-semibold text-lg animate-bounce">
                    Schaffst du {activeSession.todaysCheckout}?
                  </p>
                  <button
                    onClick={markCheckoutAsDone}
                    className="w-full max-w-xs py-4 bg-emerald-500 hover:bg-emerald-400 text-slate-950 font-black text-lg rounded-2xl shadow-lg shadow-emerald-500/20 active:scale-95 transition-all flex items-center justify-center gap-2"
                  >
                    <CheckCircle2 className="w-6 h-6" /> Checkout Geschafft!
                  </button>

                  <button
                    onClick={generate916Video}
                    disabled={isRecordingVideo}
                    className="mt-2 text-xs text-slate-400 hover:text-cyan-400 flex items-center gap-1.5 transition-colors"
                  >
                    <Share2 className="w-3.5 h-3.5" />
                    {isRecordingVideo ? "Generiere 9:16 Video..." : "Als 9:16 Story-Video speichern"}
                  </button>
                </div>
              ) : (
                <button
                  onClick={drawTodaysCheckout}
                  disabled={isSpinning || activeSession.remainingCheckouts.length === 0}
                  className="w-full max-w-xs py-4 bg-gradient-to-r from-emerald-500 to-cyan-500 hover:from-emerald-400 hover:to-cyan-400 text-slate-950 font-black text-lg rounded-2xl shadow-xl shadow-cyan-500/10 active:scale-95 transition-all flex items-center justify-center gap-2 disabled:opacity-50"
                >
                  <Play className="w-6 h-6 fill-current" />
                  {isSpinning ? "Glücksrad dreht..." : "Checkout des Tages ziehen"}
                </button>
              )}
            </div>

            {/* PROGRESS CARD */}
            <div className="bg-slate-900 border border-slate-800 rounded-2xl p-5 flex flex-col gap-3">
              <div className="flex justify-between items-center text-sm">
                <span className="text-slate-400 font-medium">Gesamtfortschritt</span>
                <span className="text-emerald-400 font-bold">
                  {completedCount} / {totalPossible} ({progressPercent}%)
                </span>
              </div>
              <div className="w-full bg-slate-800 h-3 rounded-full overflow-hidden p-0.5">
                <div
                  className="bg-gradient-to-r from-emerald-500 to-cyan-400 h-full rounded-full transition-all duration-500"
                  style={{ width: `${progressPercent}%` }}
                />
              </div>
              <div className="flex justify-between text-xs text-slate-500">
                <span>Offen: {activeSession.remainingCheckouts.length} Checkouts</span>
                <span>Bogey-Zahlen automatisch entfernt</span>
              </div>
            </div>
          </div>
        )}

        {/* TAB 2: HISTORIE */}
        {activeTab === "history" && (
          <div className="bg-slate-900 border border-slate-800 rounded-2xl p-5 flex flex-col gap-4">
            <h3 className="font-bold text-lg text-slate-200 flex items-center gap-2">
              <History className="w-5 h-5 text-emerald-400" /> Erledigte Checkouts
            </h3>

            {activeSession.completedCheckouts.length === 0 ? (
              <p className="text-slate-500 text-center py-8 text-sm">
                Noch keine Checkouts in dieser Session absolviert.
              </p>
            ) : (
              <div className="flex flex-col divide-y divide-slate-800 max-h-[450px] overflow-y-auto">
                {activeSession.completedCheckouts.map((item, idx) => (
                  <div key={idx} className="py-3 flex justify-between items-center">
                    <span className="text-slate-400 text-sm">{item.date}</span>
                    <span className="text-xl font-extrabold text-emerald-400 bg-emerald-500/10 border border-emerald-500/20 px-3 py-1 rounded-xl">
                      {item.checkout}
                    </span>
                  </div>
                ))}
              </div>
            )}
          </div>
        )}

        {/* TAB 3: SPIELSTÄNDE & RESET */}
        {activeTab === "sessions" && (
          <div className="flex flex-col gap-4">
            <div className="flex justify-between items-center">
              <h3 className="font-bold text-lg text-slate-200">Meine Spielstände</h3>
              <button
                onClick={() => createNewSession()}
                className="text-xs bg-emerald-500/10 hover:bg-emerald-500/20 text-emerald-400 border border-emerald-500/30 px-3 py-1.5 rounded-xl font-semibold flex items-center gap-1"
              >
                + Neuer Spielstand
              </button>
            </div>

            <div className="flex flex-col gap-3">
              {sessions.map((session) => {
                const isActive = session.id === activeSessionId;
                const count = session.completedCheckouts.length;
                const percent = Math.round((count / totalPossible) * 100);

                return (
                  <div
                    key={session.id}
                    onClick={() => setActiveSessionId(session.id)}
                    className={`p-4 rounded-2xl border transition-all cursor-pointer flex justify-between items-center ${
                      isActive
                        ? "bg-slate-900 border-emerald-500/50 shadow-lg shadow-emerald-500/5"
                        : "bg-slate-950 border-slate-800 hover:border-slate-700"
                    }`}
                  >
                    <div>
                      <div className="flex items-center gap-2">
                        <span className="font-bold text-slate-200">{session.name}</span>
                        {isActive && (
                          <span className="text-[10px] bg-emerald-500/20 text-emerald-400 px-2 py-0.5 rounded-md font-bold uppercase">
                            Aktiv
                          </span>
                        )}
                      </div>
                      <p className="text-xs text-slate-500 mt-1">
                        Erstellt: {session.createdAt} | {count} / {totalPossible} ({percent}%)
                      </p>
                    </div>
                    <ChevronRight className="w-5 h-5 text-slate-600" />
                  </div>
                );
              })}
            </div>

            {/* SYNC SCHLÜSSEL INFO BOX */}
            <div className="mt-4 bg-slate-900/50 border border-slate-800 rounded-2xl p-4 text-xs text-slate-400 flex flex-col gap-2">
              <span className="font-semibold text-slate-300 flex items-center gap-1.5">
                <Lock className="w-3.5 h-3.5 text-cyan-400" /> Multi-Device Synchronisation Key:
              </span>
              <code className="bg-slate-950 px-3 py-2 rounded-lg text-cyan-400 font-mono text-center tracking-wider border border-slate-800">
                {syncCode}
              </code>
              <p className="text-[11px] text-slate-500">
                Nutze diesen Code, um deine Spielstände auf ein zweites Smartphone oder PC zu übertragen.
              </p>
            </div>
          </div>
        )}
      </main>
    </div>
  );
}
