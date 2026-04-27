# Clone Work Items

Quickly clone one or many work items under a new parent, directly from the
Backlog.

## What it does

Select one or more work items in the Backlog (or in a query result), right-click,
choose **Clone Work Items**, type the new parent's ID, and the extension creates
one-to-one copies underneath that parent.

### Hierarchy is preserved

If the items you select form a hierarchy on the original board
(for example **Epic → Feature → User Story → Task**), the clones rebuild the
**same hierarchy** underneath your new parent. Items whose original parent
isn't part of the selection are attached directly to the new parent.

### Field rules for every cloned work item

- **Area Path** and **Iteration Path** are inherited from the new parent
- **Assigned To** is cleared (unassigned)
- **State** is set to the work-item type's own default initial state — the
  first state in the *Proposed* category, so it works for custom processes
  where "New" doesn't exist (e.g. "To Do", "Proposed", "Open", ...)
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
