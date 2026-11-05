---
name: cc-customize-shared-component
description: Change a component from a Call Center module's shared library (order-shared, return-shared, customer-shared, alert-shared) for one route using an [extn] attribute selector, or for every route by editing the shared library.
---

# Customize a shared component

Components used by several routes live in each module's shared library, for example
`call-center-order/packages/order-shared/src/lib/components/`. Choose one of these two approaches
to change a shared component.

| Approach | When | Cost |
|---|---|---|
| **Per route** (recommended): copy into the route and switch the selector | Change needed on specific pages | Merge only the copied files |
| **In the shared library**: edit `<short>-shared` directly | Change needed everywhere | Every consuming route must be customized, and you merge the shared code on every upgrade |

If the shared component is a modal opened by an action, use [cc-customize-existing-action](../cc-customize-existing-action/SKILL.md).
If it lives in `@buc/cc-components`, use
[cc-customize-common-components](../cc-customize-common-components/SKILL.md).

**Before you start**, follow [buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md)
for each route involved.

## Per route

Example: change `call-center-summary-notes` on Order details.

1. **Copy** `order-shared/src/lib/components/summary-notes-panel/` to
   `order-details/src-custom/app/custom/summary-notes-panel/`.

2. **Change the selector** to require an `extn` attribute, and rename the class.
   IBM's selector excludes `[extn]`, so the two never clash:

   ```ts
   @Component({
     selector: 'call-center-summary-notes[extn]',   // IBM: 'call-center-summary-notes:not([extn])'
     standalone: false,
     templateUrl: './summary-notes-panel.component.html',
     styleUrls: ['./summary-notes-panel.component.scss']
   })
   export class ExtnSummaryNotesComponent { /* copied code + your changes */ }
   ```

   Check the IBM selector first. Only components with the `:not([extn])` pattern
   support this.

3. **Point the parent at your copy.** Find where it's used:

   ```
   grep -rn "<call-center-summary-notes" call-center-order/packages/order-details/src
   ```

   Override that parent template (copy `order-summary.component.html` into the same
   path under `src-custom`) and add `extn`:

   ```html
   <call-center-summary-notes extn [expanded]="expanded" [summaryDetails]="summaryDetails"
     [ruleSetValues]="ruleSetValues" (afterPricingChange)="refreshData()" [isDetailsPage]="true">
   </call-center-summary-notes>
   ```

4. **Declare it** in `src-custom/app/app-customization.impl.ts`:

   ```ts
   static readonly components = [ExtnSummaryNotesComponent];
   ```

## In the shared library

1. Edit the files in `packages/<short>-shared/src/...` and comment each change.
2. Set `runAsCustomization: true` in `overrides.json` for **every** route that
   should pick up the change, and prepare each one
   ([buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md)).
3. List every one of those routes in `package-customization.json` when you build.

Routes you don't customize keep using IBM's shared code from the base container.

## Verify

Ask the user to reload the frame and confirm your version renders on the target page.

If IBM's version still shows, the parent template wasn't overridden to add `extn`, or
the IBM selector doesn't use the `:not([extn])` pattern. If both render, your selector
is missing `[extn]`.
