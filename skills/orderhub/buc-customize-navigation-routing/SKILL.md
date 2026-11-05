---
name: buc-customize-navigation-routing
description: Navigate from custom Order Hub code to another page, whether it's in the same Angular application or a different one.
---

# Link between pages

Choose the navigation mechanism based on the target application's location. Using the
wrong mechanism commonly leaves links unresponsive without an error.

## Within the same Angular application

Use standard Angular routing; no Order Hub-specific setup is needed:

```ts
import { Router } from '@angular/router';

private router: Router;

static readonly CREATE_EDIT_OPTIMIZATION_ROUTE: string =
  BucConstants.ROOT_TOKEN + '/create-edit-optimization-profile/createOptimizationProfile';

this.router.navigate([CREATE_EDIT_OPTIMIZATION_ROUTE], {
  queryParams: { viewType: 'Create' }
});
```

Build the path from `BucConstants.ROOT_TOKEN` rather than hardcoding the leading
segment.

## Across Angular applications

Each application runs in its own frame, so the router can't navigate between them.
Post a message to the shell instead:

```ts
import { getPostMessageDomain } from '@buc/common-components';

window.top.postMessage({
  action: 'NAVIGATE',
  data: {
    path: '/order-management/workspace',
    queryParams: { }
  }
}, getPostMessageDomain());
```

Always pass `getPostMessageDomain()` as the target origin.

**The target must be registered as a feature with the shell.** Shipped pages
already are. For a page of your own — or an IBM page that isn't registered — add it
to `features.json` first, using `"hidden": true` so it's reachable without
appearing in the menu. See [buc-create-features-json](../buc-create-features-json/SKILL.md).

Navigation to an unregistered path is dropped without an error. If a cross-app link
does nothing, check registration before debugging the call.

## Common target paths

Shipped pages registered with the shell:

| Page | Path |
|---|---|
| Home | `/order-management/workspace-home/workspace/welcome` |
| Workspaces | `/order-management/workspace` |
| Manage saved searches | `/order-management/workspace-manage-favorites/workspace/manage-favorites` |
| Outbound orders search | `/order-management/order-search/order/search` |
| Inbound orders search | `/order-management/order-search/order/search-inbound` |
| Returns search | `/order-management/order-search/order/search-returnorder` |
| Order search results | `/order-management/order-search-result/order/search-result` |
| Order details | `/order-management/order-details/order/order-details` |
| Order line details | `/order-management/order-line-details/order/order-line-details` |
| Work orders search | `/order-management/work-order-search/workorder/search` |
| Work order details | `/order-management/work-order-details/workorder/workorder-details` |
| Outbound shipments search | `/order-management/shipment-search/shipment/search` |
| Shipment details | `/order-management/shipment-details` |
| Inventory search | `/order-management/inventory-search/inventory/search` |
| Inventory search results | `/order-management/inventory-search-results/inventory/results` |
| Inventory item details | `/order-management/inventory-sku-details/inventory/sku` |
| Safety stock rules | `/order-management/inventory/safety-stock-rules` |
| Node search | `/order-management/node-search/node/search` |
| Node details | `/order-management/node-details/node/node-details` |
| Fulfillment landing page | `/order-management/fulfillment-home/sourcing/home` |
| Sourcing tester | `/order-management/fulfillment-sourcing-tester/sourcing/search` |
| Alerts search | `/order-management/workspace-alerts-search/workspace/alert-search` |
| Alert details | `/order-management/workspace-alert-details/workspace/alert-details` |
| Exceptions search | `/order-management/exception` |

For a path not listed, ask the user to navigate there in Order Hub and read the
path from the browser address bar.

## Verify

Ask the user to trigger the link and confirm both that the target page opens and
that any query parameters arrived — a cross-app navigation that lands on the page
but drops its parameters usually means they were nested outside `data`.
