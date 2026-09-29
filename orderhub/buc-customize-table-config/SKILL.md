---
name: buc-customize-table-config
description: Full buc-table-config.json reference - columns, data binding, formatters, sorting, editable cells, toolbar and overflow actions, and pagination. Use when adding columns to an existing table or configuring a generated one.
---

# buc-table-config.json reference

This file defines almost everything about a table: its columns, their positions,
value formatting, and available actions. Configure these here without writing a
component or changing markup.

## Where the file goes

Resolve the custom directory for this route with
[buc-resolve-config-placement](../buc-resolve-config-placement/SKILL.md), and retain
that choice for the file and its translations. For route placement, first complete
[buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md).


## Find the object name

Each top-level key is a table. Ask the user to open the page with the browser
**Console** tab open and read the `initializeTable` message:

```
Message: BaseTableComponent.initializeTable():  Initializing configuration for order-table
```

## Object attributes

| Attribute | Notes |
|---|---|
| `name` | Required. Unique identifier, same as the key |
| `aliases` | Apply this same config to other tables, for example inbound as well as outbound |
| `tokenId` | Service token for editable columns |
| `headers` | The columns |
| `toolbarActions` | Buttons shown when rows are selected |
| `overflowMenuActions` | Per-row overflow menu entries |
| `maxToolbarDisplayActions` | How many toolbar actions show before collapsing |
| `pagination` | Page length and page-size choices |
| `enableTableExport` | `false` removes the export icon — follow [buc-configure-table-export](../buc-configure-table-export/SKILL.md) |

## Column attributes

| Attribute | Notes |
|---|---|
| `id` | Required. Unique column ID |
| `name` | Header text. Use a translation key |
| `dataBinding` | Attribute to read from the API response. Omit for a new formatter column; explicitly set `"dataBinding": null` when replacing a shipped binding |
| `sequence` | Absolute position, from 1. Lands on the next non-fixed slot if taken. Appended last if unset |
| `default` | Visible in the default configuration. Default `true` |
| `fixed` | `true` means users can't reorder or remove it. Also marks it default. Default `false` |
| `sortable` | Default `true` |
| `sortKey` | The key to sort on |
| `sorted` | Sort by this column by default. Default `false` |
| `style` | CSS for the column, for example `{ "min-width": "100px" }` |
| `isEditable` | Makes the cell editable. Needs `tokenId` and a service |
| `fieldConfig` / `options` | Control type and value list for editable cells |

**`dataBinding` only works if the attribute is actually in the API response.** If a
column renders blank, that's the first thing to check — see
[buc-configure-getpage-templates](../buc-configure-getpage-templates/SKILL.md).

**Sorting:** multi-column sorting is off. If several columns set `"sorted": true`,
the first one in sequence wins.

## Formatters

Only supplied override keys replace shipped values. A custom header with both a
non-null `dataBinding` and `formatter` is rejected; omitting an existing binding
does not clear it. Rejection happens in `_validateHeader()` *before* the merge, so
overriding a shipped column with both keys set leaves that column exactly as
shipped — the change is dropped and only logged. A header is also stripped of
`sorted` whenever `default` is not truthy, which includes omitting `default`
entirely, not just setting it to `false`.

Use `formatter` instead of `dataBinding` — the binding moves to `formatter.value`.

**currency**

```json
"formatter": {
  "type": "currency",
  "value": "PriceInfo.TotalAmount",
  "currencyCode": "PriceInfo.Currency",
  "display": "code"
}
```

`display`: `code` (USD), `symbol` ($), `symbol-narrow`, or any string — an empty
string suppresses the indicator. `digitsInfo` takes
`{minIntegerDigits}.{minFractionDigits}-{maxFractionDigits}`; left unset, the
currency's ISO 4217 digits are used.

**dateTime**

```json
"formatter": { "type": "dateTime", "value": "PriceInfo.ReportingConversionDate", "dateFormat": "L" }
```

`dateFormat` uses Moment JS formats, default `L LT`. `useLocalTime` defaults false.

**quantity**

```json
"formatter": { "type": "quantity", "value": "OrderedQty" }
```

**enum** — map response values to translated literals:

```json
"formatter": {
  "type": "enum",
  "value": "DraftOrderFlag",
  "enumeration": { "Y": "CUSTOM_LIST.BOOLEAN_STRING.Y", "N": "CUSTOM_LIST.BOOLEAN_STRING.N" }
}
```

**template** — a lodash template with `item` (the row) and `translate`:

```json
"formatter": {
  "type": "template",
  "value": "<%= _.get(item, 'PriceInfo.TotalAmount') > 1000 ? translate('CUSTOM.HIGH') : translate('CUSTOM.NORMAL') %>"
}
```

Use `template` only when the other formatters can't express the value; it's the
hardest to debug when the data structure changes.

## Actions

**toolbarActions** appear when rows are selected:

```json
{
  "id": "myCustomToolbarAction1",
  "elem": "button",
  "type": "primary",
  "value": "CUSTOM_ORDER.TABLE.TOOLBARACTION.VALUE",
  "resourceId": "",
  "iconTemplate": "editIconTemplate",
  "metadata": {
    "custom": "true",
    "actionTargetType": "external",
    "actionTarget": "/customRoute",
    "contextColumnIds": "customerZipCode,customerPhoneNo"
  }
}
```

`elem` is always `button`; `type` is a Carbon button variant (IBM uses `primary`).
`iconTemplate` takes a `bucTemplate` string from
`icon-templates.component.html` in the common components.

This JSON only *declares* the action. Unless `actionTarget` routes somewhere, the
click is handled in the component — implement `onToolbarActionClicked`, or
`onOverflowMenuActionSelected` for a row action. Note that `toolbarContent` (the icon strip on the toolbar,
including refresh) is a separate array handled by `onToolbarContentClicked`.

**overflowMenuActions** are per row and take `id`, `label`, `resourceId`, and the
same `metadata`.

For both: `actionTargetType` is `internal` when the route is in the same package,
`external` when it isn't. `contextColumnIds` names the columns whose values get
passed to the route. `resourceId` gates visibility.

## Pagination

Per table:

```json
"pagination": { "pageLength": 10, "itemsPerPage": [10, 20, 25, 50, 75, 100] }
```

To configure every table at once, put the same block in
`shell-ui/assets/app-bootstrap-config.json`. It applies to every table that has no
per-table `pagination` of its own. Per-table config wins over the global block.

## Worked example

```json
{
  "order-table": {
    "name": "order-table",
    "aliases": ["inbound-order-table"],
    "headers": [
      {
        "id": "customerZipCode",
        "name": "CUSTOM_ORDER.SEARCH.CUSTOMER_ZIP_CODE",
        "sortKey": "CustomerZipCode",
        "dataBinding": "CustomerZipCode",
        "sequence": 1,
        "default": true,
        "sorted": true,
        "style": { "min-width": "100px" }
      },
      {
        "id": "draftOrder",
        "name": "CUSTOM_ORDER.SEARCH.DRAFT_ORDER",
        "dataBinding": "DraftOrderFlag",
        "sequence": 3,
        "sortable": false,
        "fixed": true
      }
    ]
  }
}
```

## Two special cases

- **Order details with more than 100 order lines** renders from a different table.
  To see custom columns there, apply the same configuration to
  `orderline-table-serverimpl`.
- **Node metrics tables** key columns by status code: the `id` must start with `S`,
  and any periods in the status code become underscores.

## Verify

Ask the user to reload the frame and open the table, then confirm the column
position, header text, and values. A blank column is `dataBinding`; a missing
column is `default: false` or a user preference; an unstyled header string is a
translation key that isn't defined.
