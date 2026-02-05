# Tudux - Plan de Produccion

Ultima actualizacion: Febrero 2026

## Progreso Actual

- **Fase 1: Research y Benchmark** - HECHO
  - Investigacion de apps existentes y definicion de features

- **Fase 2: Prototipo Estatico** - HECHO
  - Implementacion en `static.html` (React 18 + Babel standalone)
  - UI completa: Desktop (dias + categorias) y Mobile (1 dia con swipe)
  - Modelo de datos: items con recurrencia, excepciones, esfuerzo, categorias
  - Drag and drop, shimmer animations, popovers, dark mode
  - Safe-area para iPhone

- **Fase 3: Pruebas de UX** - HECHO
  - Testing de interacciones y debugging del prototipo

---

## Decisiones Tomadas

### Autenticacion - DECIDIDO

**Cloudflare Access**
- Login tipo "el browser pide user y pass" antes de ver la app
- Email OTP (codigo por email) - sin crear cuentas
- Gratis hasta 50 usuarios
- Configuracion en 5 minutos via dashboard
- Protege la app ANTES de que llegue al servidor

### Base de Datos - DECIDIDO

**IndexedDB (local) + Supabase (cloud)**

Arquitectura offline-first con sync a la nube:

```
IndexedDB (offline) <-> Supabase (cloud sync) -> Google Calendar (export opcional, futuro)
```

- **IndexedDB:** Persistencia local con Dexie.js
- **Supabase:** PostgreSQL gratuito, sync en tiempo real
- **Offline:** Cola de operaciones pendientes, sync al reconectar

### Por que NO Google Calendar como DB

- Modelo de datos incompatible (no soporta `sortOrder`, `exceptions`, `hiddenFromSidebar`)
- Si crean evento en Calendar a las 10am, apareceria en "Importante" aunque no sea tarea
- Requiere OAuth completo (mas complejo)
- Rate limits de API

### PWA - DECIDIDO

**Si, con soporte iOS**
- Agregar a Home Screen funciona desde iOS 11.3+
- Pantalla completa (sin barra Safari)
- Splash screen e icono custom
- **Limitacion iOS:** No soporta Push Notifications en PWAs

### Hosting - DECIDIDO

**Cloudflare Pages**
- Deploy automatico desde GitHub
- Integracion nativa con Cloudflare Access
- CDN global gratuito

### Google Calendar - FUTURO

Export opcional dejado para despues de tener la app funcionando con persistencia propia.

---

## Preguntas Pendientes

### Notificaciones - POR DECIDIR

**Contexto:** La app de Google Calendar no muestra bien las notificaciones actualizadas, hay que entrar a la app para ver lo correcto.

**Opciones:**
- **Push Notifications nativas:** Funciona en desktop (Chrome/Firefox/macOS). En iOS PWA NO funciona.
- **Telegram Bot:** Mensajes que llegan bien, funciona en todos los dispositivos. Requiere crear un bot y que el usuario lo agregue.
- **Ambos:** Push donde funcione + Telegram como fallback.

### Integracion Apple Health/Sleep - POR DECIDIR

**Ideas:**
- Energia del dia automatica segun horas dormidas
- Icono cuando se completaron calorias/ejercicio del dia

**Realidad tecnica:**
- Apple Health no tiene API web - solo apps nativas iOS pueden acceder
- **Opcion A:** Shortcut de iOS que lee Health y envia datos a Supabase
- **Opcion B:** App nativa minima que sincronice (mas complejo)
- **Opcion C:** Usar servicio intermediario (ej: Apple Health a Strava a webhook)

### Eventos de Gmail/Calendar - POR DECIDIR

**Ideas:**
- Mostrar eventos de Google Calendar como items read-only
- Popover mostraria: participantes, link de Google Meet si hay
- Estos eventos tendrian horario exacto (a diferencia de tareas)

**Preguntas abiertas:**
- Deberia poder asignar horario a cualquier tarea si quiero?
- O resolverlo poniendo horario en el titulo? Ej: "10:35 Reunion trabajo"

**Consideraciones:**
- Si agregamos horarios opcionales, el modelo de datos cambiaria
- El mapeo estados a franjas horarias se complicaria con eventos con hora especifica
- Opcion simple: horario en titulo, sin cambios al modelo

---

## Fases de Implementacion Pendientes

4. **Build System + Persistencia Local**
   - Crear proyecto Vite con React
   - Migrar codigo de `static.html` a componentes modulares
   - Implementar IndexedDB (Dexie.js) para items, categorias, energia
   - Hostear fonts localmente (IBM Plex Sans)

5. **PWA**
   - Crear `manifest.json`
   - Generar iconos (192px, 512px, apple-touch-icon 180px)
   - Service worker con cache de assets
   - Meta tags para iOS

6. **Deploy con Auth**
   - Push a GitHub
   - Conectar repo a Cloudflare Pages
   - Configurar Cloudflare Access (email permitido)

7. **Sync Cloud + Offline**
   - Configurar Supabase (PostgreSQL gratuito)
   - Crear tablas: `items`, `categories`, `day_energy`
   - Implementar sync bidireccional
   - Cola de operaciones offline
   - Resolucion de conflictos (Last Write Wins)

---

## Estructura de Proyecto Propuesta

```
tudux/
  public/
    manifest.json
    icons/
      icon-192.png
      icon-512.png
      apple-touch-icon.png
    fonts/
      IBMPlexSans-*.woff2
  src/
    App.jsx
    main.jsx
    components/
      ItemCard.jsx
      SlotSection.jsx
      CategorySection.jsx
      mobile/
        BottomSheet.jsx
        MobileFooter.jsx
    hooks/
      useLocalStorage.js
      useOfflineSync.js
    lib/
      db.js          # IndexedDB con Dexie
      sync.js        # Logica de sync
    utils/
      dates.js
      recurring.js
  index.html
  package.json
  vite.config.js
  sw.js              # Service Worker
```

---

## Archivos Criticos

| Archivo | Proposito |
|---------|-----------|
| `static.html` | Codigo fuente completo (prototipo actual) |
| `context.md` | Documentacion del modelo de datos y UI |
| `planning.md` | Plan de produccion y decisiones |
| `src/lib/db.js` | IndexedDB con Dexie (a crear) |
| `src/hooks/useOfflineSync.js` | Cola de operaciones offline (a crear) |
| `manifest.json` | Configuracion PWA (a crear) |

---

## Verificacion

1. **Local:** `npm run dev` - app funciona, datos persisten al refrescar
2. **PWA iPhone:** Agregar a home, abrir offline, verificar que carga
3. **Auth:** Acceder a URL, verificar que pide login
4. **Offline sync:** Crear item sin conexion, reconectar, verificar que sincroniza
