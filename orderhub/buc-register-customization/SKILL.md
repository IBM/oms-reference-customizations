---
name: buc-register-customization
description: Register BUC route components, providers, and module imports without duplicate Angular declarations.
---

# Register route customizations

Inspect the route's generated feature module and `src-custom/app/app-customization.impl.ts`.
The setup schematic spreads `AppCustomizationImpl.components`, `.providers`, and `.imports`
into the feature module's corresponding NgModule arrays. Preserve existing entries.

- Declare each non-standalone component in exactly one NgModule. Use `components` when it needs
  the host feature module's scope; keep a valid generated declaration when its module already
  supplies the required imports. Do not move declarations solely because they are generated.
- If moving a declaration, remove its previous declaration/import and ensure the destination
  provides the component's dependencies. Follow the installed Angular version's standalone rules.
- Register DI providers in `providers` and module dependencies in `imports`.
- A tab token or `ExtensionModule.forRoot` mapping identifies a component; it does not declare it.
  Keep both the Angular declaration and the task-specific mapping.
- Action handlers follow [buc-add-custom-action](../buc-add-custom-action/SKILL.md), including the
  original handler's root/feature token scope.

Build and test the page that uses the component. A successful build alone does not verify dynamic lookup.

Verify that each new component is declared exactly once and that existing registrations still work.

For route customization, first complete [buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md).
