---
name: cc-customize-global-search
description: Customize the Call Center global search bar and home page search, including matched fields, query types, type-ahead, URL hiding, and corresponding placeholder text.
---

# Global and home page search

Global search (top banner) and the home page search portlets match orders and
returns on a set of fields. By default, these are `OrderNo`, `CustomerEMailID`, and
`CustomerPhoneNo`, all matched with `FLIKE` (starts with).

| Search box | Config key |
|---|---|
| Global search bar | `searchConfig.<order\|return>.complexQuery.globalSearch` |
| Home page search | `searchConfig.<order\|return>.complexQuery.homeSearch` |

For fields on the Order, Return, or Customer search pages, use
[cc-customize-search-fields](../cc-customize-search-fields/SKILL.md) instead.

## 1. Change the fields and query types

Edit `callcenter-code/shell-ui/assets/app-bootstrap-config.json`. Create the folder
and file if they don't exist, and merge with any existing keys.

```json
{
  "searchConfig": {
    "order": {
      "complexQuery": {
        "globalSearch": [
          { "field": "OrderNo", "qryType": "FLIKE" }
        ],
        "homeSearch": [
          { "field": "OrderNo",         "qryType": "EQ" },
          { "field": "CustomerEMailID", "qryType": "EQ" },
          { "field": "CustomerPhoneNo", "qryType": "EQ" }
        ]
      }
    },
    "return": {
      "complexQuery": {
        "globalSearch": [ { "field": "OrderNo", "qryType": "FLIKE" } ],
        "homeSearch":   [ { "field": "OrderNo", "qryType": "FLIKE" } ]
      }
    }
  }
}
```

`qryType`: `EQ`, `FLIKE`, `LIKE`. Fewer fields and `EQ` give faster searches.

On the results page, the table toolbar keeps the settings from where the search started:
home search uses `homeSearch`, and **View more results** from global search uses
`globalSearch`.

## 2. Match the placeholder text

When the fields change, update the hints so users know what to type.

| Where | File | Keys |
|---|---|---|
| Global search bar | `shell-ui/assets/i18n/en.json` | `categorySearch.Search_placeholder_for_orders`, `categorySearch.Search_placeholder_for_returns` |
| Home portlets | `call-center-custom/custom-home-portlet/src/assets/i18n/en.json` | the portlet's search placeholder |
| Results toolbar (order) | `call-center-order/.../custom/i18n/en.json` | `ORDER_SEARCH.GENERAL.LABEL_TOOLBAR_PLACEHOLDER_GLOBAL`, `ORDER_SEARCH.GENERAL.LABEL_TOOLBAR_PLACEHOLDER` |
| Results toolbar (return) | `call-center-return/.../custom/i18n/en.json` | `RETURN_SEARCH.GENERAL.LABEL_TOOLBAR_PLACEHOLDER_GLOBAL`, `RETURN_SEARCH.GENERAL.LABEL_TOOLBAR_PLACEHOLDER` |

```json
{ "categorySearch": { "Search_placeholder_for_orders": "Search by order number" } }
```

For module literal placement, see [buc-customize-by-configuration](../buc-customize-by-configuration/SKILL.md).
Changing home portlet literals means building the portlet app
([cc-customize-home-portlet-code](../cc-customize-home-portlet-code/SKILL.md)).

## 3. Optional behavior

```json
{
  "globalSearch": { "typeaheadDelay": 500 },
  "deepLinking":  { "removeSearchQueryInUrl": true }
}
```

- `typeaheadDelay` (ms): search as the user types, in the search bar, the customer
  lookup on Create order and Create return, and the results filter. Without it,
  users press Enter.
- `removeSearchQueryInUrl`: keeps the search term out of the browser URL, for
  example when the term is an email or phone number.

## Verify

Ask the user to reload Call Center, search from the banner and from the home page with
each configured field, and confirm the expected matches. The network tab shows the
fields and query types sent.

If the old behavior remains, `searchConfig` sits under the wrong module key (`order` or `return`), or
the JSON is invalid.
