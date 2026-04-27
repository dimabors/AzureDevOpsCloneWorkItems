# Clone Work Items – Azure DevOps Extension

An Azure DevOps extension that adds a **Clone Work Items** action to the Backlog
(and to query results / context menus). Select one or more work items, pick a
new parent, and the extension creates one-to-one clones underneath it.

## Features

- Multi-select on the Backlog → right-click → **Clone Work Items**
- Pick a **new parent work item ID** in a dialog
- Cloned items inherit:
  - **Area Path** from the new parent
  - **Iteration Path** from the new parent
- Cloned items always get:
  - **Assigned To** cleared (unassigned)
  - **State / Reason** reset to the work-item type's default new state
- Optional checkboxes to copy non-hierarchy links:
  - Related, Predecessor/Successor, Affects, Tested By, Remote, Hyperlinks,
    Attachments, Tags
- Parent / Child links are **never** copied — the new parent you specify is
  used instead
- Final summary lists every source → new work item ID with clickable links

## Project Structure

```
clone-work-items/
  vss-extension.json    # Extension manifest
  package.json          # tfx-cli + vss-web-extension-sdk
  src/
    menu-action.html    # Registers the menu handler, opens the dialog
    clone-dialog.html   # Dialog UI + cloning pipeline (REST)
  img/
    icon.png            # 128x128 marketplace icon (add your own)
  docs/
    overview.md         # Marketplace overview page
```

## Getting Started

```powershell
cd clone-work-items
npm install
npm run package        # builds the .vsix locally
# or
npm run publish        # publishes to the marketplace (needs PAT)
```

Then upload the `.vsix` to your Azure DevOps organization via
**Organization Settings → Extensions → Manage Extensions → Upload extension**,
or share the marketplace listing with your org.

## How It Works

1. The menu contribution (`ms.vss-web.action`) is registered against
   `work-item-context-menu`, `backlog-item-menu`, `product-backlog-item-menu`
   and the query-result menus, so the action shows up wherever a backlog or
   query selection exists.
2. When clicked, it collects the `workItemIds` from the action context and
   opens the dialog contribution via `IHostDialogService.openDialog`.
3. The dialog loads the parent work item (`getWorkItem`) to read its
   `System.AreaPath` and `System.IterationPath`.
4. For each source work item, it calls `getWorkItem(id, expand=All)`, builds
   a JSON-Patch document copying every field except the system / overridden
   ones, adds the parent link, and posts to `createWorkItem(...)`.
5. Results (success + errors) are summarised with deep-links to the new items.

## Required Scopes

- `vso.work_write` — read + create work items in the project

## License

[MIT](LICENSE)

Inspired by the structure of
[dimabors/wiki-links-devops](https://github.com/dimabors/wiki-links-devops).
