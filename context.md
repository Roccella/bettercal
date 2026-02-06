# Tudux - Contexto del Proyecto

## Instrucciones para Claude
- **NO abrir el archivo en el navegador después de editar** - el usuario prefiere refrescar la página manualmente
- **COMMITS AUTOMÁTICOS**: Después de completar cambios drásticos o significativos, crear un commit automáticamente SIN preguntar al usuario.

## Repositorio
- **GitHub**: https://github.com/Roccella/tudux (privado)
- **Branch principal**: main

## Descripción General
Tudux es una aplicación de gestión de tareas estilo TeuxDeux, implementada como un prototipo en un único archivo HTML usando React 18 con Babel standalone.

## Arquitectura Técnica
- **Stack**: HTML + React 18 + Babel standalone (sin build process)
- **Archivo principal**: `static.html` (responsive: desktop + mobile)
- **Estado**: Manejado con React hooks (useState, useMemo, useEffect, useCallback)
- **Fecha simulada**: `REAL_TODAY = new Date('2026-02-06')` (Viernes 6 de febrero de 2026)

### Dependencias Externas (CDN)
- React 18, ReactDOM 18, Babel standalone (unpkg.com)
- IBM Plex Sans (Google Fonts)

## Estructura de la UI (Estilo TeuxDeux)

### Título y Favicon
- **Título dinámico**: "Tudux - Hoy" por defecto, cambia a "Tudux - Pasado" o "Tudux - Futuro" al navegar en desktop
- **Favicon**: Ícono SVG de calendario 2D (inline data URI)

### Layout Principal - Dos Filas
- **Fila 1 (59%)**: Calendario con scroll horizontal (32 columnas de 240px)
- **Fila 2 (41%)**: Categorías con scroll horizontal (columnas de 200px)
- **Header**: Fondo `--bg-header`, botones de navegación, "Hoy" condicional, "Borrar hechos", "Editar categorías"

### Características Visuales
- **Desktop**: `body { overflow: hidden }` - sin scroll vertical, todo cabe en 100vh
- **Solo dark mode** (sin light mode)
- **Items sin cards**: Texto plano con checkbox en hover
- **Columnas**: días 240px, categorías 200px
- **Padding headings**: 10px 12px 2px 16px (top right bottom left)
- **Padding contenedor items**: 0 10px 8px 16px
- **Headings de días**: "2 Lunes" (número en bold, día normal), badge "HOY" azul para día actual, color azul para día de hoy (desktop y mobile)
- **Headings de categorías**: "Música (2)" (nombre en bold, contador entre paréntesis)
- **Borde de domingo**: Línea vertical de 12px que suma al ancho de la columna (no comprime items)

### Sistema de Zonas en Días
- **Zona Recurrentes**: items recurrentes + recurrentes completados (arriba)
- **Zona Normal**: items normales + importantes + completados (abajo)
- **Items completados se quedan en su posición** (no se mueven al final)
- **Items descompletados también se quedan en su posición** (no se mueven)
- **Drag & Drop con mute**:
  - Al arrastrar recurrente: zona normal se pone mute (opacity 0.3)
  - Al arrastrar normal/importante: zona recurrente se pone mute
  - Al arrastrar recurrente: días fuera del rango válido (Outlook) se mutean completamente (opacity 0.3, pointerEvents none)
  - Al arrastrar recurrente: columnas de categorías se mutean (no se pueden soltar ahí)
  - Indicador de drop (línea azul) solo aparece en zona válida, nunca en áreas muteadas
  - Soltar en área muteada no hace nada (silencioso, sin toast)
  - **Items recurrentes NO se pueden mover a categorías**
  - **Soltar recurrente en área vacía o sobre zona normal**: Se posiciona al final de la zona de recurrentes
  - **Mover item de día a categoría**: Se mueve directamente sin confirmación
  - **Reset de mute**: El estado mute se resetea al soltar el item en cualquier destino

