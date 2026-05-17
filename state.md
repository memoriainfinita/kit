# State — KIT (Keyboard Input Tester)

## System

- **Stack:** HTML5 + JS vanilla + Tailwind Play CDN + Google Fonts CDN
- **Archivo único:** `index.html` — sin build, sin dependencias locales
- **Uso:** abrir directamente en navegador (doble clic)

## Structure

```
KIT/
  index.html                          — app completa autocontenida
  docs/superpowers/plans/
    2026-05-17-kit-html-port.md       — plan de implementación
  state.md
```

## App Description

Herramienta de diagnóstico de entradas (teclado + ratón). Todo client-side, sin backend.

**Funcionalidades:**
- Última tecla/botón pulsado en grande con code y keyCode
- KPM en tiempo real (ventana de 5 segundos)
- Caja de texto que acumula lo escrito (con Backspace y Enter)
- Historial de eventos (últimos 50) con timestamps de ms
- Filtros: Releases, Holds
- Modo Teclado: mapa visual completo (QWERTY + nav + numpad + ratón) con tracking de teclas probadas y % de cobertura
- Controles: Pausar/Reanudar, Limpiar, toggle de modo
- Aviso de foco de ventana
- Sin menú contextual (para poder testear click derecho)

## Patterns

- [vanilla] App 100% serverless, un solo archivo HTML. Sin framework, sin build. Confirmed 2026-05.
- [cdn] Tailwind Play CDN + Google Fonts CDN. No requiere instalación. Confirmed 2026-05.
- [easter-egg] Easter eggs activados por secuencia de teclado (buffer de chars). Confirmed 2026-05.
- [canvas-animation] Animaciones complejas con canvas + requestAnimationFrame, no CSS puro. Confirmed 2026-05.

## History

### 2026-05-17 — sesión 1
- Proyecto creado como port de `014 minimal input tester` (Next.js + React)
- Motivo: eliminar node_modules (627 MB), simplificar stack, hacer app serverless
- `index.html` implementado y verificado funcionando en navegador
- Todas las funcionalidades del original portadas y confirmadas

### 2026-05-17 — sesión 2
- Easter egg KITT (Knight Rider) añadido a `index.html`
- Triggers: escribir `kitt` o `kit te necesito` (buffer de teclado, case insensitive)
- Implementación: canvas con segmentos LED discretos (Larson scanner)
  - ~26 celdas de 18×10px con huecos de 2px
  - Decay por célula (×0.82/frame) → estela de fosforescencia residual
  - Velocidad ligada al KPM en tiempo real (350 + kpm×1.8 px/s)
  - Barra centrada, 560px max-width 80vw, border-radius inferior
  - Cerrar: Escape o clic sobre la barra
- Panel de historial: botón ✕ para ocultar, pestaña lateral vertical para reabrir

## TODO

- [x] Easter egg KITT implementado
- [ ] Ajustes visuales adicionales al scanner KITT (si se quiere seguir refinando)
- [ ] Añadir más funcionalidades (pendiente de definir)

## Origen

Port de `014 minimal input tester` (Next.js + React + Tailwind v4).
Motivación: eliminar 627 MB de node_modules, simplificar stack, hacer app serverless y portable.
