# Better Cal - Contexto del Proyecto

## Repositorio
- **GitHub**: https://github.com/Roccella/bettercal (privado)
- **Branch principal**: main

## Descripción General
Better Cal es una aplicación de gestión de tareas y atención, implementada como un prototipo en un único archivo HTML usando React 18 con Babel standalone (transformación JSX en el navegador).

## Arquitectura Técnica
- **Stack**: HTML + React 18 + Babel standalone (sin build process)
- **Archivo principal**: `prototype.html` (responsive: desktop + mobile)
- **Backup**: `prototype-bkp1.html` (versión solo desktop)
- **Estado**: Manejado con React hooks (useState, useMemo, useEffect, useCallback)
- **Fecha simulada**: `REAL_TODAY = new Date('2026-02-03')` (Martes 3 de febrero de 2026)

### Dependencias Externas (CDN)
Actualmente el prototipo usa CDN para:
- React 18 (unpkg.com)
- ReactDOM 18 (unpkg.com)
- Babel standalone (unpkg.com) - solo para desarrollo
- IBM Plex Sans (Google Fonts)

> **⚠️ RECORDATORIO PARA PRODUCCIÓN**: Cuando se implemente la versión de producción, pasar todas las dependencias de CDN a local:
> - Bundlear React/ReactDOM con Vite o similar (elimina Babel standalone)
> - Hostear IBM Plex Sans localmente (solo weights 400, 500, 600, 700)
> - Esto mejora: disponibilidad, velocidad, privacidad, seguridad, y funcionamiento offline

## Estructura de la UI

### Layout Principal
- **Calendario**: Vista de 12 días con scroll horizontal (columnas de 200px)
- **Sidebar derecha**: Categorías con grupos Backlog, Agendado y Hecho (en ese orden)

### Slots del Calendario
Cada día tiene 4 slots:
- `morning` (Mañana)
- `afternoon` (Tarde)
- `evening` (Noche)
- `done` (Hecho) - altura fija de 145px

### Sistema de Energía
- 3 niveles por día: 1 (bajo 🪦), 2 (medio 😎), 3 (alto 🔥)
- Afecta capacidad de esfuerzo del día

## Modelo de Datos

### Item
```javascript
{
  id: number,
  title: string,
  description: string,
  category: string,
  effort: number (1-3),
  scheduledDate: string | null (YYYY-MM-DD),
  scheduledSlot: string | null,
  sortOrder: number,
  completed: boolean,
  completedDate: string | null,
  createdAt: string,
  isArchived: boolean,
  hiddenFromSidebar: boolean, // Para "Limpiar hechos"
  repeat: RepeatConfig | null,
  exceptions: { [date]: Exception } | null,
  dateOverrides: { [date]: { sortOrder: number } } | null
}
```

### RepeatConfig
```javascript
{
  type: 'days' | 'weeks' | 'weekdays',
  every: number, // Para days/weeks
  days: number[], // Para weekdays: 0=Dom, 1=Lun, ..., 6=Sab
  startDate: string | null, // Para "este y siguientes"
  endDate: string | null // Para terminar series
}
```

### Exception (para items recurrentes)
```javascript
{
  deleted: boolean, // Instancia eliminada
  movedTo: { date: string, slot: string }, // Instancia movida
  completed: boolean, // Instancia completada
  completedDate: string
}
```

## Sistema de Items Recurrentes

### Generación de Instancias
- `generateRecurringInstances()` crea instancias visuales desde items base
- IDs de instancias: `{baseId}-{YYYY-MM-DD}` o `{baseId}-{YYYY-MM-DD}-moved`
- Las instancias heredan propiedades del item base pero pueden tener overrides

### Interacciones de Movimiento

#### Mismo día, mismo slot
- Solo reordena (sin popover)

#### Mismo día, diferente slot (incluyendo Hecho)
- Aplica automáticamente "Solo este evento"
- No muestra popover de opciones

