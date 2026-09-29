---
name: buc-add-custom-hotkeys
description: Add keyboard shortcuts to an Order Hub page through the hotkeys section of buc-page-definitions.json, including CSS targeting and troubleshooting.
---

# Add custom hot keys

Hot keys are declared in the `hotkeys` object of `buc-page-definitions.json` and
work by targeting an element with a CSS selector, then clicking or focusing it.

Hot keys are available for any Order Hub screen except those in `buc-app-sfo`.

## 1. Choose where the file goes

**Mandatory first check:** does the page already start the listener with its page name?

```
grep -n "initializeHotKeyListener" packages/<route>/src/app/app.component.ts
```

Most IBM routes already do. If it's missing, or passes a different name, the change
needs code (step 4), so use the route location and complete
[buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md).

Otherwise, the file's location determines whether you take ownership of a route:

| Situation | Put the file at |
|---|---|
| Hot keys are your only change to this module | `packages/<app>-root-config/src/assets/custom/buc-page-definitions.json` |
| You're already customizing the route | `packages/<route>/src-custom/assets/custom/buc-page-definitions.json` |

Prefer the first location. A root-config file isn't tied to a route, so future Order Hub
releases apply automatically with nothing to re-synchronize.

Check the page is listed in
`buc-app-<module>/packages/<module>-shared/assets/buc-app-<module>/buc-page-definitions.json`
before starting.

## 2. Define the hot keys

```json
{
  "create-contract-order": {
    "name": "create-contract-order",
    "actions": [],
    "tabs": [],
    "hotkeys": {
      "go-next": {
        "id": "go-next",
        "description": "CREATE_CONTRACT_ORDER.KEY_BINDINGS.GO_NEXT",
        "keybinding": "shift+n",
        "type": "click",
        "elementIdentifier": "[tid='contract-order-create-next'] button"
      },
      "focus-first-element": {
        "id": "focus-first-element",
        "description": "CREATE_CONTRACT_ORDER.KEY_BINDINGS.FOCUS_FIRST_ELEMENT",
        "keybinding": "shift+1",
        "type": "focus",
        "elementIdentifier": "buc-label#contractOrderName input"
      }
    }
  }
}
```

| Property | Notes |
|---|---|
| `id` | Unique ID for the hot key |
| `description` | Translation key describing what it does |
| `type` | `click` for buttons and links, `focus` for input fields |
| `elementIdentifier` | CSS selector for the target element |
| `keybinding` | Keys joined by `+`. Modifiers: `shift`, `ctrl`, `alt`, `meta` |
| `keybinding_$(locale)` | Optional per-locale override; falls back to `keybinding` |

**Targeting:** prefer `tid` attributes (`[tid='button-id'] button`) — they're stable
across releases in a way that generated class names aren't. Be specific enough to
match one element.

**Choosing keys:** avoid browser shortcuts, favor `shift` combinations, and stay
consistent across similar pages.

## 3. Add the descriptions

Under `KEY_BINDINGS` in the page's `en.json`:

```json
{
  "CREATE_CONTRACT_ORDER": {
    "KEY_BINDINGS": {
      "GO_NEXT": "Navigate to the next page.",
      "FOCUS_FIRST_ELEMENT": "Move to the name field."
    }
  }
}
```

Start each description with a verb, keep it short, and end with a period.

## 4. Start the listener

Skip this step if the check in step 1 found the listener with the right page name.

Otherwise copy `packages/<route>/src/app/app.component.ts` to
`packages/<route>/src-custom/app/app.component.ts` (never edit `src/`) and add the
call, keeping the existing constructor and lifecycle code:

```ts
import { KeybindingActionService } from '@buc/common-components';

export class AppComponent extends BucCommonClassesAppComponentClazz implements AfterViewInit {
  constructor(private readonly keybindingService: KeybindingActionService) {
    super(singleSpaStandaloneModeHelperClazzService);
  }

  ngAfterViewInit(): void {
    this.keybindingService.initializeHotKeyListener('create-contract-order');
  }
}
```

The name passed here must match the page key in `buc-page-definitions.json`.

## 5. Verify

Build first so syntax errors surface, then ask the user to open the page, press
each combination, and report what happened.

If a hot key does nothing, check the following in order:

- Does the selector match exactly one element? Ask the user to test it in Dev Tools.
- Is the selector valid? Escape spaces (`input#field\ Name`) or rewrite without them.
- Is the element visible and enabled at the time you press the key?
- Does the combination collide with a browser shortcut?
- Does the page name match `initializeHotKeyListener()`?
- Is the target non-interactive (a `<div>`)? It needs a `tabIndex` to be focusable.

If the description doesn't show, check the translation key path, that
`KEY_BINDINGS` sits under the right parent object, and that `en.json` is valid JSON.

## Action shortcuts and listener lifetime

`type: "action"` with `actionName` dispatches a registered action through
`ActionProcessorService`; it needs no `elementIdentifier`. `shift+h` is reserved
for the built-in shortcut-help modal. `setConfig()` clears all previous hotkeys
before registering the new page, so a second listener replaces the first.
For `buc-button`, the click handler checks the development-only
`ng-reflect-is-disabled` attribute; verify disabled behavior in production too.

For route customization, first complete [buc-prepare-module-customization](../buc-prepare-module-customization/SKILL.md).
