---
name: buc-add-custom-tab
description: Add a custom tab to an existing Order Hub details page through buc-page-definitions.json, without taking ownership of the page.
---

# Add a custom tab to an existing page

Tabs are declared in `buc-page-definitions.json`, so this is differential
customization. You don't copy the IBM page, so it continues to receive IBM updates.

**Before you start**, follow [buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md).

## 1. Check the page supports tabs

Only pages listed in the module's page definitions support custom tabs:

```
buc-app-<module>/packages/<module>-shared/assets/buc-app-<module>/buc-page-definitions.json
```

If the page isn't there, tabs aren't available for it — use
[buc-customize-by-overrides](../buc-customize-by-overrides/SKILL.md) instead.

## 2. Declare the tab

Create or edit
`packages/<route>/src-custom/assets/custom/buc-page-definitions.json`:

```json
{
  "order-details": {
    "name": "order-details",
    "actions": [],
    "tabs": [
      {
        "resourceId": "BUCORD0014IP0001AT0009",
        "heading": "custom.LABEL_CUSTOM_TAB",
        "value": "custom-tab",
        "id": "order-details-custom-tab",
        "content": null,
        "token": "custom-tab-token"
      }
    ]
  }
}
```

| Property | Notes |
|---|---|
| `name` | Required. Must match the page key (`order-details`). Without it the whole entry is silently ignored |
| `resourceId` | Controls who sees the tab. The resource ID must exist and be granted to the user's group |
| `heading` | Tab label. Use a translation key |
| `value` | Unique string stored in context while the tab is active |
| `id` | Unique ID for the tab |
| `content` | Keep `null`. The framework fills it with your component's `tabContent` when the tab opens |
| `token` | Injection token your component registers under |
| `showAfterTab` | Optional. ID of the tab to sit after; otherwise appended last |

Add the label to `src-custom/assets/custom/i18n/en.json`:

```json
{ "custom": { "LABEL_CUSTOM_TAB": "Custom tab" } }
```

## 3. Confirm the tab appears

Before building content, check the tab itself renders. Ask the user to reload the
frame, open the page, and confirm the tab is in the tab bar in the expected
position.

If it doesn't appear, see [Diagnose missing tabs](#diagnose-missing-tabs).

## 4. Build the tab content

Create the component under `src-custom/app/features/<feature>/custom-tab/`.

The class must expose `tabContent` as a template reference and accept `data`:

```ts
import { Component, Input, TemplateRef, ViewChild } from '@angular/core';

@Component({
  selector: 'buc-custom-tab',
  templateUrl: './custom-tab.component.html',
  styleUrls: ['./custom-tab.component.css'],
  standalone: false   // required: Angular 20 defaults to standalone, which can't go in `components`
})
export class CustomTabComponent {
  @ViewChild('tabContent', { static: true }) public tabContent: TemplateRef<any>;
  @Input() data;
}
```

Keep `static: true`. The framework reads `tabContent` as soon as it creates the
component, before change detection runs, so with `static: false` the tab is blank.

The template must wrap everything in an `ng-template` named `tabContent`:

```html
<ng-template #tabContent>
  <h1>This is the custom tab content</h1>
</ng-template>
```

Both are required. If the `ng-template` is missing or its reference is renamed,
the tab opens blank.

`data` is always `{ pageObject, routeData, context }`: the page's details, the
route data, and the page context. Read from it rather than re-fetching. IBM's own tabs
take `@Input() details` instead. Don't copy that input when using one as a reference.

## 5. Register the component

In `src-custom/app/app-customization.impl.ts`, add it to **both** `components` and
`providers`, keyed by the token from step 2:

```ts
import { CustomTabComponent } from './features/order/custom-tab/custom-tab.component';

export class AppCustomizationImpl {
  static readonly components = [CustomTabComponent];
  static readonly providers = [
    { provide: 'custom-tab-token', useValue: CustomTabComponent }
  ];
  static readonly imports = [];
}
```

The `provide` value must match `token` in `buc-page-definitions.json` exactly.

## 6. Verify

Ask the user to reload the frame, open the page, and click the new tab — your
content should render. If the tab is there but empty, check the token match first,
then the `ng-template` reference name.

For a table or fields inside the tab, use [buc-generate-table-component](../buc-generate-table-component/SKILL.md) or
[buc-generate-summary-component](../buc-generate-summary-component/SKILL.md).

## Diagnose missing tabs

None of these failures raise an error on the page.

- **No tab at all**:
  - Check that `name` matches the page key.
  - Check that the user is granted `resourceId`.
  - Check `showAfterTab`. An ID that matches no existing tab drops your tab rather than
    appending it. Remove `showAfterTab`, or take a real ID from the page's
    `prepare*Tabs()` method in `src/`.
- **Tab shows but is blank**:
  - Ask the user to check the browser console, where component errors are logged.
  - Confirm the provider token matches `token` exactly.
  - Confirm the `#tabContent` reference name and `static: true`.
