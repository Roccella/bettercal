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
- **Fecha simulada**: `REAL_TODAY = new Date('2026-02-03')` (Martes 3 de febrero de 2026)

### Dependencias Externas (CDN)
- React 18, ReactDOM 18, Babel standalone (unpkg.com)
- IBM Plex Sans (Google Fonts)

## Estructura de la UI (Estilo TeuxDeux)

### Layout Principal - Dos Filas
- **Fila 1 (50%)**: Calendario con scroll horizontal (columnas de 200px)
- **Fila 2 (50%)**: Categorías con scroll horizontal (columnas de 200px)
- **Header**: Fondo `--bg-header`, botones de navegación, "Hoy" condicional, "Borrar hechos", "Editar categorías"

### Características Visuales
- **Solo dark mode** (sin light mode)
- **Items sin cards**: Texto plano con checkbox en hover
- **Columnas de 200px** para días y categorías
- **Headings de días**: "2 Lunes" (número en bold, día normal), badge "HOY" azul para día actual
- **Headings de categorías**: "Música (2)" (nombre en bold, contador entre paréntesis)
- **Borde de domingo**: Línea vertical de 10px gris para separar semanas

### Items Recurrentes
- **Texto verde** (color `--accent-green`)
- **Sufijo "(R)"** después del título
- **Sin emoji 🔄**

### Items Importantes
- **Texto rojo** (color `--accent-red`)
- **Se mueven arriba** al marcar como importante
- **Campo `isImportant`** en el modelo de datos

### Botones Hover en Items (Desktop)
- **Borrar** (trash icon) - a la izquierda del botón importante
- **Importante** (círculo con !) - a la derecha extrema
- Visibles solo en hover (opacity 0 → 1)

### Interacciones
- **Click en header de día**: Crea item nuevo al principio
- **Click en área vacía del día**: Crea item nuevo al final
- **Click en header de categoría**: Crea item nuevo (sin fecha)
- **Click en área vacía de categoría**: Crea item al final
- **Checkbox en hover**: Marca como completado
- **Completar en categorías**: Item se queda en lugar (hiddenFromSidebar), no va al calendario

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
  isImportant: boolean, // NUEVO: marca item como importante (texto rojo, sube arriba)
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
}
```

## Componentes Principales

### ItemCard
- Checkbox en hover (a la izquierda)
- Título con color según estado (normal/completado/recurrente/importante/pasado)
- Botones hover: borrar y importante (derecha)
- Props: `item`, `categories`, `onComplete`, `onEdit`, `onDelete`, `onToggleImportant`, etc.

### DayColumn
- Header: "2 Lunes" + badge HOY
- Lista de items
- Click en header = agregar al principio
- Click en área vacía = agregar al final

### CategoryColumnSimple
- Header: "Música (2)" - click abre AddEditItemPopover (no edita categoría)
- Solo muestra items backlog (sin fecha, no completados)
- Click en área vacía = agregar al final

### CategoriesModal
- Abre desde botón "Editar categorías" en header
- Listado de categorías con drag & drop para reordenar
- Cada fila: drag handle + color swatch + input nombre + botón borrar
- Botones: Nueva categoría, Guardar

### Popovers (Simplificados)
- `AddEditItemPopover`: Sin selector de prioridad, sin "Marcar como hecho"
- `BottomSheet` (mobile): Mismo formato simplificado

## Funciones Clave

### handleToggleImportant(id)
- Toggle `isImportant` en item
- Si marca como importante: mueve arriba (sortOrder mínimo - 1)
- Si desmarca: mantiene posición actual

### clearCategoryDone()
- Elimina items completados de categorías (hiddenFromSidebar)
- Muestra toast con cantidad eliminada

### isTodayVisible (computed)
- Verifica si REAL_TODAY está en el array de días visibles
- Usado para mostrar/ocultar botón "Hoy" condicionalmente

## Versión Mobile

### Layout Responsive
- Breakpoint: 600px
- Mobile: Vista de 1 día con swipe + footer con tabs

### Características Mobile
- **Swipe navegación** con scroll-snap
- **Footer**: selector mes + tabs (iconos SVG 2D) + botón "+"
- **BottomSheet**: Editor sin prioridad ni "Marcar como hecho"
- **Iconos SVG 2D**: Calendario (rect + líneas), Categorías (grid 2x2)

### Safe Area (iPhone)
- `viewport-fit=cover` + `env(safe-area-inset-*)` para notch y home indicator

## Sin Usar (Removido)
- Light mode (solo dark)
- Sistema de prioridad (Important/Pendiente select)
- Botón "Marcar como hecho" en popovers
- Sticky del día de hoy
- Emoji 🔄 para recurrentes
- Click en categoría para editar (ahora crea item)
