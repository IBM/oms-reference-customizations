---
name: cc-configure-home-portlets
description: Reorder, resize, or hide Call Center home page portlets (order, customer, return search, alerts) through cc-home-custom-widget-config.json, without writing code.
---

# Home page portlets by configuration

The hub home page renders portlets from `cc-base-widget-config.json` (IBM). You
override entries in `cc-home-custom-widget-config.json`. To view IBM's file, log in
and find `cc-base-widget-config.json` in the network tab.

```json
{
  "widgets": [
    { "id": "search-order",    "fixed": false, "configuration": {}, "colSpan": 1, "sequence": "1", "resourceIds": ["ICC000014"] },
    { "id": "search-customer", "fixed": false, "configuration": {}, "colSpan": 1, "sequence": "2", "resourceIds": ["ICC000050"] },
    { "id": "search-return",   "fixed": false, "configuration": {}, "colSpan": 1, "sequence": "3", "resourceIds": ["ICC000040"] },
    { "id": "alerts",          "fixed": false, "configuration": {}, "colSpan": 2, "sequence": "4", "resourceIds": ["ICC000070"] }
  ]
}
```

| Attribute | Meaning |
|---|---|
| `id` | Portlet ID. Never change it for IBM portlets |
| `sequence` | Ascending order on the page |
| `colSpan` | Columns the portlet spans. IBM: 1, alerts 2 |
| `resourceIds` | Shown only to users with these permissions |
| `extensionType` | **Required in your file.** `CONFIG` for JSON-only changes, `CODE` for code ([cc-customize-home-portlet-code](../cc-customize-home-portlet-code/SKILL.md)) |

## 1. Write the override

Create `cc-home-custom-widget-config.json` and copy in only the portlets you
are changing, with `extensionType: "CONFIG"`:

```json
{
  "widgets": [
    { "id": "alerts",        "fixed": false, "configuration": {}, "colSpan": 2, "sequence": "1", "resourceIds": ["ICC000070"], "extensionType": "CONFIG" },
    { "id": "search-return", "fixed": false, "configuration": {}, "colSpan": 1, "sequence": "4", "resourceIds": ["EXTN_HIDDEN"], "extensionType": "CONFIG" }
  ]
}
```

| Goal | Change |
|---|---|
| Move a portlet | `sequence` |
| Widen or narrow | `colSpan` |
| Hide from everyone | Set `resourceIds` to a dummy ID nobody has |
| Hide from some groups | Use a real resource ID and grant it only to the groups that should see it |

## 2. Put it where the hub reads it

| Environment | Location |
|---|---|
| DTK (mounted hub folder) | `callcenter-code/call-center-hub/assets/cc-home-custom-widget-config.json` |
| DTK (no mount) | Copy to `/opt/app-root/src/call-center-hub/assets/custom/` in `om-call-center-base` |
| Traditional install | `$HTML_DIRECTORY/call-center/call-center-hub/assets/custom/` |
| Deployment package | Keep a copy at `callcenter-code/call-center-hub/` |

```
docker cp cc-home-custom-widget-config.json om-call-center-base:/opt/app-root/src/call-center-hub/assets/custom/
```

## Verify

Ask the user to reload Call Center and check the order and sizes of the home page
portlets. Hidden portlets should be gone for users without the resource ID.

If nothing changes, `extensionType` is missing or the file isn't where the hub reads it.

To style the portlets, see [cc-customize-styles](../cc-customize-styles/SKILL.md).
To add hot keys on the home page, use page key `home-page`
([buc-add-custom-hotkeys](../buc-add-custom-hotkeys/SKILL.md)).
