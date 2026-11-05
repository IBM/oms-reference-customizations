---
name: buc-prepare-module-customization
description: Prepare an existing Order Hub/Call Center module and route for customization. Set up overrides.json, the src-custom folder, assets, environments, and the dev server, then connect Order Hub/Call Center to the toolkit. Do this before writing route code or route custom assets for an IBM page.
---

# Enable customization for an existing module

Complete this setup once per route. Make subsequent changes in `src-custom/`.

## 1. Find the module and route

Ask the user which Order Hub/Call Center page they want to change if it isn't clear. Map it to a module:

| Order Hub menu | Module | Port |
|---|---|---|
| Home, Workspaces, Alerts, Settings > Alert rules | `buc-app-workspace` | 8900 |
| Nodes and capacity | `buc-app-node` | 8200 |
| Orders, Shipments | `buc-app-order` | 8300 |
| Inventory | `buc-app-inventory` | 8600 |
| Fulfillment, Promise and Fulfill | `buc-app-fulfillment` | 9000 |
| Exceptions | `buc-app-exception` | 9100 |
| Settings > Display settings, User roles, Customization, About; Configuration > Distribution groups | `buc-app-settings` | 8400 |
| Configurations > Nodes, Carriers, Custom attributes, Tenant | `buc-app-configurations` | 9200 |
| Security > Users | `buc-app-user` | 9600 |

Call Center modules: `call-center-order` (5100), `call-center-return` (6100), `call-center-customer` (7100), `call-center-alert` (8100).

Each module contains one package per route under `packages/`. The route names are the
keys in the module's `overrides.json`.

## 2. Install dependencies

From `devtoolkit_docker/<orderhub-code|callcenter-code>/<module>`:

```
yarn config set "strict-ssl" false
yarn install --update-checksums
```

Run `yarn cache clean` first if the DTK was just upgraded. Ignore
`Failed to compile entry-point @carbon/icons-angular/` errors — they're harmless.

## 3. Turn on customization for the route

In `<module>/overrides.json`, set `runAsCustomization` to `true` for your route:

```json
"order-search-result": {
  "runAsCustomization": true
}
```

Enable only the routes you are actively customizing, with a maximum of five at
a time.

## 4. Mirror the folders you need

Under `packages/<route>/`, recreate the `src/` path of any file you customize
inside `src-custom/`, then copy the file across. Example:

```
src/app/features/search/search.component.ts
src-custom/app/features/search/search.component.ts
```

Copy only the files you need to change. Relative-path errors in the copied files
are expected at this stage and resolve once the module compiles.

## 5. Wire assets and environments

**Do this before generating any component or writing any custom JSON.**
Complete [buc-wire-custom-assets](../buc-wire-custom-assets/SKILL.md) in full. It covers the
baseline copy, the `merged` and `merged-prod` `angular.json` mappings, and both environment files.
Complete its **Definition of done** checklist to verify route setup before generating
a component or writing custom JSON.

Key rule from that skill: IBM-shipped files go in `src-custom/assets/<module-name>/`
(the baseline snapshot); all custom JSON and schematic output goes in
`src-custom/assets/custom/`. Never write custom content into the baseline snapshot path.
The one exception is the generated search-panel key (see
[buc-configure-generated-assets](../buc-configure-generated-assets/SKILL.md)).

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

For route overlays set `environment.customization = true;` on the exported `environment` instance
(after instantiation, e.g. `export const environment = new OnPremEnvironment(); environment.customization = true;`,
not as a class property) in both `environment.ts` and `environment.prod.ts`. Verify all wiring even when `src-custom` already exists.
Create `src-custom/assets/custom/` and its `i18n/en.json` only as needed.

## 6. Start the dev server

`overrides.json` and `angular.json` are read at startup, so a restart is required
after any change to them.

Ask the user to run this in their own terminal from the module folder and to tell
you when it's up — it's long-running and holds the terminal:

```
yarn stop-app
yarn start-app
```

Compiling takes several minutes. It's ready when each customized route reports
`<route>: √ Compiled successfully.` Ignore the "Angular Live Development Server is
listening on localhost:<port>" message — you don't open that URL directly.

If the build dies with `exit code 134`, raise the heap and restart:

```
export NODE_OPTIONS=--max_old_space_size=8048     # bash
$Env:NODE_OPTIONS="--max_old_space_size=8048"     # PowerShell
```

## 7. Connect Order Hub/Call Center to the toolkit

These are browser steps. Walk the user through them and wait for confirmation:

1. Open `https://localhost:7443/order-management`. The port differs if
   `OH_BASE_HTTPS_PORT` is set in `devtoolkit_docker/compose/om-compose.properties`,
   so ask if the default doesn't load.
2. Click **Customize** in the top banner, set the module to **ON**, and save.
3. Open `https://localhost:<module port>` in a new tab and accept the
   self-signed certificate. The module won't load in Order Hub/Call Center until this is done.
4. Refresh the Order Hub/Call Center tab.

The menu item now reads **(DEV mode)**. That page is being served from the local
dev server, so your changes appear on reload.

Ask the user to confirm they see **(DEV mode)** before moving on. If they don't,
the certificate from step 3 is the usual cause.

**Call Center:** open `https://localhost:6443/call-center/login`, click **Customize** in the header, and set the module **ON**. If a page shows `Failed to load module <route>`, have the user open the `main.js` link in the error, accept the certificate, and sign in again.

## Next

[buc-customize-by-configuration](../buc-customize-by-configuration/SKILL.md), [buc-add-content-via-extension-point](../buc-add-content-via-extension-point/SKILL.md), or
[buc-customize-by-overrides](../buc-customize-by-overrides/SKILL.md).
