---
name: buc-customize-locales-i18n
description: Add and test a custom language or locale in Order Hub/Call Center, covering the shell, each module, and the login page.
---

# Add a custom locale

Translations are retrieved per module. Before starting a new locale, plan for
one file per module plus a shell file.

## 1. Register the language with the shell

Create `shell-ui/assets/app-bootstrap-config.json` in `<orderhub-code|callcenter-code>`:

```json
{
  "productsData": {
    "products": [
      {
        "id": "omsi",
        "supportedLanguages": "de,en,es,fr,it,ja,ko,pl,pt_BR,ru,tr,zh_TW,zh,da,et,fi,pa,<new_language_code>",
        "defaultLanguage": "en"
      }
    ]
  }
}
```

List **every** language the application should offer, not just the new one. A code
missing from `supportedLanguages` is ignored and the shell falls back to the
browser language.

## 2. Translate the shell

Create `shell-ui/assets/i18n/`, then get the source strings from a running
instance. Ask the user to follow these steps in their browser:

1. Open Dev Tools, select the **Network** tab, and refresh the page.
2. Find the request to
   `<ORDERHUB_DOMAIN>/order-management/assets/i18n/<language_code>.json`
   (Call Center: `<CC_DOMAIN>/call-center/assets/i18n/<language_code>.json`).
3. Copy its contents.

Save it as `<new_language_code>.json` in `shell-ui/assets/i18n/` and translate the
values.

In the same folder, add the language name for the login page picker:

```json
{
  "loginPage": {
    "Label_language_custom": "Custom language"
  }
}
```

## 3. Translate each module

Repeat this approach for each module. Ask the user to navigate to the module and
copy the response from:

```
<ORDERHUB_DOMAIN>/<buc-app|call-center>-<module>/assets/i18n/<language_code>.json
```

Save each as
`<buc-app|call-center>-<module>/packages/<module>-root-config/src/assets/custom/i18n/<new_language_code>.json`
and translate.

Repeat for every module that needs the language. Skipped modules render in the
fallback language, creating a visible inconsistency without raising an error.

### Call Center

- Shell source: `<CC_DOMAIN>/call-center/assets/i18n/<language_code>.json`.
- Hub: merge `cc-hub/i18n/hub/`, `cc-hub/i18n/`, `common-components/i18n/`, and `svc-angular/i18n/` (each `<language_code>.json` under `<CC_DOMAIN>/call-center-hub/assets/`) into `callcenter-code/call-center-hub/assets/call-center-hub/i18n/<new_language_code>.json`.
- Custom locales are on-premises only.

## 4. Test locally

Copy the shell assets into the container:

```
docker exec om-<orderhub|call-center>-base bash -c 'mkdir -p /opt/app-root/src/shell-ui/assets/custom'
docker cp <orderhub-code|callcenter-code>/shell-ui/assets/. om-<orderhub|call-center>-base:/opt/app-root/src/shell-ui/assets/custom
```

And the hub assets:

```
docker exec om-<orderhub|call-center>-base bash -c 'mkdir -p /opt/app-root/src/<orderhub-hub|call-center-hub>/assets/custom/'
docker cp <orderhub-code|callcenter-code>/<orderhub-hub|call-center-hub>/assets/. om-<orderhub|call-center>-base:/opt/app-root/src/<orderhub-hub|call-center-hub>/assets/custom/
```

Then enable the module in the **Customization** modal and run `yarn start-app`.

Walk the user through the rest — it spans the login page and user configuration:

1. On the login page, select the custom language.
2. Create a locale in the application and update the user role to use the locale
   code.
3. Confirm the UI renders in the new language across the modules you translated.

## Traditional installations

Use the same content in these locations:

- Merge `<orderhub-code|callcenter-code>/shell-ui/assets` into
  `$HTML_DIRECTORY/ext/<orderhub|call-center>/shell-ui/assets` on the NGINX server. If that
  directory isn't empty, **merge** — don't overwrite it.
- Set `"devMode": true` in `app-bootstrap-config.json`.
- Module translations go in the same
  `<module>-root-config/src/assets/custom/i18n/` path.

## Deploy

Package and deploy as usual by following [buc-deploy-customization](../buc-deploy-customization/SKILL.md).

For a route already using `environment.customization = true`, custom locale bundles
must also be available under that route's `src-custom/assets/custom/i18n/`; it does
not request the module-level custom bundle. Complete
[buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md) and
[buc-wire-custom-assets](../buc-wire-custom-assets/SKILL.md), then verify the requested
translation URL. Shell language registration remains separate.
