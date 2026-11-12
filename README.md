# Modern Redmine Theme — Redmine 7 (MF)

A clean, modern UI theme for **Redmine 7** with dark mode, 27 colour schemes (plus any of
~1,250 Gogh palettes), a sidebar on the left or right, and design tokens.

This is a personal fork of
[`acosonic/redmine-modern-theme`](https://github.com/acosonic/redmine-modern-theme)
(branch `redmine-7`), maintained separately from the Redmine 6-based theme used before.
All local work lives in **`stylesheets/custom.css`** and one clearly marked block at the
end of `javascripts/theme.js`, so the upstream files stay untouched and easy to rebase.
See [Local customisations](#local-customisations).

## How it is built

Redmine 7 changed what a theme can lean on, and this branch is built around that:

- **Core's stylesheet sits in a cascade layer.** `application.css` starts with
  `@import url(/application.css) layer(core)`. Theme rules are unlayered, so they win over
  core whatever its specificity — the theme has **no `!important`** outside one short block
  that answers core's own `!important` rules. (The Redmine 6 branch needed 2,195 of them.)
- **Colours go through core's own variables.** Redmine 7 paints with the Open Color scale
  (`--oc-*`). The theme sets about a dozen inputs (`--rm-bg`, `--rm-fg`, `--rm-accent`, …)
  and re-derives that scale from them with `color-mix()`. Dark mode and every colour scheme
  are just different inputs; core, plugins that use `--oc-*`, and the theme all follow.
- **Core's own components are used, not replaced.** The account menu, the mobile flyout
  menu, the sidebar collapse button and the Gantt chart are Redmine 7's; the theme styles
  them. `theme.js` only adds what core has not got.

## Status

Beta. The core pages are done and contrast-checked (WCAG AA) in light, dark and all 27
schemes. On this fork the plugins used on our instance (RedmineUP Agile, Redmineflux
Testcase, Vault, Checklists, Pins, the taskboard) are styled too. Some less-used pages
(repository browser, roadmap, wiki history, admin tables) have only the general treatment.
Upstream reports and screenshots go to the
[issues](https://github.com/acosonic/redmine-modern-theme/issues).

## Features

- Light / dark mode: follows the operating system until you choose, then remembered per
  browser, with no flash on load. It also remembers your colour scheme for each mode.
- 27 colour schemes from [Gogh](https://github.com/Gogh-Co/Gogh) — Dracula, Nord, Gruvbox,
  Catppuccin, Rosé Pine, Tokyo Night, Solarized Light, GitHub and more — each
  contrast-checked to WCAG AA. Pick one on **My account**; the four used last sit in the
  account menu. Or search **all ~1,250 Gogh palettes** by name and pick an accent.
- Sidebar on the left (default) or the right — choose on **My account**; collapse it with
  Redmine's own button.
- Sticky top bar, header and project tab bar on desktop; sticky sidebar that falls back to
  scrolling with the page when it is taller than the window.
- Priority arrows and a colour dot per tracker in issue lists and on the issue page.
- Syntax highlighting, Gantt bars and the calendar coloured from the active palette.
- Project list as a card grid that reads left to right.
- Collapsible Subtasks / Related issues sections, toast notifications
  (`window.rmToast(message, type, duration)`).
- Phones (below 900px): Redmine 7's own header and flyout menu in the theme's colours.
- Written with logical properties throughout, so right-to-left languages mirror as in core.
- Self-hosted Inter font, no external requests.

## Install

Clone this fork into `themes/` under a **different folder name** than the Redmine 6 theme,
so both stay available and Redmine lists them as separate themes:

```bash
cd /path/to/redmine/themes
git clone https://github.com/matbott/redmine-modern-theme-mf-v7.git redmine-modern-theme-mf-v7
```

Restart Redmine, then choose
**Administration → Settings → Display → Theme → redmine-modern-theme-mf-v7**.
The previous theme keeps working; switch back at any time.

Updating: `git pull` in the theme folder and restart Redmine. In production with
precompiled assets, clear the cache and recompile first:

```bash
docker exec -it <container> /bin/bash -c '
  cd /usr/src/redmine && rm -rf public/assets/* &&
  RAILS_ENV=production bundle exec rake tmp:cache:clear &&
  RAILS_ENV=production bundle exec rake assets:precompile'
docker restart <container>
```

## Requirements

- Redmine **7.0** or later.
- A browser from 2024 or later: the theme relies on cascade layers, `color-mix()`,
  `light-dark()`, CSS nesting and `:has()` (Chrome/Edge 123+, Firefox 120+, Safari 17.5+).

## Customising

Change the palette by overriding the inputs at the top of `stylesheets/application.css`
(section 1) — for example in a custom stylesheet loaded after the theme:

```css
:root {
  --rm-accent: light-dark(#7C3AED, #A78BFA);
  --rm-topbar: light-dark(#2E1065, #1E1B4B);
}
```

Everything else — hover shades, tints, focus rings, the Open Color scale core uses — is
derived from those.

## Development

Tools live in `src/` (Redmine does not publish that folder): `gen-schemes.py` builds the
scheme blocks from Gogh, `shot.mjs` takes screenshots and runs probes, `contrast.mjs`
audits WCAG contrast, `seed.rb` fills a local development database.

## Local customisations

Everything below lives in `stylesheets/custom.css` (imported right after `schemes.css`),
except the two script blocks at the end of `javascripts/theme.js`
(sidebar width and the project hero). See [MAINTENANCE.md](MAINTENANCE.md) for the full
map, token translations and how to update from upstream.

- **Selected issue rows** — `tr:has(input[type=checkbox]:checked)`, so the row paints as
  soon as its checkbox is ticked (core only marks it on submit).
- **Sidebar width** — per-browser slider on **My account** (180–360 px), stored as
  `--rm-sidebar-w` and applied before first paint.
- **RedmineUP Checklists** — the new-item field takes the row.
- **Admin index** — icon cards in a responsive grid.
- **Project overview hero** (“Vistazo moderno”) — hero, KPI cards, issue/members/activity
  summary and a donut chart of open issues (toggle: by tracker / by assignee); accent read
  from the active palette, so it follows dark mode and schemes.
- **Plugin compatibility** — RedmineUP Agile, Redmineflux Testcase, Vault, Pins, View
  Customize code blocks.
- **Taskboard / board** (`#rdb*`, `.rdb-*`) — cards, headers, footers, dropdowns and the
  compact list on the theme tokens; answers the plugin's inline colours where needed.

## Author

Made by **Aleksandar Pavic**, author of the
[Redmine Cookbook](https://www.packtpub.com/en-us/product/redmine-cookbook-9781785285264)
(Packt Publishing).

## Credits

- Palettes: [Gogh](https://github.com/Gogh-Co/Gogh) (MIT)
- Font: [Inter](https://rsms.me/inter/) (OFL)
- Ideas (no code): priority arrows and tracker dots from Opale, gradient bars from gitmike

## Licence

MIT
