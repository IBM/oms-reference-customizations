---
name: buc-add-page-to-route
description: Add a new Angular page and child route inside an existing Order Hub/Call Center route package (for example a page in order-search), with breadcrumbs and permission control. For a new route package with its own dev server port and deployment entry, use buc-add-route-package instead.
---

# Add a page to an existing route

This adds an Angular page and child route inside an existing route package,
such as `order-search` in `buc-app-order`. To create a new route package, with its own
dev server port and deployment entry, follow
[buc-add-route-package](../buc-add-route-package/SKILL.md) instead.

The page lives entirely in `src-custom/`. You aren't overriding anything, so this
survives IBM updates. **Before you start**, follow [buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md).

## 1. Generate the component

From the module root, choose an explicit custom parent directory:

```
ng g @schematics/angular:component <page-name> \
  --project <route> \
  --path packages/<route>/src-custom/app/features \
  --standalone=false \
  --skip-import
```

Angular 20 otherwise defaults to standalone components. The explicit false value
makes this component compatible with `AppCustomizationImpl.components`.
`--skip-import` leaves module registration untouched: add the generated component
to that array, preserving existing entries. There is no declaration to remove.

## 2. Add the route

In `src-custom/app/app-customization.impl.ts`:

```ts
import { RouterModule, Routes } from '@angular/router';
import { BucCommonComponentsModule, BucFeatureComponentsModule, BucIconsModule } from '@buc/common-components';
import { BucCommonClassesConnectorResourcePermissionsGuardFn } from '@buc/svc-angular';
// Use the real name from packages/<module-short>-shared/src/public-api.ts,
// e.g. for buc-app-order: OrderSharedModule.
import { SharedModule } from '@buc/<module-short>-shared';
import { MyPageComponent } from './features/my-page/my-page.component';

export const customRoutes: Routes = [
  {
    canActivate: [BucCommonClassesConnectorResourcePermissionsGuardFn],
    path: 'my-page',
    component: MyPageComponent,
    data: { id: 'CUSXXX0001' }      // resource ID that gates this page
  }
];

export class AppCustomizationImpl {
  static readonly components = [MyPageComponent];
  static readonly providers = [];
  static readonly imports = [
    RouterModule.forChild(customRoutes),
    BucCommonComponentsModule,
    BucFeatureComponentsModule,
    BucIconsModule,
    SharedModule
  ];
  static readonly exports = [RouterModule];
}
```

This matches IBM's own search routes. Don't add the module's URL guard (for example
`OrderShipmentUrlGuardService`): it maps only IBM detail-page paths to resource IDs,
so it blocks any other path, including yours.

`data.id` is checked by the permissions guard. Create the resource and grant it
before testing or the route returns no permission.

## 3. Record the route path

Put constants in `src-custom/app/features/<page-name>/custom-constants.ts` so the
route string isn't duplicated across your table config, actions, and breadcrumb:

```ts
export class CustomConstants {
  static readonly ROUTE_MY_PAGE = '/<module>/<route>/<feature>/my-page';
  static readonly ACTION_IDS_MY_PAGE = 'create';
}
```

The full route is `/<module>/<route>/<feature path>/<route path>`. Use this
value as `actionTarget` when linking to the page from a table action
([buc-customize-by-configuration](../buc-customize-by-configuration/SKILL.md)).

## 4. Check it loads

Once the module recompiles, ask the user to navigate to the page and confirm they
see the default `<page-name> works!` text. Resolve any routing or permission
problems before building the page.

## 5. Lay out the page

Use the standard Order Hub/Call Center page shell with a loading state, breadcrumb header,
content rows, and footer. Write templates with Angular's built-in control flow (`@if` / `@else`, `@for`, `@switch`).

```html
@if (initialized) {
  <div class="screen-header">
    @if (breadCrumbList) {
      <buc-page-header
        class="screen-breadcrumb"
        [breadcrumbList]="breadCrumbList"
        [headerOrder]="['breadcrumb']">
      </buc-page-header>
    }
  </div>

  <div class="screen">
    <div class="cds--row"><div class="cds--col"><!-- form --></div></div>
    <div class="cds--row"><div class="cds--col"><!-- table --></div></div>
  </div>

  <div class="screen-footer"><!-- buttons --></div>
} @else {
  <div class="screen-loading">
    <buc-loading [isActive]="true"></buc-loading>
  </div>
}
```

In the component class, resolve translations up front and append the breadcrumb
to the trail the user arrived on:

```ts
export class MyPageComponent implements OnInit {
  public initialized = false;
  public breadCrumbList = [];
  private nlsMap: any = { 'custom.LABEL_MY_PAGE': '' };

  constructor(
    private breadcrumbSvc: BreadcrumbService,
    private translateSvc: TranslateService,
    private route: ActivatedRoute
  ) {}

  async ngOnInit(): Promise<void> {
    await this._initTranslations();
    this._initBC();
    this.initialized = true;
  }

  private async _initTranslations() {
    const keys = Object.keys(this.nlsMap);
    const json = await this.translateSvc.get(keys).toPromise();
    keys.forEach(k => this.nlsMap[k] = json[k]);
  }

  private _initBC() {
    this.breadcrumbSvc.updateLast(this.nlsMap['custom.LABEL_MY_PAGE'], CustomConstants.ROUTE_MY_PAGE);
    this.breadCrumbList = this.breadcrumbSvc.get();
  }
}
```

`BreadcrumbService` comes from `@buc/<module-short>-shared`.

## 6. Footer buttons

```html
<div class="screen-footer">
  <div class="cds--row">
    <div class="cds--col">
      <buc-button class="padding-right--1rem" [type]="'secondary'" [btnSize]="'normal'"
                  (click)="onCancel()">{{ 'custom.LABEL_CANCEL' | translate }}</buc-button>
      <buc-button [type]="'primary'" [btnSize]="'normal'"
                  (click)="onSave()">{{ 'custom.LABEL_SAVE' | translate }}</buc-button>
    </div>
  </div>
</div>
```

Cancel returns to where the user came from:

```ts
onCancel() {
  this.router.navigate([Constants.RESULTS_ROUTE]);
}
```

## 7. Add content and data

- Table → [buc-generate-table-component](../buc-generate-table-component/SKILL.md), embedded as
  `<buc-my-page-table [parentPage]="this"></buc-my-page-table>`
- Detail fields → [buc-generate-summary-component](../buc-generate-summary-component/SKILL.md)
- Strings → `src-custom/assets/custom/i18n/en.json`

Ask the user to reload and confirm the breadcrumb trail and layout before you wire
up data.
