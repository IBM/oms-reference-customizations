---
name: cc-navigation
description: Make a custom Call Center route reachable - add its left-menu entry in features.json and register it with call-center-hub through routes.json and import-map.json, for local development and deployment.
---

# Register a route in the menu and hub

A custom route is reachable only when all three files agree:

| File | Purpose |
|---|---|
| `callcenter-code/shell-ui/assets/features.json` | Left-menu entry that opens a hub path |
| `routes.json` | Maps the hub path to an app name |
| `import-map.json` | Maps the app name to its `main.js` |

The schematic writes `routes.json` and `import-map.json` to
`packages/<short>-root-config/src/assets/` when you create the route or module.

## 1. Add the menu entry

`callcenter-code/shell-ui/assets/features.json` is an array:

```json
[
  {
    "id": "extn-carrier-search",
    "title": "features.LABEL_CARRIER_SEARCH",
    "icon": "topnav_watson-20",
    "type": "IFRAME",
    "navigation": { "route": { "link": "/call-center-hub/carrier-search", "path": "/carrier-search" } },
    "enablementCondition": { "byResourceId": "EXTNCC0010" }
  }
]
```

| Property | Notes |
|---|---|
| `id` | Unique. Prefix it, for example `extn-` |
| `title` | Translation key, defined in `shell-ui/assets/i18n/<lang>.json` |
| `icon` | A name from `/opt/app-root/src/shell-ui/assets/icons/shell-icon-set.svg` in `om-call-center-base` |
| `type` | Always `IFRAME` |
| `navigation.route.link` | Must start with `/call-center-hub/`. The suffix matches a `routes.json` path |
| `navigation.route.path` | What the address bar shows |
| `enablementCondition.byResourceId` | Optional. Comma-separated resource IDs the user needs |
| `hidden` | Optional. `true` keeps the route reachable by URL but off the menu |

```json
{ "features": { "LABEL_CARRIER_SEARCH": "Carrier search" } }
```

## 2. Register with the hub

Put one `routes.json` and one `import-map.json` in the hub mount path,
`callcenter-code/call-center-hub/assets/`, merging every custom route into them:

```json
{ "routes": [ { "path": "carrier-search", "appName": "carrier-search" } ] }
```

For local development, point each import at its dev server (port and context root
from `app-config.json`):

```json
{
  "imports": {
    "carrier-search":        "https://localhost:8200/ext-callcenter-carrier/carrier-search/main.js",
    "order-release-details": "https://localhost:5100/call-center-order/order-release-details/main.js"
  }
}
```

If the folder isn't mounted, copy both folders into `om-call-center-base`:

```
docker cp callcenter-code/shell-ui/assets/. om-call-center-base:/opt/app-root/src/shell-ui/assets/custom
docker cp callcenter-code/call-center-hub/assets/. om-call-center-base:/opt/app-root/src/call-center-hub/assets/custom
```

## 3. Before deploying

Remove the `https://localhost:...` imports from the hub folder. The deployed module
carries its own `routes.json` and `import-map.json` from
`<short>-root-config/src/assets/`.

## Verify

Ask the user to open each dev server URL once to accept its certificate, reload Call
Center, and open the menu item. The route should load in a hub tab.

If the menu item is missing, `features.json` isn't loaded or `enablementCondition` excludes the
user. A blank tab means the `routes.json` path doesn't match the `link` suffix, or the
certificate wasn't accepted.
