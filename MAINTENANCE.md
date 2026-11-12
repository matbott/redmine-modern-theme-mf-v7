# Mantenimiento de este fork

Notas para retomar el trabajo sin tener que reconstruir el contexto.

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
javascripts/theme.js          upstream + 2 bloques marcados al final
README.md                     instalación y resumen
MAINTENANCE.md                este archivo
```

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
  - Dona: conteos por tipo desde `table.issue-report` del overview (sin request); por
    asignado desde `/projects/<id>/issues/report/assigned_to` (una request, cacheada).
    Colores en el array `PALETTE` (`--rm-*`).
  - Chips: un `total_count` por indicador vía `/issues.json?...&limit=1` (sin asignar,
    vencidas, sin fecha, nuevas/cerradas 7 d) y el cierre promedio sobre las últimas 30
    cerradas. Los links usan el formato de query de Redmine
    (`f[]=campo&op[campo]=operador&v[campo][]=valor`); si un filtro deja de existir, el
    chip muestra “—”.
  - Tamaño de la dona: `.vz-donut` en `custom.css`.
- **Taskboard**: sección *Taskboard / board* y *Taskboard compact*.
- **Plugins**: sección *Plugin compatibility*; usar `html.dark-mode` cuando el plugin pisa
  colores y `!important` solo para ganarle a CSS del plugin o a estilos inline.
- **Sidebar**: lado en el theme (My account) + ancho en nuestro bloque de `theme.js`.

## Pendientes / ideas

- Revisar el taskboard en móvil (el theme nuevo tiene su propio responsive).
- Si se agregan más plugins, sumar un bloque por plugin con su comentario.
- El hero podría mostrar más datos (próximamente).
