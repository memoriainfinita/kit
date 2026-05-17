# KIT — Keyboard Input Tester HTML Port Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Crear `index.html` vanilla autocontenido que replique exactamente el Input Tester de React/Next.js, sin build ni dependencias locales.

**Architecture:** Un único archivo HTML con CDN links para Tailwind Play y Google Fonts, HTML estático para el layout, y un bloque `<script>` con toda la lógica. El DOM se actualiza con funciones targeted (no full re-render) para mantener performance durante input continuo.

**Tech Stack:** HTML5 + JS vanilla + Tailwind Play CDN + Google Fonts CDN

---

### Task 1: Crear index.html completo

**Files:**
- Create: `index.html`

- [ ] **Step 1: Escribir index.html**

```html
<!DOCTYPE html>
<html lang="es" class="dark">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>KIT — Keyboard Input Tester</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;700&family=JetBrains+Mono:wght@400;700&display=swap" rel="stylesheet">
  <script src="https://cdn.tailwindcss.com"></script>
  <style>
    body { font-family: 'Inter', sans-serif; }
    .font-mono { font-family: 'JetBrains Mono', monospace !important; }
    #typed-text-container { scrollbar-width: thin; scrollbar-color: #404040 transparent; }
    #history-list { scrollbar-width: thin; scrollbar-color: #404040 transparent; }
    .animate-pulse { animation: pulse 2s cubic-bezier(0.4,0,0.6,1) infinite; }
    @keyframes pulse { 0%,100%{opacity:1} 50%{opacity:.5} }
  </style>
</head>
<body class="bg-neutral-950 text-neutral-50 antialiased select-none overflow-hidden">

  <div id="focus-warning" style="display:none" class="fixed top-0 left-0 w-full bg-yellow-500/10 border-b border-yellow-500/20 text-yellow-500/80 text-xs text-center py-1.5 z-50 pointer-events-none font-mono">
    ⚠️ Haz clic en cualquier parte para que el navegador detecte el teclado
  </div>

  <main id="main" class="min-h-screen flex flex-col md:flex-row">

    <!-- Left Panel -->
    <div class="flex-1 flex flex-col items-center p-8 border-b md:border-b-0 md:border-r border-neutral-800/50 relative">

      <div class="absolute top-6 left-6 flex items-center gap-3">
        <div id="rec-dot" class="w-2.5 h-2.5 rounded-full bg-green-500 animate-pulse"></div>
        <span id="rec-label" class="text-xs font-mono text-neutral-400 uppercase tracking-widest">Escuchando</span>
      </div>

      <div class="absolute top-6 right-6 flex flex-col items-end">
        <div id="kpm-value" class="text-3xl font-bold font-mono text-neutral-200">0</div>
        <div class="text-xs font-mono text-neutral-500 uppercase tracking-widest">KPM (Pulsaciones/Min)</div>
      </div>

      <!-- Text View -->
      <div id="text-view" class="flex-1 flex flex-col items-center justify-center w-full mt-16 mb-8">
        <div id="waiting-display" class="text-center text-neutral-500">
          <div class="text-2xl mb-2">Esperando entrada...</div>
          <div class="text-sm font-mono">Presiona cualquier tecla o botón del ratón</div>
        </div>
        <div id="last-event-display" style="display:none" class="text-center">
          <div id="last-device-action" class="text-sm font-mono text-neutral-500 mb-4 uppercase tracking-widest"></div>
          <div id="last-value" class="text-6xl md:text-8xl font-bold tracking-tight mb-6 text-neutral-100"></div>
          <div id="last-details" class="text-lg font-mono text-neutral-400 bg-neutral-900/50 px-6 py-3 rounded-lg border border-neutral-800/50 inline-block"></div>
        </div>
        <div id="typed-text-container" class="w-full max-w-2xl h-32 bg-[#0a0a0a] border border-neutral-800/80 rounded-lg p-4 overflow-y-auto font-mono text-sm text-neutral-300 whitespace-pre-wrap break-words shadow-inner mt-8">
          <span id="typed-text-placeholder" class="text-neutral-600 italic">El texto escrito aparecerá aquí...</span>
          <span id="typed-text-content" style="display:none"></span>
        </div>
      </div>

      <!-- Keymap View -->
      <div id="keymap-view" style="display:none" class="flex-1 flex flex-col items-center justify-center w-full mt-16 mb-8 overflow-x-auto">
        <div class="mb-8 flex flex-col items-center gap-2">
          <div id="keys-count" class="text-3xl font-mono font-bold text-neutral-200">0 / 0</div>
          <div id="keys-label" class="text-xs font-mono text-neutral-500 uppercase tracking-widest">Teclas Probadas (0%)</div>
          <div class="w-64 h-2 mt-2 bg-neutral-900 rounded-full overflow-hidden border border-neutral-800">
            <div id="keys-progress" class="h-full bg-green-500 transition-all duration-300" style="width:0%"></div>
          </div>
        </div>
        <div id="keyboard-layout" class="flex flex-col xl:flex-row gap-8 bg-neutral-900/30 p-8 rounded-2xl border border-neutral-800/50 min-w-max"></div>
      </div>

      <!-- Controls -->
      <div class="absolute bottom-6 flex gap-4">
        <button id="btn-toggle-view" class="text-xs font-mono uppercase tracking-widest px-4 py-2 rounded border border-neutral-800 hover:bg-neutral-900 text-neutral-400 hover:text-neutral-200 transition-colors cursor-pointer">Modo Teclado</button>
        <button id="btn-recording" class="text-xs font-mono uppercase tracking-widest px-4 py-2 rounded border border-neutral-800 hover:bg-neutral-900 text-neutral-400 hover:text-neutral-200 transition-colors cursor-pointer">Pausar</button>
        <button id="btn-clear" class="text-xs font-mono uppercase tracking-widest px-4 py-2 rounded border border-neutral-800 hover:bg-neutral-900 text-neutral-400 hover:text-neutral-200 transition-colors cursor-pointer">Limpiar</button>
      </div>
    </div>

    <!-- Right Panel: History -->
    <div class="w-full md:w-96 bg-[#050505] flex flex-col h-[50vh] md:h-screen">
      <div class="p-4 border-b border-neutral-800/50 bg-[#0a0a0a] flex flex-col gap-3">
        <div class="flex justify-between items-center">
          <h2 class="text-xs font-mono uppercase tracking-widest text-neutral-500">Historial</h2>
          <span id="history-count" class="text-xs font-mono text-neutral-600">0 eventos</span>
        </div>
        <div class="flex gap-4 text-xs font-mono text-neutral-400">
          <label class="flex items-center gap-1.5 cursor-pointer hover:text-neutral-200 transition-colors">
            <input type="checkbox" id="check-releases" checked class="accent-neutral-500"> Releases
          </label>
          <label class="flex items-center gap-1.5 cursor-pointer hover:text-neutral-200 transition-colors">
            <input type="checkbox" id="check-holds" class="accent-neutral-500"> Holds
          </label>
        </div>
      </div>
      <div id="history-list" class="flex-1 overflow-y-auto p-4 font-mono text-sm">
        <div class="text-neutral-600 text-center mt-10">Sin eventos</div>
      </div>
    </div>

  </main>

<script>
// --- KEY MAP DATA ---
const MAIN_KEYS = [
  [{c:'Escape',l:'Esc'},{c:'F1',l:'F1'},{c:'F2',l:'F2'},{c:'F3',l:'F3'},{c:'F4',l:'F4'},{c:'F5',l:'F5'},{c:'F6',l:'F6'},{c:'F7',l:'F7'},{c:'F8',l:'F8'},{c:'F9',l:'F9'},{c:'F10',l:'F10'},{c:'F11',l:'F11'},{c:'F12',l:'F12'}],
  [{c:'Backquote',l:'`'},{c:'Digit1',l:'1'},{c:'Digit2',l:'2'},{c:'Digit3',l:'3'},{c:'Digit4',l:'4'},{c:'Digit5',l:'5'},{c:'Digit6',l:'6'},{c:'Digit7',l:'7'},{c:'Digit8',l:'8'},{c:'Digit9',l:'9'},{c:'Digit0',l:'0'},{c:'Minus',l:'-'},{c:'Equal',l:'='},{c:'Backspace',l:'Back',w:'80px'}],
  [{c:'Tab',l:'Tab',w:'64px'},{c:'KeyQ',l:'Q'},{c:'KeyW',l:'W'},{c:'KeyE',l:'E'},{c:'KeyR',l:'R'},{c:'KeyT',l:'T'},{c:'KeyY',l:'Y'},{c:'KeyU',l:'U'},{c:'KeyI',l:'I'},{c:'KeyO',l:'O'},{c:'KeyP',l:'P'},{c:'BracketLeft',l:'['},{c:'BracketRight',l:']'},{c:'Backslash',l:'\\',w:'64px'}],
  [{c:'CapsLock',l:'Caps',w:'80px'},{c:'KeyA',l:'A'},{c:'KeyS',l:'S'},{c:'KeyD',l:'D'},{c:'KeyF',l:'F'},{c:'KeyG',l:'G'},{c:'KeyH',l:'H'},{c:'KeyJ',l:'J'},{c:'KeyK',l:'K'},{c:'KeyL',l:'L'},{c:'Semicolon',l:';'},{c:'Quote',l:"'"},{c:'Enter',l:'Enter',w:'80px'}],
  [{c:'ShiftLeft',l:'Shift',w:'112px'},{c:'KeyZ',l:'Z'},{c:'KeyX',l:'X'},{c:'KeyC',l:'C'},{c:'KeyV',l:'V'},{c:'KeyB',l:'B'},{c:'KeyN',l:'N'},{c:'KeyM',l:'M'},{c:'Comma',l:','},{c:'Period',l:'.'},{c:'Slash',l:'/'},{c:'ShiftRight',l:'Shift',w:'112px'}],
  [{c:'ControlLeft',l:'Ctrl',w:'64px'},{c:'MetaLeft',l:'Win',w:'64px'},{c:'AltLeft',l:'Alt',w:'64px'},{c:'Space',l:'Space',w:'256px'},{c:'AltRight',l:'Alt',w:'64px'},{c:'MetaRight',l:'Win',w:'64px'},{c:'ContextMenu',l:'Menu',w:'64px'},{c:'ControlRight',l:'Ctrl',w:'64px'}]
];
const NAV_KEYS = [
  [{c:'Insert',l:'Ins'},{c:'Home',l:'Home'},{c:'PageUp',l:'PgUp'}],
  [{c:'Delete',l:'Del'},{c:'End',l:'End'},{c:'PageDown',l:'PgDn'}],
  [{c:'',l:''},{c:'',l:''},{c:'',l:''}],
  [{c:'',l:''},{c:'ArrowUp',l:'↑'},{c:'',l:''}],
  [{c:'ArrowLeft',l:'←'},{c:'ArrowDown',l:'↓'},{c:'ArrowRight',l:'→'}]
];
const NUM_KEYS = [
  [{c:'NumLock',l:'Num'},{c:'NumpadDivide',l:'/'},{c:'NumpadMultiply',l:'*'},{c:'NumpadSubtract',l:'-'}],
  [{c:'Numpad7',l:'7'},{c:'Numpad8',l:'8'},{c:'Numpad9',l:'9'},{c:'NumpadAdd',l:'+'}],
  [{c:'Numpad4',l:'4'},{c:'Numpad5',l:'5'},{c:'Numpad6',l:'6'},{c:'NumpadEnter',l:'Ent'}],
  [{c:'Numpad1',l:'1'},{c:'Numpad2',l:'2'},{c:'Numpad3',l:'3'},{c:'NumpadDecimal',l:'.'}],
  [{c:'Numpad0',l:'0',w:'80px'},{c:'',l:''},{c:'',l:''},{c:'',l:''}]
];
const MOUSE_KEYS = [
  [{c:'Mouse0',l:'Left'},{c:'Mouse1',l:'Mid'},{c:'Mouse2',l:'Right'}],
  [{c:'Mouse3',l:'Back'},{c:'Mouse4',l:'Fwd'}]
];
const ALL_TESTABLE_KEYS = new Set(
  [...MAIN_KEYS,...NAV_KEYS,...NUM_KEYS,...MOUSE_KEYS].flat().map(k=>k.c).filter(Boolean)
);

