---
name: buc-generate-summary-component
description: Generate an Order Hub field component or summary side panel with the summary-component schematic, for displaying record attributes on a details page.
---

# Generate a field or summary component

`@buc/schematics:summary-component` generates one of two components, both driven by
`buc-field-details.json`:

| `--is-summary-panel` | Result |
|---|---|
| `true` (default) | **Summary component**: attributes in the right side panel |
| `false` | **Field component**: attributes in the main content area, usually under a tab |

## Before generating

Generated components are route-based customization. **Do not run the schematic** until
you have followed [buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md)
and [buc-wire-custom-assets](../buc-wire-custom-assets/SKILL.md) for the target route, then
used [buc-configure-generated-assets](../buc-configure-generated-assets/SKILL.md) to resolve
`--json-file-path` and `--translation-file-path`. Complete all validation checks in the prerequisite skills before
continuing.

## 1. Run the schematic

From the module root (for example `devtoolkit_docker/orderhub-code/buc-app-order`):

```
ng g @buc/schematics:summary-component \
  --name <name> \
  --is-summary-panel <true|false> \
  --path packages/<route>/src-custom/app/features/<feature> \
  --json-file-path packages/<route>/src-custom/assets/custom \
  --translation-file-path packages/<route>/src-custom/assets/custom/i18n \
  --project <route> \
  --skip-import
```

| Option | Notes |
|---|---|
| `--name` | Required. `my-sample` → `MySampleFieldsComponent`, config key `my-sample-fields` |
| `--json-file-path` | Required. For customizations: `packages/<route>/src-custom/assets/custom` |
| `--translation-file-path` | Required. For customizations: `packages/<route>/src-custom/assets/custom/i18n` |
| `--project` | Required. The route package, e.g. `order-details` |
| `--path` | Optional. Relative to the module. Defaults to `src/app`, not `src-custom`, so pass it |
| `--json-file-name` | Optional. Defaults to `buc-field-details.json` |

## 2. Register the generated components

`--skip-import` suppresses automatic NgModule registration. There is no generated
declaration to remove. Add the component to
`AppCustomizationImpl.components`, keeping the existing entries. See
[buc-register-customization](../buc-register-customization/SKILL.md).
Without the flag, a fresh custom tree can fail with `Could not find an NgModule`.
The schematic writes its JSON immediately; compilation is needed to serve it.

## 3. Label a field component's tab

A field component renders inside a tab, and the tab needs a label. Add a `header`
to the generated HTML pointing at a translation key:

```html
[header]="'MY_PAGE.MY_TABS.DETAILS'"
```

Summary panels don't need this.

## 4. Configure the attributes

Edit your entry in `buc-field-details.json` to choose the attributes to display,
set their order, and define their labels. Labels are translation keys — define them in
`src-custom/assets/custom/i18n/en.json`.

## 5. Verify

Ask the user to reload the frame and open the details page, then confirm that the tab
label and attributes display correctly. A blank value almost always means the attribute path in
the config doesn't match the API response — ask them to check the response in the
**Network** tab.

## Going further

- Field attributes, editable fields, populating dropdowns from APIs →
  [buc-customize-field-details](../buc-customize-field-details/SKILL.md)
- An attribute the API isn't returning → [buc-configure-getpage-templates](../buc-configure-getpage-templates/SKILL.md)
