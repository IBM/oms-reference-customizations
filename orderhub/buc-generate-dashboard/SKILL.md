---
name: buc-generate-dashboard
description: Generate an Order Hub dashboard and its widget cards with the dashboard-component and dashboard-widget-component schematics, then define the card layout in dashboard-configs.json.
---

# Generate a dashboard and its widgets

A dashboard is a host component plus one widget component per card. The host owns
the grid and the saved layout. Each widget owns its own data. Generate the host first.

## Collect first

Ask the user for anything missing. Do not guess.

- The route package, and where the dashboard appears (an extension ID or its own route)
- Each card: title, what it shows, and the API and response path that supply the value
- Who may see each card (a resource ID), or confirm everyone may

## Before generating

Generated components are route-based customization. **Do not run the schematic** until
you have followed [buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md)
and [buc-wire-custom-assets](../buc-wire-custom-assets/SKILL.md) for the target route, then
used [buc-configure-generated-assets](../buc-configure-generated-assets/SKILL.md) to resolve
`--json-file-path`. Complete all validation checks in the prerequisite skills before
continuing.

## 1. Generate the host

```
ng g @buc/schematics:dashboard-component \
  --name <name> \
  --dashboard-id <name>-dashboard \
  --dashboard-save-key <save-key> \
  --path packages/<route>/src-custom/app/features/dashboard \
  --json-file-path packages/<route>/src-custom/assets/custom \
  --project <route> \
  --skip-import
```

| Option | Notes |
|---|---|
| `--name` | Suffixed with `Dashboard` automatically — `order-metrics` → `OrderMetricsDashboardComponent` |
| `--json-file-path` | Where `dashboard-configs.json` is created or updated |
| `--json-file-name` | Defaults to `dashboard-configs.json` |
| `--dashboard-id` | Always pass it. It sets the TypeScript dashboard constant. The JSON layout key comes from `--name` |
| `--dashboard-save-key` | Always pass it. The generated service needs it. It sets the saved-layout key |
| `--path` | Where the component files go |

For `--name order-metrics`, the generated JSON key is `order-metrics-dashboard`, so
pass `--dashboard-id order-metrics-dashboard`. Any other value produces a constant
that doesn't match the layout. Omit `--module`: the schematic patches that module
even when `--skip-import` is passed. Neither dashboard schematic accepts `--standalone`.

## 2. Generate each widget

Once per card:

```
ng g @buc/schematics:dashboard-widget-component \
  --name <widget-name> \
  --path packages/<route>/src-custom/app/features/dashboard \
  --project <route> \
  --skip-import
```

Each run creates a widget component and its data service. The schematic is
`dashboard-widget-component`, not `dashboard-widget`. Widgets don't touch
`dashboard-configs.json`; you place them in the layout in step 4.

## 3. Register the host and widgets

The schematic creates **`dashboard.config.ts`**. Extend its existing
`dashboardsComponents: DashboardComponents[]` export, keeping its generated constants:

```ts
import { DashboardComponents } from '@buc/common-components';
import { OpenOrdersWidgetComponent } from './open-orders-widget/open-orders-widget.component';

export const dashboardsComponents: DashboardComponents[] = [
  {
    id: 'order-metrics-dashboard',
    components: [{ id: 'open-orders-widget', component: OpenOrdersWidgetComponent }]
  }
];
```

Each component `id` must match a card `id` in the layout (step 4).

In **`app-customization.impl.ts`**, declare the host and every widget, and import the
dashboard module, keeping existing entries:

```ts
export class AppCustomizationImpl {
  static readonly components = [OrderMetricsDashboardComponent, OpenOrdersWidgetComponent];
  static readonly imports = [BucDashboardModule.forRoot(dashboardsComponents), ButtonModule];
}
```

A widget missing from `components` shows an empty card, with no build error. To place
the dashboard on an existing page, map the host component to the extension ID —
follow [buc-add-content-via-extension-point](../buc-add-content-via-extension-point/SKILL.md).

## 4. Define the layout

Replace the sample card in your entry in `dashboard-configs.json`:

```json
{
  "order-metrics-dashboard": {
    "dashboardID": "order-metrics-dashboard",
    "layout": [
      {
        "id": "open-orders-widget",
        "resourceId": "BUCB2B0001IP0008",
        "cardName": "DASHBOARD.OPEN_ORDERS.TITLE",
        "cardDescription": "DASHBOARD.OPEN_ORDERS.CARD_DESCRIPTION",
        "default": { "max": { "x": 0, "y": 0, "h": 4, "w": 8, "minH": 4, "minW": 4 } }
      }
    ]
  }
}
```

| Property | Notes |
|---|---|
| `id` | Must match the widget's `id` in `dashboard.config.ts` |
| `resourceId` | Controls who sees the card. Omit it to show the card to everyone |
| `cardName` / `cardDescription` | Translation keys, not literal text |
| `default.max` | Grid position and size. `x`/`y` place it, `w`/`h` size it, `minW`/`minH` set the resize floor |

The grid is 16 columns wide (`"w": 16` is full width). Cards flow by `y` then `x`.

## 5. Load and show the widget's data

The generated widget service and template are placeholders. Fill in both.

**Service.** Widgets are re-created on resize and in edit mode, so cache the result. For a
count, call `getPage` with a count template and page size 1:

```ts
import { CommonService, Templates } from '@buc/order-shared';

async getData(): Promise<number> {
  if (this.data === null) {
    const input = { Order: { DocumentType: '0001' } };
    const res: any = await CommonService.getPage(input, Templates.AccountSalesOrdersCount, 1, 1).toPromise();
    this.data = Number(res?.Output?.OrderList?.TotalNumberOfRecords) || 0;
  }
  return this.data;
}
```

For other calls, follow [buc-call-apis](../buc-call-apis/SKILL.md).

**Component.** Show the card (`isScreenInitialized`), then load, tracking loading and errors:

```ts
count: number;
widgetLoading = true;
dataLoadingError = false;

async initialize() {
  this.isScreenInitialized = true;
  try {
    this.count = await this.widgetService.getData();
  } catch (e) {
    this.dataLoadingError = true;
  }
  this.widgetLoading = false;
}
```

**Template.** Replace `text="TITLE"` with a translation key, then fill `<buc-ai-card-content>`:

```html
<buc-ai-card-title [text]="'custom.DASHBOARD.OPEN_ORDERS.TITLE' | translate"></buc-ai-card-title>
...
<buc-ai-card-content>
  @if (widgetLoading) {
    <buc-loading [isActive]="true" size="sm"></buc-loading>
  } @else if (dataLoadingError) {
    <buc-notification [notificationObj]="{ type: 'error', message: ('custom.DASHBOARD.ERROR' | translate), lowContrast: true }"></buc-notification>
  } @else {
    {{ count }}
  }
</buc-ai-card-content>
```

Wrap charts and visualizations in `@if (render)` so resize and edit mode can force a redraw.

## 6. Verify

Ask the user to reload the frame and open the page hosting the dashboard, then confirm:

1. Every card appears, in the positions the layout specifies, and shows its value.
2. Dragging a card and reloading keeps the new position (`--dashboard-save-key` works).

A card that's missing entirely is usually a `resourceId` that isn't granted. A card
that's present but empty is usually a widget missing from `components`.
