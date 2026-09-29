---
name: buc-customize-search-fields
description: Full search_fields.json reference - field types, populating options from APIs, query operators, conditional display with showWhen, and targeting the right search API.
---

# search_fields.json reference

Configure search forms to add criteria such as pickers, query fields, and date
ranges, and choose which search each applies to, without writing code.

## Where the file goes

Resolve the custom directory for this route with
[buc-resolve-config-placement](../buc-resolve-config-placement/SKILL.md), and retain
that choice for the file and its translations. For route placement, first complete
[buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md).

## Field types

| Type | What the user gets |
|---|---|
| `label` | Display-only label; no request path required |
| `text` | Free text |
| `number` | Numeric input |
| `date` | A single date |
| `dateRange` | From and to dates |
| `dropdown` | Pick from a list. Single or multiple |
| `comboBox` | Type or pick from suggestions. Single or multiple |
| `radio` | One of several. `orientation` is `horizontal` (default) or `vertical` |
| `toggle` | Two modes |
| `dropdownQuery` | A value plus an operator — Is, Contains, Starts with |
| `comboBoxQuery` | Combo box plus an operator |
| `nodeQryPicker` | Node text plus an operator |
| `itemPicker` | Pick from items |
| `nodePicker` | Pick from nodes |
| `addressPicker` | Search and select an address |
| `organizationPicker` | Search and select an organization |

The `*Query` types let users choose how to match, usually avoiding the need for
three separate fields.

## Core attributes

| Attribute | Notes |
|---|---|
| `label` | Field label |
| `type` | One of the above |
| `request` | Nonblank API request path required for new fields except `type: "label"`; missing paths are silently filtered out |
| `insertAfter` | Position after the referenced field ID; supported by `CustomField` and `FieldSorter` |
| `target` | Which search API the value applies to |
| `operator` | `EQ` (equal), `LIKE` (contains), `FLIKE` (starts with) |
| `value` | What gets sent, based on the input |
| `oobSeq` | Position in the form |
| `locked` | Keeps the field in the non-reorderable part of the form |
| `showWhen` | Regular expression controlling when the field appears |

`request` must match an input property of the API named by `target`. With a mismatch,
the field renders and accepts input, but its value never reaches the payload.

For a query field, `operator` sets the operator dropdown's default.

## Choosing target

`target` decides which search API receives the value. For order search pages:

| Target | Applies when "Search for" is | API |
|---|---|---|
| `orders` | Order | `getOrderList` |
| `lines` | Order lines | `getOrderLineList` |
| `releases` | Order releases | `getOrderReleaseList` |
| `receipts` | Receipts | `getOrderReceiptList` |
| `alerts` | Alerts in the results tab | the orders API |
| `all` | every search | — |

`target: "all"` combined with `showWhen` is a useful pattern: the request applies
everywhere, but the field only renders for the criteria you choose, so a user can
only set it where it makes sense.

```json
{ "request": "some-order-line-attribute", "showWhen": "^order-line.*", "target": "all" }
```

## Conditional display

`showWhen` matches `<search-for>.<search-by>`:

- `inbound-order-receipt.*` — when searching for Order receipt
- `inbound-order-receipt.Item` — Order receipt, searching by Item
- `always` — regardless of criteria

Order search identifiers:

| Search-for | Search-by |
|---|---|
| `outbound-order` | Status, Item, Date, Address |
| `order-line` | Status, Item, Date |
| `order-release` | Status, Item, Date, Logistics |
| `inbound-order` | Status, Item, Date |
| `inbound-order-line` | Status, Item, Date |
| `inbound-order-receipt` | Item, Receipt |

Shipment search identifiers:

| Search-for | Search-by |
|---|---|
| `shipment` | Status, Item, Date, Carrier |
| `shipment-container` | Status, allAttributes |
| `inbound-shipment` | Status, Item, Date, Carrier |
| `inbound-shipment-container` | allAttributes |

## Populating options

Use `list` for static options or `fetch` to load them from an API. API loading is
available only on `dropdown`, `dropdownQuery`, and `radio`:

```json
"fetch": {
  "api": "getQueryTypeList",
  "type": "oms",
  "parameters": {},
  "populate": "qryType",
  "response": {
    "listAttribute": "StringQueryTypes.QueryType",
    "map": { "id": "QueryType", "label": "QueryTypeDesc" }
  }
}
```

`populate` is `list` (the values) or `qryType` (the operator dropdown).

**One `fetch` per field.** On a query field you populate either the values or the
operators from the API — the other must be hardcoded:

```json
"queryList": [
  { "id": "EQ",    "label": "Is" },
  { "id": "FLIKE", "label": "Starts with" },
  { "id": "LIKE",  "label": "Contains" }
]
```

To translate fetched labels, add `translation` inside `fetch`: `key` names the
attribute to translate, `prefix` is prepended to build the translation key.

```json
"translation": { "prefix": "my.prefix_", "key": "label" }
```

## Worked example

```json
{
  "orders": {
    "fields": [
      {
        "label": "Alias value",
        "type": "dropdownQuery",
        "target": "orders",
        "request": "ItemAliasList.ItemAlias.AliasValue",
        "operator": "LIKE",
        "oobSeq": 3.1,
        "showWhen": "outbound-order.Item",
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

## Verify

Ask the user to reload the frame, open the search page, and confirm the field
appears — and, if you used `showWhen`, that it appears only under the right
criteria. Then have them run a search with it filled in and check the **Network**
tab to confirm that the payload carries the attribute.

Field renders but never reaches the payload → `request` or `target`. Field never
renders → `showWhen`. Empty dropdown → the `fetch` response mapping.

## Overriding a shipped field

Match its existing `id`. The loader applies exactly `hidden`, `oobSeq`, and a
shallow merge of `internalConfig`; other top-level custom-field properties do not
replace shipped behavior. Use the simpler CustomField schema only for new IDs.
