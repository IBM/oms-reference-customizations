---
name: buc-customize-login-page
description: >
  Customize OrderHub's login page or shell labels, banner text, and logo assets.
---

# Customize the Login Page

Login branding belongs to the shell under `shell-ui/assets`, separate from module apps.

## Step 1 — Override text labels

Create `shell-ui/assets/i18n` if it doesn't exist. Add/edit `en.json` (and per-locale siblings):

```json
{
  "loginPage": { "Label_Title_orderHub": "<your custom title>" },
  "cuiTopBanner": { "brand": "<your custom brand text>", "orderHub": "<your custom text>" }
}
```

## Step 2 — Replace the logo

1. Create `shell-ui/assets` if it doesn't exist.
2. Add/edit `app-bootstrap-config.json`:
   ```json
   { "bannerFileName": "/order-management-customization/shell-ui/assets/icon2.png" }
   ```

Prefix `bannerFileName` with `order-management-customization`; a bare relative path silently
fails to resolve.

## Verify

Reload the login page and check the updated title, banner text, and logo.

## Next step

For shell analytics or tracking scripts, use [buc-inject-custom-html-js](../buc-inject-custom-html-js/SKILL.md).
