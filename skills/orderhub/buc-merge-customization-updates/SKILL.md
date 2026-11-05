---
name: buc-merge-customization-updates
description: Merge existing Order Hub customizations onto a new code base after a fix pack or release, and audit which routes carry customization debt.
---

# Merge customizations onto a new code base

When a fix pack or release arrives, your existing customizations keep working, so
no change is strictly required. To receive IBM's fixes **for the routes you
overrode**, merge those customizations into the new code.

The work required depends on the customization approach:

| Approach used | Work at upgrade |
|---|---|
| New applications or pages | None |
| Customization by configuration | None |
| Differential customization | Copy your files across and recompile |
| Customization by overrides | Compare and merge, file by file |

## 1. Decide whether to merge

1. Review the release notes for the fixes included.
2. Check the list of changed files in those notes.
3. Determine whether any changed file is one you customized **by code**.

If none overlap, you're done. If they do, and the fixes matter, merge.

## 2. Audit what's customized

If you aren't sure what's been customized, check the environment. Walk the user
through these steps:

1. Log in to Order Hub and select the tenant in the banner.
2. Go to **Settings > About**.
3. Open **Order hub customization** to see which modules are customized.
4. Expand each module to see the specific pages or routes, with the date each was
   last uploaded.

Anything listed here is a route that won't receive new versions until you merge.
Ask the user to share that list before planning the work.

## 3. Get the new source

**Containers** — in the updated toolkit:

```
./om-compose.sh extract-orderhub-code oms
```

**Traditional** — from the runtime repository:

```
mkdir -p orderhub-code/lib
tar -xf <runtime>/repository/orderhub/code/orderhub/lib.tar --directory=orderhub-code/lib
tar -xf <runtime>/repository/orderhub/code/orderhub/on-prem-source.tar --directory=orderhub-code
```

From the Sterling Order Management 10.0.2604.0 release onward, `lib.tar` carries
only the library versions used by the current code base, not every earlier version.
If you need an earlier one, download the `lib.tar` for your Angular version from
the Self Service download page, under **OMS Modern UI Libraries**.

## 4. Merge

**Differential customizations** — copy your customized files into the new code
directory, keeping the same structure. Most live in `src-custom` folders. Then
recompile.

**Overrides** — one file at a time:

1. Copy your customized files into the new code directory, same structure.
2. For each one, copy IBM's new version into `src-custom`.
3. Use a comparison tool to diff your version against IBM's new version.
4. Merge your changes into IBM's new file — not the other way round. Starting from
   IBM's file is what picks up the fixes.

**Files outside `src-custom`** — anything IBM-provided that you edited in place
needs the same treatment: merge your changes into the new file. **Mandatory:** check
each of these per customized module, because the new code base ships them without
your edits:

- `angular.json`: the `merged` and `merged-prod` asset arrays for each customized route
- `overrides.json`: `runAsCustomization` / `useProd` entries

## 5. Build, test, deploy

Build locally, test the built image in the toolkit, then deploy, as covered in
[buc-deploy-customization](../buc-deploy-customization/SKILL.md).

A merge can compile successfully while losing existing behavior, so compilation
alone isn't enough to verify it. For each route on the audit list from step 2, ask
the user to exercise the customization and confirm it still behaves as it did
before the upgrade, **and** that the IBM fix you merged for is now present. Both
checks matter: the merge must include the fix and preserve your customization.

## Staying current

- Periodically review the `package.json` files in your own applications and move
  them to newer library versions as IBM modules move to higher Angular versions.
  Open source fixes only reach you if you take them.
- Keep the list of overridden files short. Every entry is work at every upgrade —
  when a customization can be moved to configuration or an extension point, that
  removes it from this process permanently.

For route customization, first complete [buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md).
