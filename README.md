# Clone Work Items – Azure DevOps Extension

An Azure DevOps extension that adds a **Clone Work Items** action to the Backlog
(and to query results / context menus). Select one or more work items, pick a
new parent, and the extension creates one-to-one clones underneath it.

## Features

- Multi-select on the Backlog → right-click → **Clone Work Items**
- Pick a **new parent work item ID** in a dialog
- **Original hierarchy is preserved** within the selection. If you select an
  Epic → Feature → User Story → Task chain, the clones rebuild the same
  Epic → Feature → User Story → Task chain under your new parent. Items
  whose original parent isn't in the selection are attached directly to the
  new parent.
- Cloned items inherit:
  - **Area Path** from the new parent
  - **Iteration Path** from the new parent
- Cloned items always get:
  - **Assigned To** cleared (unassigned)
  - **State** set to the work-item type's own default initial state
    (the first state in the *Proposed* category — works for any custom
    process where "New" doesn't exist)
- Optional checkboxes to copy non-hierarchy links:
  - Related, Predecessor/Successor, Affects, Tested By, Remote, Hyperlinks,
    Attachments, Tags
- Original Parent / Child links from outside the selection are **never**
  copied — your new parent (or the cloned ancestor) is used instead
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
3. The dialog loads the new parent work item (`getWorkItem`) to read its
   `System.AreaPath` and `System.IterationPath`.
4. It fetches every selected source with `expand=All`, then performs a
   topological sort using each item's `System.LinkTypes.Hierarchy-Reverse`
   relation so a parent within the selection is always cloned before its
   children.
5. For each source it looks up the work-item type's default initial state
   via `getWorkItemType` (first state in the *Proposed* category), builds a
   JSON-Patch document copying every field except the system / overridden
   ones, attaches the correct parent link (cloned ancestor when available,
   otherwise the user-specified new parent), and posts to
   `createWorkItem(...)` with `bypassRules=false` so workflow defaults apply.
6. A `sourceId → newId` map is maintained so children can be re-parented to
   the freshly cloned ancestor.
7. Results (success + errors) are summarised with deep-links to the new
   items.

## Required Scopes

- `vso.work_write` — read + create work items in the project

## License

[MIT](LICENSE)

Inspired by the structure of
[dimabors/wiki-links-devops](https://github.com/dimabors/wiki-links-devops).
