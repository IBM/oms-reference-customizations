---
name: buc-register-customization
description: >
  Register BUC route components, providers, and module imports without duplicate Angular declarations.
---

# Register Route Customizations

Before registering, ensure the route is already prepared — see [buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md).

## Step 1 — Locate the registration file

Open `src-custom/app/app-customization.impl.ts`. The setup schematic spreads
`AppCustomizationImpl.components`, `.providers`, and `.imports` into the feature module's
corresponding NgModule arrays. Preserve all existing entries.

## Step 2 — Add your declarations

- Declare each non-standalone component in exactly one NgModule. Use `components` when it needs
  the host feature module's scope; keep a valid generated declaration when its module already
  supplies the required imports. Do not move declarations solely because they are generated.
- If moving a declaration, remove its previous declaration/import and ensure the destination
  provides the component's dependencies. Follow the installed Angular version's standalone rules.
- Register DI providers in `providers` and module dependencies in `imports`.
- A tab token or `ExtensionModule.forRoot` mapping identifies a component; it does not declare it.
  Keep both the Angular declaration and the task-specific mapping.

## Step 3 — Verify

Build and test the page that uses the component. A successful build alone does not verify dynamic lookup.
