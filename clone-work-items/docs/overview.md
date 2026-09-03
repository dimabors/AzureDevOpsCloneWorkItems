# Clone Work Items

Quickly clone one or many work items under a new parent, directly from the
Backlog.

## Demo

[![Cloning a work item hierarchy from the Azure DevOps backlog](https://raw.githubusercontent.com/dimabors/AzureDevOpsCloneWorkItems/main/media/clone-work-items-demo.gif)](https://github.com/dimabors/AzureDevOpsCloneWorkItems/blob/main/media/clone-work-items-demo.mp4)

Selecting a hierarchy on the Backlog, choosing a new parent and link options in
the dialog, and reviewing the summary of cloned items.
Click the animation for the
[full-resolution recording](https://github.com/dimabors/AzureDevOpsCloneWorkItems/blob/main/media/clone-work-items-demo.mp4).

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

### Filter links by target work-item type

When the dialog opens it analyzes everything your selected items link to and
shows one checkbox per **target type** — for example
*Test Case (4 linked items)* or *Software Requirement (7 linked items)*.
Uncheck a type and links pointing at items of that type are not copied.
This combines with the checkboxes above: a link is copied only if its link
kind **and** its target's type are both enabled.

### Broken or unreadable links never break the clone

Links pointing at work items that no longer exist — or that you don't have
permission to read — are detected up front, shown as a warning, and skipped
automatically. The work items themselves are always cloned; every skipped
link is listed in the summary so nothing disappears silently.

## Clone linked Test Cases too

If the selected items have **Tested By** links to Test Cases, you can choose
to clone the test cases as well instead of linking to the originals:

- The cloned work items get *Tested By* links pointing at the **new** test
  case copies (the reverse *Tests* links are created automatically).
- Optionally give a **different parent work item ID** for the cloned test
  cases — otherwise they go under the same new parent.
- Optionally enter a **Test Plan ID and Test Suite ID** — the cloned test
  cases are then added to that suite automatically.
- Test case steps, fields, tags, hyperlinks and attachments are copied;
  their old work-item links are not (they would point back at the originals).

## Confirmation

When cloning finishes, a summary lists every source → new work item ID with
clickable links so you can jump straight into the new items, plus a list of
any links that were skipped and why.

## Required permissions

- `vso.work_write` — read and create work items in the project
- `vso.test_write` — add cloned test cases to a test plan / suite
