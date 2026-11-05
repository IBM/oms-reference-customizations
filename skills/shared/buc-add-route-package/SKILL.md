---
name: buc-add-route-package
description: Create a new route package in an existing Order Hub/Call Center monorepo (an IBM module like buc-app-order, or your own custom app), including the schematic scaffolding, dev server configuration, menu item, and deployment. For a page inside an existing route package, use buc-add-page-to-route instead.
---

# Add a new route package to an existing monorepo

This adds a *new* route package. For a page inside a route package that already exists,
follow [buc-add-page-to-route](../buc-add-page-to-route/SKILL.md) instead.

The new route doesn't override an existing page, so there's nothing
to re-merge when IBM ships an update.

Both targets use the same schematic, but one flag and the deployment files differ:

| Target | `--is-custom-app` | Deployment |
|---|---|---|
| An IBM module (`buc-app-order`, `buc-app-catalog`, ...) | Leave unset — defaults to `false` | Needs `package-customization.json` and a `build-merged` script |
| Your own custom app | **Set `true`** | Follow [buc-deploy-customization](../buc-deploy-customization/SKILL.md) |

For a custom app you haven't created yet, start with
[buc-create-new-app](../buc-create-new-app/SKILL.md).

## Call Center differences

| Step | Call Center |
|---|---|
| 2. Schematic flags | Add `--channel-app=call-center` (and `--prefix=call-center` for IBM modules). No `--is-custom-app` |
| 3. `overrides.json` | `"runAsCustomization": true` for the new route, not `useProd` |
| 4. Menu item | Register with the hub (`routes.json`, `import-map.json`) — [cc-navigation](../cc-navigation/SKILL.md) |
| 6. Deployment | `package-customization.json` entry `{ "type": "code", "custom": true }`; `build-merged` is `"yarn build"` |

## 1. Declare the route

Open the monorepo's `app-config.json` — for an IBM module that's
`devtoolkit_docker/<orderhub-code|callcenter-code>/<buc-app-order|call-center-order>/app-config.json` — and add an entry
under `routes`, with a port not already in use:

```json
"routes": {
  "custom-page1": { "devServerConfig": { "port": 9301, "contextRoot": "/custom-page1" } },
  "custom-page2": { "devServerConfig": { "port": 9302, "contextRoot": "/custom-page2" } }
}
```

You can add several at once — the schematic generates a package for each new entry
and leaves existing routes untouched.

## 2. Run the schematic

From the monorepo directory:

```
ng g @buc/schematics:add-app-to-existing-monorepo \
  --app-config-json app-config.json \
  --module-short-name <short-name> \
  --prefix <selector-prefix> \
  --mode on-prem
```

Add `--is-custom-app=true` when the monorepo is your own app rather than an IBM one.

| Option | Notes |
|---|---|
| `--app-config-json` | Required. The file from step 1 |
| `--module-short-name` | Required. The text after the module name's **last dash** — `buc-app-order` → `order`, `custom-monorepo` → `monorepo` |
| `--mode` | `on-prem` |
| `--is-custom-app` | Defaults to `false`, which targets an IBM repository. **Set `true` for your own app**; the default targets IBM repositories |
| `--prefix` | HTML selector prefix. Default `buc` |

If you didn't scaffold the monorepo yourself, check its existing `package.json` for
the short name rather than re-deriving it, in case it was renamed.

## 3. Point the route at your local build

In the monorepo's `overrides.json`, set the new route to serve from your dev server:

```json
{
  "routes": {
    "custom-page1": { "useProd": false },
    "custom-page2": { "useProd": false }
  },
  "devServerConfig": { "hostName": "bucgenerichost" }
}
```

Edit the file in place, keeping its line endings and indentation. Don't re-serialize it
with a script.

Follow both rules (Call Center: see the table above):

- **Always `"useProd": false`** for a custom route. This is **not** the
  `runAsCustomization: true` you'd use when overriding an *existing* route — that's
  a different operation, and using it here won't serve your new route. See
  [buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md)
  for the override case.
- **Always `"hostName": "bucgenerichost"`** under `devServerConfig`.

## 4. Add the menu item

If the route needs its own navigation entry, create the development `features.json`.
For the full property set — icon, parent, sequence, translations, and `hidden` for a
route reachable only by link — follow
[buc-create-features-json](../buc-create-features-json/SKILL.md) (Call Center: [cc-navigation](../cc-navigation/SKILL.md)).

```json
[
  {
    "id": "custom-order",
    "title": "Custom Order",
    "icon": "shopping--cart-16",
    "parentFeatureId": "oms-order-search",
    "navigation": {
      "route": {
        "link": "https://localhost:8300/buc-app-order/custom-route/order/home",
        "path": "/buc-app-order/custom-route/order/home"
      }
    },
    "enablementCondition": { "byResourceId": "BUCORD0005,BUCORD0006" },
    "type": "IFRAME"
  }
]
```

Load it into the container:

```
docker exec om-<orderhub|call-center>-base bash -c 'mkdir -p /opt/app-root/src/shell-ui/assets/custom'
docker cp <orderhub-code|callcenter-code>/shell-ui/assets/dev/. om-<orderhub|call-center>-base:/opt/app-root/src/shell-ui/assets/custom/
```

## 5. Run it

Ask the user to run `yarn start-app` in their own terminal and tell you when it's
up, then to confirm the new route loads from the menu. For a custom app, they may
need to accept the certificate at `https://localhost:<new route port>` first.

If it exits with code 134, the Node heap needs raising. See
[buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md).

## 6. Prepare for deployment

Build the page first with the component schematics
([buc-generate-search-panel](../buc-generate-search-panel/SKILL.md),
[buc-generate-table-component](../buc-generate-table-component/SKILL.md), and so on), then:

**`features.json`** — copy the dev version from
`<orderhub-code|callcenter-code>/shell-ui/assets/dev/features.json` to
`<orderhub-code|callcenter-code>/shell-ui/assets/features.json` and change the link from the
absolute dev URL to a relative path:

```json
"link": "/order-management-customization/buc-app-order/custom-route/order/home"
```

**`package-customization.json`** *(IBM modules only)* — list the route with
`"custom": true`, which marks it as new rather than an override:

```json
{
  "repositoryName": "buc-app-order",
  "customizationContextRoot": "/<order-management|call-center>-customization",
  "routes": {
    "custom-route": { "type": "code", "custom": true }
  }
}
```

**`packages/<route-name>/package.json`** *(IBM modules only)* — add a `build-merged`
script with the same value as the existing `build` script. The name must match exactly:
`package-customization` builds the `build-merged` target and silently skips routes without it. A new route has no IBM
code to merge against, so unlike an overridden route it takes no `--deploy-url`.

Then build and deploy by following [buc-deploy-customization](../buc-deploy-customization/SKILL.md).
