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
- **Imágenes**: avatares (generados con IA con la cara del usuario) de los
  ejercicios (`foto-*.jpg`, redimensionados a 760 px de alto y ~20 KB cada uno)
  e iconos (`icono*.png`, `apple-touch-icon.png`). Los **21 ejercicios** tienen
  foto con la cara del usuario. La lista de qué imagen usa cada ejercicio
  está en el objeto `FOTOS` de `index.html`; si un ejercicio no está en `FOTOS`,
  se dibuja con la figura SVG.
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

**9. Equipo disponible en el perfil**
En el perfil, además de altura/peso/condiciones, se elige **con qué equipo se cuenta**
(pesas o botellas de agua, banda elástica, bicicleta). Las rutinas **dejan fuera los
ejercicios que necesiten algo que no se tiene** (igual que con las condiciones médicas), y
en el catálogo/detalle se avisa con «🧰 Necesitas: …».
- Cada ejercicio pide su equipo (`EQUIPO_EJ`): el **remo** necesita **banda elástica**; el
  **curl** y el **press**, pesas o botellas de agua; la **bici**, bicicleta. Si marcas que
  no tienes banda, el remo desaparece de las rutinas (comportamiento predecible).
- En el perfil, los equipos que tienes se marcan en verde con un **✓**; lo que dejes sin
  marcar se quita de las rutinas.
- Al **guardar el perfil**, todas las vistas dependientes se refrescan al momento
  (catálogo, rutinas, nutrición y el detalle abierto): el cambio se nota sin reabrir la app.
- **Sustitución automática:** cuando un ejercicio se cae (por equipo o por una condición),
  la rutina **no se queda coja**: se sustituye por otro de la **misma categoría** y que
  comparta el máximo de músculos, si hay alguno disponible que no esté ya en la rutina. En
  la pantalla de explicación se avisa con «🔄 Sustituye a …». Solo se deja el hueco si no
  hay ningún ejercicio alternativo posible. Código: `buscarSustituto()` en `construirPasos()`.
- Si el perfil **aún no** tiene el equipo configurado, se asume que se tiene todo (para no
  vaciar las rutinas). En la pantalla de perfil, por eso, los equipos salen marcados por
  defecto: basta con **desmarcar** lo que no se tenga.
- Código: `EQUIPOS`, `EQUIPO_EJ`, `EQUIPO_NOMBRE`, `faltaEquipo()`, `noDisponible()`
  (combina condiciones + equipo), y el bloque de equipo en el perfil.

**10. Progreso motivador (pestaña Progreso)**
Para animar a seguir cada semana, la pestaña Progreso ahora muestra, además de las
estadísticas y los objetivos OMS:
- Una **cabecera con un mensaje de ánimo** que cambia según los días que llevas esta
  semana y tu racha.
- Una **gráfica de las últimas 8 semanas** (días que te moviste cada semana, la actual
  destacada) para ver la evolución.
- **Logros / medallas** desbloqueables (primera sesión, rachas, 10/25/50 días, «semana
  redonda» con los 4 pilares).
- Código: `pintarMotiva()`, `pintarGrafica()`, `pintarLogros()`, `LOGROS`, y auxiliares
  `lunesDe()`, `diasActivosEntre()`, `mejorRacha()`, `semanaCompleta()`.

**11. Guardar las sesiones en la app Salud del iPhone (vía Atajos)**
Al terminar una rutina, en un iPhone aparece el botón **«❤️ Guardar en Salud»**. Como la
app Salud (HealthKit) no es accesible desde una web, se usa un **puente con la app Atajos**:
el botón abre un atajo llamado **`Plenitud Salud`** (que el usuario crea una vez) y le pasa
los datos de la sesión (tipo de rutina, minutos y calorías estimadas) en JSON. El atajo
registra esa actividad en Salud (energía activa, que además suma al anillo de Movimiento).
- Las calorías son una **estimación**: `minutos × peso × 0,06` (el peso, del perfil; 70 kg
  por defecto). Los minutos son la duración prevista de la rutina.
- El botón solo se muestra en iPhone/iPad (`esIOS()`); en otros dispositivos no aparece.
- Código: `esIOS()`, `urlSalud()` y el botón en `pFinish()`.
- **Cómo crear el atajo `Plenitud Salud`** (una sola vez, en el iPhone):
  1. App **Atajos** → **+** → ponle de nombre exactamente **Plenitud Salud**.
  2. Actívalo para que reciba entrada (al recibir la orden desde la app, la «Entrada del
     atajo» será un texto JSON).
  3. Acción **«Obtener diccionario de la entrada»**.
  4. Acción **«Obtener valor del diccionario»** con la clave `kcal`.
  5. Acción **«Registrar muestra de salud»** → tipo **Energía activa** → valor: el resultado
     anterior → fecha: la actual.
  6. (Opcional) **«Mostrar notificación»**: «Sesión guardada en Salud».
  La primera vez, iOS pedirá permiso para escribir en Salud.

**12. Registro y evolución del peso**
En la pestaña **Progreso** hay una sección **«Mi peso»** para apuntar el peso cada cierto
tiempo y ver la evolución: peso actual, variación desde el primer registro (baja en verde,
sube en naranja) y una **gráfica de línea** con los últimos registros.
- Se guarda un registro por día (`plenitud60_pesos`); volver a apuntar el mismo día
  sustituye el valor. El último peso se sincroniza con el perfil, así la calculadora de
  proteína/IMC usa siempre el peso más reciente. Guardar el peso en el perfil también lo
  añade al histórico.
- Código: `getPesos()`, `addPeso()`, `pintarPeso()`, `graficaPeso()`, `registrarPeso()`.

---

## Datos guardados en el teléfono (localStorage)

| Clave | Contenido |
|-------|-----------|
| `plenitud60_historial` | Ejercicios realizados (con fecha). |
| `plenitud60_perfil` | Altura, peso, condiciones y equipo disponible. |
| `plenitud60_plan` | Rutina asignada a cada día de la semana. |
| `plenitud60_recuperados` | Días de esta semana ya recuperados. |
| `plenitud60_sesion` | Sesión de ejercicio en curso (para reanudar si se cierra la app). |
| `plenitud60_pesos` | Histórico de peso (para ver la evolución). |
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
