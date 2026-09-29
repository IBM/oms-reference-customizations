---
name: buc-add-custom-action
description: Add a custom action to an existing Order Hub page's Actions menu through buc-page-definitions.json, and handle it with an action service.
---

# Add a custom action to an existing page

Page actions are declared in `buc-page-definitions.json`, so this is differential
customization — the IBM page stays untouched and keeps receiving updates.

For an action on a table row rather than the page, configure it in `buc-table-config.json` — see
[buc-customize-table-config](../buc-customize-table-config/SKILL.md).

**Before you start**, follow [buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md).

## 1. Check the page supports actions

Only pages listed in the module's page definitions support custom actions:

```
buc-app-<module>/packages/<module>-shared/assets/buc-app-<module>/buc-page-definitions.json
```

## 2. Declare the action

Create or edit
`packages/<route>/src-custom/assets/custom/buc-page-definitions.json`:

```json
{
  "order-details": {
    "name": "order-details",
    "actions": [
      {
        "id": "custom-action",
        "label": "custom.LABEL_CUSTOM_ACTION",
        "resourceId": "BUCORD0014AT0001"
      }
    ]
  }
}
```

| Property | Notes |
|---|---|
| `id` | Unique. This is the name your service subscribes to |
| `label` | Menu label. Use a translation key |
| `resourceId` | Controls who sees the action. The resource ID must exist and be granted to the user's group |

Add the label to `src-custom/assets/custom/i18n/en.json`:

```json
{ "custom": { "LABEL_CUSTOM_ACTION": "Custom action" } }
```

## 3. Confirm the action appears

Ask the user to reload the frame, open the page, click **Actions**, and confirm the
new entry is in the menu. It has no handler yet, so clicking it is expected to do
nothing.

If it's missing, the `resourceId` isn't granted to their user group.

## 4. Handle the action

Create `src-custom/app/services/custom-action.service.ts` and subscribe to the
action ID:

```ts
import { Injectable } from '@angular/core';
import { ActionProcessorService } from '@buc/common-components';

@Injectable()
export class CustomActionService {
  constructor(private actionProcessorService: ActionProcessorService) {
    this.actionProcessorService.select<any>('custom-action').subscribe(({ params }) => {
      const componentId = params.component;   // the page's component ID
      const pageData = params.data;           // { pageObject, routeData, context }
      // your logic here
    });
  }
}
```

`params.data` carries the page's own data — `pageObject` (details data),
`routeData` (the router object), and `context` where the page has one. Use it
instead of re-fetching what the page already loaded.

## 5. Register the service

Pick the token from the route's own modules:

```
grep -rl "BucActionsModule.forChild" packages/<route>/src/app
```

If it finds a match, a new route action can use `CUSTOM_FEATURE_ACTIONS`. If it
doesn't, use `CUSTOM_ACTIONS`. When overriding an existing action, use the token for the module
that registered it: `CUSTOM_ACTIONS` for `forRoot`, or `CUSTOM_FEATURE_ACTIONS` for
`forChild`.

For a route with `forChild`, register it in `src-custom/app/app-customization.impl.ts`:

```ts
import { CUSTOM_FEATURE_ACTIONS } from '@buc/common-components';
import { CustomActionService } from './services/custom-action.service';

export class AppCustomizationImpl {
  static readonly components = [];
  static readonly providers = [
    CustomActionService,
    { provide: CUSTOM_FEATURE_ACTIONS,
      useValue: [{ name: 'custom-action', action: CustomActionService }], multi: true }
  ];
  static readonly imports = [];
}
```

**Mandatory:** list the service class itself in `providers`, as above. The token only
maps a name to a class. Without the provider, the factory logs an error and leaves
the handler null.

Preserve existing registrations. A second `BucActionsModule.forChild()` replaces
the shipped non-multi feature-action list; the custom multi-provider adds or
overrides handlers by name.

The `name` must match the action `id` in `buc-page-definitions.json` exactly. A
mismatch leaves the menu entry unresponsive, with no error when clicked.

## 6. Verify

Ask the user to reload the frame and run the action, then confirm what happened.
Add a visible result first — a notification via [buc-call-apis](../buc-call-apis/SKILL.md), or
navigation to a page — so there's something to confirm.
