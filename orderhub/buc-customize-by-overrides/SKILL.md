---
name: buc-customize-by-overrides
description: Take over an existing Order Hub component, data service, or action by copying it into src-custom and editing it. Use when configuration and extension points can't do the job.
---

# Customize by overrides

You copy an IBM file into `src-custom/`, edit it, and Order Hub loads yours instead.
You can change it freely, but **you now own that file.** At each DTK release, diff
your copy against the new IBM version and re-merge by hand.

Copy only what the change needs: one component, or preferably one service. Try
configuration ([buc-customize-table-config](../buc-customize-table-config/SKILL.md), [buc-customize-search-fields](../buc-customize-search-fields/SKILL.md),
[buc-customize-field-details](../buc-customize-field-details/SKILL.md)) and [buc-add-content-via-extension-point](../buc-add-content-via-extension-point/SKILL.md) first.

**Before you start**, follow [buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md).

## Override a component

1. Find the IBM source under `packages/<route>/src/app/features/.../`.
2. Recreate that same path under `src-custom/` and copy the files across. Skip
   `.scss` unless you're changing styling — fewer files, less to re-merge.
3. Edit your copy. Fix import paths as you go; red squiggles from relative paths
   clear once the module compiles.

When adding a UI element, copy the markup of a similar element already on the page
rather than writing it from scratch — the surrounding grid classes and `buc-*`
components carry layout and theming:

```html
<div class="combo-box bx--col-md-3">
  <buc-label class="size--sm"
             [label]="'MY_PAGE.MY_SECTION.MY_LABEL' | translate"
             [(inputValue)]="myValue"
             [isDisabled]="isEdit"></buc-label>
</div>
```

Bind to a new field on the component class and add the translation key to
`src-custom/assets/custom/i18n/en.json`.

To trace how an existing field flows end to end, search the component for its
variable name and follow every hit — typically a population method, a submit
method, and the data service. Mirror that same set for your field.

## Override a data service

Data services usually live in the shared library, not the route package, so the
copy moves across packages and the imports need fixing.

1. Create `src-custom/app/data-services/` and copy the service into it, for example
   from `packages/<module-short>-shared/src/lib/data-services/`.
2. Repoint imports that were relative inside the shared lib to its package name:

```ts
import { Templates } from '@buc/<module-short>-shared/lib/constants/order.constants';
import { CommonService } from '@buc/<module-short>-shared/lib/data-services/common-service.service';
```

3. Add your parameter to the request payload in the relevant method.
4. Register it in `src-custom/app/app-customization.impl.ts` against the **original
   IBM class**. Your copy has the same name but is a different class, so on its own
   it replaces nothing:

```ts
import { CreateOrderDataService as IbmCreateOrderDataService } from '@buc/<module-short>-shared';
import { CreateOrderDataService } from './data-services/create-order-data.service';

export class AppCustomizationImpl {
  static readonly components = [];
  static readonly providers = [
    CreateOrderDataService,
    { provide: IbmCreateOrderDataService, useExisting: CreateOrderDataService }
  ];
  static readonly imports = [];
}
```

5. In any component you also overrode, repoint its import to your copy:

```ts
import { CreateOrderDataService } from '../../../../data-services/create-order-data.service';
```

**Mandatory:** verify the new behavior on a component you overrode *and* on one you
didn't. Components that don't inject `IbmCreateOrderDataService` keep the IBM service.

## Override a component from the shared folder

A component in the shared package can't be overridden in place — your copy would
collide with the original, which is still there. Use the **copy into a new
component** approach instead:

1. Copy the whole component from the shared folder into `src-custom/`.
2. **Rename it and its selector.** Prefix the class with `Custom` and make the
   selector unique. Both must differ from the shared original or Angular has two
   declarations of the same thing.
3. Import it and declare it in `app-customization.impl.ts`.
4. Fix its imports. The original used relative paths to its neighbors inside the
   shared folder; now that it lives outside, those become
   `@buc/<module-short>-shared` references.
5. Override the file that *uses* the component and point it at your new selector.

Step 5 puts your version on the page. Renaming alone has no effect because
nothing references the new component yet.

## Override an action

Actions are dispatched by name and resolved through a service, so you can change
what an existing action does without touching the components that trigger it. This
is the smallest override available; use it when possible.

Provide your service against the action name in
`src-custom/app/features/ext-*.module.ts`, using the `CUSTOM_ACTIONS`
token (or `CUSTOM_FEATURE_ACTIONS` for feature-level actions):

```ts
providers: [
  MyDataService,
  ScheduleActionService,
  { provide: CUSTOM_ACTIONS, useValue: [{ name: 'schedule', action: ScheduleActionService }], multi: true }
]
```

Each entry is `{ name, action }` — the action name, and the service implementing it.
**Mandatory:** list the action service class in `providers` too, and keep
`multi: true`. Both tokens are multi-providers, so a non-multi entry fails.

This binding must live in the `ext-*.module.ts` feature module so that the
`TranslateService` injected into the action has its bundles loaded. Copy the route's
existing `src/app/features/ext-*.module.ts` into `src-custom/`, keeping its file and
class name (see
[buc-add-content-via-extension-point](../buc-add-content-via-extension-point/SKILL.md), step 5).

## Verify

Ask the user to reload the frame and run the flow end to end. For anything that
submits, ask them to check the **Network** tab to confirm that the request payload carries your
field and that the response comes back clean.

## Keep the debt visible

Record which files you copied and from which DTK version. At each upgrade, diff the
new IBM file against your copy before merging. The fewer files on that list, the
cheaper every upgrade is.