// --- STATE ---
let lastEvent = null;
let history = [];
let isRecording = true;
let recordReleases = true;
let recordHolds = false;
let typedText = '';
let testedKeys = new Set();
let showKeyMap = false;
let pressTimestamps = [];

// --- DOM REFS ---
const $ = id => document.getElementById(id);

// --- BUILD KEYBOARD ---
function buildKeyboard() {
  const layout = $('keyboard-layout');

  function makeRow(row) {
    const div = document.createElement('div');
    div.style.cssText = 'display:flex;gap:6px;justify-content:center;margin-bottom:6px;';
    row.forEach(key => {
      if (!key.c) {
        const spacer = document.createElement('div');
        spacer.style.cssText = 'width:40px;height:40px;';
        div.appendChild(spacer);
        return;
      }
      const k = document.createElement('div');
      k.id = 'key-' + key.c;
      k.textContent = key.l;
      k.style.cssText = `display:flex;align-items:center;justify-content:center;height:40px;width:${key.w||'40px'};border-radius:4px;border:1px solid;font-size:10px;font-family:'JetBrains Mono',monospace;transition:all 0.15s;background:#030712;border-color:#262626;color:#737373;`;
      div.appendChild(k);
    });
    return div;
  }

  // Main keyboard
  const mainDiv = document.createElement('div');
  MAIN_KEYS.forEach(row => mainDiv.appendChild(makeRow(row)));
  layout.appendChild(mainDiv);

  // Nav + Numpad wrapper
  const rightDiv = document.createElement('div');
  rightDiv.style.cssText = 'display:flex;flex-direction:column;gap:24px;';

  const navNumDiv = document.createElement('div');
  navNumDiv.style.cssText = 'display:flex;gap:32px;';

  const navDiv = document.createElement('div');
  NAV_KEYS.forEach(row => navDiv.appendChild(makeRow(row)));
  navNumDiv.appendChild(navDiv);

  const numDiv = document.createElement('div');
  NUM_KEYS.forEach(row => numDiv.appendChild(makeRow(row)));
  navNumDiv.appendChild(numDiv);

  rightDiv.appendChild(navNumDiv);

  // Mouse
  const mouseDiv = document.createElement('div');
  mouseDiv.style.cssText = 'padding-top:16px;border-top:1px solid #262626;';
  const mouseLabel = document.createElement('span');
  mouseLabel.textContent = 'Ratón';
  mouseLabel.style.cssText = 'display:block;font-size:10px;font-family:"JetBrains Mono",monospace;color:#525252;text-transform:uppercase;letter-spacing:0.1em;margin-bottom:4px;';
  mouseDiv.appendChild(mouseLabel);
  MOUSE_KEYS.forEach(row => {
    const rowDiv = document.createElement('div');
    rowDiv.style.cssText = 'display:flex;gap:6px;margin-bottom:6px;';
    row.forEach(key => {
      const k = document.createElement('div');
      k.id = 'key-' + key.c;
      k.textContent = key.l;
      k.style.cssText = 'display:flex;align-items:center;justify-content:center;height:40px;width:48px;border-radius:4px;border:1px solid #262626;font-size:10px;font-family:"JetBrains Mono",monospace;color:#737373;background:#030712;transition:all 0.15s;';
      rowDiv.appendChild(k);
    });
    mouseDiv.appendChild(rowDiv);
  });
  rightDiv.appendChild(mouseDiv);
  layout.appendChild(rightDiv);
}

