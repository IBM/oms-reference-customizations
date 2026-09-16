---
name: buc-add-custom-hotkeys
description: >
  Configure BUC page keyboard shortcuts and verify the matching keybinding listener.
---

# Add Custom Hot Keys

Configure shortcuts in JSON. Screens within `buc-app-sfo` do not support them.

Hotkey configuration lives in `buc-page-definitions.json` inside the route's custom directory:

```
<module>/packages/<route>/src-custom/assets/custom/buc-page-definitions.json
```

## Step 1 — Add the `hotkeys` block

Create or edit `buc-page-definitions.json` in the custom directory selected above.

```json
{
  "create-contract-order": {
    "name": "create-contract-order",
    "actions": [], "tabs": [],
    "hotkeys": {
      "go-next": {
        "id": "go-next",
        "description": "KEY_BINDINGS.GO_NEXT",
        "keybinding": "shift+n",
        "type": "click",
        "elementIdentifier": "[tid='contract-order-create-next'] button"
      },
      "focus-first-element": {
        "id": "focus-first-element",
        "description": "KEY_BINDINGS.FOCUS_FIRST_ELEMENT",
        "keybinding": "shift+1",
        "type": "focus",
        "elementIdentifier": "buc-label#contractOrderName input"
      }
    }
  }
}
```

| Field | Notes |
|---|---|
| `type` | `click` simulates a click on the target; `focus` moves keyboard focus (for inputs). |
| `elementIdentifier` | CSS selector. Prefer `tid` test-id attributes — they're stable across UI changes. Escape spaces with `\ `. |
| `keybinding` | e.g. `"shift+n"`, `"ctrl+s"`. Modifiers: `shift`, `ctrl`, `alt`, `meta`. |
| `keybinding_$(locale)` | Optional per-locale override. |

Add `description` translation keys under `KEY_BINDINGS` in the selected custom directory's `i18n/en.json`.

## Step 2 — Initialize the page's keybinding listener

Check listener initialization before selecting a JSON-only approach. If it is absent, explain
that route code is required and resolve that change with the user. Prepare the route in code
mode and keep the JSON in its selected location only after verifying the loader supports it.
If not already active, inject `KeybindingActionService` from `@buc/common-components` in the
page's `app.component.ts` and call it in `ngAfterViewInit`:

```ts
this.keybindingService.initializeHotKeyListener('create-contract-order'); // page name must match the top-level key above
```

## Troubleshooting

- Confirm the `elementIdentifier` selector matches a visible, enabled element.
- Escape spaces in selectors.
- Check for conflicts with built-in browser shortcuts.
- Confirm the page name passed to `initializeHotKeyListener()` matches the JSON's top-level key
  exactly.
- Non-interactive elements (e.g. a plain `<div>`) need `tabIndex` set to become focusable —
  otherwise `type: "focus"` silently does nothing.
- For missing translations, check the key path and that `KEY_BINDINGS` is a valid parent object
  in the JSON.

