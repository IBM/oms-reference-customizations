---
name: cc-customize-customer-create-fields
description: Add optional or mandatory fields to the Call Center customer creation forms, which come from the @buc/cc-components library.
---

# Add fields to customer creation

The customer creation forms (for example `business-create-org-details`) come from
`@buc/cc-components`. Optional fields only need JSON configuration. Mandatory fields also need a
form control in the library.

## 1. Declare the field

Declare the field in `buc-field-details.json`. An optional field can go in root-config. A mandatory field means the route is customized for the library change, so put it in that route's `src-custom/assets/custom/` ([buc-resolve-config-placement](../buc-resolve-config-placement/SKILL.md)):

```json
{
  "business-create-org-details": {
    "name": "business-create-org-details",
    "attributes": [
      {
        "id": "extnVatNo",
        "name": "CUSTOM.CUSTOMER.LABEL_VAT_NO",
        "value": "",
        "type": "label",
        "dataBinding": "Customer.VatNumber",
        "index": 4,
        "options": { "displayEmptyValue": true }
      }
    ]
  }
}
```

For an **optional** field, `"type": "label"` is all you need. Skip to Verify.

## 2. Make it mandatory

Set `"type": "labelWithTemplate"`, then change the library.

**Before you start**, follow [cc-customize-common-components](../cc-customize-common-components/SKILL.md)
to build and link `@buc/cc-components`.

In `libs/cc-components/src/lib/modules/cc-feature-components/customer-create/organization-details/organization-details.component.ts`:

```ts
// in init
this.parentForm.addControl('extnVatNo',
  new UntypedFormControl(this.customerObj?.Customer?.VatNumber ?? '', [Validators.required, isEmptyValidator]));

// in getDataForAttribute
case 'extnVatNo':
  data = { template: this.requiredTextfieldTpl, templateData: { name: id } };
  break;
```

`requiredTextfieldTpl` is the template IBM uses for `orgName`. Add your own in the HTML
if you need a different control.

## Verify

Ask the user to open **Create customer**, confirm the field appears in the expected position, then try
to save with it empty. A mandatory field should block the save. Then check the network
tab to confirm the value reaches the create request. If it doesn't, extend the mashup
input ([cc-extend-mashups](../cc-extend-mashups/SKILL.md)).
