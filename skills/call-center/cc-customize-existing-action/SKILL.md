---
name: cc-customize-existing-action
description: Replace the behavior or modal of an existing Call Center action (for example Adjust pricing, Add note, Manage holds) by re-registering the action name with CUSTOM_ACTIONS or CUSTOM_FEATURE_ACTIONS and a copied modal component.
---

# Override an existing action or its modal

IBM actions are services registered under a constant name. To replace IBM's service, register
your own under the **same name**. It usually opens your modified copy of IBM's modal.

This is an override: you own the copied modal and must re-merge it on upgrade. To add
a new action instead, use [buc-add-custom-action](../buc-add-custom-action/SKILL.md).

**Before you start**, follow [buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md)
for the route where the action is used (for example `order-details`).

## 1. Find the action and its token

Root (shared) actions are registered with `BucActionsModule.forRoot` in the module's
shared library, for example
`call-center-order/packages/order-shared/src/lib/actions/order.actions.module.ts`.
Feature actions use `BucActionsModule.forChild` in the route, for example
`packages/order-details/src/app/modules/order-details.action.module.ts`.

```
grep -rn "BucActionsModule.forRoot\|BucActionsModule.forChild" -A40 call-center-order/packages --include=*.ts
```

| Registered with | Override token | Examples (order) |
|---|---|---|
| `forRoot` (shared) | `CUSTOM_ACTIONS` | `ADJUST_PRICING`, `ACTION_ADD_NOTE`, `CANCEL_ORDER`, `MODIFY_CUSTOMER_ADDRESS`, `PAYMENT_ACTIVITY` |
| `forChild` (route) | `CUSTOM_FEATURE_ACTIONS` | `MANAGE_HOLDS`, `APPEASE_CUSTOMER`, `ADDITIONAL_LINE_DETAILS` |

The name is a constant from `Constants` in `@call-center/<short>-shared`.

## 2. Copy the modal

Copy the modal folder from the shared library into the route's `src-custom`:

```
call-center-order/packages/order-shared/src/lib/components/adjust-pricing-modal/
  → call-center-order/packages/order-details/src-custom/app/custom/adjust-pricing-modal/
```

Rename the class (for example `ExtnAdjustPricingModalComponent`), fix its relative
imports to come from `@call-center/order-shared`, and make your changes. Mark each
change with a comment.

## 3. Write the action service

`src-custom/app/custom/actions/extn-adjust-pricing-action.service.ts`:

```ts
import { Injectable, OnDestroy } from '@angular/core';
import { ActionProcessorService } from '@buc/common-components';
import { ModalService } from 'carbon-components-angular';
import { Subscription } from 'rxjs';
import { ActionParams, Constants } from '@call-center/order-shared';
import { ExtnAdjustPricingModalComponent } from '../adjust-pricing-modal/adjust-pricing-modal.component';

@Injectable()
export class ExtnAdjustPricingActionService implements OnDestroy {
  private subscriptions: Subscription[] = [];

  constructor(private actionProcessorService: ActionProcessorService, private modalService: ModalService) {
    this.subscriptions.push(
      this.actionProcessorService.select<ActionParams>(Constants.ADJUST_PRICING)
        .subscribe(({ params }) => this.open(params))
    );
  }

  private open(params) {
    this.modalService.destroy();
    this.modalService.create({
      component: ExtnAdjustPricingModalComponent,
      inputs: {
        modalText: params.data.modalText,
        modalData: { ...params.data.modalData, successCallback: () => this.onSuccess(params) }
      }
    });
  }

  private onSuccess(params) {
    // tell the calling component to refresh
    this.actionProcessorService.dispatchUpdate(params.component, {
      action: Constants.ADJUST_PRICING, refresh: true, success: true
    });
  }

  ngOnDestroy() { this.subscriptions.forEach(s => s.unsubscribe()); }
}
```

Read IBM's original action service for the exact `inputs` its modal expects, and
keep them.

## 4. Register

In the existing `src-custom/app/app-customization.impl.ts`:

```ts
import { CUSTOM_ACTIONS, BucCommonComponentsModule, BucFeatureComponentsModule } from '@buc/common-components';
import { Constants } from '@call-center/order-shared';
import { ExtnAdjustPricingModalComponent } from './custom/adjust-pricing-modal/adjust-pricing-modal.component';
import { ExtnAdjustPricingActionService } from './custom/actions/extn-adjust-pricing-action.service';

export class AppCustomizationImpl {
  static readonly components = [ExtnAdjustPricingModalComponent];
  static readonly providers = [
    ExtnAdjustPricingActionService,
    { provide: CUSTOM_ACTIONS, useValue: [{ name: Constants.ADJUST_PRICING, action: ExtnAdjustPricingActionService }], multi: true }
  ];
  static readonly imports = [BucCommonComponentsModule, BucFeatureComponentsModule];
}
```

Use `CUSTOM_FEATURE_ACTIONS` instead for a `forChild` action. The service class must
also be in `providers`.

The override only applies in routes where you register it. If the same action is
used from several routes, register it in each customized route.

## Verify

Ask the user to reload the frame, run the action, and confirm your modal opens. On
success, the page should refresh as before.

If IBM's modal still opens, the token is wrong (`CUSTOM_ACTIONS` vs `CUSTOM_FEATURE_ACTIONS`) or
the service isn't in `providers`. If the page doesn't refresh, `dispatchUpdate` isn't called.

## Shared components that aren't modals

To change a non-modal shared component (a panel, a widget) for one route, use
[cc-customize-shared-component](../cc-customize-shared-component/SKILL.md).
