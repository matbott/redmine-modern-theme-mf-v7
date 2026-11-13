# Mantenimiento de este fork

Notas para retomar el trabajo sin tener que reconstruir el contexto.

## Estado actual (2026-11-13)

- Repo: `matbott/redmine-modern-theme-mf-v7`, rama `main`.
- Último commit funcional: `26c9dfa`, tag **`v1.2.3`** (calendario dark: pisa el CSS
  del plugin RedmineUP con `!important`). El commit de esta documentación va después.
- Server: Redmine **7.0.3**, container `redmine-redmine-1`, theme instalado en
  `/usr/src/redmine/themes/redmine-modern-theme-mf-v7`.
- **Pendiente inmediato**: deployar el último `main` en el server con el bloque de
  *Deploy* y hacer `Ctrl+F5`. El tag `v1.2.3` todavía puede no estar deployado.
- El theme anterior `redmine-modern-theme-mf` sigue instalado y se puede volver a él
  desde Administración → Configuración → Mostrar → Tema.

### Verificación pendiente (para mañana)

1. **Mi página** (`/my/page`): que las 6 tarjetas cuenten y que cada link abra el
   listado con resultados (usan el id del usuario, no `me`). Con Redmine Admin deben
   dar: vencidas 20, cerradas 7 días 8, estancadas 2 (verificado contra la API).
2. **Listado de incidencias**: agregar la columna **Fecha fin** (Opciones → Columnas) y
   comprobar rojo vencida, naranja ≤7 días, amarillo ≤30 días. Una fecha a +30 días no
   debe tener color ni barra. La barra roja izquierda debe aparecer **solo** en vencidas
   reales (no en `behind-schedule`).
3. **Vistazo**: columnas 50/50, dona debajo de Incidencias, panel de atención 4+4+4,
   panel **Personalizar** con los 16 checks (incluye los 3 de colores).
4. **Búsqueda del header**: al enfocar debe crecer hasta ~400 px (si solo llega a 220,
   el deploy no tomó la regla con `#header`). El buscador de proyectos y los campos
   autocompletar deben mostrar una sola lupa (sin mosaico).
5. **Mi cuenta**: al final de la sección *Sidebar position* debe verse
   `Modern Theme MF v1.2.3` en chico. Ya **no** debe aparecer en el pie del sidebar.
6. **Listado de incidencias**: en las columnas **Asignado** y **Autor** debe aparecer
   un círculo con iniciales antes del nombre. El color tiene que coincidir con el
   avatar del menú de cuenta (misma clase `avatar-color-<id % 16>` de Redmine).
7. **Listado de incidencias**: tarjetas de abiertas por tipo arriba de los filtros y
   link “Total abiertas”.
8. **Calendario** (`/projects/<id>/issues/calendar`) en modo oscuro: las celdas del mes,
   las de otros meses, la cabecera y los bordes deben usar superficies oscuras del
   theme, no blanco/gris claro.

### Decisiones vigentes

- La configuración del Vistazo es **por proyecto y por navegador**:
  `localStorage['rm-vz-stats:<ident>']`. No hay backend ni configuración compartida.
  Para compartirla habría que usar un `ProjectCustomField` o un plugin.
- Los datos de las tarjetas del listado son una **foto fija** de abiertas por tipo; no
  siguen los filtros de la página (decisión tomada).
- “Mi vistazo” no tiene toggles propios; el umbral de estancadas está fijo en 7 días.
- Los colores de vencimiento solo corren en listados de **proyecto** y necesitan la
  columna Fecha fin visible.

### Versionado

- La versión visible está en `javascripts/theme.js`, bloque `LOCAL CUSTOMISATIONS —
  sidebar width` (`var VERSION = '1.2.3'`).
- Al publicar cambios: subir `VERSION`, commit, `git tag -a vX.Y.Z` y push del tag.
- Se muestra al final del fieldset `#rm-layout` de **Mi cuenta** (sidebar position /
  width), como `Modern Theme MF vX.Y.Z`.