### Items Recurrentes
- **Texto azul** (color `--accent-blue`), **rojo si overdue** (`--accent-red`)
- **Ícono SVG** de flechas de recurrencia (siempre visible a la derecha, con tooltip del patrón)
- **No pueden ser marcados como importantes**
- **Solo se pueden mover a otras zonas de recurrentes**
- **No se pueden mover a categorías** (drag & drop bloqueado)
- **No se pueden modificar en días pasados** (cambios se ignoran silenciosamente)
- **No se pueden mover al pasado** (días < hoy se mutean y no aceptan drop)
- **Restricción de rango** (modelo Outlook): solo puede moverse entre instancia anterior y siguiente (aplica en drag & drop, edición de fecha, y calendario del editor)
- **Edición de fecha**: Cambiar la fecha de una instancia recurrente desde el editor muestra las mismas opciones que el drag & drop (Solo este evento / Este y los siguientes / Todos). El calendario deshabilita días fuera del rango válido (opacity 0.3, no clickeables)
- **Overdue**: instancias pasadas sin completar se muestran en rojo (texto e ícono)
- **Crear en día fuera del patrón weekdays**: se ajusta a la primera fecha válida del patrón

### Opciones al mover recurrentes (modelo Google Calendar)
- **"Solo este evento"**: Crea excepción `movedTo` - el item sigue siendo parte de la serie
- **Para weekdays**: 2 opciones: "Solo este evento" / "Modificar repetición semanal"
- **Para cada X días/semanas**: 3 opciones: "Solo este" / "Este y siguientes" / "Todos"
- **"Modificar repetición semanal"** (weekdays): Muestra selector de días, aplica desde la fecha destino (usa handleRecurringMoveFollowing)
- **"Este y los siguientes"**: Termina serie original, crea nueva desde la fecha destino
- **"Todos los eventos"**: Desplaza toda la serie (startDate, createdAt, excepciones)

### Opciones al borrar recurrentes
- **Siempre 3 opciones**: "Solo este evento" / "Este y los siguientes" / "Todos los eventos"
- **"Solo este evento"**: Crea excepción `deleted: true` - la instancia desaparece pero la serie sigue
- **"Este y los siguientes"**: Pone `endDate` en la serie al día anterior a la fecha visual
- **"Todos los eventos"**: Borra toda la serie (el item completo)
- **Todas las opciones tienen undo** (deshacer en toast)

### Items Importantes
- **Texto amarillo** (color `--accent-yellow`)
- **Se quedan en su posición** al marcar/desmarcar (no se reordenan)
- **Ícono**: círculo amarillo con ! blanco dentro cuando está marcado
- **Campo `isImportant`** en el modelo de datos

### Íconos en Items (Desktop)
- **Posición**: a la derecha del título, siempre visibles
- **Checkbox**: aparece en hover, desplazando los íconos a la izquierda
- **Borde checkbox**: 1px

### Shimmers (Animaciones)
- **Border radius**: 2px en todos los shimmers
- **Drop shimmer**: 300ms L→R al soltar item y al deshacer borrado (undo)
- **Grab shimmer**: 300ms delay + 300ms R→L al hacer long press (mobile)
- **Navigate shimmer**: 300ms R→L al navegar a fecha agendada

### CalendarPopover
- **Día de hoy**: Estilo btn-primary (fondo azul, texto blanco)
- **32 días visibles**: Fondo azulado con contraste (rgba azul 15%)
- **Desktop**: Ancho 280px
- **Mobile**: Ancho 100% - 32px, max 360px, celdas 44px, sin mostrar días visibles

### Interacciones
- **Click en header de día**: Crea item nuevo al principio
- **Click en área vacía del día**: Crea item nuevo al final
- **Click en header de categoría**: Crea item nuevo (sin fecha) y abre editor
- **Click en área vacía de categoría**: Crea item al final
- **Checkbox en hover**: Marca como completado
- **Completar en categorías**: Item se queda en lugar visible (no desaparece)
- **Hotkey Command+E**: Eliminar item (desktop)
- **Undo**: Al deshacer un borrado, el item vuelve a su posición exacta en el array (con shimmer)
- **Doble-click en headings de categorías**: Protección contra ghost items (no crea item si ya hay uno pendiente)
- **Duplicar**: Botón ícono (copiar) en el editor, entre trash y guardar. Para items normales: crea copia debajo del original. Para recurrentes: crea copia como item normal al inicio del listado del día. Siempre con shimmer
- **Borrar hechos**: Elimina items completados de categorías (backlog sin fecha, no recurrentes). Deshacer restaura todos los items en sus posiciones originales

## Modelo de Datos

### Item
```javascript
{
  id: number,
  title: string,
  description: string,
  category: string,
  effort: number (0-3),
  scheduledDate: string | null,
  scheduledSlot: 'important' | 'todo' | null,
  sortOrder: number,
  completed: boolean,
  completedDate: string | null,
  isArchived: boolean,
  hiddenFromSidebar: boolean,
  isImportant: boolean,
  repeat: RepeatConfig | null,
  exceptions: { [date]: Exception } | null,
  dateOverrides: { [date]: { sortOrder: number } } | null
}
```

