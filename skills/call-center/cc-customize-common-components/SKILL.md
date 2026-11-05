---
name: cc-customize-common-components
description: Change a component in the shared @buc/cc-components library (customer create, address edit, payment, store search, home portlets, and more), link the rebuilt library into the Call Center modules that use it, and package it with those routes.
---

# Customize @buc/cc-components

Components used across modules (customer creation, address display and edit,
manage payment method, store search, gift options, the iframe loader, home
portlets) come from `@buc/cc-components`. Every module depends on a prebuilt copy:

```
"@buc/cc-components": "file:../lib/buc/cc-components/cc-components-<version>.tgz"
```

The source is in `call-center-custom/libs/cc-components/src/lib/modules/`:

| Folder | Contents |
|---|---|
| `cc-feature-components/` | `customer-create`, `customer-details`, `address-edit`, `address-display`, `manage-payment-method`, `store-search-result`, `gift-options`, `override-price-modal`, `iframe-loader`, ... |
| `cc-home-components/` | Home page portlets |

Changing even one component makes you responsible for maintaining the whole
library and merging it at every upgrade. Try JSON, extension points, or an `[extn]` override
in the route first.

## 1. Build and link the library

```
cd callcenter-code/call-center-custom
yarn install
yarn cc-components:dev            # watch build to dist/libs/cc-components, leave it running
```

In a second terminal:

```
cd callcenter-code/call-center-custom/dist/libs/cc-components
yarn link                         # "success Registered @buc/cc-components"
```

If yarn says it's already registered, run `yarn unlink` in the previous location,
then `yarn link` again.

## 2. Make each consuming module use your build

In each module that uses the changed component (for example
`call-center-customer`), add to `package.json` scripts:

```json
"postinstall": "yarn link @buc/cc-components"
```

Then:

```
yarn install
```

Customize the routes that render the component: `runAsCustomization: true` and the
environments flag
([buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md)).
Then run `yarn start-app`.

## 3. Edit

Change files under `call-center-custom/libs/cc-components/src/lib/modules/...`.
The watch build rebuilds `dist`, and the linked modules pick up the changes. Comment every
change for the upgrade merge.

## 4. Build for deployment

1. List each consuming route in its module's `package-customization.json`:

   ```json
   { "repositoryName": "call-center-order", "customizationContextRoot": "/call-center-customization",
     "routes": { "order-search": { "type": "code" } } }
   ```

2. Build the library for production and relink:

   ```
   cd callcenter-code/call-center-custom
   yarn cc-components:prod
   cd dist/libs/cc-components && yarn link
   ```

   Check that the link points at this build. If not, `yarn unlink` and then
   `yarn link`.

3. Build the consuming modules. If `call-center-custom` only holds library changes
   (no payment or portlet code), **leave it out** of the module list:

   ```
   ./build-customization.sh build-ui call-center-order,call-center-customer
   ./build-customization.sh package-jar call-center-order,call-center-customer
   ```

The rebuilt library is bundled into each customized route. Routes you didn't
customize keep IBM's copy.

## Verify

Ask the user to reload the frame and open a page in a customized route that renders
the changed component, and confirm the change. Then have them open the same component
from a route you did **not** customize: it should still show IBM's version.

If nothing changes, the module isn't linked (rerun `yarn install` so `postinstall` links it)
or the watch build stopped.