### Backlog acordado / ideas

- Mi página: toggles por usuario, dona o mini-gráficos, horas de la semana, umbral de
  estancadas configurable.
- Colores de vencimiento en `/issues` global y en los listados de Mi página; banda de
  60/90 días.
- Vistazo global en Home (si algún día se retoma, además de Mi página).
- Grafana: sincronizar `theme=dark|light` y `kiosk` en los iframes, envolverlos con
  tokens del theme y definir altura por dashboard. Ver la sección de ideas al final.
- Navegación: Ctrl+K, proyectos favoritos, chips de filtros rápidos, cabecera sticky,
  barra de acciones al seleccionar filas, detalle de incidencia con barra sticky.
- Revisar taskboard en móvil.

## Origen

- Base: [`acosonic/redmine-modern-theme`](https://github.com/acosonic/redmine-modern-theme),
  rama `redmine-7`, commit `2685821` (MIT).
- Este repo es **independiente** del theme anterior `redmine-modern-theme-mf` (linaje
  Redmine 6 con parches). Los dos pueden estar instalados a la vez en `themes/`.
- Pensado para **Redmine 7.0.3** (verificado en el server de producción).
- El theme no usa `!important` en los archivos del autor: el core va en `@layer(core)` y
  todo lo que está fuera de la capa gana. Nuestro `custom.css` es unlayered.

## Estructura

```
stylesheets/application.css   upstream — solo se agregó @import url(custom.css)
stylesheets/schemes.css       upstream
stylesheets/custom.css        TODO lo local (≈800 líneas, con secciones comentadas)
javascripts/theme.js          upstream + bloques locales (Vistazo, sidebar width,
                              tarjetas del listado, Mi página)
README.md                     instalación y resumen
MAINTENANCE.md                este archivo (empezar por “Estado actual”)
```

Archivos clave para retomar: `stylesheets/custom.css` (todo lo visual),
`javascripts/theme.js` (bloques al final + `init` del Vistazo), esta guía, y el bloque
de *Deploy* más abajo.

Regla: **no editar los archivos del autor** salvo el `@import`. Así se puede rebasear
contra upstream sin conflictos.

## Mapa de tokens

El theme nuevo no tiene los tokens del anterior (`--c-*`) sino `--rm-*`; además deriva la
paleta de core (`--oc-*`) con `color-mix()`, y `light-dark()` resuelve los modos.

| Viejo (`mf`)        | Nuevo (`-v7`)            |
| ------------------- | ------------------------ |
| `--c-bg`            | `--rm-bg`                |
| `--c-surface`       | `--rm-surface`           |
| `--c-surface-2`     | `--rm-surface-2`         |
| `--c-border`        | `--rm-border`            |
| `--c-border-2`      | `--rm-border-2`          |
| `--c-text`          | `--rm-fg`                |
| `--c-text-muted`    | `--rm-muted`             |
| `--c-text-subtle`   | `--rm-muted`             |
| `--c-primary`       | `--rm-accent`            |
| `--c-primary-hover` | `--rm-accent-hover`      |
| `--c-primary-light` | `--rm-accent-soft`       |
| `--c-primary-ring`  | `--rm-ring`              |
| `--c-focus-ring`    | `--rm-ring`              |
| `--c-success`       | `--rm-green`             |
| `--c-warning`       | `--rm-yellow`            |
| `--c-danger`        | `--rm-red`               |
| `--c-info`          | `--rm-blue`              |
| `--c-on-primary`    | `--rm-on-accent`         |
| `--c-topbar-bg`     | `--rm-topbar`            |
| `--c-topbar-text`   | `--rm-topbar-fg`         |
| `--c-header-bg`     | `--rm-surface`           |
| `--c-sidebar-bg`    | `--rm-surface`           |
| `--r-sm` / `--r` / `--r-md` | `--rm-r-sm` / `--rm-r` / `--rm-r-md` |
| `--shadow-*`        | `--rm-shadow-*`          |
| `--t-fast` / `--t`  | `--rm-t`                 |

`custom.css` define aliases locales (`--sp-*`, `--text-*`, `--r*`, `--shadow-*`,
`--t-fast`) para que el código portado siga legible. Si upstream renombra un `--rm-*`,
se cambia **solo el alias**.

### Orden de carga (importante)

`@import` va antes que el resto de `application.css`, o sea que `custom.css` se evalúa
**antes** que las reglas del autor. Para pisar una regla del autor de igual especificidad
hace falta más especificidad o `!important` (documentado en cada caso). Contra el core no
hay problema: el core está en una capa.

## Qué se portó

| Feature | Dónde |
| --- | --- |
| Filas seleccionadas (`:has`) | `custom.css` — Issue list |
| Ancho de sidebar (180–360 px) | `theme.js` — bloque final + `custom.css` (slider) |
| Admin index en tarjetas | `custom.css` — Admin index |
| Hero + KPIs + dona + chips de operación (“Vistazo moderno”) | `custom.css` — Project overview + bloque final de `theme.js` |
| Plugins RedmineUP Agile / Testcase / Vault | `custom.css` — Plugin compatibility |
| Pins toolbar | `custom.css` — final |
| View Customize — bloques `<pre>` | `custom.css` — Plugin compatibility |
| Taskboard (`#rdb*`, `.rdb-*`) | `custom.css` — Taskboard / board |
| Lista compacta del taskboard (`.rdb-compact`) | `custom.css` — Taskboard compact |
| RedmineUP Checklists (campo nuevo ítem) | `custom.css` — Checklists |

El plugin `redmine_task_weight` (repo aparte) usa `--rm-*` con fallback a `--c-*`, así
funciona con los dos themes.

## Qué NO se portó (y por qué)

En Redmine 7 el core se adapta solo vía `--oc-*`/`light-dark()`, así que estos parches del
theme anterior ya no hacen falta:

- Overrides de dark mode del core 6 (sección enorme del `mf`).
- Hover de filas de `table.list` (upstream lo hace con `--rm-accent-soft`).
- Separador de día en Actividad, fix del icono en flash, padding compacto del sidebar,
  barras de progreso `div`: **se sacaron** después de probar el theme nuevo, ya estaban
  resueltos.
- Scrollbar, botón “new object”, layout de Mi cuenta, clear-query del sidebar, inputs de
  Actividad, drag de My page: los resuelve el core 7 o el propio theme.

También se descartaron por específicos del theme viejo: `.contextual { clear: both }`,
`#main.nosidebar #content`, la corrección del ancho del botón “new object”, y la sección
de overrides por esquema (`html[data-scheme]`), que el nuevo resuelve con sus tokens.

## Actualizar desde upstream

```bash
git remote add upstream https://github.com/acosonic/redmine-modern-theme.git
git fetch upstream redmine-7
git merge upstream/redmine-7     # o: git rebase upstream/redmine-7
```

Conflictos esperados: **solo** `stylesheets/application.css` (la línea del `@import`) y
`javascripts/theme.js` (los bloques al final). `custom.css` no debería chocar.

Después de actualizar:

1. Revisar que sigan existiendo los `--rm-*` usados (ver mapa arriba).
2. Probar en el server (deploy abajo) y mirar: overview de proyecto, lista de issues
   (hover/selección), admin, taskboard (normal y compacta), Pins, Distribución de Carga.
3. Si upstream agrega estilos para un plugin, quitar el bloque duplicado de `custom.css`.

## Deploy

```bash
docker exec -it redmine-redmine-1 /bin/bash << 'EOF'
set -e
cd /usr/src/redmine/themes
if [ -d redmine-modern-theme-mf-v7/.git ]; then
  cd redmine-modern-theme-mf-v7
  git fetch origin
  git reset --hard origin/main
else
  rm -rf redmine-modern-theme-mf-v7
  git clone https://github.com/matbott/redmine-modern-theme-mf-v7.git
fi

# Plugin task_weight: si está como repo se actualiza; si se copió a mano,
# se baja solo la vista nueva.
VIEW=$(find /usr/src/redmine/plugins -path '*/app/views/task_weights/index.html.erb' 2>/dev/null | head -1)
if [ -n "$VIEW" ]; then
  PLUGIN_DIR=$(dirname "$(dirname "$(dirname "$VIEW")")")
  if [ -d "$PLUGIN_DIR/.git" ]; then
    git -C "$PLUGIN_DIR" fetch origin && git -C "$PLUGIN_DIR" reset --hard origin/main
  else
    curl -fsSL https://raw.githubusercontent.com/matbott/redmine_task_weight/main/app/views/task_weights/index.html.erb -o "$VIEW"
  fi
fi

cd /usr/src/redmine
rm -rf public/assets/*
RAILS_ENV=production bundle exec rake tmp:cache:clear
RAILS_ENV=production bundle exec rake assets:precompile
exit
EOF
docker restart redmine-redmine-1
```

El theme se elige en **Administración → Configuración → Mostrar → Tema →
`redmine-modern-theme-mf-v7`**. El anterior queda instalado para volver con un clic.
`Ctrl+F5` después de cambiar.

## Dónde tocar cada cosa

- **Colores de todo**: tokens en `application.css` §1, o `schemes.css` para esquemas.
  No agregar colores fijos en `custom.css`.
- **Hero/KPIs/dona/chips**: markup y fetch en el bloque final de `theme.js`; estilos en la
  sección *Project overview* de `custom.css`.
  - Dona: se inserta en la columna izquierda, justo debajo de la caja *Incidencias*
    (`.splitcontentleft .issues.box`). Conteos por tipo desde `table.issue-report` del
    overview (sin request); por asignado desde `/projects/<id>/issues/report/assigned_to`
    (una request, cacheada). Colores en el array `PALETTE` (`--rm-*`).
  - Chips: un `total_count` por indicador vía `/issues.json?...&limit=1` (sin asignar,
    vencidas, sin fecha, nuevas/cerradas 7 d, estancadas) y el cierre promedio sobre las
    últimas 30 cerradas. Los links usan el formato de query de Redmine
    (`f[]=campo&op[campo]=operador&v[campo][]=valor`); si un filtro deja de existir, el
    chip muestra “—”.
  - Estancadas: abiertas con `updated_on <= hoy - N` (N configurable: 3/7/15/30/60).
  - **Panel de atención** (columna derecha): tres listas de 4 ítems cada una — vencidas,
    por vencer en 7 días y estancadas — con el total real en el título (vía
    `total_count`). Filtros API: `due_date<=ayer`, `due_date=><hoy|hoy+7` y
    `updated_on<=hoy-N`, ordenadas por fecha. El umbral de estancadas es el mismo que
    el del chip; al cambiarlo se recargan ambas cosas sin refrescar la página.
  - **Visibilidad por proyecto**: botón *Personalizar* en el hero. Cada indicador
    (KPI básicas, dona, cada chip, estancadas y cada lista del panel de atención) se
    puede prender/apagar; se guarda en `localStorage['rm-vz-stats:<ident>']`, o sea
    **por proyecto y por navegador**. No requiere server ni afecta a otros usuarios.
    Si todo el panel de atención queda apagado, se oculta la caja completa. Para un
    default distinto, editar `vzDefault` (bloque de configuración antes de los KPIs en
    `theme.js`); para un valor compartido habría que moverlo a un plugin/custom field.
  - **Colores de vencimiento** (listado de incidencias): toggles *dueLate*, *dueWeek* y
    *dueMonth* en el panel *Personalizar*. El bloque del listado pinta
    `td.due_date` con rojo (`--rm-red`), naranja (`--rm-red` + `--rm-yellow`) o amarillo
    (`--rm-yellow`) según falten 0-7 o 0-30 días. La columna Fecha fin tiene que estar
    visible en el listado. Las filas vencidas reciben `tr.vz-row-late` (barra roja a la
    izquierda): no usamos `behind-schedule` de Redmine porque ese flag se basa en el
    `done_ratio` y también marca incidencias que todavía no vencieron.
  - Tamaño de la dona: `.vz-donut` en `custom.css`.
- **Tarjetas de abiertas por tipo** (listado de incidencias): bloque final de `theme.js`
  (`issue list KPI cards`), sólo en `/projects/<id>/issues`. Lee
  `/projects/<id>/issues/report/tracker` (una request), ordena por abiertas y agrega una
  tarjeta de Total. Es una foto fija: no sigue los filtros de la página. Cada tarjeta
  linkea al listado filtrado por ese tipo y estado abierto. Se prende/apaga con
  *Tarjetas del listado* en el panel *Personalizar* del Vistazo.
- **Mi vistazo** (Mi página): bloque final de `theme.js` (`My page`). Inserta una fila de
  KPI arriba de los bloques de `/my/page` con datos del usuario logueado: asignadas
  abiertas, vencidas, vencen en 7 días, reportadas abiertas, cerradas en 7 días y
  estancadas +7 días. Resuelve el id del usuario desde el menú (`.flyout-menu__avatar
  a[href^="/users/"]`, fallback `a.user`) y lo usa en los links (`f[]` + `op[]=` +
  `v[]`) y en `/issues.json?assigned_to_id=<id>`; si no lo encuentra cae a `me`. No
  usar `op[assigned_to_id]=me`: el operador va vacío (`=`) y `me` es el valor.
- **Calendario**: sección *Calendar* al final de `custom.css`. El core pinta
  `ul.cal .calbody.this-month` con `--oc-white` y el resto con `--oc-gray-1`, valores
  fijos que no cambian en dark. Además, el CSS de RedmineUP se carga después del theme
  y repinta fines de semana (`#f1f1f1`), columna de semana (`#def`) y hoy (`#ffd`).
  Por eso estas reglas usan `!important`: es CSS de plugin, el caso permitido. Se usan
  `--rm-surface`, `--rm-surface-2`, `--rm-border` y `--rm-muted`; aplica a calendarios
  de proyecto y globales.
- **Avatares del listado**: bloque final de `theme.js` (`initials avatars in issue list
  user columns`). Antepone un `span` con iniciales a `td.assigned_to a.user` y
  `td.author a.user`. Reutiliza las clases nativas `avatar s16 avatar-color-<id % 16>`,
  así que el color es exactamente el que Redmine usa en el menú de cuenta; no hay
  requests extra. Iniciales: primera letra del primer y del último token del nombre;
  una sola palabra → sus dos primeras letras. El CSS solo ajusta margen y font-size.
- **Taskboard**: sección *Taskboard / board* y *Taskboard compact*.
- **Plugins**: sección *Plugin compatibility*; usar `html.dark-mode` cuando el plugin pisa
  colores y `!important` solo para ganarle a CSS del plugin o a estilos inline.
- **Sidebar**: lado en el theme (My account) + ancho en nuestro bloque de `theme.js`.

## Grafana (idea para retomar)

Hoy los iframes se inyectan con View Customize y con el CMS de RedmineUP. El iframe es
un documento aparte, así que el CSS del theme no puede tocar su fondo. Plan:

- Bloque en `theme.js` que detecte claro/oscuro y reescriba el `src` de los iframes de
  Grafana agregando `theme=dark|light`, `kiosk` y `refresh=...`.
- Envolver el iframe en un contenedor con tokens (`--rm-surface`, borde, radio, sombra).
- Altura por dashboard: Grafana no expone su alto por `postMessage`, así que conviene un
  mapa URL→alto o una configuración por tablero.
- En Grafana, paneles con fondo transparente + `kiosk` es lo que mejor se funde.

## Notas sueltas

- Revisar el taskboard en móvil (el theme nuevo tiene su propio responsive).
- Si se agregan más plugins, sumar un bloque por plugin con su comentario.
- El hero podría mostrar más datos (próximamente).
