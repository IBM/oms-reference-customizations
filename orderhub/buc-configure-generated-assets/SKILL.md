---
name: buc-configure-generated-assets
description: Choose schema and translation output paths for BUC component schematics writing into src-custom.
---

# Configure generated assets

Generated components need route code. If the user chose module-level, JSON-only work, explain
that before switching to route customization, and keep any independent module-level JSON changes.

1. Prepare the route with [buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md) and [buc-wire-custom-assets](../buc-wire-custom-assets/SKILL.md).
2. Inspect the installed generator's dry run and runtime loader. For a new base-schema entry,
   set `--json-file-path` to `packages/<route>/src-custom/assets/custom` after the shared
   subtree has been copied. Existing-page extension entries belong in `assets/custom/`.
3. Set `--translation-file-path` to `packages/<route>/src-custom/assets/custom/i18n`.
4. Preserve existing entries. Verify that served route assets contain the generated schema IDs and
   translations; do not edit shared originals to fix a missing entry.

Dashboard custom layouts use `assets/custom/dashboard-configs.json`. Keep that output when
the loader layers custom layouts over the baseline; the base-schema rule does not apply there.

Verify that the generated base schema and custom translations are requested at their intended URLs.

If a component skill sent you here, return to it to generate and register the component and
implement its behavior.
