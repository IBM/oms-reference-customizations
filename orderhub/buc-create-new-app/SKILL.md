---
name: buc-create-new-app
description: Create a new standalone custom Order Hub application repository with the ng-new schematic, then run it locally inside Order Hub. Use for capabilities that don't belong inside an IBM module.
---

# Create a custom app

A custom app is your own monorepo, hosted in the Order Hub shell as a menu item.
Your code is completely separate from IBM code, so product updates never affect it
and there is nothing to re-merge.

Choose this over customizing an IBM module when you're adding a new capability
rather than changing an existing page.

## 1. Install the schematics

Order Hub customizations target **Angular 20**. Install the matching CLI and a
Node.js version compatible with it:

```
npm install -g @angular/cli@20
```

Then, from the Order Hub code directory (`devtoolkit_docker/orderhub-code`):

```
npm config set "strict-ssl" false
npm uninstall -g @buc/schematics
npm install -g ./lib/buc/schematics/schematics-v5latest.tgz
```

`schematics-v5latest.tgz` is the Angular 20 build.

## 2. Describe the app

Create a folder named after your module and an `app-config.json` inside it. Add
one entry per route, each on its own port:

```json
{
  "name": "custom-monorepo",
  "devServerConfig": { "port": 9300, "contextRoot": "/custom-monorepo" },
  "prodServerConfig": { "hostName": "static.omsbusinessusercontrols.ibm.com" },
  "routes": {
    "custom-page1": { "devServerConfig": { "port": 9301, "contextRoot": "/custom-page1" } },
    "custom-page2": { "devServerConfig": { "port": 9302, "contextRoot": "/custom-page2" } }
  }
}
```

`name` must be the module name — the same as the folder.

## 3. Generate the workspace

From inside that folder:

```
ng new --collection=@buc/schematics \
  --app-config-json=app-config.json \
  --module-short-name=<short-name> \
  --prefix=<selector-prefix> \
  --mode=on-prem
```

| Option | Notes |
|---|---|
| `--app-config-json` | The file from step 2. **This flag is what selects a monorepo** — see below |
| `--module-short-name` | Text after the last dash of the module name — `buc-app-settings` → `settings` |
| `--mode` | **Must be `on-prem`** |
| `--prefix` | HTML selector prefix. Default `buc` |
| `--shared-library-name` | Default `<module-short-name>-shared` |
| `--generate-root-config` | Default `true` |
| `--skip-install` / `--skip-git` / `--commit` | Optional |

**`ng new` dispatches on `--app-config-json`.** With it, you get a Lerna monorepo
that can hold several routes. Without it, you get a single-route app instead — and
that path requires `--module-name` rather than `--app-config-json`. If you meant a
monorepo and omitted the flag, the command still succeeds and quietly builds the
wrong shape.

Neither flag is marked required on `ng new` itself because it dispatches to
another schematic. The selected schematic defines the requirement.

**Don't rename or prefix `module-short-name` afterwards.** The generated
`root-config` package is named from it, and deployment breaks if they diverge.

Wait for `Packages installed successfully.` Ignore
`Failed to compile entry-point @carbon/icons-angular/` — it doesn't affect icons.

## 4. Add the menu item

The app won't appear in Order Hub until it's in `features.json`. Create the
development version at `<orderhub-code>/shell-ui/assets/dev/features.json` now,
then load it into the container:

```
docker exec om-orderhub-base bash -c 'mkdir -p /opt/app-root/src/shell-ui/assets/custom'
docker cp <orderhub-code>/shell-ui/assets/dev/. om-orderhub-base:/opt/app-root/src/shell-ui/assets/custom/
```

If the containers aren't running, start them first from
`devtoolkit_docker/compose`:

```
./om-compose.sh start orderhub
```

## 5. Run it

Ask the user to run this in their own terminal from the app folder and tell you when
it's up — it's long-running:

```
yarn start-app
```

Ignore the "Angular Live Development Server is listening on localhost:<port>"
message; you reach the app through Order Hub, not that URL.

On `exit code 134` / `lerna ERR! yarn run start .... exited 134`, raise the heap and
retry:

```
export NODE_OPTIONS=--max_old_space_size=8048     # bash
$Env:NODE_OPTIONS="--max_old_space_size=8048"     # PowerShell
```

Use `yarn stop-app` after stopping the job to make sure nothing is left running.

## 6. Open it in Order Hub

Walk the user through these browser steps and wait for confirmation:

1. Open `https://localhost:9300` (the `devServerConfig` port from
   `app-config.json`) and accept the self-signed certificate.
2. Open `https://localhost:7443/order-management`. The port differs if
   `OH_BASE_HTTPS_PORT` is set in
   `devtoolkit_docker/compose/om-compose.properties`, so ask if it doesn't load.
3. Click the new menu item in the left navigation.

If the menu item is missing, `features.json` didn't reach the container — redo
step 4. If it's there but the page is blank, the certificate in step 1 wasn't
accepted.

## Next

- Pages and components → [buc-generate-search-panel](../buc-generate-search-panel/SKILL.md),
  [buc-generate-search-result](../buc-generate-search-result/SKILL.md), [buc-generate-table-component](../buc-generate-table-component/SKILL.md),
  [buc-generate-summary-component](../buc-generate-summary-component/SKILL.md)
- Data → [buc-call-apis](../buc-call-apis/SKILL.md)
