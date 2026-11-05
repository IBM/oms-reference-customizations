---
name: buc-customize-login-page
description: Rebrand the Order Hub/Call Center login page through shell-ui assets, including the title and banner text, logo image, and shell-level CSS. Use for organization branding rather than page customization.
---

# Customize the login page

The login page renders in the shell, before any module loads, so none of the
module customization mechanisms reach it. To change its branding, edit
`shell-ui/assets` in the Order Hub/Call Center code directory.

Everything here lives under `<orderhub-code|callcenter-code>/shell-ui/assets/`. Create the
directories if they don't exist.

## 1. Change the brand name

Create `shell-ui/assets/i18n/` and add a bundle per language — `en.json`,
`fr.json`, and so on:

```json
{
  "loginPage": {
    "Label_Title_orderHub": "Acme Fulfillment"
  },
  "cuiTopBanner": {
    "brand": "Acme",
    "orderHub": "Fulfillment"
  }
}
```

| Key | Where it shows |
|---|---|
| `loginPage.Label_Title_orderHub` | The title on the login page itself |
| `cuiTopBanner.brand` | The first part of the banner, across the application |
| `cuiTopBanner.orderHub` | The second part of the banner |

`cuiTopBanner` isn't login-only — it's the top banner everywhere, so changing it
rebrands the whole application. Confirm with the user that's what they want.

**Call Center:** the keys are `loginPage.Label_Title_callCenter`, `cuiTopBanner.brand`, `cuiTopBanner.callCenter`, and `cuiTopBanner.callCenter_title` (browser title). For a logo, set `brandImage` to an SVG path starting with `/call-center-customization/`.

For translating other parts of the application, follow
[buc-customize-locales-i18n](../buc-customize-locales-i18n/SKILL.md).

## 2. Change the logo

Create `shell-ui/assets/app-bootstrap-config.json` and set `bannerFileName`. The
first character determines how the value is interpreted:

```json
{ "bannerFileName": "/<order-management|call-center>-customization/shell-ui/assets/icon2.png" }
```

```json
{ "bannerFileName": "logo.png" }
```

| Form | Meaning |
|---|---|
| Starts with `/` | Treated as a URL or absolute path, used as given |
| A bare file name | The file **must** be in `assets/custom/images` |

A logo usually fails to appear when this rule is missed: a bare file name placed
anywhere other than `assets/custom/images` silently fails to resolve.

This is the same file that [buc-inject-custom-html-js](../buc-inject-custom-html-js/SKILL.md)
uses for `extensionsFileName`. If it already exists, add the key rather than
replacing the file.

## 3. Restyle with CSS

Create `shell-ui/assets/custom_styles.css` for shell-level overrides. It applies to
shell elements including the login page:

```css
.login-page-img {
  object-fit: contain !important;
  background-color: white !important;
}
```

`!important` is usually needed here — the shell's own styles load after yours.

## 4. Verify

These are shell assets, not module code, so the dev server won't pick them up the
way it picks up route changes. Ask the user to copy them into the running container
and reload:

```
docker exec om-<orderhub|call-center>-base bash -c 'mkdir -p /opt/app-root/src/shell-ui/assets/custom'
docker cp <orderhub-code|callcenter-code>/shell-ui/assets/. om-<orderhub|call-center>-base:/opt/app-root/src/shell-ui/assets/custom/
```

Then ask them to sign out, open the login page, and confirm that the title,
banner, and logo all changed. If the text changed but the logo didn't, check the
`bannerFileName` path rule in step 2 first.

## Deploy

Shell assets ship with the rest of the customization. Build the JAR and deploy by
following [buc-deploy-customization](../buc-deploy-customization/SKILL.md). Ask the user to confirm
the branding survived the deployment, since a path that resolved locally can still
break behind the customization context root.
