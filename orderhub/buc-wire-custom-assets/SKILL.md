---
name: buc-wire-custom-assets
description: >
  Copy shared module assets and wire merged builds for BUC route-based JSON or code customization.
---

# Wire Route Assets

Run after [buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md) for route-based JSON or code.
Before reusing completed setup, check the copied subtree and both build mappings — the environment flag alone is not enough.

## Copy the baseline

Discover the full module name and package prefix, then copy the complete subtree:

```text
packages/<module-short-name>-shared/assets/<module-name>/
    -> packages/<route>/src-custom/assets/<module-name>/
```

For example, `order-shared/assets/buc-app-order/` goes into the route's
`src-custom/assets/buc-app-order/`. Include JSON, i18n, images, and other supplied files.
Create parent directories, avoid duplicate module nesting, and preserve existing snapshots and
user changes. Custom translations alone are not enough.

Existing-page extensions belong in `src-custom/assets/custom/`; translations go in its `i18n/`.

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

For route code, copy missing `src/environments/` files to `src-custom/environments/`, preserving
changes, and add `environment.customization = true;` as a post-instantiation assignment after the
`export const environment = new OnPremEnvironment();` line in both `environment.ts` and
`environment.prod.ts`. Do not add `customization` as a class property.
Changes limited to `search_fields.json`, `buc-table-config.json`, or `buc-field-details.json`
do not require this flag. Preserve a flag used by existing code; check other JSON consumers' loaders.

Have the user restart the dev server after build/override changes. Verify baseline/custom JSON and i18n
requests resolve and the requested behavior works. Return to the calling skill.