#### Diferente día, slot normal
Al soltar el item:
1. El item aparece temporalmente en la posición exacta donde se soltó (usando `pendingRecurringDrop` con `_dropIndex`)
2. Los sortOrders del slot destino se recalculan para posicionar correctamente el item temporal
3. El popover aparece debajo del item soltado
4. Si se cancela, el item vuelve a su posición original

Muestra popover con 2 opciones:
1. **Solo este evento**: Marca instancia como `deleted` en la serie y crea un item independiente (sin recurrencia) en la fecha/slot destino
2. **Este y siguientes**: Termina serie original, crea nueva serie desde fecha destino
   - Para items tipo `weekdays`: Muestra selector de días, botón "Aplicar" deshabilitado hasta que se modifiquen los días
   - Para otros tipos (`days`, `weeks`): Muestra input de frecuencia con el valor actual, botón "Aplicar" siempre habilitado (permite mover manteniendo la misma frecuencia)

Nota: No se incluye "Todos los eventos" porque mover eventos pasados no tiene sentido semántico (mismo comportamiento que Google Calendar).

#### Diferente día, slot Hecho
- Solo muestra opción "Solo este evento"
- Las otras opciones no tienen sentido (no puede haber serie recurrente en slot "done")

### Fechas en Drag & Drop
El dragData incluye:
- `exceptionDate`: Clave en exceptions (para "solo este evento")
- `visualDate`: Donde aparece el item visualmente (para "este y siguientes"/"todos")

### Completar Items Recurrentes
- Desde calendario: Click en checkbox marca esa instancia (crea excepción `completed`)
- Desde sidebar Agendado: Checkbox deshabilitado (no se puede completar)
- Desde sidebar Backlog: Sin checkbox para recurrentes
- Desmarcar: Quita `completed` de la excepción

## Componentes Principales

### ItemCard
- Muestra checkbox, título, metadata y barra de esfuerzo
- Estructura de 3 contenedores: izquierda (checkbox), centro (título+meta), derecha (effort con margin-top: 4px para centrado vertical)
- Props: `item`, `categories`, `onComplete`, `onEdit`, `inSidebar`, `isToday`, `isPast`, `showDate`
- Atributo `data-item-id` para localizar el elemento en el DOM (usado por RecurringActionPopover)

### SlotSection
- Contenedor de items para un slot específico
- Maneja drag & drop y creación de nuevos items
- Botón "+" para agregar items

### SidebarGroup
- Grupo colapsable en sidebar (Backlog, Agendado, Hecho)
- Maneja drag & drop entre grupos

### CategorySection
- Sección de categoría en sidebar
- Contiene los 3 grupos en orden: Backlog, Agendado, Hecho
- Recibe `groupsExp` y `onGroupToggle` props para estado de expansión persistente

### Popovers
- `AddEditItemPopover`: Crear/editar items (incluye selector de fecha, horario, esfuerzo, recurrencia). Tiene botones: 🗑 (si editando), ✕ (cancelar), Guardar/Agregar
- `EditCategoryPopover`: Crear/editar categorías (con alerta de descartar cambios)
- `RecurringActionPopover`: Opciones al mover items recurrentes
- `RemoveDatePopover`: Confirmación al mover item agendado/recurrente a backlog
- `DiscardConfirmDialog`: Confirmación de descartar cambios
- `CalendarPopover`: Selector de fecha
- `MiniCalendar`: Calendario pequeño dentro de AddEditItemPopover

## Variables CSS Importantes

### Colores de fondo de tarjetas
- `--bg-card`: Sidebar
- `--bg-card-today`: Items de hoy (más claro)
- `--bg-card-past`: Items pasados
- `--bg-card-future`: Items futuros

### Otros
- `--checkbox-bg`: Fondo del checkbox
- `--effort-bg-today`: Fondo de barra de esfuerzo en items de hoy
- `--accent-green`: Checkbox completado
- `--accent-blue`: Indicadores de recurrencia

## Funciones Clave

