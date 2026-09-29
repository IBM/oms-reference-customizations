---
name: buc-customize-field-details
description: Full buc-field-details.json reference - adding fields to the summary panel of details pages, binding them to API data, and making them editable. Use for order, shipment, inventory, exception, and alert detail pages.
---

# buc-field-details.json reference

Configure a details page's summary panel to add fields, including editable fields
that save changes through an update API, without changing the page's markup.

## Where the file goes

Resolve the custom directory for this route with
[buc-resolve-config-placement](../buc-resolve-config-placement/SKILL.md), and retain
that choice for the file and its translations. For route placement, first complete
[buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md).

## Object names

Each top-level key identifies a summary panel:

| Page | Key |
|---|---|
| Order details | `order-summary` |
| Order line details | `orderline-summary` |
| Order audit details | `order-audit-summary` |
| Order receipt details | `order-receipt-summary` |
| Order release details | `orderrelease-summary` |
| Shipment details | `shipment-summary` |
| Shipment line details | `shipmentline-summary` |
| Shipment container details | `shipment-container-summary` |
| Exception details | `exception-summary` |
| Inventory audit details | `inv-audit-summary` |
| Sourcing test results | `sourcing-test-results-summary` |
| Alert details | `alert-details-summary` |

## Field attributes

| Attribute | Notes |
|---|---|
| `id` | Required. Unique field ID. Components reference the field by this |
| `name` | Label text. Use a translation key |
| `dataBinding` | Attribute to read from the API response, in dot notation (`Order[0].OrderNo`). When present the value is resolved automatically and `getDataForAttribute()` is **not** called for that id — omit it for computed values |
| `value` | Hardcode a value instead. Use `dataBinding` for anything from the API |
| `type` | The declared `BucAttributeFieldDetails.type` union is `label`, `link`, `textbox`, `dropdown`, `combobox`, `combobox_multiselect`, `radio`, `checkbox`, `datepicker`, `labelWithTemplate`, `heading`. Shipped configs also use `node`, `number`, `quantity`, and `textarea`, which render correctly despite not being in the declared union; inspect installed types for additional controls |
| `attrTID` | Optional tag for identifying the element in automated tests |
| `options` | Required for editable types — see below |
| `fullPath` | Required only when the field must appear in the field-config modal: `"<id>.<pageConfigName>"` |
| `formatter` | Same formatter shapes as the table config |

**`dataBinding` only resolves if the attribute is in the API response.** A field
that renders empty is almost always missing from `getPage-templates.json` — see
[buc-configure-getpage-templates](../buc-configure-getpage-templates/SKILL.md).

## Editable fields

Use `options` for editable controls such as `textbox`, `dropdown`, `combobox`, and
`datepicker`. Display-only `heading`, `link`, `labelWithTemplate`, and `label` do
not become editable by adding options:

```json
{
  "id": "serialNo",
  "name": "Serial number",
  "type": "textbox",
  "dataBinding": "SerialNo",
  "options": {
    "editPermission": "OTHERS",
    "editAttr": "SerialNo"
  }
}
```

| Option | Notes |
|---|---|
| `editPermission` | The allowed order modification type for this change |
| `editAttr` | The attribute name sent in the update API request when the user saves |

`editPermission` maps to OMS order modification types. On 10.0.2403.2 and later,
for pages that have no allowed modification types, use `ALWAYS_ALLOW`.

For a dropdown, populate the choices from an API:

```json
"options": {
  "editPermission": "ALWAYS_ALLOW",
  "editAttr": "PaymentStatus",
  "fetch": {
    "api": "getPaymentStatusList",
    "type": "oms",
    "parameters": {},
    "response": {
      "listAttribute": "PaymentStatus",
      "map": { "id": "CodeType", "label": "Description" }
    }
  }
}
```

`type` is always `oms` for customizations. `response.listAttribute` is the array in
the response. `map.id` is the stored value and `map.label` is what the user sees.
**Mandatory:** nest these under `response`. The renderer reads
`fetch.response.listAttribute` and `fetch.response.map`, so a flat `fetch` throws.

## Exposing a field to tenant administrators

A field you add here is on for everyone. To let a tenant administrator switch it
on or off under **Settings > Display settings**, register it in
`buc-tenant-config.json` as well.

## Verify

Ask the user to reload the frame and open the details page, then confirm the label,
the value, and — for an editable field — that a change saves and survives a reload.

If a saved change doesn't persist, check `editAttr` against what the update API expects, then
`editPermission`. If the field is read-only when it shouldn't be, `editPermission`
is the cause.

## Field visibility, order, and value resolution

The array is `attributes`, merged by `id`. Use `hidden: true` to hide an attribute;
omitting it does not delete a shipped field. New fields can use `index`, but the
installed insertion expression uses `cf.index || (baseFields.length + i)`, so
`index: 0` falls back to append. Verify ordering against the installed implementation.
Code can use the generated `applyAttributeOverrides()` and `removeAttributes([ids])`.

Supported field settings include `formatter`, `options.list` (static choices),
`options.fetch` (API choices), `tooltipText`/`tooltipMsg`, `nameTemplate`/`valueTemplate`,
and `parentId`. Values with `dataBinding`, `formatter`, or `options` take the
`getDataForCustomAttribute()` path rather than `getDataForAttribute()`.

Generated summaries extend `EditableFieldRenderer` → `AbstractSummarySection` →
`BaseFieldDetailsComponent` in `@buc/common-components`. `OrderEditableFieldRenderer`
is an application subclass of `EditableFieldRenderer`, not the generated base.
`onFieldDetailsAttributeLoadComplete()` reads `editPermission`/`editAttr` at the top
level or inside `options`, but only processes attributes that have an `options`
object. These JSON keys populate `permissions`/`requestMap`; they are supported.
`dataBinding` already supplies the default request-map path.
