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

```powershell
npm run publish -- --token <PAT>
```

You need a Personal Access Token with **Marketplace (Publish)** scope and
publisher access to `DimaBors` (or change the `publisher` field in
`vss-extension.json` to your own publisher id).

## Code layout

- `src/menu-action.html` — registers the backlog/query menu handler and opens
  the dialog. Keep it tiny; it should only marshal the selected work-item ids
  into the dialog.
- `src/clone-dialog.html` — UI + cloning pipeline. All REST work happens here
  via the `TFS/WorkItemTracking/RestClient`.
- `vss-extension.json` — manifest. Bump `version` for every published build
  (or use `--rev-version` in the npm scripts).

## Adding a new optional link type

1. Add a checkbox in `src/clone-dialog.html` under the `.checks` block.
2. Map its id in `shouldCopyRelation(rel)` to the corresponding
   `rel.rel` string (see the
   [link reference types docs](https://learn.microsoft.com/azure/devops/boards/queries/link-type-reference)).

## Reporting issues

Please file issues at
<https://github.com/dimabors/AzureDevOpsCloneWorkItems/issues> with:

- Azure DevOps organization type (cloud / Server)
- Steps to reproduce
- A screenshot of the dialog's progress/summary panel if cloning failed
