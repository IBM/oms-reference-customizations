---
name: buc-prepare-module-customization
description: >
  Prepare a BUC route for src-custom development, including scaffolding, asset wiring, and local preview.
---

# Prepare Route Customization

For JSON work, place configuration files in the route's `src-custom/` directory (route-based config).
Root-config-only work needs module dependencies and local preview, not route activation or copying.

## Route setup

1. Inspect `app-config.json`, `angular.json`, `overrides.json`, and existing `src-custom` files.
   If scaffolding is absent, run the schematic at the module root with the target `mode`
   (`on-cloud` or `on-prem`) and actual `moduleShortName`:

   ```sh
   ng g @buc/schematics:setup-customization \
     --mode on-prem \
     --module-short-name order
   ```

   Do not remove route packages unless requested.
2. Install missing dependencies through devtoolkit environment.
   ```sh
   npm install -g @angular/cli@20
   npm install -g ./lib/buc/schematics/schematics-v5latest.tgz
   ```
3. Set `routes.<route>.runAsCustomization` to `true` in `overrides.json`, preserving other entries.
   Enable only routes needed for the task.
4. For code changes, mirror affected files from `src/` into `src-custom/`; leave IBM originals and
   generated `src-merged/` untouched. Differential additions need their own files, not copied host
   components. Use [registration](../buc-register-customization/SKILL.md) when adding components,
   providers, or imports.
5. For route JSON and code alike, complete [asset wiring](../buc-wire-custom-assets/SKILL.md).
   That skill defines the required module-subtree copy, both build mappings, and environment exception.
6. Request the user to restart the dev server after override/build configuration changes.

## Local preview

Use the browser to verify these steps, or pass them to the user if UI access is unavailable:

- Open the configured shell URL, typically `https://localhost:7443/order-management`.
- Enable the module in **Customize**. Open its local server URL to resolve any certificate prompt
  for the known development server, then reload the shell.
- Confirm the module shows **DEV mode**, assets load from the expected server, and the changed UI works.

To verify a published package, disable that module's local customization toggle and reload.
Continue independent implementation work while arranging preview access.
