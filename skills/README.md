# Skills

Skills for AI coding assistants that customize IBM Sterling Order Management Order Hub and Call Center (NextGen) with the developer toolkit. Each skill has its own folder containing a `SKILL.md` file. The assistant reads that file when a task matches the skill's description.

The skills are split into three directories:

| Directory | Contents |
| --- | --- |
| [orderhub](orderhub/README.md) | Skills that apply only to Order Hub |
| [call-center](call-center/README.md) | Skills that apply only to Call Center |
| [shared](shared/README.md) | Skills that apply to both products |

## Setting up the skills

Install the shared skills plus the skills for the product you're customizing:

| Project | Copy the contents of |
| --- | --- |
| Order Hub | `orderhub/` and `shared/` |
| Call Center | `call-center/` and `shared/` |

Copy the skill folders themselves, not the directories that contain them. Keep all skills side by side in one flat skills directory: they link to each other with relative paths such as `../buc-prepare-module-customization/SKILL.md`. Don't rename the skill folders, and don't move them into subfolders.

Use your assistant's project skills directory as the target. The commands below use `<project>/.<ide>/skills/`, where `.<ide>` is your assistant's folder in the project root, such as `.bob` for IBM Bob. Check your assistant's documentation for the exact location.

For an Order Hub project, from this `skills` directory:

```bash
mkdir -p <project>/.<ide>/skills
cp -r orderhub/*/ shared/*/ <project>/.<ide>/skills/
```

For a Call Center project:

```bash
mkdir -p <project>/.<ide>/skills
cp -r call-center/*/ shared/*/ <project>/.<ide>/skills/
```

On Windows PowerShell, for an Order Hub project:

```powershell
New-Item -ItemType Directory -Force <project>\.<ide>\skills
Get-ChildItem orderhub, shared -Directory | Copy-Item -Destination <project>\.<ide>\skills -Recurse
```

Replace `orderhub` with `call-center` for a Call Center project.

The per-directory `README.md` files are documentation only, so you don't need to copy them.

Some shared skills link to skills for the other app, which your setup doesn't include:

- **Call Center notes in an Order Hub setup.** Shared skills mark Call Center differences with a **Call Center** note, which can link to a `cc-*` skill. Ignore these notes in an Order Hub project.
- **Order Hub skills in a Call Center setup.** Most links to an Order Hub skill give the Call Center alternative next to it, for example "(Call Center: cc-extend-mashups)". Follow the alternative. Some links have no alternative:
  - Links to `buc-deploy-customization` (from `buc-add-route-package`, `buc-create-new-app`, and several other shared skills) cover Order Hub deployment only. No Call Center deployment skill is included.

## Where to start

- **Customizing an IBM page:** `buc-prepare-module-customization` sets up the module and route.
- **A new capability outside IBM code:** `buc-create-new-app` creates your own app, and `buc-add-route-package` adds route packages to it or to an IBM module.
- **Any UI work:** `buc-find-building-blocks` searches the component library before you write code.

The skills assume the developer toolkit and the `@buc/schematics` collection are already installed.
