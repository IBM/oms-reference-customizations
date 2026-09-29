---
name: buc-configure-table-export
description: Control the Order Hub table data export feature - what users get from it, and how to turn it off globally or per table for data-governance reasons.
---

# Table data export

Export is **enabled by default on every Order Hub table**. When enabled, the table
shows a download icon and users can export its contents.

The `Table export (ICC000118)` permission gate applies only to Call Center.
Order Hub uses `model.enableTableExport` directly. If its icon is missing, check
the effective table-export configuration and precedence below.

## What users get

From the **Export table data** modal, the user picks a file name and a format:

| Format | Result |
|---|---|
| CSV | One file. If the table has inner tables, a `.zip` with one CSV per table |
| XLSX | One workbook, with inner tables as additional sheets |
| JSON | One file, inner tables included |

These formats work by default and need no configuration.

## Where the setting comes from

Four sources control whether a table can export. They are checked in this order;
the first value that is set wins:

| Priority | Source | Where |
|---|---|---|
| 1 | The table's own custom config | `buc-table-config.json` in your custom assets |
| 2 | The global switch | `tableConfig.enableTableExport` in `app-bootstrap-config.json` |
| 3 | The table's shipped config | IBM's `buc-table-config.json` — don't edit |
| 4 | The product default | Enabled, for Order Hub |

A per-table entry takes precedence over the global switch, and both take precedence
over the shipped configuration. Choose the level that matches the intended scope.

## Turning export off everywhere

One key, not one entry per table. In `shell-ui/assets/app-bootstrap-config.json`
(the same file [buc-inject-custom-html-js](../buc-inject-custom-html-js/SKILL.md)
uses):

```json
{
  "tableConfig": { "enableTableExport": false }
}
```

This is the right tool for a blanket data-governance rule. Individual tables can
still be turned back **on** afterwards with a per-table entry, because per-table
config outranks it.

## Disabling export for one table

Use this when only specific tables carry data that shouldn't leave the
application.

### 1. Choose where the config goes

If the table is on a route you've already taken over, put it in that route:

```
packages/<route>/src-custom/assets/custom/buc-table-config.json
```

Otherwise use root-config, which doesn't take ownership of a route, so future Order
Hub releases keep applying to it:

```
packages/<module-short>-root-config/src/assets/custom/buc-table-config.json
```

### 2. Find the table's schema name

Ask the user to follow these steps in their browser:

1. Open the page with the table, with the **Console** tab open.
2. Find the message containing `BaseTableComponent.initializeTable`.
3. Read the object name from it.

For example, on the outbound order search results page:

```
Message: BaseTableComponent.initializeTable():  Initializing configuration for order-table
```

### 3. Set the property

```json
{
  "orderline-table": {
    "name": "orderline-table",
    "enableTableExport": false
  }
}
```

`name` and `enableTableExport` are all you need. Don't copy the shipped `headers`
array in — the columns merge separately and an entry that only sets export doesn't
need to mention them at all. (IBM's own example includes an empty `"headers": []`;
it's harmless, and omitting it produces the same result.)

Add one entry per table. Confirm the intended scope with the user before adding
entries — if they want export gone everywhere, use the global switch above instead.

### 4. Verify

Ask the user to reload the frame, open each affected table, and confirm the export
icon is gone. Then ask them to check a table you *didn't* change still has it —
that confirms you disabled what you meant to and nothing more.

Then build and deploy as usual.

For route customization, first complete [buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md).
