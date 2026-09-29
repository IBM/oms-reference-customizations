---
name: buc-wire-custom-assets
description: Copy an Order Hub module's shared assets into a route's src-custom and wire its merged builds, so route-based JSON or code customization is served. Use before generating components or writing custom JSON for a route.
---

# Wire route assets

Complete this skill before generating any component or writing any custom JSON for a route.

**Before you start**, the route must be `true` in the module's `overrides.json`. If it isn't, follow
[buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md) first.

Before reusing completed setup, check the copied subtree and both build mappings; the environment
flag alone is not enough.

## Copy the baseline

Discover the full module name and package prefix, then copy the complete subtree of IBM-shipped assets:

```text
packages/<module-short>-shared/assets/<module-name>/
    -> packages/<route>/src-custom/assets/<module-name>/
```

For example, `order-shared/assets/buc-app-order/` goes into the route's
`src-custom/assets/buc-app-order/`. Include JSON, i18n, images, and other supplied files.
Create parent directories, avoid duplicate module nesting, and preserve existing snapshots and
user changes. Custom translations alone are not enough.

**Two separate asset paths — never mix them:**

| Path | Purpose |
|---|---|
| `src-custom/assets/<module-name>/` | IBM-shipped baseline snapshot (copied above). Never write custom content here. |
| `src-custom/assets/custom/` | All custom and extension JSON (`buc-table-config.json`, `buc-field-details.json`, etc.) and translations (`src-custom/assets/i18n/en.json`). This is where schematic output and hand-written overrides go. |

## Wire both builds

In `angular.json`, configure `projects.<route>.architect.build.configurations.merged.assets`
and `.merged-prod.assets`:

```json
"assets": [
  { "glob": "**", "input": "packages/<route>/src-merged/assets", "output": "assets" },
  { "glob": "*.json", "input": "packages/<route>/src-merged/assets/<module-name>", "output": "assets/<route>" },
  { "glob": "**", "input": "node_modules/@buc/svc-angular/assets", "output": "assets" },
  { "glob": "**", "input": "node_modules/@buc/common-components/assets", "output": "assets" }
]
```

Preserve required project-specific mappings and avoid duplicate outputs that overwrite the snapshot
with shared originals. Never edit `src-merged`. If missing, inspect and repair the installed
merge/start workflow; verify any toolkit-specific development mapping against actual served assets.

## Environment and verification

For every route overlay under `src-custom/assets/custom/`, including JSON-only
changes and custom i18n, copy missing `src/environments/` files into
`src-custom/environments/`, preserving edits, and set `environment.customization = true;`
after instantiating `environment` in both `environment.ts` and `environment.prod.ts`
(set on the instance, e.g. `export const environment = new OnPremEnvironment(); environment.customization = true;`,
not as a property inside the `OnPremEnvironment` class).

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

The baseline snapshot under `assets/<module-name>/` is a separate load path and
does not itself require this flag. Custom i18n is excluded from the merged translation
bundle (`**/custom/i18n/*.json`); it always uses the selected custom loader path.

## Verify

Route setup is complete only when all four conditions below are met. Check each one; a `src-custom/` or
`assets/custom/` directory that already exists is not proof of any of them:

1. `<route>` is `true` in the module's `overrides.json` — from [buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md).
2. The IBM baseline subtree is copied under `src-custom/assets/<module-name>/`.
3. `merged` and `merged-prod` are both mapped in `angular.json`.
4. `environment.customization = true` is set on the exported instance in `environment.ts` and `environment.prod.ts`.

Have the user restart the dev server after build/override changes. Verify baseline/custom JSON and i18n
requests resolve and the requested behavior works. Return to the calling skill. 

If another skill sent you here, return to it.