// --- UPDATE KEY VISUAL ---
function setKeyTested(code, tested) {
  const el = document.getElementById('key-' + code);
  if (!el) return;
  if (tested) {
    el.style.background = 'rgba(34,197,94,0.2)';
    el.style.borderColor = 'rgba(34,197,94,0.5)';
    el.style.color = 'rgb(74,222,128)';
    el.style.boxShadow = '0 0 10px rgba(34,197,94,0.1)';
  } else {
    el.style.background = '#030712';
    el.style.borderColor = '#262626';
    el.style.color = '#737373';
    el.style.boxShadow = '';
  }
}

// --- UPDATE KEYMAP STATS ---
function updateKeymapStats() {
  const tested = Array.from(testedKeys).filter(k => ALL_TESTABLE_KEYS.has(k)).length;
  const total = ALL_TESTABLE_KEYS.size;
  const pct = total > 0 ? Math.round((tested / total) * 100) : 0;
  $('keys-count').textContent = tested + ' / ' + total;
  $('keys-label').textContent = 'Teclas Probadas (' + pct + '%)';
  $('keys-progress').style.width = pct + '%';
}

// --- ADD EVENT ---
function addEvent(ev) {
  if (!isRecording) return;
  const now = Date.now();
  if (ev.action === 'Press' && ev.device === 'Keyboard') {
    pressTimestamps.push(now);
  }
  const entry = { ...ev, id: crypto.randomUUID(), timestamp: new Date(now) };
  lastEvent = entry;
  history = [entry, ...history].slice(0, 50);
  updateLastEventDisplay();
  updateHistoryList();
}

