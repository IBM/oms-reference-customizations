---
name: buc-generate-search-from-config
description: Generate a complete Order Hub search experience from one JSON configuration file, including the search page, results page, table, API service, and translations.
---

# Generate a search experience from config

`@buc/schematics:search-from-config` takes one JSON file and generates the search
panel, the results page, the table, an optional API service, the `search_fields.json`
and `buc-table-config.json` entries, and the translation keys.

Use this for a complete search flow. To generate an individual part, use
[buc-generate-search-panel](../buc-generate-search-panel/SKILL.md), [buc-generate-search-result](../buc-generate-search-result/SKILL.md), or
[buc-generate-table-component](../buc-generate-table-component/SKILL.md).

## Collect first

Ask the user for anything missing. Do not guess.

- The route package, and where the search panel appears (its own route or an extension ID)
- The API, and the JSON paths to the row array and the total count in its response
- The search fields: label, type, and the API attribute each one fills
- The table columns: label and the row attribute each one shows

## Before generating

Generated components are route-based customization. **Do not run the schematic** until
you have followed [buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md)
and [buc-wire-custom-assets](../buc-wire-custom-assets/SKILL.md) for the target route, then
used [buc-configure-generated-assets](../buc-configure-generated-assets/SKILL.md) to resolve
`metadata.paths.jsonFile` and `metadata.paths.translationFile`. Complete all validation checks in the prerequisite skills before
continuing.

## 1. Write the config

Create a file such as `my-search.json` in the repository. `metadata` is required.
`oms-api`, `search`, and `search-results` are each optional.

```json
{
  "metadata": {
    "name": "order-new-search",
    "displayName": "Order Search",
    "module": "order",
    "paths": {
      "component": "packages/order-search/src-custom/app/features/order",
      "table": "packages/order-search/src-custom/app/features/order",
      "jsonFile": "packages/order-search/src-custom/assets/buc-app-order",
      "translationFile": "packages/order-search/src-custom/assets/custom/i18n"
    },
    "options": { "sharedLib": "order-shared", "routing": true, "prefix": "buc" }
  },
  "oms-api": {
    "endpoints": { "search": { "apiName": "getOrderList", "method": "POST" } },
    "requestMapping": { "pageNumber": "PageNumber", "pageSize": "PageSize" },
    "responseMapping": {
      "data": "Output.OrderList.Order",
      "totalCount": "Output.OrderList.TotalNumberOfRecords"
    },
    "requestDefaults": { "Order": { "DocumentType": "0001" } }
  },
  "search": {
    "fileName": "search_fields.json",
    "route": "/order-search/order/order-new-search",
    "breadcrumb": { "label": "Order Search" },
    "saveSearch": { "enabled": false },
    "customizeSearchCriteria": { "preferenceName": "order-new-search" },
    "searchFieldsJson": { "groups": {}, "fields": [] }
  },
  "search-results": {
    "route": "/order-search/order/order-new-search-result",
    "breadcrumb": { "label": "Order Search Results" },
    "saveSearch": { "enabled": true },
    "pagination": { "type": "server-side", "defaultPageSize": 10 },
    "refresh": { "enabled": false },
    "tableConfigJson": {
      "name": "order-new-search-table",
      "headers": [
        { "id": "orderNo", "name": "Order Number", "sortKey": "OrderNo", "dataBinding": "OrderNo" }
      ]
    }
  }
}
```

Key fields:

| Field | Notes |
|---|---|
| `metadata.name` | Base name for everything generated. All other names derive from it |
| `metadata.displayName` | Required |
| `metadata.module` | Optional module basename (without `.module.ts`). Components are declared there only if that module file exists in the output folder; otherwise register them yourself (step 3) |
| `metadata.paths.*` | All required. For customizations, point them into `src-custom` |
| `metadata.paths.jsonFile` | The route's copy of the module assets, `src-custom/assets/<module-name>`. The generated panel reads its fields from `./assets/<module-name>/search_fields.json`; in `assets/custom/` they load only as an overlay and fields without `request` are dropped. The schematic adds its keys and keeps existing entries. Never the shared original in `<module-short>-shared` |
| `options.sharedLib` | Required. For example `order-shared` |
| `options.routing` | Default `true`. Updates a routing module only if one already exists |
| `oms-api` | Without it, the table has no data service and stays empty |
| `oms-api.responseMapping.data` | JSON path to the array in the API response |
| `pagination.type` | `server-side` or `client-side` |
| `tableConfigJson.name` | Ignored. The key and name are always `{name}-table` |

`metadata.name` is normalized to end in `-search`, and every name derives from it.
`order-new` and `order-new-search` both give `order-new-search`, `order-new-search-result`,
`order-new-search-table`, and `order-new-search-table.service`.

## 2. Generate

Run with `--dry-run` first, then without it:

```
ng g @buc/schematics:search-from-config --config=my-search.json --project <route>
```

| Option | Notes |
|---|---|
| `--config` | Required. Path to the JSON file. Prompts if omitted |
| `--dry-run` | Optional. Validates the config without writing files |

This schematic accepts no path flags. Every output location comes from
`metadata.paths` in the config file.

## 3. Finish the generated code

1. **Build the query.** `buildQueryPayload()` in the generated service ignores the
   criteria. Map each field into the request. Criteria arrive as
   `[{ id, options: [{ fieldId, value: { qryType, content } }] }]`:
   ```ts
   const options = (searchCriteria || []).reduce((all, c) => all.concat(c?.options || []), []);
   const orderNo = options.find(o => o.fieldId === 'orderNo')?.value;
   if (orderNo?.content) {
     payload.Order.OrderNo = orderNo.content;
     payload.Order.OrderNoQryType = orderNo.qryType || 'FLIKE';
   }
   ```
2. **Reload on new criteria.** The generated `onSearch()` in the results page is empty,
   so applying filters never reaches the table. Call the table's
   `updateSearchCriteria()`, skipping the first load:
   ```ts
   @ViewChild(OrderNewSearchTableComponent) table: OrderNewSearchTableComponent;
   public async onSearch(): Promise<void> {
     this.hasSearched = true;
     if (this.table && !this.table.initialLoad) { this.table.updateSearchCriteria(this.searchCriteria); }
   }
   ```
3. **Routes.** IBM routing modules are in `src/`, so the generator adds no route. Add
   the search and results routes through `RouterModule.forChild(customRoutes)` in
   `AppCustomizationImpl.imports` — follow [buc-add-page-to-route](../buc-add-page-to-route/SKILL.md).
   The paths must match `search.route` and `search-results.route`.
4. **Register.** Add the three components to `AppCustomizationImpl.components`, and the
   generated service to `providers` when you used `oms-api`, keeping existing entries. See
   [buc-register-customization](../buc-register-customization/SKILL.md). If
   `<metadata.module>.module.ts` exists in the output folder, the generator already declared
   the components and provided the service there. Remove those entries first.

**Optional: session prefix.** The panel, results page, and table all use `sample-search`,
so they match each other. Change all three (and the results page's `'global-sample-search'`)
only if another search flow uses the same prefix.

To show the panel on an existing page, map it to the extension ID and follow "Mount a generated page component" in
[buc-add-content-via-extension-point](../buc-add-content-via-extension-point/SKILL.md#mount-a-generated-page-component).

## 4. Verify

Ask the user to reload the frame, run a search, and confirm results come back. If
the table is empty but the call succeeds, `responseMapping.data` is pointing at the
wrong path. Ask them for the response shape from the **Network** tab.
