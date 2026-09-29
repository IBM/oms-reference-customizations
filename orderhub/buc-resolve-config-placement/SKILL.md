---
name: buc-resolve-config-placement
description: Decide whether a custom JSON asset goes in root-config or in a route's src-custom, by inferring what the module already uses. Run this before writing any file under assets/custom, and record the answer so the question is never asked twice.
---

# Resolve configuration placement

A custom JSON asset can live in one of two places. Other skills write
`<config-dir>` in their paths and send you here to resolve it:

| Placement | `<config-dir>` |
|---|---|
| Root-config | `packages/<module-short>-root-config/src/assets/custom/` |
| Route | `packages/<route>/src-custom/assets/custom/` |

`<module-short>` is the text after the last dash of the module name:
`buc-app-order` → `order` → `packages/order-root-config/`. `<route>` is the
package directory under `packages/`, which is also the route's key in
`overrides.json`.

The difference is ownership. Root-config isn't tied to a route, so IBM updates
keep applying and the route never goes into `package-customization.json` at
deployment. A route's `src-custom/` only takes effect once you own that route.

Work through the steps below and stop when the placement is resolved. Most
requests are settled by steps 1 to 3 without asking the user anything.

## 1. Does the change need code?

Root-config carries JSON and nothing else. If any part of the change is code,
the answer is **route** and there is no preference to offer:

| Change | Why it has to be a route |
|---|---|
| A page action with a handler | The action calls a service you register |
| A custom tab | The tab renders a component |
| An extension point, a new page, a lifecycle override | All Angular code |
| A schematic-generated component (table, search panel, search result, summary, dashboard) | The schematic emits Angular code into the route package |
| Hot keys on a page that doesn't already start the listener | `initializeHotKeyListener()` is a code change |

Hot keys are the case that looks like configuration and isn't. Root-config alone
is enough only when the page already calls `initializeHotKeyListener()` with a
matching page name.

## 2. Does the file support both places?

| File | Both? | Notes |
|---|---|---|
| `buc-table-config.json` | yes | |
| `buc-field-details.json` | yes | |
| `search_fields.json` | yes | |
| `getPage-templates.json` | yes | |
| `buc-page-definitions.json` | yes | hot keys only — see step 1 |
| `i18n/<language>.json` | yes | keep strings with the assets they label |
| `buc-tenant-config.json` | no | `buc-app-settings/packages/settings-configurations/src-custom/assets/custom/` only |
| `features.json`, login assets, injected HTML or JS | no | shell locations — follow their own skill |

If a file supports only one location, return that location and continue.

## 3. Is this file already placed?

From `devtoolkit_docker/orderhub-code`, for the module you're changing:

```
find <module>/packages/*-root-config/src/assets/custom -type f
find <module>/packages/*/src-custom/assets/custom -type f
```

Read the target route's effective `environment.customization` before selecting
an existing file. A file's existence does not prove it is requested. Preserve the
user's placement when compatible; if it is not active, complete the required setup.

`environment.customization` selects the custom asset URL on the SINGLE_SPA
development path; it does not merge module and route custom directories:

| Flag | Config URL | Custom translation URL |
|---|---|---|
| `true` | `<ctx>/<module>/<route>/assets/custom/<file>` | `<ctx>/<module>/<route>/assets/custom/i18n/<file>` |
| `false` | `<ctx>/<module>/assets/custom/<file>` | `<ctx>/<module>/assets/custom/i18n/<file>` |

The route overlay is never requested with the flag unset. With it set, the
module-level custom overlay is not requested for that route. This is separate
from merging the selected overlay over shipped defaults. Inspect the effective
environment for the target route before retaining an existing placement.
`bootstrapConfig.noExtensions` disables config-based loading on the on-premise false branch.
The implementation uses `custom/i18n`, despite the loader JSDoc saying `i18n/custom`.
Production module-level URLs also depend on deployment context and dev-mode settings;
verify the packaged URL rather than copying a development URL into deployment config.

If both locations contain the filename, identify which routes consume each copy.
Do not delete or move one merely because both exist; other routes may depend on it.
Record the target route and selected directory for the task.

## 4. Infer from the module

Nothing is placed yet for this file, so read what the module is already doing:

```
find <module>/packages/*-root-config/src/assets/custom -type f
find <module>/packages/*/src-custom/assets/custom -type f
grep -n '"runAsCustomization": true' <module>/overrides.json
```

| What you find | Placement |
|---|---|
| Files under root-config, no route enabled | Root-config |
| Enabled routes with files under their `src-custom/assets/custom/`, nothing in root-config | Route — the one that owns the page you're changing |
| Both, for different files | Inspect the target route's effective environment; it selects one overlay location for all these file families |
| Nothing anywhere, no route enabled | Unresolved — go to step 5 |

**Directories prove nothing here.** A clean toolkit already ships
`<module-short>-root-config/src/assets/custom/i18n/` and
`<route>/src-custom/assets/custom/i18n/`, both empty, and every route in
`overrides.json` set to `false`. Count files and flags, never folders.

## 5. Ask — once

Only now, and only because nothing on disk answers it. Ask the user to choose,
in one question, with the cost stated:

> This module has no customizations yet. JSON-only changes can go in
> **root-config**, which isn't tied to a route and keeps applying through IBM
> updates, or in the **route's `src-custom/`**, which needs the route enabled for
> customization and re-merged at every upgrade. Which do you want for
> `<module>`?

Recommend root-config unless they're already planning code changes to that route.
Then record the answer (next section) so no later skill asks again.

## If the answer is route

A route's `src-custom/` is inert until that route is enabled. Writing JSON there
with `runAsCustomization: false` produces no error and no effect — it's the most
common silent failure in this whole flow.

Before writing anything, confirm `<route>` is `true` in `overrides.json` and that
`environment.customization = true` is set. If not, run
[buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md)
first.

Module-level placement needs no route activation solely for JSON, but the target
route must have `environment.customization` false and extensions enabled.

## Verify

After the file is written, confirm it landed in the resolved `<config-dir>`, and check
whether the same filename also exists in the other location:

```
find <module>/packages/*-root-config/src/assets/custom -name '<file>'
find <module>/packages/*/src-custom/assets/custom -name '<file>'
```

Confirm that the loader requests the selected copy for the target route. Two hits
can serve different routes; do not delete either without checking its consumers.
