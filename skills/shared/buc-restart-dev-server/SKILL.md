---
name: buc-restart-dev-server
description: Restart a BUC module after build configuration changes and diagnose startup failures.
---

# Restart the module server

After `overrides.json` or `angular.json` changes, run from the affected module root:

```sh
yarn stop-app
yarn start-app
```

Wait for **every** `runAsCustomization: true` or `"useProd": false` route in `overrides.json` to show **"Compiled successfully"**
in the output — this can take several minutes. A listening message alone does not confirm compilation
or page loading. Leave unrelated modules running.

If startup fails, inspect the preceding error. Exit code 134 alone does not prove heap exhaustion.
For a confirmed Node heap error, increase the limit within available memory, preserving other
`NODE_OPTIONS` flags. Set the variable before restarting — use the syntax for your shell:

```sh
# macOS / Linux / Git Bash
export NODE_OPTIONS="--max_old_space_size=8048"
```

```cmd
:: Windows CMD
SET NODE_OPTIONS=--max_old_space_size=8048
```

```powershell
# Windows PowerShell
$Env:NODE_OPTIONS = "--max_old_space_size=8048"
```

Restart and check that the error is resolved. Review each listener and Git warning in context rather than dismissing all warnings as a group.

Ask the user to confirm compilation and browser behavior if those checks are unavailable.