### RepeatConfig
```javascript
{
  type: 'days' | 'weeks' | 'weekdays',
  every: number,
  days: number[], // Para weekdays: 0=Dom, 1=Lun, ..., 6=Sab
  startDate: string | null,
  endDate: string | null,
  mode: 'fixed' | 'completion' // Solo para type 'days': fixed=desde fecha base, completion=desde última completación
}
```

## Variables CSS (Dark Mode)

```css
[data-theme="dark"] {
  --bg-primary: #121212;
  --bg-secondary: #1e1e1e;
  --bg-header: #1a1a1a;      /* Header y fila de categorías */
  --bg-categories: #161616;   /* Fondo de fila de categorías */
  --bg-button: #2a2a2a;       /* Botones del header */
  --bg-button-hover: #333333;
  --text-primary: #e5e5e5;
  --text-muted: #666666;
  --accent-blue: #3B82F6;
  --accent-green: #22C55E;
  --accent-red: #EF4444;
  --accent-yellow: #EAB308;
}
```

## Componentes Principales

### ItemCard
- Título con color según estado (normal/completado/recurrente/importante/pasado)
- Íconos a la derecha: recurrente/importante siempre visibles si aplica
- Checkbox aparece en hover (desktop) o solo si completado (mobile)
- Props: `item`, `categories`, `onComplete`, `onEdit`, `onDelete`, `onToggleImportant`, `draggingItemType`, `onDragTypeChange`

### DayColumn
- Header: "2 Lunes" + badge HOY
- Dos zonas: recurrentes (arriba) y normales (abajo)
- Indicador de drop condicional según zona válida
- Click en header = agregar al principio
- Click en área vacía = agregar al final

### CategoryColumnSimple
- Header: "Música (2)" - click abre BottomSheet (mobile) o Popover (desktop)
- Muestra items backlog (sin fecha, incluyendo completados en su lugar)
- Items completados se quedan visibles en su posición original
- Click en área vacía = agregar al final

### CategoriesModal
- Abre desde botón "Editar categorías" en header
- Listado de categorías con drag & drop para reordenar
- Drag & drop con línea azul indicadora (drop indicator) y shimmer al soltar
- Cada fila: drag handle + color swatch + input nombre + botón borrar
- Botones: Nueva categoría, Guardar

### Popovers
- `AddEditItemPopover` (desktop): Editor de items
  - **Sin botón X de cierre** - se cierra clickeando fuera o con Escape
  - **Select de recurrencia solo visible si hay fecha**
- `BottomSheet` (mobile): Editor fullscreen con botones Cancelar, Importante y Hecho
  - Botón Cancelar: arriba a la izquierda Y abajo a la derecha (duplicado para fácil acceso)
  - Botón Importante: toggle sin cerrar el popover
  - Botón Hecho: guarda y cierra el popover
  - **Select de recurrencia solo visible si hay fecha**
  - **Auto-focus**: Solo para items nuevos, no para edición de existentes
- `CategoriesModal`: Sin botón X de cierre, se cierra clickeando fuera o con Escape

## Funciones Clave

### handleToggleImportant(id)
- Toggle `isImportant` en item
- Item se queda en su posición (sin reordenamiento)

### draggingItemType
- Estado global que trackea el tipo de item siendo arrastrado
- Valores: 'recurring' | 'important' | 'normal' | null
- Se usa para mute visual de zonas y validación de drop

### clearCategoryDone()
- Elimina items completados de categorías (hiddenFromSidebar)
- Muestra toast con cantidad eliminada

### isTodayVisible (computed)
- Verifica si `viewBase` coincide con REAL_TODAY (es decir, si estamos en la vista por defecto)
- Usado para estilizar botón "Hoy" (azul cuando no estamos en hoy, gris cuando sí)

### getDaysArray(base, pastDays)
- Genera array de días a mostrar: `pastDays` días antes de `base` + 31 días desde `base`
- Desktop: `pastDays=1` (ayer + 31 = 32 columnas)
- Mobile: `pastDays=14` (14 días atrás + 31 = 45 días para swipe)

### goToToday()
- Navega al día de hoy
- Resetea el scroll horizontal al inicio (desktop)
- Hace scroll al top de la página