// --- RENDER: LAST EVENT ---
function updateLastEventDisplay() {
  if (!lastEvent) return;
  $('waiting-display').style.display = 'none';
  $('last-event-display').style.display = 'block';
  $('last-device-action').textContent = lastEvent.device + ' · ' + lastEvent.action;
  $('last-value').textContent = lastEvent.value;
  $('last-details').textContent = lastEvent.details;
}

// --- RENDER: HISTORY ---
function updateHistoryList() {
  const list = $('history-list');
  $('history-count').textContent = history.length + ' eventos';
  if (history.length === 0) {
    list.innerHTML = '<div class="text-neutral-600 text-center mt-10">Sin eventos</div>';
    return;
  }
  list.innerHTML = history.map(ev => {
    const actionColor = ev.action==='Press' ? 'color:rgb(34,197,94,0.8)' : ev.action==='Release' ? 'color:rgb(239,68,68,0.8)' : ev.action==='Hold' ? 'color:rgb(234,179,8,0.8)' : 'color:rgb(59,130,246,0.8)';
    const ts = ev.timestamp.toLocaleTimeString(undefined,{hour12:false,fractionalSecondDigits:3});
    return `<div style="padding:8px 0;border-bottom:1px solid rgba(38,38,38,0.3);">
      <div style="display:flex;justify-content:space-between;align-items:baseline;margin-bottom:4px;">
        <span style="color:#d4d4d4;font-weight:700;">${ev.value}</span>
        <span style="color:#525252;font-size:11px;">${ts}</span>
      </div>
      <div style="display:flex;justify-content:space-between;font-size:11px;">
        <span style="${actionColor}">${ev.action}</span>
        <span style="color:#737373;">${ev.details}</span>
      </div>
    </div>`;
  }).join('');
}

