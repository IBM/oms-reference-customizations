---
name: buc-configure-generated-assets
description: Choose schema and translation output paths for BUC component schematics writing into src-custom.
---

# Configure generated assets

Generated components require route code. If the user chose module-level JSON-only work, explain the
code requirement before switching to route customization. Preserve independent module-level JSON changes.

1. Prepare the route with [buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md) and [buc-wire-custom-assets](../buc-wire-custom-assets/SKILL.md).
2. Set the JSON output path based on the file the generated component loads:

   | Generator | Output setting and path |
   |---|---|
   | `search-panel` | `--json-file-path packages/<route>/src-custom/assets/<module-name>` |
   | `search-from-config` | Set `metadata.paths.jsonFile` in its input JSON to `packages/<route>/src-custom/assets/<module-name>` |
   | `table-component`, `search-result-component`, `summary-component`, `dashboard-component` | `--json-file-path packages/<route>/src-custom/assets/custom` |

   The two search-panel generators are the only exception to the `assets/custom/` rule.
   They write into the route snapshot because their panels read `search_fields.json` from that path.
3. Set `--translation-file-path` to `packages/<route>/src-custom/assets/custom/i18n`.
4. Preserve existing entries. Verify that served route assets contain the generated schema IDs and
   translations; do not edit shared originals to fix a missing entry.

Dashboard custom layouts use `assets/custom/dashboard-configs.json`. Keep that output when
the loader layers custom layouts over the baseline; the base-schema rule does not apply there.

Verify that the generated base schema and custom translations are requested at their intended URLs.

If a component skill sent you here, return to it to generate and register the component and
implement its behavior.
