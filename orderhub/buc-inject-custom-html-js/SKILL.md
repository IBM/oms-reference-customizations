---
name: buc-inject-custom-html-js
description: Inject custom HTML and JavaScript into the Order Hub application frame for analytics, usage tracking, and enterprise integrations that must run across all pages.
---

# Inject custom HTML and JavaScript

The shell injects your HTML into the application frame that surrounds every page,
outside the per-application iframes. Scripts run once at startup and apply across
all pages, supporting analytics that per-page injection can't provide.

Typical uses include analytics frameworks such as Segment.js, usage tracking,
enterprise tool integrations, and shared scripts that must load globally.

## Limits

- The HTML must be a **static snippet**. Angular components don't work here.
- External libraries must be reachable at run time from the browser.
- A bad script degrades the whole application, not one page. This runs in the
  frame that wraps everything.

## 1. Write the snippet

Create an HTML file, for example `shell-extensions.html`:

```html
<script>
  (function() {
    const banner = document.createElement("div");
    banner.innerText = "Custom extension loaded";
    banner.style.padding = "8px";
    if (document.body) {
      document.body.prepend(banner);
    } else {
      console.warn("document.body not available yet");
    }
  })();
</script>
```

Guard against `document.body` not existing yet — the snippet runs at startup and
can execute before the body is available.

## 2. Place it

```
orderhub-code/shell-ui/assets/codeFragments/shell-extensions.html
```

If the HTML references images or other static files, put them in
`assets/custom/images`.

## 3. Point the shell at it

Set the path in `shell-ui/assets/app-bootstrap-config.json`.

For local testing, the file name alone:

```json
{ "extensionsFileName": "shell-extensions.html" }
```

For deployment, the full path:

```json
{ "extensionsFileName": "/order-management-customization/shell-ui/assets/codeFragments/shell-extensions.html" }
```

Use the full path for deployment. The snippet won't load if you deploy with only
the file name.

## 4. Test locally

```
docker exec om-orderhub-base bash -c 'mkdir -p /opt/app-root/src/shell-ui/assets/custom'
docker cp shell-ui/assets/. om-orderhub-base:/opt/app-root/src/shell-ui/assets/custom/
```

Ask the user to reload Order Hub and confirm the script ran — check the
**Console** for your output, and the **Network** tab to confirm that any external library
loaded. Also ask them to click through a couple of pages and confirm nothing else
broke, since this affects every page.

## 5. Deploy

Switch `extensionsFileName` to the deployment path, then package and deploy.
