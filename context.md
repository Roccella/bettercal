# Better Cal - Contexto del Proyecto

## Instrucciones para Claude
- **NO abrir el archivo en el navegador después de editar** - el usuario prefiere refrescar la página manualmente
- **COMMITS AUTOMÁTICOS**: Después de completar cambios drásticos o significativos, crear un commit automáticamente SIN preguntar al usuario.

## Repositorio
- **GitHub**: https://github.com/Roccella/bettercal (privado)
- **Branch principal**: main

## Descripción General
Better Cal es una aplicación de gestión de tareas estilo TeuxDeux, implementada como un prototipo en un único archivo HTML usando React 18 con Babel standalone.

## Arquitectura Técnica
- **Stack**: HTML + React 18 + Babel standalone (sin build process)
- **Archivo principal**: `static.html` (responsive: desktop + mobile)
- **Estado**: Manejado con React hooks (useState, useMemo, useEffect, useCallback)
- **Fecha simulada**: `REAL_TODAY = new Date('2026-02-06')` (Viernes 6 de febrero de 2026)

### Dependencias Externas (CDN)
- React 18, ReactDOM 18, Babel standalone (unpkg.com)
- IBM Plex Sans (Google Fonts)

## Estructura de la UI (Estilo TeuxDeux)

### Layout Principal - Dos Filas
- **Fila 1 (50%)**: Calendario con scroll horizontal (32 columnas de 240px)
- **Fila 2 (50%)**: Categorías con scroll horizontal (columnas de 200px)
- **Header**: Fondo `--bg-header`, botones de navegación, "Hoy" condicional, "Borrar hechos", "Editar categorías"

### Características Visuales
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
  - Indicador de drop (línea azul) solo aparece en zona válida
  - **Items recurrentes NO se pueden mover a categorías**
  - **Soltar recurrente en área vacía o sobre zona normal**: Se posiciona al final de la zona de recurrentes
  - **Mover item de día a categoría**: Se mueve directamente sin confirmación
  - **Reset de mute**: El estado mute se resetea al soltar el item en cualquier destino

### Items Recurrentes
- **Texto azul** (color `--accent-blue`)
- **Ícono SVG** de flechas de recurrencia (siempre visible a la derecha)
- **No pueden ser marcados como importantes**
- **Solo se pueden mover a otras zonas de recurrentes**
- **No se pueden mover a categorías** (drag & drop bloqueado)
- **No se pueden modificar en días pasados** (cambios se ignoran silenciosamente)

### Items Importantes
- **Texto amarillo** (color `--accent-yellow`)
- **Se quedan en su posición** al marcar/desmarcar (no se reordenan)
- **Ícono**: círculo amarillo con ! blanco dentro cuando está marcado
- **Campo `isImportant`** en el modelo de datos

### Íconos en Items (Desktop)
- **Posición**: a la derecha del título, siempre visibles
- **Checkbox**: aparece en hover, desplazando los íconos a la izquierda
- **Borde checkbox**: 1px

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
  endDate: string | null
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
- Cada fila: drag handle + color swatch + input nombre + botón borrar
- Botones: Nueva categoría, Guardar

### Popovers
- `AddEditItemPopover` (desktop): Editor de items
  - **Select de recurrencia solo visible si hay fecha**
- `BottomSheet` (mobile): Editor fullscreen con botones Importante y Hecho
  - Botón Importante: toggle sin cerrar el popover
  - Botón Hecho: guarda y cierra el popover
  - **Select de recurrencia solo visible si hay fecha**
  - **Auto-focus**: Solo para items nuevos, no para edición de existentes

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
- Verifica si REAL_TODAY está en el array de días visibles
- Usado para mostrar/ocultar botón "Hoy" condicionalmente

### goToToday()
- Navega al día de hoy
- Resetea el scroll horizontal al inicio (desktop)
- Hace scroll al top de la página

## Versión Mobile

### Layout Responsive
- Breakpoint: 600px
- Mobile: Vista de 1 día con swipe + footer con tabs

### Características Mobile
- **Swipe navegación** con scroll-snap
- **Header flotante**: Botones "Hoy" (si no es hoy) + mes flotan fijos arriba a la derecha, no se repiten en cada día
- **FAB flotante**: Botón "Agregar" (padding 22px 36px) posicionado a 120px del fondo, con touchAction manipulation
- **Footer**: 112px de alto, íconos arriba (padding 24px 20px 0), zonas de tap completas
- **Padding top**: 10px en heading de día y contenedor de categorías
- **Botones flotantes (Hoy/Mes)**: pointerEvents none en container, auto en botones (permite scroll through)
- **Scroll bloqueado**: html/body con overflow:hidden y position:fixed en mobile, solo scroll en contenedores internos
  - Icono calendario: arriba a la derecha de su mitad
  - Icono categorías: arriba a la izquierda de su mitad
- **BottomSheet**: Editor con botones Importante/Hecho (colores completos cuando activos)
- **Iconos SVG 2D**: Calendario (rect + líneas), Categorías (grid 2x2)
- **Items**: fontSize 0.875rem, padding 8px 0, gap 8px, lineHeight 1.3
- **Íconos en items**: Solo visibles si el estado está activo (recurrente/importante/completado)
- **Toast**: Posición más arriba (132px + safe-area) para no tapar footer
- **Bottom sheet focus**: Delay de 350ms para esperar animación slideUp antes de focus
- **Categorías mobile**: Sin cards, sobre el fondo directamente, con padding top extra entre secciones
- **Heading de día**: Muestra borde inferior al hacer scroll, sin botones (están en header flotante)

### Interacciones Mobile
- **Tocar item**: Abre BottomSheet (pero no si el calendario está abierto, en ese caso solo cierra calendario)
- **Tocar heading categoría**: Crea item y abre BottomSheet
- **Calendario popover**: Se cierra al tocar fuera sin hacer shimmer al siguiente item
- **FAB Agregar**: Crea nuevo item y abre BottomSheet
- **Long press (300ms+)**: Activa drag mode con shimmer visual
- **Tap rápido**: No genera shimmer, solo abre el editor

### Safe Area (iPhone)
- `viewport-fit=cover` + `env(safe-area-inset-*)` para notch y home indicator

## Sin Usar (Removido)
- Light mode (solo dark)
- Sistema de prioridad (Important/Pendiente select)
- Botón "Marcar como hecho" en popovers desktop
- Sticky del día de hoy
- Emoji 🔄 para recurrentes
- Click en categoría para editar (ahora crea item)
- Reordenamiento automático al marcar importante
- hiddenFromSidebar al completar items en categorías (ahora se quedan visibles)
