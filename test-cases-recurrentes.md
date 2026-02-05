# Casos de prueba para Items Recurrentes

## Setup inicial
- REAL_TODAY = 2026-02-06 (Viernes)
- Items de ejemplo:
  - Gym abajo: Lunes y Jueves (days: [1, 4])
  - Gym arriba: Martes y Viernes (days: [2, 5])
  - Boxeo switch: Miércoles (days: [3])
  - Practicar teclado: Cada 3 días desde 2026-02-02

---

## 1. Mover instancia recurrente a otro día

### 1.1 "Solo este evento" - Mover a día futuro válido
- [ ] Arrastrar "Gym abajo" del Jueves 5 al Viernes 6
- [ ] Elegir "Solo este evento"
- **Esperado**: Item aparece en Viernes 6 (ya no es recurrente, texto negro)
- **Verificar**: El Lunes 9 sigue teniendo "Gym abajo" recurrente

### 1.2 "Solo este evento" - Mover instancia overdue
- [ ] Arrastrar "Gym abajo" del Jueves 5 (overdue, rojo) al Viernes 6
- [ ] Elegir "Solo este evento"
- **Esperado**: Item aparece en Viernes 6 como normal
- **Verificar**: No desaparece

### 1.3 "Este y los siguientes"
- [ ] Arrastrar "Gym abajo" del Lunes 9 al Martes 10
- [ ] Elegir "Este y los siguientes"
- [ ] Verificar que puedo modificar los días de la semana
- **Esperado**: Nueva serie empieza el Martes 10

### 1.4 "Todos los eventos"
- [ ] Arrastrar "Gym abajo" del Lunes 9 al Martes 10
- [ ] Elegir "Todos los eventos"
- **Esperado**: Toda la serie se desplaza (ahora es Martes y Viernes)

---

## 2. Restricciones de rango (modelo Outlook)

### 2.1 No puede saltar instancia siguiente
- [ ] "Gym arriba" repite Mar y Vie
- [ ] Intentar mover instancia del Viernes 6 al Miércoles 11 (después del Martes 10)
- **Esperado**: Toast "No se puede mover después de la siguiente instancia"

### 2.2 No puede mover al pasado
- [ ] Intentar arrastrar cualquier recurrente a un día pasado (ej: Jueves 5)
- **Esperado**: Día pasado está muteado (opacity 0.3), no acepta drop

### 2.3 Puede mover dentro del rango
- [ ] "Gym arriba" del Viernes 6, mover al Sábado 7 o Domingo 8
- **Esperado**: Funciona (está entre Vie 6 y Mar 10)

---

## 3. Crear recurrente weekdays en día inválido

### 3.1 Crear en día fuera del patrón
- [ ] Crear nuevo item el Viernes 6
- [ ] Elegir recurrencia "Días de semana" con Lunes y Miércoles seleccionados
- [ ] Guardar
- **Esperado**: Item aparece el Lunes 9 (no el Viernes 6)
- **Esperado**: Toast "Programado para Lun 9 Feb (primer día del patrón)"

---

## 4. Mover a día fuera del patrón weekdays

### 4.1 Pre-agregar día al patrón
- [ ] "Gym abajo" repite Lun y Jue (days: [1, 4])
- [ ] Arrastrar instancia del Lunes 9 al Miércoles 11
- [ ] Elegir "Este y los siguientes"
- **Esperado**: Mensaje amarillo "Miércoles no está en tu patrón. Se agregará."
- **Esperado**: Miércoles ya está seleccionado en el selector de días
- **Esperado**: Botón "Aplicar" está habilitado

---

## 5. Modo fixed vs completion (cada X días)

### 5.1 Fixed mode
- [ ] Crear item "Tarea cada 3 días" con modo "Desde fecha fija"
- [ ] Completar la instancia 2 días tarde
- **Esperado**: Siguiente instancia sigue el calendario original

### 5.2 Completion mode
- [ ] Crear item "Tarea cada 3 días" con modo "Desde última completación"
- [ ] Completar la instancia 2 días tarde (ej: debía ser Lunes, completo Miércoles)
- **Esperado**: Siguiente instancia es 3 días desde Miércoles (Sábado)

---

## 6. Visual de overdue

### 6.1 Items pasados sin completar
- [ ] Ver instancias recurrentes en días pasados
- **Esperado**: Texto rojo, ícono de recurrencia rojo

### 6.2 Items pasados completados
- [ ] Completar una instancia recurrente en día pasado
- **Esperado**: Texto gris (muted), ícono azul (no rojo)

---

## 7. Edición desde el editor (no drag)

### 7.1 Cambiar propiedades - Solo este evento
- [ ] Click en instancia recurrente para abrir editor
- [ ] Cambiar título
- [ ] Guardar
- [ ] Elegir "Solo este evento"
- **Esperado**: Solo esa instancia cambia, las demás mantienen título original

### 7.2 Cambiar propiedades - Este y los siguientes
- [ ] Click en instancia recurrente
- [ ] Cambiar título
- [ ] Guardar → "Este y los siguientes"
- **Esperado**: Nueva serie con nuevo título desde esa fecha

---

## 8. Tooltips

### 8.1 Tooltip en ícono weekdays
- [ ] Hover sobre ícono de recurrencia de "Gym abajo"
- **Esperado**: "Lun, Jue"

### 8.2 Tooltip en ícono cada X días
- [ ] Hover sobre ícono de "Practicar teclado"
- **Esperado**: "Cada 3 días" o "Cada 3 días (desde completación)"

---

## Bugs conocidos a investigar

1. **Item desaparece al mover "Solo este evento"**
   - Reproducir: Mover instancia overdue a otro día
   - Ver consola para logs de debug