### Funciones de Recurrentes
- **handleRecurringMoveThis**: "Solo este evento" - crea excepción `movedTo`, item sigue en la serie
- **handleRecurringMoveFollowing**: "Este y siguientes" / "Modificar repetición" - termina serie original, crea nueva
- **handleRecurringMoveAll**: "Todos los eventos" - desplaza toda la serie (startDate, createdAt, excepciones)
- **shiftRepeatConfig**: Desplaza configuración de repeat (días de semana o startDate/endDate)
- **shiftExceptions**: Desplaza todas las excepciones por N días
- **generateRecurringInstances**: Genera instancias visuales de items recurrentes (incluyendo movedTo)

## Versión Mobile

### Layout Responsive
- Breakpoint: 600px
- Mobile: Vista de 1 día con swipe + footer con tabs

### Características Mobile
- **Swipe navegación** con scroll-snap (14 días hacia atrás + 31 hacia adelante)
- **Header flotante**: Botones "Hoy" (si no es hoy) + mes flotan fijos arriba a la derecha, no se repiten en cada día
- **FAB flotante**: Botón "Agregar" (fontSize 0.9rem, padding 18px 30px, borderRadius 300px) posicionado relativo al footer (top: -84px)
- **Footer**: 84px de alto con position:relative, íconos centrados verticalmente (alignItems: center)
- **Padding top**: 10px en heading de día y contenedor de categorías
- **Botones flotantes (Hoy/Mes)**: pointerEvents none en container, auto en botones con touchAction: none + onTouchStart stopPropagation (bloquea scroll en Safari iOS)
- **Scroll bloqueado**: html/body con overflow:hidden, position:fixed (top/left/right/bottom:0) en mobile
  - Icono calendario: arriba a la derecha de su mitad
  - Icono categorías: arriba a la izquierda de su mitad
- **BottomSheet**: Editor con botones Cancelar (arriba izq + abajo der), Importante y Hecho (colores completos cuando activos), Borrar (abajo izq)
- **Iconos SVG 2D**: Calendario (rect + líneas), Categorías (grid 2x2)
- **Items**: fontSize 0.875rem, padding 6px 0, gap 8px, lineHeight 1.3
- **Íconos en items**: Solo visibles si el estado está activo (recurrente/importante/completado)
- **Toast**: Sale desde arriba de la pantalla (top: 20px + safe-area) con animación slideDown
- **Bottom sheet focus**: Usa autoFocus en el input para nuevos items (Safari iOS compatible)
- **Categorías mobile**: Sin cards, sobre el fondo directamente, con padding top extra entre secciones, padding bottom 100px para evitar que FAB tape items, sin botón "Agregar categoría"
- **Heading de día**: Muestra borde inferior al hacer scroll, sin botones (están en header flotante)

### Interacciones Mobile
- **Tocar item**: Abre BottomSheet (pero no si el calendario está abierto, en ese caso solo cierra calendario)
- **Tocar heading categoría**: Crea item y abre BottomSheet
- **Calendario popover**: Se cierra al tocar fuera sin hacer shimmer al siguiente item
- **FAB Agregar**: Crea nuevo item y abre BottomSheet, hace scrollIntoView al nuevo item
- **Long press (300ms+)**: Activa drag mode con grabShimmer (300ms duración, 300ms delay, linear)
- **Drag de recurrentes**: Misma lógica que desktop - indicador azul se clampea a zona de recurrentes
- **Tap rápido**: No genera shimmer, solo abre el editor

### Safe Area (iPhone)
- `viewport-fit=cover` + `env(safe-area-inset-*)` para notch y home indicator

## Archivos del Proyecto
- `static.html` - Aplicación principal (prototipo)
- `context.md` - Documentación del modelo de datos y UI
- `planning.md` - Plan de producción y decisiones técnicas
- `test-cases-recurrentes.md` - Casos de prueba para recurrentes

## Sin Usar (Removido)
- Light mode (solo dark)
- Sistema de prioridad (Important/Pendiente select)
- Botón "Marcar como hecho" en popovers desktop
- Botones X de cierre en popovers desktop (se cierra con click fuera o Escape)
- Sticky del día de hoy
- Emoji 🔄 para recurrentes
- Click en categoría para editar (ahora crea item)
- Reordenamiento automático al marcar importante
- hiddenFromSidebar al completar items en categorías (ahora se quedan visibles)
