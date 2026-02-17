# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Dwains Lovelace Dashboard is a Home Assistant custom integration (v3.8.0, requires HA 2025.4.0+) that auto-generates a Lovelace UI dashboard from Home Assistant's configuration. It is distributed via HACS and is in maintenance mode — the original developer accepts community PRs but is not actively developing new features.

## Build Commands

JavaScript frontend lives in `custom_components/dwains_dashboard/js/`:

```bash
npm run build    # Production build via webpack
npm run watch    # Development watch mode
```

There is no test suite and no lint scripts. There are no CI/CD workflows.

## Architecture

### Python Backend (`custom_components/dwains_dashboard/`)

- **`__init__.py`** (1808 lines) — Core entry point. Handles async setup, registers websocket API handlers and Home Assistant services, discovers areas/devices/entities, builds the dashboard configuration tree, and loads plugins.
- **`config_flow.py`** — Single-instance HA config flow for initial setup and options (sidebar title/icon).
- **`process_yaml.py`** — Custom YAML loader with Jinja2 template support. Handles `# dwains_dashboard`, `# dwains_theme`, and `# lovelace_gen` comment markers that control processing behavior.
- **`notifications.py`** — Custom notification service (`notification_create`, `notification_dismiss`, `notification_mark_read`) with read/unread state tracking.
- **`load_dashboard.py`** / **`load_plugins.py`** — Dashboard and plugin/JS loading helpers.
- **`sensor.py`** — Version-check sensor that polls a remote endpoint (throttled to every 800 minutes).

### JavaScript Frontend (`custom_components/dwains_dashboard/js/`)

- Built with Webpack 5, output is the compiled `dwains-dashboard.js` bundle.
- Uses **LitElement** (v2) for WebComponents.
- Uses **Sortable.js** for drag-and-drop and **card-tools** for Home Assistant card utilities.
- Styled with **Tailwind CSS 2** via PostCSS.
- Multiple card entry points are defined in `webpack.config.js`.

### Lovelace Configuration (`custom_components/dwains_dashboard/lovelace/`)

- `ui-lovelace.yaml` — Top-level dashboard definition that includes view files.
- `views/` — YAML files for each dashboard view (homepage, devices, more pages, etc.). These are processed through `process_yaml.py` with Jinja2 templating before being sent to the Lovelace frontend.

### Data Flow

1. HA loads the integration via `__init__.py` `async_setup_entry`.
2. Python discovers entities/areas/devices from HA's state machine.
3. YAML view templates are rendered via Jinja2 in `process_yaml.py`.
4. Rendered config is served to the frontend via HA's Lovelace websocket API.
5. The JS bundle registers custom WebComponent cards that HA's frontend renders.

## Key Dependencies

**Python:** voluptuous, PyYAML (with annotatedyaml custom Loader), Jinja2, aiofiles, aiohttp

**JavaScript:** lit-element ^2, lit-html ^1, @mdi/js ^6, sortablejs ^1.14, custom-card-helpers ^1.8, js-cookie ^3

## Development Notes

- Testing is done manually against a live Home Assistant instance — there is no automated test runner.
- After changing Python files, reload the integration in HA (`Developer Tools → YAML → Reload All YAML`), or call the `dwains_dashboard.reload` service.
- After changing JS files, run `npm run build` then clear the browser cache before testing.
- The integration registers under the domain `dwains_dashboard` (see `const.py`).