### Manejo de Recurrentes
- `generateRecurringInstances(items, startDate, endDate)`: Genera instancias visuales. Para tipo `days`, usa `startDate` o `createdAt` como base y calcula `daysSinceBase % every === 0`. Las fechas se normalizan con `'T00:00:00'` para evitar problemas de timezone.
- `handleRecurringMoveThis()`: Marca instancia como `deleted` y crea item independiente en destino
- `handleRecurringMoveFollowing()`: Termina serie y crea nueva
- `handleRecurringMoveAll()`: Mueve toda la serie
- `shiftExceptions(exceptions, dayOffset, targetSlot)`: Desplaza excepciones
- `shiftRepeatConfig(repeat, dayOffset)`: Desplaza configuración de repeat
- `pendingRecurringDrop`: Estado temporal para mostrar item en destino mientras se muestra el popover de opciones. Incluye `_dropIndex` para posicionar correctamente el item en el slot destino

### Utilidades de Fecha
- `formatDate(date)`: Retorna YYYY-MM-DD
- `formatDateLabel(dateStr)`: Retorna "Lun 2 Feb"
- `getDayDiff(date1, date2)`: Diferencia en días
- `shiftDate(dateStr, days)`: Suma días a una fecha

### Sidebar
- `clearDone()`: Limpia items hechos de sidebar (usa `hiddenFromSidebar` flag)
- Items recurrentes muestran "Recurrente" en vez de fecha

## Control de Popovers
- Variable global `lastPopoverCloseTime` previene reapertura accidental (threshold 200ms)
- Funciona tanto en desktop (popovers) como en mobile (BottomSheet a pantalla completa)
- Botón "+" de nueva categoría deshabilitado cuando hay popover de item abierto

## Notas de Implementación

### Checkbox en Sidebar
- Agendado: Visible pero deshabilitado para recurrentes
- Backlog: Oculto para recurrentes
- Ambos: Funcional para items no recurrentes

### Barras de Esfuerzo
- Colores más oscuros para items de hoy
- Fondo más oscuro (`--effort-bg-today`) para contraste en tarjetas blancas

### Alertas de Descartar
- Items: Muestra si hay cambios desde estado inicial
- Categorías: Muestra si nombre o color cambiaron
- No muestra si solo se abrió el popover sin cambios

### Popover de Edición de Items
- **Selector de fecha**: MiniCalendar con opción "Sin fecha (Backlog)"
- **Selector de horario**: Mañana/Tarde/Noche (solo visible si hay fecha o recurrencia)
- **Alerta de recurrencia**: Muestra aviso cuando se quita fecha a un item recurrente
- **Detección de cambios**: Incluye título, descripción, categoría, esfuerzo, fecha, horario, tipo de repetición, frecuencia y días de semana
- **Botón X de cancelar**: En desktop y mobile, entre trash y guardar

### Flujos de Quitar Fecha/Recurrencia
1. **Drag & drop a Backlog**: Muestra `RemoveDatePopover` con mensaje apropiado (fecha o recurrencia)
2. **Editar → Sin fecha → Guardar**: Muestra alerta inline, al guardar limpia `repeat` y mueve a backlog
3. Items movidos a backlog aparecen primero en la lista

## Persistencia con localStorage

### Estados Persistidos
- `bettercal_expanded`: Estado de categorías expandidas/colapsadas (objeto `{ categoryId: boolean }`)
- `bettercal_groups`: Estado de grupos dentro de cada categoría (objeto `{ categoryId: { backlog: bool, scheduled: bool, done: bool } }`)

### Implementación
- `loadFromStorage(key, defaultValue)`: Helper para cargar estado con fallback
- Estados se inicializan con función para cargar de localStorage
- `useEffect` guarda cambios automáticamente cuando los estados cambian
- Los botones "Expandir todo" / "Colapsar todo" actualizan ambos estados

