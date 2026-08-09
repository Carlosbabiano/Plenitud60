# Plenitud 60+ — Documentación de la app

Aplicación web (una sola página, `index.html`) de ejercicio y nutrición pensada para
personas mayores de 60 años. Funciona como PWA: se puede instalar en el móvil y se usa
sin conexión. Todos los datos se guardan solo en el teléfono del usuario
(`localStorage`), no hay servidor.

Este documento recoge **qué hace la app y todos los cambios que vamos haciendo**.
Se actualiza en cada cambio.

---

## Cómo está construida

- **Un único fichero**: `index.html` contiene el HTML, el CSS (dentro de `<style>`) y
  todo el JavaScript (dentro de `<script>`). No hay dependencias externas.
- **Imágenes**: fotos reales de algunos ejercicios (`foto-*.png`) e iconos
  (`icono*.png`, `apple-touch-icon.png`). El resto de ejercicios usan figuras
  dibujadas con SVG dentro del propio código.
- **`manifest.json`**: configuración de la PWA (nombre, iconos, color).

### Secciones de la app (pestañas inferiores)
1. **Rutinas** — rutina de hoy según el plan semanal, rutinas pendientes y lista de
   todas las rutinas.
2. **Ejercicios** — catálogo de ejercicios por categoría (Fuerza, Cardio, Equilibrio,
   Flexibilidad) con su ficha de detalle.
3. **Progreso** — sesiones, racha, objetivos OMS e historial.
4. **Nutrición** — guía vegetariana y calculadora de proteína/IMC.
5. **Perfil** — altura, peso, molestias/condiciones y recordatorios en el calendario.

### Datos principales (en el JavaScript de `index.html`)
- `EJERCICIOS` — lista de ejercicios con sus pasos, dosis, contraindicaciones, etc.
- `ROUTINES` — rutinas (cada una es una lista de ejercicios con su duración).
- `PLAN_DEFAULT` — plan semanal por defecto (lunes a domingo).
- `EJ_TIEMPO` — conjunto de ejercicios que se cronometran (ver más abajo).
- `REPS_INFO` — series y repeticiones de cada ejercicio de repeticiones.

---

## Registro de cambios

### 2026-08-09

**1. Pantalla de explicación antes de cada ejercicio**
Antes de empezar cualquier ejercicio (tanto en una rutina como al practicar uno suelto)
aparece una pantalla que lo explica: dibujo, categoría, dosis (🎯) y los pasos
numerados, con un botón grande **▶ Empezar**. Es a tu ritmo (no corre ningún contador
mientras lees) y sustituye al antiguo descanso automático entre ejercicios.
- Código: funciones `pRenderIntro()`, `pStart()`, `irAPaso()` en el reproductor.

**2. El contador de tiempo solo aparece en los ejercicios de tiempo**
- Ejercicios **de tiempo** (caminar, marcha, escaleras, bici, apoyo a una pierna y los
  estiramientos de cuello/piernas/pantorrilla): al empezar sale la **cuenta atrás** con
  botón de Pausar.
- Ejercicios **de repeticiones** (sentadillas, flexiones, curl, etc.): **no** hay
  contador de tiempo.
- La clasificación está en el conjunto `EJ_TIEMPO`. Para cambiar si un ejercicio es de
  tiempo o de repeticiones, se añade o se quita su id de ese conjunto (una sola línea).

**3. Contador de series en los ejercicios de repeticiones**
En los ejercicios de repeticiones con más de una serie, la pantalla muestra las
repeticiones por serie (p. ej. «10–12 repeticiones»), **«Serie X de Y»** y unos puntos
que se van rellenando. Se pulsa **«Serie hecha ✓»** al acabar cada serie; en la última
el botón pasa a **«Terminar ejercicio ✓»** y avanza al siguiente ejercicio.
- Las series y repeticiones de cada ejercicio están en `REPS_INFO`. Para ajustar cuántas
  series tiene un ejercicio, se cambia su número ahí.
- Los ejercicios con una sola serie (p. ej. movilidad de hombros) siguen mostrando
  solo el botón «Hecho ✓», sin contador de series.
- Código: `REPS_INFO`, `seriesDe()`, `repsDe()`, `serieHecha()`, `pRenderRun()`.

**4. Recuperar rutinas pendientes en otro día**
Si algún día de esta semana no se hizo su rutina, en la pantalla de **Rutinas** aparece
un aviso **«⏰ Rutinas pendientes de esta semana»** con cada rutina que quedó suelta y un
botón **«Hacer ahora»** para hacerla ese mismo día. Al hacerla, deja de aparecer como
pendiente. El aviso se reinicia cada semana (lunes).
- Los días recuperados se guardan por semana en `localStorage`
  (`plenitud60_recuperados`).
- Código: `pendientesSemana()`, `hacerPendiente()`, `getRecuperados()` /
  `marcarRecuperado()` / `estaRecuperado()`, y el bloque `#pendientesCard` en
  `pintarRutinas()`.
- Nota: además, siempre se puede lanzar cualquier rutina cualquier día desde
  «Todas las rutinas», y cambiar la rutina de un día tocándolo en el plan semanal.

---

## Datos guardados en el teléfono (localStorage)

| Clave | Contenido |
|-------|-----------|
| `plenitud60_historial` | Ejercicios realizados (con fecha). |
| `plenitud60_perfil` | Altura, peso y condiciones. |
| `plenitud60_plan` | Rutina asignada a cada día de la semana. |
| `plenitud60_recuperados` | Días de esta semana ya recuperados. |
| `plenitud60_voz` | Si la voz que anuncia los ejercicios está activada. |

---

## Despliegue (GitHub → Netlify)

- **Repositorio de GitHub:** https://github.com/Carlosbabiano/Plenitud60
- **Hosting:** Netlify (despliega automáticamente cada vez que se sube un cambio al
  repositorio).

### Cómo subir cambios nuevos
Desde la carpeta del proyecto, tras editar los ficheros:

```bash
git add -A
git commit -m "Descripción del cambio"
git push
```

Al hacer `push`, Netlify detecta el cambio y vuelve a publicar la web en unos segundos.
La primera vez, GitHub pide iniciar sesión a través de Git Credential Manager; después
queda recordado.

### Primera conexión con Netlify (solo una vez)
En Netlify: **Add new site → Import an existing project → GitHub →** elegir el
repositorio `Plenitud60`. Dejar el directorio de publicación en la raíz (no hay que
compilar nada, es HTML plano).
