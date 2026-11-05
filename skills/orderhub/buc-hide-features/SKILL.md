---
name: buc-hide-features
description: Hide built-in Order Hub features from the left navigation menu using a custom features.json entry.
---

# Hide a feature from the navigation menu

To hide a shipped menu item, add an entry with its ID and `hidden` set to
true. You don't copy or modify the IBM `features.json` — your file is merged over
it.

To add your own menu item instead, follow [buc-create-features-json](../buc-create-features-json/SKILL.md).

## 1. Find the feature ID

IDs come from the shipped file inside the running container:

```
docker cp om-orderhub-base:/opt/app-root/src/shell-ui/assets/features.json features.json
```

Find the menu item's entry and note its `id`. Hiding a parent also hides its
children, so check whether you want to hide the parent or a single child.

## 2. Add the hide entry

In your custom `features.json`, add only the ID and the flag:

```json
[
  {
    "id": "oms-inventory-ss-rules",
    "hidden": "true"
  }
]
```

Use the same file for hidden and custom menu items; both types of entry can
appear together.

## 3. Test locally

```
docker exec om-orderhub-base bash -c 'mkdir -p /opt/app-root/src/shell-ui/assets/custom'
docker cp <path to features.json> om-orderhub-base:/opt/app-root/src/shell-ui/assets/custom/
```

Ask the user to reload Order Hub and confirm the menu item is gone and that the
rest of the navigation is intact, because a wrong ID can hide more than intended.

## 4. Build

```
./build-customization.sh build-image <image_name> <image_tag> <ui-modules>
```

If you have no customized modules — hiding a menu item on its own doesn't create
one — omit the module list:

```
./build-customization.sh build-image om-orderhub-ext custom
```

Then deploy as usual: follow [buc-deploy-customization](../buc-deploy-customization/SKILL.md).

## Note on access

`hidden` removes the item from the menu but the page stays reachable by URL. If the
goal is to stop people using a feature rather than to tidy the menu, control it
with permissions instead.