### Feedback Visual de Drop
- Animación shimmer (destello de izquierda a derecha) cuando se suelta un item
- Clase CSS `.just-dropped` con pseudo-elemento `::after` animado
- Estado `justDroppedItemId` en App, se limpia después de 600ms
- Funciona en calendario y sidebar, para items normales y recurrentes

## Versión Mobile

### Layout Responsive
- Breakpoint: 600px
- Desktop (>600px): Layout original con 12 días + sidebar (columnas de 200px)
- Mobile (≤600px): Vista de 1 día con swipe + footer con tabs

### Características Mobile
- **Swipe navegación**: Scroll horizontal nativo con CSS `scroll-snap-type: x mandatory`. Permite ver parcialmente el día siguiente/anterior mientras se arrastra, con snap al soltar.
- **Footer con 3 secciones**:
  - Izquierda: Selector de mes (solo mes, sin año) + botón "Hoy" (si no es hoy)
  - Centro: Tabs 📅/📁 centrados respecto a la ventana (usando CSS grid)
  - Derecha: Botón "+" primary para agregar items
- **BottomSheet a pantalla completa**: Editor de items que ocupa toda la pantalla (reemplaza el bottom sheet parcial)
- **Botón + según vista**: En calendario crea item en morning del día actual, en categorías crea en backlog
- **Header de categorías**: Solo visible en vista categorías, con "Limpiar hechos" y botón "+" para nueva categoría

### Interacción Touch en Items (Mobile)
- **Long-press para drag**: Los items requieren mantener presionado ~200ms antes de poder arrastrarlos. Esto evita que un swipe rápido sobre un item arrastre el item en vez de hacer scroll del día.
- **Vibración feedback**: Al activarse el drag después del long-press, el dispositivo vibra brevemente (si soporta `navigator.vibrate`).
- **Click en touchend**: El tap en items se activa al soltar (touchend), no al tocar. Si hay movimiento durante el touch, se cancela el click (es un swipe).
- **touch-action: pan-x**: Los items permiten scroll horizontal nativo mientras se tocan.
- **Prevención de interacción post-cierre**: `lastPopoverCloseTime` evita que al cerrar el BottomSheet se active un item debajo

### Alineamiento Visual Mobile
- Todos los elementos están alineados a 12px del borde izquierdo:
  - Header del día (Lun 2)
  - Labels de slots (Mañana, Tarde, Noche)
  - Items dentro de slots
  - Sección Hecho

### Componentes Mobile
- `MobileDayColumn`: Renderiza un día completo (header con padding 14px + slots + done)
- `BottomSheet`: Editor de items a pantalla completa con botones: 🗑 (si editando), ✕ (cancelar), Guardar/Agregar
- `MobileFooter`: Barra inferior con selector de mes, tabs y botón agregar

### Estados Mobile
- `isMobile`: Detecta viewport ≤600px
- `mobileViewMode`: 'calendar' | 'categories'
- `currentMobileDay`: Día actualmente visible
- `bottomSheetItem`: Item siendo editado en BottomSheet
- `mobileScrollRef`: Ref para scroll programático

### CSS Mobile
- `.mobile-day-scroll`: Container horizontal con `scroll-snap-type: x mandatory` y `scroll-snap-stop: always`
- `.mobile-day-column`: Cada día ocupa 100% del ancho con `scroll-snap-align: start`
- `.mobile-footer`: Footer fijo con grid de 3 columnas (1fr auto 1fr) para centrar tabs
- `.mobile-tab-btn`: Botones de tabs con estilo similar a energía (activo=opacidad 100%, inactivo=35% + grayscale)
- `.bottom-sheet`: Panel a pantalla completa con flexbox column y safe-area-inset
- Slots en mobile tienen padding lateral 12px via CSS específico

### Tipografía Responsive
- Todos los `font-size` usan `rem` (no `px`) para escalar con el tamaño base
- Desktop: `html { font-size: 130%; }`
- Mobile (≤600px): `html { font-size: 150%; }` para mejor legibilidad en pantallas pequeñas
