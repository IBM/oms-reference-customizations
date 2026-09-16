---
name: buc-inject-custom-html-js
description: >
  Add a static HTML or JavaScript fragment to the BUC shell and configure its assets.
---

# Inject Custom HTML/JavaScript at the Shell Level

Use this for scripts that run once at shell startup, outside app iframes and their sandboxing.
For module-specific UI, use the module's Angular code through the other skills in this collection.

## Constraints

- Use a static HTML snippet without Angular components.
- Test carefully: an invalid script can slow or break the entire shell.
- External libraries must be reachable at runtime from the browser loading the shell.

## Step 1 — Write the snippet

```html
<!-- shell-extensions.html -->
<script>
  // your tracking/analytics code
</script>
```

## Step 2 — Place it and any supporting assets

```
shell-ui/assets/codeFragments/shell-extensions.html
```

Supporting static assets (images, etc.) go under `src/custom/assets/images`.

## Step 3 — Configure for local testing

```
shell-ui/assets/app-bootstrap-config.json
```

```json
{ "extensionsFileName": "shell-extensions.html" }
```

Copy into the base container:
```sh
docker exec om-orderhub-base bash -c 'mkdir -p /opt/app-root/src/shell-ui/assets/custom'
docker cp <orderhub-code>/shell-ui/assets/. om-orderhub-base:/opt/app-root/src/shell-ui/assets/custom
```

## Step 4 — Configure for deployment

Switch `extensionsFileName` to the deployed path:

```json
{ "extensionsFileName": "/order-management-customization/shell-ui/assets/codeFragments/shell-extensions.html" }
```

Package and deploy through the normal Self Service / customization pipeline.

## Verify

Reload the shell and check DevTools → Network/Console to confirm the script loads and executes once
at startup, applying across all module pages.
