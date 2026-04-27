# Clone Work Items

Quickly clone one or many work items under a new parent, directly from the
Backlog.

## What it does

Select one or more work items in the Backlog (or in a query result), right-click,
choose **Clone Work Items**, type the new parent's ID, and the extension creates
one-to-one copies underneath that parent.

For every cloned work item:

- **Area Path** and **Iteration Path** are inherited from the new parent
- **Assigned To** is cleared (unassigned)
- **State / Reason** are reset to the work-item type's default new state
- All other fields (Title, Description, custom fields, story points, ...) are
  copied as-is

## Optional link copying

Parent / Child links are never copied — the new parent you specify replaces
them. For everything else you can opt in via checkboxes:

- Related links
- Predecessor / Successor
- Affects / Affected By
- Tested By / Tests
- Remote / other work-item links
- Hyperlinks
- Attachments
- Tags

## Confirmation

When cloning finishes, a summary lists every source → new work item ID with
clickable links so you can jump straight into the new items.

## Required permissions

- `vso.work_write` — read and create work items in the project
