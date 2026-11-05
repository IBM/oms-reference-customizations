---
name: cc-configure-address-formats
description: Set per-country address display order, mandatory fields, contact fields, and numeric fields for Call Center address pages with buc-address-field-config.json.
---

# Configure address formats

`buc-address-field-config.json` controls address display and validation for each
country. This requires only JSON, so no route ownership is needed.

## 1. Create the file

`packages/<short>-root-config/src/assets/custom/buc-address-field-config.json`, or the
route's `src-custom/assets/custom/` if that route is already customized by code.

## 2. Define the formats

```json
{
  "addressFormats": [
    {
      "id": "de-DE",
      "displayOrder": {
        "preference1": "FirstName LastName",
        "preference2": "Company",
        "preference3": "AddressLine1",
        "preference4": "AddressLine2",
        "preference5": "ZipCode City",
        "preference6": "Country"
      },
      "mandatoryFields": "FirstName,ZipCode",
      "contactInfo": ["FirstName", "LastName", "EMailID", "DayPhone"],
      "numberFields": "DayPhone,EveningPhone"
    }
  ]
}
```

| Property | Notes |
|---|---|
| `id` | A locale defined in OMS. The part after the dash is matched to the address's `Country`; with no match, the first entry is used |
| `displayOrder` | Six display lines. Separate fields on a line with spaces |
| `mandatoryFields` | Comma-separated. Required when creating or editing |
| `contactInfo` | Fields shown as contact information |
| `numberFields` | Comma-separated. Treated as numeric for editing and validation |

## Verify

Ask the user to open an address whose `Country` matches the entry (for example `DE`) on an
order or customer, and confirm the line layout. The user's own locale has no effect. Then have them leave a mandatory
field empty while editing the address and confirm that saving is blocked.
