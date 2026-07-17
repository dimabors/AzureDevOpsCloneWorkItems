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
- **Link target type filter** — when the dialog opens it analyzes everything
  the selected items link to and shows a checkbox per target work-item type
  (e.g. *Test Case (4 linked items)*). Uncheck a type and links pointing at
  it are not copied. Combines with the link-kind checkboxes above.
- **Inaccessible link targets never break the clone** — links to work items
  that don't exist or that you can't read (TF401232) are detected up front
  and skipped with a warning. A retry safety net also strips any link the
  server still rejects at create time, so a bad link can never fail the item.
- **Clone linked Test Cases too** (optional) — instead of linking clones to
  the original test cases, create copies of the test cases as well:
  - optional separate **parent work item ID** for the cloned test cases
    (defaults to the main new parent)
  - optional **Test Plan ID + Test Suite ID** — cloned test cases are added
    to that suite automatically
  - the cloned work items get *Tested By* links pointing at the **new** test
    case copies (Azure DevOps adds the reverse *Tests* links automatically)
- Original Parent / Child links from outside the selection are **never**
  copied — your new parent (or the cloned ancestor) is used instead
- Final summary lists every source → new work item ID with clickable links,
  plus every link that was intentionally skipped and why

## Project Structure

```
.github/
  workflows/
    publish.yml         # CI: validates version, publishes to Marketplace, tags
clone-work-items/
  vss-extension.json    # Extension manifest (version = single source of truth)
  package.json          # tfx-cli + vss-web-extension-sdk
  src/
    menu-action.html    # Registers the menu handler, opens the dialog
    clone-dialog.html   # Dialog UI + cloning pipeline (REST)
  img/
    icon.png            # 128x128 marketplace icon
  docs/
    overview.md         # Marketplace overview page
    contribution.md     # Contributor guide
```

## Getting Started

```powershell
cd clone-work-items
npm install
npm run package        # builds the .vsix locally
```

Then upload the `.vsix` to your Azure DevOps organization via
**Organization Settings → Extensions → Manage Extensions → Upload extension**,
or share the marketplace listing with your org.

## Releasing (CI)

Publishing is automated with GitHub Actions
([.github/workflows/publish.yml](.github/workflows/publish.yml)):

1. Bump `version` in `clone-work-items/vss-extension.json` and push to `main`.
2. The workflow validates the version against the latest `v*` git tag — it
   **fails** if the version is unchanged, lower than the last release, or not
   valid `X.Y.Z` semver.
3. It packages the `.vsix`, uploads it as a build artifact, publishes it to
   the Marketplace, and only then creates and pushes the `v<version>` tag.

Requires a repository secret named `AZDO_MARKETPLACE_PAT`: an Azure DevOps PAT
scoped to **All accessible organizations** with **Marketplace → Manage**.

## How It Works

1. The menu contribution (`ms.vss-web.action`) is registered against
   `work-item-context-menu`, `backlog-item-menu`, `product-backlog-item-menu`
   and the query-result menus, so the action shows up wherever a backlog or
   query selection exists.
2. When clicked, it collects the `workItemIds` from the action context and
   opens the dialog contribution via `IHostDialogService.openDialog`.
3. On open, the dialog **analyzes link targets**: it fetches the selected
   items' relations, batch-loads every linked work item
   (`getWorkItems` with `errorPolicy=Omit`), flags targets that can't be
   read, and builds the per-type link filter checkboxes from the results.
4. On Clone, it loads the new parent work item (`getWorkItem`) to read its
   `System.AreaPath` and `System.IterationPath`.
5. It fetches every selected source with `expand=All`, then performs a
   topological sort using each item's `System.LinkTypes.Hierarchy-Reverse`
   relation so a parent within the selection is always cloned before its
   children.
6. If **Clone linked Test Cases** is enabled, the accessible test cases on
   `TestedBy-Forward` links are cloned first (under their own parent if one
   was given) and optionally added to the specified plan/suite via
   `TestHttpClient.addTestCasesToSuite`. A `origTestCaseId → newId` map is
   kept so the main clones link to the copies.
7. For each source it looks up the work-item type's default initial state
   via `getWorkItemType` (first state in the *Proposed* category), builds a
   JSON-Patch document copying every field except the system / overridden
   ones, attaches the correct parent link (cloned ancestor when available,
   otherwise the user-specified new parent), filters/rewrites relations
   (inaccessible targets skipped, unchecked target types skipped, Tested By
   rewritten to cloned test cases), and posts to `createWorkItem(...)` with
   `bypassRules=false` so workflow defaults apply.
8. If the server still rejects the create with **TF401232** (link target
   missing / not readable), the offending relation is stripped and the create
   is retried — the work item itself always gets created.
9. A `sourceId → newId` map is maintained so children can be re-parented to
   the freshly cloned ancestor.
10. Results (successes, errors, and skipped links) are summarised with
    deep-links to the new items.

## Required Scopes

- `vso.work_write` — read + create work items in the project
- `vso.test_write` — add cloned test cases to a test plan / suite

## License

[MIT](LICENSE)

Inspired by the structure of
[dimabors/wiki-links-devops](https://github.com/dimabors/wiki-links-devops).
