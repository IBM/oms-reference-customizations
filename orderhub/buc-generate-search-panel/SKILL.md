---
name: buc-generate-search-panel
description: Generate an Order Hub search criteria page with the search-panel schematic, including breadcrumbs, saved searches, and search-by groupings.
---

# Generate a search page

`@buc/schematics:search-panel` creates the page where users enter search criteria.
The generated component already includes a breadcrumb trail, save search, customize
search criteria, and search-by groupings.

For the results page, use [buc-generate-search-result](../buc-generate-search-result/SKILL.md).

## Collect first

Ask the user for anything missing. Do not guess.

- The route package (for example `shipment-search`) and the shared library (`order-shared`)
- Where the panel appears: its own route, or an extension ID on an existing page
- The search fields: label, type, and the API attribute each one fills
- The results page the Search button opens

## Before generating

Generated components are route-based customization. **Do not run the schematic** until
you have followed [buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md)
and [buc-wire-custom-assets](../buc-wire-custom-assets/SKILL.md) for the target route, then
used [buc-configure-generated-assets](../buc-configure-generated-assets/SKILL.md) to resolve
`--json-file-path` and `--translation-file-path`. Complete all validation checks in the prerequisite skills before
continuing.

## 1. Run the schematic

From the module root (for example `devtoolkit_docker/orderhub-code/buc-app-order`), dry
run first with `--dry-run`, then run it:

```
ng g @buc/schematics:search-panel \
  --name <name> \
  --shared-lib <module-short>-shared \
  --path packages/<route>/src-custom/app/features/<feature> \
  --json-file-path packages/<route>/src-custom/assets/custom \
  --translation-file-path packages/<route>/src-custom/assets/custom/i18n \
  --project <route> \
  --skip-import
```

| Option | Notes |
|---|---|
| `--name` | Required. `my-sample` → `MySampleSearchComponent`, config key `my-sample-search` |
| `--json-file-path` | Required. For customizations: `packages/<route>/src-custom/assets/custom` |
| `--translation-file-path` | Required. For customizations: `packages/<route>/src-custom/assets/custom/i18n` |
| `--shared-lib` | Required. The shared library folder, for example `order-shared` |
| `--project` | Required. The route package, e.g. `order-search` |
| `--path` | Optional. Defaults to `src/app`, not `src-custom`, so pass it |

## 2. Finish the generated code

1. **Results route.** `resultsRoute()` returns `''`. Return the real route, using the
   constants class the host page already imports (for example
   `ShipmentConstants.SHIPMENT_SEARCH_RESULT_ROUTE`, not `Constants`).
2. **Session prefix (optional).** The panel saves criteria under `global-<sessionPrefix>`
   (default `sample-search`), which a generated results page reads by default. Change it
   only if another panel uses the same prefix, and change the results page to match.
3. **Breadcrumb.** `prepareBreadCrumbList()` uses a placeholder route. Replace it.
   When mounting at an extension point, follow "Mount a generated page component" in
   [buc-add-content-via-extension-point](../buc-add-content-via-extension-point/SKILL.md#mount-a-generated-page-component).

## 3. Register

`--skip-import` suppresses NgModule registration. Add the component to
`AppCustomizationImpl.components`, keeping existing entries. See
[buc-register-customization](../buc-register-customization/SKILL.md). To show it on an
existing page, map it to the extension ID as in
[buc-add-content-via-extension-point](../buc-add-content-via-extension-point/SKILL.md).

## 4. Define the search fields

The schematic writes sample fields into `search_fields.json` under your
`<name>-search` key. Replace them with the fields you need. For field syntax,
types, and API attribute mappings, see
[buc-customize-search-fields](../buc-customize-search-fields/SKILL.md).

Labels are translation keys; define them in `src-custom/assets/custom/i18n/en.json`.

## 5. Verify

Ask the user to reload the frame, open the page, and confirm the fields render. Then
run a search and check the **Network** tab: the request payload must include each
field they filled in. A field that renders but never reaches the payload has the
wrong `request` attribute path.
