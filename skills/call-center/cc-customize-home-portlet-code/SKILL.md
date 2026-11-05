---
name: cc-customize-home-portlet-code
description: Override an IBM home page portlet's code or add a new portlet in Call Center using the custom-home-portlet app in call-center-custom, linked to the cc-components library, then run and build it.
---

# Home page portlets by code

Code portlets run from `call-center-custom/custom-home-portlet`. The hub loads
that app into each portlet marked `extensionType: "CODE"` and passes the portlet
`id` as `homePortletId`. Your template uses this ID to select the portlet.

IBM portlets come from `@buc/cc-components`, whose source is in
`call-center-custom/libs/cc-components/src/lib/modules/cc-home-components/`:

| id | Component | Selector | Folder |
|---|---|---|---|
| `search-order` | `SearchOrderWidgetComponent` | `buc-cc-search-order-widget` | `search-order-widget` |
| `search-customer` | `SearchCustomerWidgetComponent` | `buc-cc-search-customer-widget` | `search-customer-widget` |
| `search-return` | `SearchReturnWidgetComponent` | `buc-cc-search-return-widget` | `search-return-widget` |
| `alerts` | `CcAlertsWidgetComponent` | `buc-cc-alerts-widget` | `alerts-widget` |

## 1. Mark the portlets as CODE

In `cc-home-custom-widget-config.json` (location as in
[cc-configure-home-portlets](../cc-configure-home-portlets/SKILL.md)):

```json
{
  "widgets": [
    { "id": "search-order", "fixed": false, "configuration": {}, "colSpan": 1, "sequence": "1",
      "resourceIds": ["ICC000014"], "extensionType": "CODE" },
    { "id": "extn-invoices", "fixed": false, "configuration": {}, "colSpan": 1, "sequence": "5",
      "resourceIds": ["ICC000014"], "extensionType": "CODE" }
  ]
}
```

A new portlet is always `CODE`. Give it a new, unique `id`.

## 2. Link the cc-components library

Overriding an IBM portlet means editing its source in `libs/cc-components`:

```
cd callcenter-code/call-center-custom
yarn install
yarn cc-components:dev              # watch build into dist/libs/cc-components (leave running)
cd dist/libs/cc-components && yarn link
cd ../../.. && yarn link @buc/cc-components
```

If yarn says the package is already registered, run `yarn unlink` in its previous
location, then link it again. Details are in
[cc-customize-common-components](../cc-customize-common-components/SKILL.md).

## 3. Write the portlet code

`custom-home-portlet/src/app/app.component.html` already has a placeholder block.
Add one block per `CODE` portlet, following its Angular 20 control flow:

```html
<!-- override IBM portlet -->
@if (homePortletId === 'search-order') {
  <div><buc-cc-search-order-widget></buc-cc-search-order-widget></div>
}

<!-- new portlet -->
@if (homePortletId === 'extn-invoices') {
  <div><extn-invoices-widget></extn-invoices-widget></div>
}
```

- **Override:** edit the component under `libs/cc-components/.../search-order-widget/`.
  If it uses `BucNotificationService` and errors at runtime, add the needed providers
  to that component's `providers`.
- **New portlet:** create the component in `custom-home-portlet/src/app/` and
  declare it in the app module.

Labels go in `custom-home-portlet/src/assets/i18n/en.json`.

## 4. Run it

```
yarn start-home       # https://localhost:9200, port set in call-center-custom/package.json
```

Point the hub at it with `import-map.json` in the hub mount path
(`callcenter-code/call-center-hub/assets/`), or in
`/opt/app-root/src/call-center-hub/assets/custom/` in `om-call-center-base`:

```json
{ "imports": { "custom-home-portlet": "https://localhost:9200/call-center-home/custom-home-portlet/main.js" } }
```

Open that URL once to accept the certificate, then reload Call Center.

## 5. Build for deployment

1. In `call-center-custom/package.json`:

   ```json
   "package-customization": "yarn cc-components:prod && yarn build && yarn build-home",
   "postinstall": "yarn link @buc/cc-components"
   ```

2. **Delete** the development `import-map.json` from the hub mount path.
3. Keep `cc-home-custom-widget-config.json` under `callcenter-code/call-center-hub/`.
4. From `callcenter-code/container-build`:

   ```
   ./build-customization.sh build-ui call-center-custom
   ./build-customization.sh package-jar call-center-custom
   ```

If only `CONFIG` portlets changed, skip this build.

## Verify

Ask the user to reload Call Center and confirm each `CODE` portlet renders your
version.

If the portlet is empty, the `homePortletId` in `app.component.html` doesn't match the widget
`id`, or `import-map.json` doesn't point at `localhost:9200`. If IBM's version still shows,
`extensionType` isn't `CODE`.
