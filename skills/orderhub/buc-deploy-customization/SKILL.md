---
name: buc-deploy-customization
description: Package, build, test, and deploy Order Hub customizations to an on-premises environment, as a customization image (containers) or a customization folder (traditional).
---

# Deploy customizations

Deployment has four stages: declare what ships, build it, check the packaged
result, and deploy it. Test a container image in the toolkit before deploying it.

**Mandatory:** ask whether the target is **containers** (OpenShift / Operator) or
**traditional** (web server) before building. The steps differ. Don't use
`package-jar` or a Self Service upload; that is the SaaS flow.

## 1. Declare what ships

**For each existing IBM route you customized** — edit
`<orderhub-code>/<module>/package-customization.json` and list every customized
route:

```json
{
  "repositoryName": "buc-app-order",
  "customizationContextRoot": "/order-management-customization",
  "routes": {
    "create-order": { "type": "code" }
  }
}
```

`type` is always `"code"`. Leave `customizationContextRoot` at
`/order-management-customization` — it's preset and changing it breaks the build.

Then for **each existing customized route**, open
`<module>/packages/<route>/package.json` and add a `--deploy-url` to the
`build-merged` script:

```json
"build-merged": "ng build create-order --configuration=merged-prod --deploy-url=/order-management-customization/buc-app-order/create-order/"
```

The pattern is `/order-management-customization/<module>/<route>/`. Without it,
the deployed build requests its assets from the wrong path and the page loads
blank.

A **new** route added to an IBM module is different. It takes `"custom": true` and a
`build-merge` script without `--deploy-url`. See
[buc-add-route-package](../buc-add-route-package/SKILL.md). Root-config-only JSON changes
need no route entry, but the module must still be built.

**For each new custom app** — create the deployment version of `features.json` in
`<orderhub-code>/shell-ui/assets/` (not `assets/dev/`), with absolute
`/order-management-customization/...` links. See [buc-create-features-json](../buc-create-features-json/SKILL.md).

## 2. Build

From `<orderhub-code>/container-build`, build the modules first:

```
./build-customization.sh build-ui <module1>,<module2>
```

Then package them:

| Target | Command | Output |
|---|---|---|
| Containers | `./build-customization.sh build-image <image_name> <image_tag> <module1>,<module2>` | A customization docker image |
| Traditional | `./build-customization.sh package-ui <module1>,<module2>` | A folder in `build/src` |

Omit the module list to build everything.

- For containers, use an image name and tag that differ from IBM's extension image,
  with a tag starting with `custom`, for example `om-orderhub-ext custom-2026-09-26`.
- `om-orderhub-ext image cannot be found on docker.io` → set `DOCKER_BUILDKIT=0` and retry.
- Permission denied → `chmod +x build-customization.sh`
- `Command failed with exit code 137` on a module with many routes → limit
  parallelism in that module's `package.json`:
  `"build": "lerna run build --stream --concurrency=2"`

## 3. Test the image in the toolkit (containers)

This runs a second set of containers on different ports, alongside the ones you've
been developing against. Set the image name and tag you just built in
`devtoolkit_docker/compose/om-compose.properties`:

```
ORDERHUB_EXT_CUSTOM_IMAGE=<image_name>
ORDERHUB_EXT_CUSTOM_TAG=<image_tag>
```

Deploy the containers from `devtoolkit_docker/compose`:

```
./om-compose.sh setup-orderhub-custom
```

This creates `om-orderhub-base-custom` and `om-orderhub-ext-custom`.

Ask the user to open `https://localhost:7444/order-management` and confirm the
customizations behave as expected. The port is 7444 unless `OH_BASE_CUSTOM_HTTPS_PORT`
is set. This instance loads the **built** customizations rather than your local
dev server, so a successful test confirms the packaging.

For a traditional deployment, check that `build/src/ext` contains the routes and
assets you intend to copy.

## 4. Deploy

Use the target selected before building.

**Containers:**

1. Load the customization image into the cluster's image registry.
2. In the Operator configuration, set the Order Hub extension image name and tag to
   the customization image.
3. Update the environment through the Operator.

**Traditional:**

1. Copy the `ext` directory from `build/src` to the web server's HTML directory.
2. For a new app or new routes, copy the deployment `features.json` to
   `<html_directory>/ext/orderhub/shell-ui/assets/features.json`.
3. Set `"devMode": false` in
   `<html_directory>/orderhub/shell-ui/assets/app-bootstrap-config.json`.
4. Open `https://<HOST>:<PORT>/order-management/` using the real host, not
   `localhost` or `bucgenerichost`.

## 5. Verify

If the environment you deployed to is the one you were developing against, turn
DEV mode off first or you'll keep seeing your local server instead of the deployed
build:

1. In Order Hub, go to **Settings > Customization**.
2. Open the module customization toggles.
3. Set the module you customized to **OFF** and save.
4. Reload the frame.

The menu items drop the **(DEV mode)** suffix. Ask the user to confirm the
customizations are still there — they're now being served from the deployment.
