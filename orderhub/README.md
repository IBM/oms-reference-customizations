# OrderHub Skills

Skills for customizing [IBM Order Management System (OMS)](https://www.ibm.com/products/order-management) — specifically the BUC-based OrderHub UI shell and module apps.

| Skill | Description | Example prompt |
| --- | --- | --- |
| [buc-add-custom-hotkeys](buc-add-custom-hotkeys/SKILL.md) | Configure BUC page keyboard shortcuts and verify the matching keybinding listener. | Add a Shift+N shortcut on the create-order page that clicks the 'Next' button. |
| [buc-customize-login-page](buc-customize-login-page/SKILL.md) | Customize OrderHub's login page or shell labels, banner text, and logo assets. | Rebrand the OrderHub login page — change the title text and swap the logo image. |
| [buc-inject-custom-html-js](buc-inject-custom-html-js/SKILL.md) | Add a static HTML or JavaScript fragment to the BUC shell and configure its assets. | Load a Google Analytics snippet across the entire OrderHub shell. |
| [buc-prepare-module-customization](buc-prepare-module-customization/SKILL.md) | Prepare a BUC route for src-custom development, including scaffolding, asset wiring, and local preview. | Prepare the order-details route in buc-app-order for customization. |
| [buc-wire-custom-assets](buc-wire-custom-assets/SKILL.md) | Copy shared module assets and wire merged builds for BUC route-based JSON or code customization. | Prepare the order-search route in buc-app-order for route-based customization. |
| [buc-register-customization](buc-register-customization/SKILL.md) | Register BUC route components, providers, and module imports without duplicate Angular declarations. | How do I register my new custom component so it's included in the feature module? |
