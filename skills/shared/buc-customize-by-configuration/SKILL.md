---
name: buc-customize-by-configuration
description: Change Order Hub/Call Center labels, table columns, table actions, and search fields by editing JSON only, with no code. Use this before considering a code-based approach.
---

# Customize by configuration

JSON controls search forms, tables, and field attributes. Override these settings by
putting a partial file in the selected custom directory. Order Hub/Call Center merges it over the
IBM default, so include only the keys you are changing.

| What | File | Full reference |
|---|---|---|
| Table columns, actions, overflow menu | `buc-table-config.json` | [buc-customize-table-config](../buc-customize-table-config/SKILL.md) |
| Detail fields, summary panels | `buc-field-details.json` | [buc-customize-field-details](../buc-customize-field-details/SKILL.md) |
| Search criteria fields | `search_fields.json` | [buc-customize-search-fields](../buc-customize-search-fields/SKILL.md) (Call Center: [cc-customize-search-fields](../cc-customize-search-fields/SKILL.md)) |
| Attributes the API returns | `getPage-templates.json` | [buc-configure-getpage-templates](../buc-configure-getpage-templates/SKILL.md) (Call Center: [cc-extend-mashups](../cc-extend-mashups/SKILL.md)) |
| Custom tabs, page actions, hot keys | `buc-page-definitions.json` | [buc-add-custom-tab](../buc-add-custom-tab/SKILL.md), [buc-add-custom-action](../buc-add-custom-action/SKILL.md), [buc-add-custom-hotkeys](../buc-add-custom-hotkeys/SKILL.md) |
| Any visible string | `i18n/en.json` | [buc-customize-locales-i18n](../buc-customize-locales-i18n/SKILL.md) |

This skill covers common changes. For the full attribute sets, including
formatters, editable fields, and conditional display, read the reference skills. Check them
before deciding that a change needs code.

**Call Center:**

- Search fields use a different schema — [cc-customize-search-fields](../cc-customize-search-fields/SKILL.md).
- Attributes come from mashups, not `getPage-templates.json` — [cc-extend-mashups](../cc-extend-mashups/SKILL.md).
- Shell labels go in `callcenter-code/shell-ui/assets/i18n/<lang>.json`, and home portlet labels in `call-center-custom/custom-home-portlet/src/assets/i18n/<lang>.json`.

## Where to put the file

Resolve `<config-dir>` for the target route with
[buc-resolve-config-placement](../buc-resolve-config-placement/SKILL.md), then use
that directory for configuration and translations. The effective environment
selects the overlay path. If it resolves to route placement, complete
[buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md) first.

## Find the schema name

Config objects are keyed by schema name. To find a table's schema name, ask the
user to follow these steps and report what they see, since you can't read their
browser:

1. Open the page in Order Hub/Call Center with the browser **Console** tab open.
2. Look for the last message containing `BaseTableComponent.initializeTable`.
3. The schema name is in that message (for example `inv-search-result-table`).

For search forms, find a label you can see on screen (for example "Master catalog
ID") and search for it in the IBM `search_fields.json` under
`packages/<module-short>-shared/assets/<module-name>/`. The object that contains the surrounding fields is
your key.

## Rename a label or table header

Labels are translation keys, not literals. Find the key, then redefine it.

1. Open the IBM config at `packages/<module-short>-shared/assets/<module-name>/buc-table-config.json` and
   find your schema. Each header's `name` is the translation key:

```json
{ "id": "deliveryDate", "name": "SHARED.GENERAL.LABEL_reqDelDate", "sortable": true }
```

2. Redefine that key in `<config-dir>/i18n/en.json`, matching the
   dot-notation as nested objects:

```json
{
  "SHARED": {
    "GENERAL": {
      "LABEL_reqDelDate": "Desired delivery date"
    }
  }
}
```

This works for any string you can trace to a translation key, not just table
headers. Keep all label overrides in this one file.

## Add a search field

Create `<config-dir>/search_fields.json` with just your key and field:

```json
{
  "add-order": {
    "fields": [
      {
        "label": "Alias value",
        "type": "dropdownQuery",
        "target": "add-order",
        "request": "ItemAliasList.ItemAlias.AliasValue",
        "operator": "LIKE",
        "fetch": {
          "populate": "qryType",
          "api": "getQueryTypeList",
          "type": "oms",
          "parameters": {},
          "response": {
            "listAttribute": "StringQueryTypes.QueryType",
            "map": { "id": "QueryType", "label": "QueryTypeDesc" }
          }
        }
      }
    ]
  }
}
```

`request` is the API attribute path the value is sent as. It must match what the
page's search API expects. `fetch` is optional and populates dropdown choices.

## Position a search field

`oobSeq` controls the order in which fields appear. Every shipped field has one, starting
at 1 and increasing in whole numbers.

```json
{
  "orders": {
    "fields": [
      { "label": "Order name", "type": "text", "request": "OrderName",
        "target": "orders", "oobSeq": 1.1 },
      { "label": "Shipping node", "type": "nodePicker", "target": "releases",
        "request": "OrderLine.ShipNode", "oobSeq": 2.1 }
    ]
  }
}
```

Use decimals to slot a custom field between two shipped ones — `1.1` puts it
straight after the field with sequence 1.

- **A field with no `oobSeq` goes to the top of the form**, which is rarely what
  you want. Always set one.
- Keep every value unique.
- Don't use 0 — it's ambiguous with "not set".
- Read the shipped `search_fields.json` first to see what the surrounding sequence
  numbers are.

To keep a custom field in the locked part of the form, so users can't re-sequence
it away when they customize their search criteria:

```json
"locked": true
```

Check the shipped `search_fields.json` to see which section is locked.

## Add a table action

Create `<config-dir>/buc-table-config.json` with the schema name and
only the section you're changing:

```json
{
  "inv-search-result-table": {
    "name": "inv-search-result-table",
    "overflowMenuActions": [
      {
        "id": "createReservation",
        "label": "custom.LABEL_CREATE_RESERVATION",
        "resourceId": "CUSINV0001AT0001",
        "metadata": {
          "custom": "true",
          "actionTargetType": "internal",
          "actionTarget": "/<module>/<route>/<path to your page>",
          "contextColumnIds": "name"
        }
      }
    ]
  }
}
```

- `label` is a translation key — add it to `<config-dir>/i18n/en.json`.
- `resourceId` controls who sees the action. Create it and grant it before testing,
  or the action won't appear.
- `contextColumnIds` lists the row columns passed to the target. Confirm with the
  user that those columns are actually visible in their Order Hub/Call Center.
- `actionTarget` is the route to navigate to. For a page you are building, see
  [buc-add-page-to-route](../buc-add-page-to-route/SKILL.md).

## Verify

JSON assets are picked up on reload, so no server restart is needed.

Ask the user to reload the frame in Order Hub/Call Center and navigate to the page, then confirm
what they see. For a search field, ask them to run a search with the field filled in
and check the **Network** tab to confirm that the request payload carries the new attribute.

If a change doesn't appear, check in order: the schema name, the translation key
nesting, and whether the file landed in the resolved `<config-dir>`.
