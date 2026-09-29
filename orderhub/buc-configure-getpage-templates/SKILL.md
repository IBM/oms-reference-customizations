---
name: buc-configure-getpage-templates
description: Use getPage-templates.json to retrieve extra attributes from existing getPage API calls without writing code. Use when a custom column or summary field renders blank.
---

# getPage-templates.json

Pages load their data through the `getPage` API, and each API call uses a template
that lists exactly which attributes to fetch. An attribute not in the template is
not in the response — so a column or field bound to it renders blank, with no
error.

**This is the usual cause of a blank custom column or summary field.**
The configuration is correct, but the data was never requested.

You only need this file when the attribute you want isn't already retrieved.

## Where the file goes

`<config-dir>/getPage-templates.json`. Resolve `<config-dir>` with
[buc-resolve-config-placement](../buc-resolve-config-placement/SKILL.md) first. It is
either root-config or `packages/<route>/src-custom/assets/custom/`.

## 1. Find the API and template

Ask the user to follow these steps in their browser:

1. Open the page (or the tab) whose data you're extending, with the **Console**
   tab open.
2. Find the `getPage` message and read the API name and template ID from it.

```
Message: BucCommOmsRestAPIService.getPage():  Invoking API "getAuditList" using template ID "default"
```

## 2. Copy only that template

The application's `getPage-templates.json` holds templates for many APIs. Copy
across **only the one you're extending** — the rest keep coming from the defaults,
and a template you copy is one you now maintain.

The structure groups templates by API name, then by scenario:

```json
{
  "apiName": {
    "scenarioId": { },
    "default": { }
  }
}
```

`default` is used whenever the API is invoked without a scenario name.

## 3. Add the attribute

Reproduce the existing template and add your attribute with an empty string value:

```json
{
  "getAuditList": {
    "default": {
      "AuditList": {
        "Audit": {
          "Modifyts": "",
          "Modifyuserid": "",
          "AuditContextId": "",
          "AuditXml": "",
          "Reference1": "",
          "Reference2": "",
          "Reference3": "",
          "Reference4": "",
          "Reference5": "",
          "Reference6": "",
          "AuditKey": ""
        }
      }
    }
  }
}
```

Here `AuditKey` is the addition. Keep the existing attributes — this replaces the
template rather than merging into it, so anything you drop stops being fetched and
breaks the columns already using it.

Check the OMS API documentation for the full set of attributes available on that
API.

## 4. Bind to it

The attribute name is what `dataBinding` refers to in
[buc-customize-table-config](../buc-customize-table-config/SKILL.md) or [buc-customize-field-details](../buc-customize-field-details/SKILL.md). They must match exactly.

## Verify

Ask the user to reload the frame, open the page, and check the **Network** tab:
the `getPage` response should now contain the attribute. Then confirm the column
or field displays it.

If it's still missing from the response, check the attribute name against the API
documentation. If it's in the response but not on screen, check
`dataBinding`.

For route customization, first complete [buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md).