// --- RENDER: TYPED TEXT ---
function updateTypedText() {
  const container = $('typed-text-container');
  if (typedText === '') {
    $('typed-text-placeholder').style.display = '';
    $('typed-text-content').style.display = 'none';
  } else {
    $('typed-text-placeholder').style.display = 'none';
    $('typed-text-content').style.display = '';
    $('typed-text-content').textContent = typedText;
  }
  container.scrollTop = container.scrollHeight;
}

// --- KPM TIMER ---
function startKpmTimer() {
  setInterval(() => {
    const now = Date.now();
    pressTimestamps = pressTimestamps.filter(t => now - t <= 5000);
    const kpm = pressTimestamps.length * 12;
    $('kpm-value').textContent = kpm;
  }, 200);
}

// --- FOCUS ---
function updateFocusWarning(focused) {
  $('focus-warning').style.display = focused ? 'none' : 'block';
}

// --- TOGGLE VIEW ---
function toggleView() {
  showKeyMap = !showKeyMap;
  if (showKeyMap) {
    $('text-view').style.display = 'none';
    $('keymap-view').style.display = 'flex';
    $('btn-toggle-view').textContent = 'Modo Texto';
    $('btn-toggle-view').style.cssText = 'font-size:12px;font-family:"JetBrains Mono",monospace;text-transform:uppercase;letter-spacing:0.1em;padding:8px 16px;border-radius:4px;border:1px solid #e5e5e5;background:#e5e5e5;color:#171717;cursor:pointer;';
  } else {
    $('keymap-view').style.display = 'none';
    $('text-view').style.display = 'flex';
    $('btn-toggle-view').textContent = 'Modo Teclado';
    $('btn-toggle-view').style.cssText = '';
  }
}

// --- TOGGLE RECORDING ---
function toggleRecording() {
  isRecording = !isRecording;
  $('rec-dot').style.background = isRecording ? 'rgb(34,197,94)' : '#525252';
  $('rec-dot').style.animationName = isRecording ? 'pulse' : 'none';
  $('rec-label').textContent = isRecording ? 'Escuchando' : 'Pausado';
  $('btn-recording').textContent = isRecording ? 'Pausar' : 'Reanudar';
}

// --- CLEAR ---
function clearAll() {
  history = [];
  lastEvent = null;
  typedText = '';
  testedKeys = new Set();
  ALL_TESTABLE_KEYS.forEach(code => setKeyTested(code, false));
  updateHistoryList();
  updateKeymapStats();
  updateTypedText();
  $('waiting-display').style.display = 'block';
  $('last-event-display').style.display = 'none';
}

