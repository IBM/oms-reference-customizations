---
name: cc-customize-search-fields
description: Add, hide, or override search fields on the Call Center order, return, customer, alert, and enhanced order search pages with search_fields.json, including fetched drop-downs, operators, filter groups, and facets.
---

# Customize search fields

Search pages are driven by `search_fields.json`. The IBM defaults for each module
are in:

```
packages/<short>-shared/assets/call-center-<module>/search_fields.json
```

Read that file first to find the field `id`s you need for `insertAfter`,
hiding, and overriding fields.

## Where your file goes

| Situation | Path |
|---|---|
| Search route not customized by code (usual case) | `packages/<short>-root-config/src/assets/custom/search_fields.json` |
| Search route is customized by code | `packages/<route>/src-custom/assets/custom/search_fields.json` |

Add only your changes. Don't copy the whole IBM file.

## Structure

The top-level keys map to modules: `orders`, `returns`, `customers`, `alerts`,
plus `enhanced-order-search` for the enhanced search page. Each has a `fields`
array and optionally `groups`.

## Add a field

```json
{
  "orders": {
    "fields": [
      {
        "id": "displayCreditCardNo",
        "label": "CUSTOM.SEARCH.LABEL_CARD_LAST4",
        "type": "text",
        "request": "PaymentMethod.DisplayCreditCardNo",
        "target": "orders",
        "insertAfter": "postalCode",
        "helperText": "Use with name or email for better results"
      }
    ]
  }
}
```

| Property | Notes |
|---|---|
| `id` | Unique |
| `label` | Translation key or text. Put keys in `custom/i18n/en.json` |
| `type` | `text`, `number`, `date`, `dropdown`, `dropdownQuery`, `radio`, `toggle` |
| `request` | Dot-path in the search API input that receives the value |
| `target` | `orders`, `returns`, `customers`, `alerts`, or `all` |
| `insertAfter` | `id` of an IBM field. You can't anchor to another custom field. Default: end of page |
| `filterId` | Which "Search by" group the field shows in (for example `OrderInfo`, `CustomerInfo`) |
| `operator` | Default operator: `EQ` (is), `LIKE` (contains), `FLIKE` (starts with) |
| `orientation` | `horizontal` (default) or `vertical`, for `radio` |
| `value` | Default value |
| `list` | Static options: `[{ "id": "Y", "label": "Yes", "selected": "true" }]` |
| `fetch` | Options from a mashup, for `dropdown`, `dropdownQuery`, `radio` only |
| `helperText` | Text under the field |
| `hidden` | `true` hides the field |

## Options from an API

```json
"fetch": {
  "api": "icc.order.summary.getPaymentTypeList",
  "type": "mashup",
  "parameters": {
    "PaymentType.CallingEnterpriseCode": "{{enterprise}}",
    "PaymentType.CallingSellerOrganizationCode": "{{enterprise}}"
  },
  "translation": { "prefix": "ORDER_SEARCH.GENERAL.LABEL_QUERY_", "key": "label" },
  "response": {
    "listAttribute": "PaymentTypeList.PaymentType",
    "map": { "id": "PaymentType", "label": "PaymentTypeDescription" }
  }
}
```

`translation` is optional. It builds the label key as `prefix + <label value>`.

For a text box with an Is / Contains / Starts with selector, use `"type":
"dropdownQuery"` with `icc.order.order-search.getQueryTypeList` as the fetch API,
mapping `QueryTypeList.StringQueryTypes.QueryType`.

## New filter group

```json
{
  "orders": {
    "groups": { "paths": { "searchBy": [ { "content": "Search by invoice", "value": "InvoiceInfo" } ] } },
    "fields": [
      { "id": "searchByInvoiceInfo", "placementOnly": true, "type": "label", "label": "Search by invoice", "filterId": "InvoiceInfo" },
      { "id": "invoiceNo", "label": "Invoice number", "type": "text", "request": "InvoiceNo", "target": "orders", "showWhen": "always", "filterId": "InvoiceInfo" }
    ]
  }
}
```

## Override or hide an IBM field

Reference the IBM `id` and put changes in `internalConfig`:

```json
{ "id": "postalCode", "internalConfig": { "helperText": "Combine with name or email" } },
{ "id": "entryType", "internalConfig": { "id": "entryType", "hidden": true } }
```

`internalConfig` accepts `title`, `type` (`input`, `datePicker`, `dropdown`,
`dropdownQueryNumeric`, `radioFieldGroup`), `filterId`, `default`, `helperText`,
`placeholder`, `tooltipText`, `canReset`, and `hidden`. `orderLineStatus` and
`orderTotal` are `custom` types. Override their `options` or `rangeSet` as shipped.

## Enhanced order search

`enhanced-order-search` adds `type` values `dateRange` and `dropdownQueryNumeric`
(with `operators`), and lets you change the **Refine by** facets:

```json
{
  "enhanced-order-search": {
    "groups": {
      "facetAggregates": [
        { "name": "StatusFacet", "sequence": 2 },
        { "name": "ChannelFacet", "sequence": 5, "shown": false },
        { "name": "CustomerFacet", "label": "Customer zip code", "field": "CustomerZipCode",
          "type": "TERMS", "sort": { "field": "_key", "type": "ASC" }, "sequence": 1 }
      ]
    },
    "fields": [
      { "id": "loyaltyNumber", "label": "EXTN_ORDER_SEARCH.LABEL_LOYALTY_NUMBER", "type": "text",
        "request": "CustomerRewardsNo", "showWhen": "always", "locked": true,
        "operators": [
          { "content": "ENHANCED_ORDER_SEARCH.GENERAL.LABEL_CONTAINS", "value": "LIKE", "selected": true },
          { "content": "ENHANCED_ORDER_SEARCH.GENERAL.LABEL_EQUAL", "value": "EQ" }
        ] }
    ]
  }
}
```

Facet `type` is `TERMS` (group by exact value) or `DATE_RANGE`. Copy the IBM facet
names from the shipped file to reorder or hide them.

## Make the API accept the value

The search mashup must accept the `request` attribute. Extend it incrementally.
For example, for order search results:

```xml
<mashup id="icc.order.order-search-results.getPaginatedOrderList" ...>
  <API Name="getOrderList"><Input>
    <Order><PaymentMethod DisplayCreditCardNo=""/></Order>
  </Input></API>
</mashup>
```

See [cc-extend-mashups](../cc-extend-mashups/SKILL.md).

## Verify

Ask the user to reload the frame, open the search page, and confirm the field appears
in the expected group and position. Then have them run a search with it filled in and
check the network tab: the request payload must carry the attribute.

If a field renders but never reaches the payload, check `request`, `target`, and the mashup input.
An empty drop-down points to the `fetch` response mapping. If a field is missing, `insertAfter` points at
a custom field, or `filterId` names a group that doesn't exist.

For global and home page search (not these pages), see
[cc-customize-global-search](../cc-customize-global-search/SKILL.md).
