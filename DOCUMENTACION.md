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

**5. La pantalla no se apaga durante los ejercicios (Wake Lock)**
Mientras el reproductor está abierto (haciendo una rutina o un ejercicio), la app pide un
bloqueo de pantalla para que el móvil **no se apague solo** sin tener que tocarlo. Se
suelta al cerrar el reproductor, para no gastar batería el resto del tiempo. Si el móvil
se bloquea y se vuelve a abrir, el bloqueo se recupera automáticamente.
- Usa la Screen Wake Lock API (`navigator.wakeLock`). Soportada en Safari de iPhone
  (iOS 16.4+) y en Android. Requiere HTTPS (Netlify lo cumple).
- Código: `wakeLock`, `pedirWakeLock()`, `soltarWakeLock()`, el listener de
  `visibilitychange`, y las llamadas en `arrancarPlayer()` / `pClose()`.

**6. Ejercicios de tiempo por los dos lados**
Los ejercicios de tiempo que se hacen «por pierna» o «por lado» (apoyo a una pierna,
estiramiento de piernas, de pantorrilla y de cuello) ahora **cronometran cada lado**: el
contador corre para «Pierna derecha», y al terminar pasa a «Pierna izquierda» antes de
avanzar al siguiente ejercicio. Antes solo daba tiempo para un lado.
- Qué ejercicios tienen dos lados se define en el objeto `LADOS`.
- Código: `ladosDe()`, `avanzarTiempo()`, y la rama de tiempo de `pRenderRun()`.

**7. Botón «Anterior» en el reproductor**
Durante una rutina se puede **volver al ejercicio anterior** (por si te lo saltaste o
quieres repetirlo). Aparece a partir del segundo ejercicio.
- Código: `pAnterior()` y el botón en `pRenderIntro()` / `pRenderRun()`.

**8. Reanudar la sesión si se cierra la app**
Mientras haces una rutina, la app **guarda continuamente** dónde estás (ejercicio, fase,
contador, lado, serie, ejercicios ya hechos). Si la app se cierra por cualquier motivo
(se bloquea el móvil, se cierra Safari, un aviso…), al volver a abrirla aparece en la
pantalla de Rutinas un aviso **«▶ Sesión sin terminar»** con botones **Reanudar** y
**Descartar**. Reanudar reabre el reproductor justo donde se quedó.
- La sesión se guarda en `localStorage` (`plenitud60_sesion`). Se borra al **completar**
  la rutina o al **cerrarla a propósito** con la ✕ (así el aviso solo aparece tras cierres
  inesperados).
- Código: `guardarSesion()`, `cargarSesion()`, `borrarSesion()`, `reanudarSesion()`,
  `descartarSesion()`, y el bloque `#reanudarCard` en `pintarRutinas()`.

---

## Datos guardados en el teléfono (localStorage)

| Clave | Contenido |
|-------|-----------|
| `plenitud60_historial` | Ejercicios realizados (con fecha). |
| `plenitud60_perfil` | Altura, peso y condiciones. |
| `plenitud60_plan` | Rutina asignada a cada día de la semana. |
| `plenitud60_recuperados` | Días de esta semana ya recuperados. |
| `plenitud60_sesion` | Sesión de ejercicio en curso (para reanudar si se cierra la app). |
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
