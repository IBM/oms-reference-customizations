---
name: buc-create-features-json
description: Add a custom app or page to the Order Hub left navigation menu using features.json, for both local development and deployment.
---

# Add a menu item (features.json)

`features.json` is an array of menu entries for the Order Hub shell. You maintain
**two copies** that differ only in `navigation.route.link`:

| Version | Location | `link` |
|---|---|---|
| Development | `<orderhub-code>/shell-ui/assets/dev/features.json` | `https://localhost:<port>/<base href>/<path to the page>` |
| Deployment | `<orderhub-code>/shell-ui/assets/features.json` | `/order-management-customization/<base-href>/<path to the page>` |

Create and test the development version first, then copy it and change the links
for deployment.

## The entry

```json
[
  {
    "id": "custom-monorepo",
    "title": "features.custom-monorepo-title",
    "icon": "programs-30",
    "iconForRailStateOnly": true,
    "parentFeatureId": "oms-inventory-root",
    "sequence": "10.4011",
    "navigation": {
      "route": {
        "link": "https://localhost:9300/custom-monorepo/custom-page1/monorepo/home",
        "path": "/custom-monorepo/custom-page1/monorepo/home"
      }
    },
    "enablementCondition": { "byResourceId": "BUCWSP" },
    "type": "IFRAME"
  }
]
```

| Property | Notes |
|---|---|
| `id` | Unique. Prefix with `custom` so it can't collide with an IBM feature |
| `title` | A literal, or a translation key such as `features.my-title` |
| `icon` | Name from the shell icon set — see below |
| `iconForRailStateOnly` | Optional, default `false`. Show only the icon when the menu collapses |
| `parentFeatureId` | Optional. Nests this under an existing menu — see below |
| `sequence` | Position within its level |
| `navigation.route.link` | Where the page is actually served from. Differs per version |
| `navigation.route.path` | `/<base-href>/<path to the page>`. What the user sees in the address bar |
| `type` | Always `IFRAME` |
| `enablementCondition.byResourceId` | Optional. Comma-separated resource IDs the user must have |
| `hidden` | Optional, default `false`. Keeps the page reachable by URL but off the menu |

## Finding icon and parent values

Get both values from the running container:

```
docker cp om-orderhub-base:/opt/app-root/src/shell-ui/assets/features.json features.json
```

Use the `id` of the menu you want to nest under as your `parentFeatureId`. Valid icon
names are in `/opt/app-root/src/shell-ui/assets/icons/shell-icon-set.svg` inside the
same container.

## Choosing a sequence value

The menu is ordered by ascending `sequence`, which accepts any real number.
Insert an item by choosing a value between its two neighbors. Read the shipped
values from the same file:

```
10        Order Hub
10.10     Home
10.15     Dashboards
10.20     Workspaces
10.30     Orders
10.40     Inventory
10.401      Search
10.402      Reservation
10.45     Promise and fulfill
```

To place an item between Dashboards and Workspaces, use `10.16`. Between Search and
Reservation, use `10.4011`. Add decimal places as needed — there's no depth limit,
and this works at any level of the hierarchy.

## Load it for local development

```
docker exec om-orderhub-base bash -c 'mkdir -p /opt/app-root/src/shell-ui/assets/custom'
docker cp <orderhub-code>/shell-ui/assets/dev/. om-orderhub-base:/opt/app-root/src/shell-ui/assets/custom/
```

Re-run this after every edit, because the container holds a copy, not a live link.

## Translate the title

If `title` is a translation key, add it to the language files in
`<orderhub-code>/shell-ui/assets/i18n/`: `en.json`, `de.json`, `es.json`,
`fr.json`, `it.json`, `ja.json`, `ko.json`, `pl.json`, `pt.json`, `ru.json`,
`tr.json`, `zh-Hant.json`, `zh-Hans.json`.

## Verify

Ask the user to reload Order Hub and confirm the menu item appears in the right
place and opens the page.

If it doesn't appear, the file wasn't copied into the container, or
`enablementCondition` is excluding their user. If it appears but the page is blank,
they haven't accepted the certificate for the dev server port yet.
