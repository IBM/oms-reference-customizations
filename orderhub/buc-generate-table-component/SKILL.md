---
name: buc-generate-table-component
description: Generate an Order Hub data table with the table-component schematic, then configure its columns and wire it to an API. Use whenever a page needs a table.
---

# Generate a table component

`@buc/schematics:table-component` creates a table with Order Hub's sorting,
pagination, and `buc-table-config.json` support. Use the schematic instead
of writing a table by hand.

## Collect first

Ask the user for anything missing. Do not guess.

- The route package, and where the table appears (a page template or an extension ID)
- The API that returns the rows, and the JSON path to the row array in its response
- Each column: label and the row attribute it shows (check the API's getPage template returns it)
- Whether all rows come back in one call, and any row or toolbar actions

## Before generating

Generated components are route-based customization. **Do not run the schematic** until
you have followed [buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md)
and [buc-wire-custom-assets](../buc-wire-custom-assets/SKILL.md) for the target route, then
used [buc-configure-generated-assets](../buc-configure-generated-assets/SKILL.md) to resolve
`--json-file-path` and `--translation-file-path`. Complete all validation checks in the prerequisite skills before
continuing.

## 1. Pick a base class

| Class | When |
|---|---|
| `ClientSidePaginationBaseTableComponent` | All rows arrive in one API call |
| `BaseTableComponent` | The API returns one page at a time |

## 2. Run the schematic

From the module root (for example `devtoolkit_docker/orderhub-code/buc-app-order`):

```
ng g @buc/schematics:table-component \
  --name <name> \
  --extend <ClientSidePaginationBaseTableComponent|BaseTableComponent> \
  --path packages/<route>/src-custom/app/features/<feature> \
  --json-file-path packages/<route>/src-custom/assets/custom \
  --translation-file-path packages/<route>/src-custom/assets/custom/i18n \
  --project <route> \
  --skip-import
```

| Option | Notes |
|---|---|
| `--name` | Required. `my-sample` → `MySampleTableComponent`, config key `my-sample-table` |
| `--extend` | Always pass it. Without it the schematic stops and prompts for a value |
| `--path` | Parent directory relative to the module. The schematic appends `<name>-table/`, so don't repeat it. Defaults to `src/app`, not `src-custom` |
| `--json-file-path` | Required. For customizations: `packages/<route>/src-custom/assets/custom` |
| `--translation-file-path` | Required. For customizations: `packages/<route>/src-custom/assets/custom/i18n` |
| `--project` | Always pass the route package, e.g. `order-search-result`. Inferring it from the module root can pick the wrong project |
| `--json-file-name` | Optional. Defaults to `buc-table-config.json`; a name that doesn't exist is created |

Paths are relative to the directory where you run the command. Use `--help` on
the schematic to list all options.

## 3. Register the generated components

`--skip-import` suppresses automatic NgModule registration. There is no generated
declaration to remove. Add the component to
`AppCustomizationImpl.components`, keeping the existing entries. See
[buc-register-customization](../buc-register-customization/SKILL.md).
Without the flag, a fresh custom tree can fail with `Could not find an NgModule`.
The schematic writes its JSON immediately; compilation is needed to serve it.

## 4. Place it on the page

The selector is in the generated `.component.ts` (for example
`buc-my-sample-table`). Pass the host page so the table can read its state:

```html
<buc-my-sample-table [parentPage]="this"></buc-my-sample-table>
```

To add it to an existing IBM page instead, map it to the page's extension ID — follow
[buc-add-content-via-extension-point](../buc-add-content-via-extension-point/SKILL.md).

## 5. Define the columns

Replace the generated `headers` array in `buc-table-config.json`:

```json
"headers": [
  { "id": "nodeId",       "name": "custom.LABEL_NODE",      "sortKey": "nodeId",       "dataBinding": "shipNode" },
  { "id": "availableQty", "name": "custom.LABEL_AVAILABLE", "sortKey": "availableQty", "dataBinding": "availableQuantity" }
]
```

- `id` — the column key. Keep it in sync with `TABLE_HEADERS` in the component
- `dataBinding` — dot path to the value on each row returned by `fetchTableData()`
- `name` — a literal, or a translation key defined in `custom/i18n/en.json`

Remove the generated `SampleActionId` toolbar and overflow actions from the JSON, and
the `ACTION_SAMPLE` / `TH_SAMPLE_HEADER` code that refers to them.

Then update `TABLE_HEADERS` in the component to match:

```ts
public readonly TABLE_HEADERS: any = {
  TH_NODE_ID: 'nodeId',
  TH_AVAILABLE: 'availableQty'
};
```

A column with `dataBinding` (or a `formatter`) reads its value straight from the row.
A column without either is filled by `getDataForColumn()`, a `switch` on
`TABLE_HEADERS`. For those columns, a mismatched `id` leaves the cell blank without an
error. When a column is blank, check its `dataBinding` path first, then the IDs.

## 6. Supply the data

The generated `fetchTableData()` returns `tableDatadetails`, which nothing sets, so the
table is empty. Override it to return rows whose keys match your `dataBinding` values.
The generated `ngOnInit` already loads the table, so don't call it again:

```ts
protected fetchTableData(): Observable<any[]> {
  return this.dataService.getOrders(this.enterpriseCode).pipe(   // your service
    tap(rows => this.multiModel.totalDataLength = rows.length),
    catchError(() => { this.multiModel.totalDataLength = 0; return of([]); })
  );
}
```

With `BaseTableComponent`, the rows are one page: set `totalDataLength` to the API's total
record count (for example `TotalNumberOfRecords`), not `rows.length`.

Wire the refresh icon in `initializeTableConfig()`:
`templateData: { onClick: () => this.loadTableAsync() }`.

To combine several calls, use `forkJoin`:

```ts
protected fetchTableData(): Observable<any[]> {
  return forkJoin([this.getReservation(), this.getAvailability()]).pipe(
    map(([reservations, availability]: any[]) =>
      reservations.map((item: any) => ({
        shipNode: item.shipNode,
        availableQuantity: availability.find(a => a.nodeId === item.shipNode)?.qty
      }))
    )
  );
}
```

Always `catchError` on each call and reset `this.multiModel.totalDataLength = 0` so
a failure shows an empty table instead of loading indefinitely. For building
the calls themselves, follow [buc-call-apis](../buc-call-apis/SKILL.md).

## 7. Common adjustments

To remove the row-selection checkbox, add this to the `<buc-table>` element in
the generated HTML:

```html
[showSelectionColumn]="false"
```

Row actions are configured in `buc-table-config.json`, not in code — see
[buc-customize-table-config](../buc-customize-table-config/SKILL.md).

## 8. Verify

Ask the user to reload the frame and open the page, then confirm the columns,
headers, and row data. If the table renders but is empty, ask them to check the
**Network** tab for the underlying API call.

## Going further

Most of what a table can do is configuration, not code:

- Column attributes, formatters, actions, pagination → [buc-customize-table-config](../buc-customize-table-config/SKILL.md)
- An attribute the API isn't returning → [buc-configure-getpage-templates](../buc-configure-getpage-templates/SKILL.md)