// --- EVENT LISTENERS ---
window.addEventListener('keydown', e => {
  if (e.repeat && !recordHolds) return;
  e.preventDefault();
  e.stopPropagation();
  if (isRecording) {
    if (!testedKeys.has(e.code)) {
      testedKeys.add(e.code);
      setKeyTested(e.code, true);
      updateKeymapStats();
    }
    if (e.key.length === 1 && !e.ctrlKey && !e.metaKey && !e.altKey) {
      typedText += e.key;
    } else if (e.key === 'Backspace') {
      typedText = typedText.slice(0, -1);
    } else if (e.key === 'Enter') {
      typedText += '\n';
    }
    updateTypedText();
  }
  addEvent({ device:'Keyboard', action: e.repeat ? 'Hold' : 'Press', value: e.key===' ' ? 'Space' : e.key, details:`Code: ${e.code} | KeyCode: ${e.keyCode}` });
}, { passive: false });

window.addEventListener('keyup', e => {
  if (!recordReleases) return;
  e.preventDefault();
  addEvent({ device:'Keyboard', action:'Release', value: e.key===' ' ? 'Space' : e.key, details:`Code: ${e.code} | KeyCode: ${e.keyCode}` });
}, { passive: false });

window.addEventListener('mousedown', e => {
  e.preventDefault();
  if (isRecording && !testedKeys.has('Mouse'+e.button)) {
    testedKeys.add('Mouse'+e.button);
    setKeyTested('Mouse'+e.button, true);
    updateKeymapStats();
  }
  const btns = ['Left Click','Middle Click','Right Click','Back Button','Forward Button'];
  addEvent({ device:'Mouse', action:'Press', value: btns[e.button]||`Button ${e.button}`, details:`X: ${e.clientX}, Y: ${e.clientY}` });
}, { passive: false });

window.addEventListener('mouseup', e => {
  if (!recordReleases) return;
  const btns = ['Left Click','Middle Click','Right Click','Back Button','Forward Button'];
  addEvent({ device:'Mouse', action:'Release', value: btns[e.button]||`Button ${e.button}`, details:`X: ${e.clientX}, Y: ${e.clientY}` });
}, { passive: false });

window.addEventListener('wheel', e => {
  const dir = e.deltaY > 0 ? 'Down' : e.deltaY < 0 ? 'Up' : null;
  if (dir) addEvent({ device:'Wheel', action:'Scroll', value:`Scroll ${dir}`, details:`Delta: ${e.deltaY}` });
}, { passive: true });

window.addEventListener('contextmenu', e => e.preventDefault());

window.addEventListener('focus', () => updateFocusWarning(true));
window.addEventListener('blur', () => updateFocusWarning(false));

document.getElementById('check-releases').addEventListener('change', e => recordReleases = e.target.checked);
document.getElementById('check-holds').addEventListener('change', e => recordHolds = e.target.checked);
document.getElementById('btn-toggle-view').addEventListener('click', e => { e.stopPropagation(); toggleView(); });
document.getElementById('btn-recording').addEventListener('click', e => { e.stopPropagation(); toggleRecording(); });
document.getElementById('btn-clear').addEventListener('click', e => { e.stopPropagation(); clearAll(); });
document.getElementById('main').addEventListener('click', () => window.focus());
document.getElementById('main').addEventListener('mouseenter', () => window.focus());

// --- INIT ---
buildKeyboard();
startKpmTimer();
updateFocusWarning(document.hasFocus());
</script>

</body>
</html>
```

- [ ] **Step 2: Abrir index.html en el navegador y verificar**

Abrir `index.html` directamente en el navegador (doble clic o arrastrar).

Checklist de verificación:
- [ ] Carga sin errores en consola
- [ ] Indicador "Escuchando" visible arriba izquierda
- [ ] KPM en arriba derecha empieza en 0
- [ ] Al pulsar una tecla: aparece en grande en el centro
- [ ] El texto escrito aparece en la caja de texto
- [ ] El historial de eventos se actualiza en el panel derecho con timestamps
- [ ] Checkboxes Releases/Holds funcionan
- [ ] Botón Pausar detiene el registro; Reanudar lo activa
- [ ] Botón Limpiar resetea todo
- [ ] Botón "Modo Teclado" muestra el mapa visual
- [ ] En modo teclado: las teclas pulsadas se iluminan en verde
- [ ] El contador de teclas probadas y la barra de progreso se actualizan
- [ ] Click derecho no abre menú contextual
- [ ] Botones del ratón se registran
- [ ] Scroll se registra como Scroll Up / Scroll Down
