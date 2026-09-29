---
name: buc-add-content-via-extension-point
description: Add your own content into an existing Order Hub page or modal through its extension points, keeping your code fully separate from IBM code so product updates don't need a merge.
---

# Customize by differential (extension points)

Every Order Hub component exposes extension points where you can inject a component.
Your code stays in `src-custom/` and never touches IBM code, so IBM updates apply
without a merge. Prefer this over overrides whenever you're *adding* to a page
rather than changing how it works.

**Before you start**, follow [buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md).

## Collect first

Ask the user for anything missing. Do not guess.

- The route and host page component (e.g `order-search`, `OrderSearchComponent`)
- The extension ID (e.g. `order_search_os_bottom`)
- What the section shows, and which host values or API it reads them from

## 1. Find the extension point

Walk the user through these browser steps and ask them for the identifier:

1. In Order Hub, navigate to the page or open the modal you want to extend.
2. Press **Ctrl + D**. The available extension points are drawn on screen with
   their identifiers.
3. Note the identifier nearest to where the content should go (for example
   `schedule_modal_shared_bottom`).
4. Press **Ctrl + Shift + D** to hide them again.

If nothing appears, the module isn't running in DEV mode — go back to
[buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md).

## 2. Build your component

Create it under `src-custom/app/custom/<name>/` (`.html` / `.scss` / `.ts`,
`standalone: false`). Use `buc-*` and Carbon components so it matches the host page.
For tables, fields, search pages, or dashboards, generate the component instead:
[buc-generate-table-component](../buc-generate-table-component/SKILL.md), [buc-generate-summary-component](../buc-generate-summary-component/SKILL.md),
[buc-generate-search-panel](../buc-generate-search-panel/SKILL.md), [buc-generate-dashboard](../buc-generate-dashboard/SKILL.md).

Declare an `@Input()` for every value the component needs from the host.

## 3. Write the extension service

The service connects your component to the host page. Extend `ExtensionService`:

```ts
import { Injectable } from '@angular/core';
import { ExtensionService } from '@buc/common-components';

@Injectable()
export class MyExtensionService extends ExtensionService {
  userInputs: any = { parentPage: null, orderNo: '' };

  /** Runs whenever the host changes. Push host state down into your component. */
  createInput() {
    this.userInputs = { parentPage: this.parentContext, orderNo: this.parentContext?.order?.OrderNo };
    this.userInputObs$.next(this.userInputs);
  }

  /** Expose your handlers back to the host. */
  handleOutput() {
    this.userOutputs = { cancel: this.onCancel.bind(this) };
    this.userOutputObs$.next(this.userOutputs);
  }

  onCancel() {
    this.parentContext.closeModal();
  }
}
```

- `parentContext` is the host component instance — its public state and methods.
- `createInput()` runs whenever the host updates. Read from `parentContext`, then
  `next()` on `userInputObs$`.
- `handleOutput()` publishes callbacks the host can invoke.

To react when the host runs one of its own methods, wrap it. Keep the original and
always call it:

```ts
overrideMethods() {
  if (!this.originalFn && this.parentContext.scheduleOrder) {
    this.originalFn = this.parentContext.scheduleOrder.bind(this.parentContext);
    this.parentContext.scheduleOrder = (event) => {
      this.userInputs.clicked = true;
      this.userInputObs$.next(this.userInputs);
      this.originalFn(event);          // never skip this
    };
  }
}
```

Call `overrideMethods()` from the end of `createInput()`.

## 4. Register the extension

In `src-custom/app/app-customization.impl.ts`, keeping existing entries:

```ts
import { ExtensionModule } from '@buc/common-components';
import { ExtensionConstants } from './features/order/extension.constants';
import { MyComponent } from './custom/my/my.component';
import { MyExtensionService } from './custom/data-services/my-extension.service';

export class AppCustomizationImpl {
  static readonly components = [MyComponent];
  static readonly providers = [];
  static readonly imports = [
    ExtensionModule.forRoot([
      { id: ExtensionConstants.ORDER_SEARCH_OS_BOTTOM, component: MyComponent, service: MyExtensionService }
    ])
  ];
}
```

Use the `ExtensionConstants` member (from the route's `features/<feature>/extension.constants.ts`) matching the identifier from step 1.

## 5. Add the feature module for data services

Services that inject `TranslateService` must be provided from the feature module,
or translation bundles won't be loaded when the service is constructed. Every route
already has `src/app/features/ext-*.module.ts`, imported by class name from the feature
module. The name varies by route, for example `ext-order` or `ext-shipment-search-result`.
Copy that file to the same path under `src-custom/`, keep its file name and class name,
and add your providers:

```ts
@NgModule({
  imports: [CommonModule, TranslateModule],
  providers: [MyDataService]
})
export class ExtOrderModule {}
```

This module is also where you bind the `CUSTOM_ACTIONS` and
`CUSTOM_FEATURE_ACTIONS` injection tokens if you're replacing an action's behavior
— follow [buc-customize-by-overrides](../buc-customize-by-overrides/SKILL.md).

## 6. Verify

Ask the user to reload the frame and open the page, then confirm the section renders
where expected and the original page still works.

Add your strings to `src-custom/assets/custom/i18n/en.json`. For API calls from your
component, follow [buc-call-apis](../buc-call-apis/SKILL.md).

## Keep extension mappings together

`ExtensionModule.forRoot()` provides `EXTENSION_TOKEN` with a single `useValue`,
not a multi-provider. Preserve all route mappings in one `forRoot([...])` array;
a later call in the same injector replaces the earlier list. Constants are per
feature folder, for example `features/order/extension.constants.ts`.

**One component per extension ID.** The directive renders only the first mapping for
an ID. To show several things at one ID, map a wrapper component that contains them.

## Mount a generated page component

Generated search panels and search results are full pages. When you mount one at an
extension point:

1. Add `@Input() parentPage: any;` to it and pass the host through the service's
   `userInputs`. The directive throws on any input the component doesn't declare.
2. In its `prepareBreadCrumbList()`, return early when `parentPage` is set so the
   mounted page does not show a second breadcrumb:
   ```ts
   if (this.parentPage) { this.breadCrumbList = []; return; }
   ```
3. Replace any placeholder route, such as `resultsRoute()` returning `''`, with a real route.

## src-custom cross-tree imports and IDE errors

`src-custom/` is **never compiled directly**. It is merged into `src-merged/` by
`yarn start` before the Angular compiler sees it. Because of this:

- Imports that reference `src/` siblings (for example
  `./features/order/extension.constants`) are **valid** even though the IDE reports
  "Cannot find module". The path resolves correctly once the merge runs.
- **Never edit `src-merged/` by hand** to fix an IDE import error. `src-merged/` is
  generated output — manual edits are overwritten the next time `yarn start` merges
  the trees.
- To confirm the import works, run `yarn start` and check the compiler output
  rather than the IDE squiggle.
