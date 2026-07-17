# Contributing

Thanks for considering a contribution!

## Local setup

```powershell
cd clone-work-items
npm install
```

## Packaging a local .vsix

```powershell
npm run package
```

This produces a `.vsix` file you can upload via
**Azure DevOps → Organization Settings → Extensions → Upload extension** for
private testing.

## Publishing

Releases are automated via GitHub Actions
(`.github/workflows/publish.yml`):

1. Bump `version` in `vss-extension.json` and push to `main`.
2. The workflow fails fast if the version is **unchanged**, **lower** than
   the latest released `v*` tag, or not valid `X.Y.Z` semver.
3. On success it packages the `.vsix`, uploads it as a build artifact,
   publishes to the Marketplace, and then pushes the `v<version>` tag.

The workflow needs a repository secret **`AZDO_MARKETPLACE_PAT`** — an Azure
DevOps PAT created with Organization set to **All accessible organizations**
and the **Marketplace → Manage** scope.

Manual publishing still works if you prefer:

```powershell
npm run publish -- --token <PAT>
```

You need publisher access to `DimaBors` (or change the `publisher` field in
`vss-extension.json` to your own publisher id).

## Code layout

- `src/menu-action.html` — registers the backlog/query menu handler and opens
  the dialog. Keep it tiny; it should only marshal the selected work-item ids
  into the dialog.
- `src/clone-dialog.html` — UI + cloning pipeline. All REST work happens here
  via `TFS/WorkItemTracking/RestClient` (and `TFS/TestManagement/RestClient`
  for adding cloned test cases to a suite). Key pieces:
  - `analyzeLinkTargets()` — runs when the dialog opens; batch-loads every
    linked work item (`errorPolicy=Omit`), marks unreadable targets, and
    builds the per-type link filter checkboxes.
  - `buildPatchDoc()` — builds the JSON-Patch for one clone; skips links to
    inaccessible targets and unchecked target types, and rewrites Tested By
    links to cloned test cases via `opts.tcCloneMap`.
  - `createWithLinkRetry()` — safety net: if the server rejects a create with
    TF401232, the offending relation is stripped and the create is retried.
  - `cloneTestCases()` / `addClonesToSuite()` — optional test-case cloning
    and plan/suite assignment.

## Adding a new optional link type

1. Add a checkbox in `src/clone-dialog.html` under the `.checks` block.
2. Map its id in `shouldCopyRelation(rel)` to the corresponding
   `rel.rel` string (see the
   [link reference types docs](https://learn.microsoft.com/azure/devops/boards/queries/link-type-reference)).

## Manifest scopes

`vss-extension.json` currently requests:

- `vso.work_write` — read + create work items
- `vso.test_write` — add test cases to suites

Adding a scope means org admins must re-authorize the extension on update —
only add one when a feature genuinely needs it.

## Reporting issues

Please file issues at
<https://github.com/dimabors/AzureDevOpsCloneWorkItems/issues> with:

- Azure DevOps organization type (cloud / Server)
- Steps to reproduce
- A screenshot of the dialog's progress/summary panel if cloning failed
