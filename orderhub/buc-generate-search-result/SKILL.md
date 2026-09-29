---
name: buc-generate-search-result
description: Generate an Order Hub search results page and its table in one step with the search-result-component schematic.
---

# Generate a search results page

`@buc/schematics:search-result-component` generates two components at once:

- the **search results component**: breadcrumbs, filter panel, page chrome
- a **table component** (`BaseTableComponent`), already embedded in the results
  template by its selector

Pair it with [buc-generate-search-panel](../buc-generate-search-panel/SKILL.md) for the criteria page.

## Collect first

Ask the user for anything missing. Do not guess.

- The route package (for example `shipment-search-result`) and the shared library
- Where the page appears: its own route, or an extension ID on an existing page
- The API that returns the rows, and the JSON path to the row array in its response
- The columns: label and the row attribute each one shows
- Whether all rows come back in one call (client-side paging) or one page at a time

## Before generating

Generated components are route-based customization. **Do not run the schematic** until
you have followed [buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md)
and [buc-wire-custom-assets](../buc-wire-custom-assets/SKILL.md) for the target route, then
used [buc-configure-generated-assets](../buc-configure-generated-assets/SKILL.md) to resolve
`--json-file-path` and `--translation-file-path`. Complete all validation checks in the prerequisite skills before
continuing.

## 1. Run the schematic

From the module root, dry run first with `--dry-run`, then run it:

```
ng g @buc/schematics:search-result-component \
  --name <name> \
  --path packages/<route>/src-custom/app/features/<feature> \
  --table-path packages/<route>/src-custom/app/features/<feature> \
  --json-file-path packages/<route>/src-custom/assets/custom \
  --translation-file-path packages/<route>/src-custom/assets/custom/i18n \
  --shared-lib <module-short>-shared \
  --project <route> \
  --skip-import
```

| Option | Notes |
|---|---|
| `--name` | Required. `my-sample` → `MySampleSearchResultComponent` and `MySampleTableComponent` |
| `--json-file-path` | Required. For customizations: `packages/<route>/src-custom/assets/custom` |
| `--translation-file-path` | Required. For customizations: `packages/<route>/src-custom/assets/custom/i18n` |
| `--shared-lib` | Required. The shared library folder, for example `order-shared` |
| `--path` | Optional. Defaults to `src/app`, not `src-custom`, so pass it |
| `--table-path` | Optional. Defaults to `--path` |
| `--project` | Always pass the route package. Inferring it from the module root can pick the wrong project |
| `--routing` | Optional, default `true`. Updates a routing module only if one exists in the same tree. IBM routing modules are in `src/`, so for customizations add the route yourself |

## 2. Register

`--skip-import` suppresses NgModule registration. Add **both** the results component
and the table to `AppCustomizationImpl.components`, keeping existing entries. See
[buc-register-customization](../buc-register-customization/SKILL.md).

To show it on an existing page, map it to the extension ID and follow "Mount a generated page component" in
[buc-add-content-via-extension-point](../buc-add-content-via-extension-point/SKILL.md#mount-a-generated-page-component).

## 3. Wire the table to data

As generated, the table shows an empty grid: the results template passes only
`[toggleFilter]`, and `fetchTableData()` returns `tableDatadetails`, which is never set.

1. Add `[parentPage]="this"` to the table tag in the results template.
2. Replace the `headers` in `buc-table-config.json` and `TABLE_HEADERS` in the table.
   Remove the `SampleActionId` toolbar and overflow actions and their code.
3. Override `fetchTableData()` to call the API. Read criteria from
   `this.parentPage.searchCriteria` when the query depends on them. Follow
   [buc-generate-table-component](../buc-generate-table-component/SKILL.md), steps 5–6.
4. Wire the refresh icon: `templateData: { onClick: () => this.loadTableAsync() }`.
5. The generated `selectPage()` only updates `pageNo`. For server-side paging, end it with
   `this.loadTableAsync()` and use `this.pageNo` and `this.pageSize` in `fetchTableData()`.

### Client-side paging

If all rows come back in one call, switch the table to
`ClientSidePaginationBaseTableComponent`, then replace the two generated server-side
handlers:

```ts
selectPage(page) {
  this.gotoPage(page);
  this.multiModel.isLoading = false;
}

onColSort(index: number) {
  this.applySortAndPagination();
}
```

## 4. Search session handshake (optional)

The panel publishes to `global-<sessionPrefix>`, and the generated results page reads the
literal `'global-sample-search'`, which matches the generated panel's default. If you change
the panel's prefix, change this string to match, or the page opens with no criteria.

## 5. Verify

Ask the user to reload the frame, run a search, and confirm the results page: the
columns show rows, and paging and sorting work. If the grid is empty, ask them to
check the **Network** tab for the API call and its response shape.

## Going further

- Columns, formatters, and actions → [buc-customize-table-config](../buc-customize-table-config/SKILL.md)
