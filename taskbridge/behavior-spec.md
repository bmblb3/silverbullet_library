# Silverbullet Taskwarrior Bridge Behavior Spec

This document is the printable mental model for the bridge. It describes the
current intended behavior in reviewable scenarios. Tests remain the executable
source of truth; when behavior changes, update the relevant test and then this
document in the same small change.

## Terms

- Silverbullet is the reference surface. It holds Markdown tasks plus notes,
  project links, tags, dates, and other helpful context as plain text.
- Taskwarrior is the execution surface. It owns planning fields such as tags,
  due, scheduled, project, wait, priority, annotations, recurrence, and status.
- A Taskking task link in Markdown is the durable identity for a linked task:
  `[first_uuid_segment](${TASKWARRIOR_APP_URL}/tasks/<uuid>)`.
- The Taskwarrior `silverbullet` UDA is a bridge-owned backlink cache. It points
  back to the current Markdown page and character index, or carries a one-time
  command such as `1` or `0`.
- Markdown line numbers in user-facing output are 1-based. Character indexes are
  zero-based and point to the start of the task marker, matching the backlink
  character index.

---

## Standard Failure Output

Failures that can be tied to a bridge source use a standard JSON shape:

- Top level: `ok: false`, `message`, `error_kind`, and usually `issues`.
- Issue level: `kind`, `message`, issue-specific fields, and `sources`.
- Silverbullet source: `source: "silverbullet"`, `page`, `line`, `character`.
- Taskwarrior source: `source: "taskwarrior"`, `uuid`, and `taskking_url` when
  `TASKWARRIOR_APP_URL` is configured.

Silverbullet frontends should primarily branch on `error_kind`,
`issues[].kind`, and `issues[].sources[].source`. The contextual fields are for
display and repair guidance.

Generic setup failures that do not point to a Markdown task or Taskwarrior task
may use `error_kind: "error"` without `issues`.

---

## Scenario: Empty Space Is A No-Op

Given a Silverbullet space with no Markdown tasks and no Taskwarrior tasks.

When `bridge sync --space <PATH>` runs.

Then sync succeeds, creates no files, creates no tasks, and reports no changes.

An immediate second sync is also a no-op.

---

## Scenario: Markdown `#tw` Imports One Task

Given a Markdown checkbox task with the one-time `#tw` command and no Taskking
link.

When sync runs.

Then the bridge creates or reuses a matching Taskwarrior task, removes `#tw`,
adds the Taskking link to the Markdown line, records the task in
`.sbtw_state.json`, and writes the Taskwarrior `silverbullet` backlink.

Then an immediate second sync is a no-op.

---

## Scenario: `#tw` Is Not Permanent State

Given an already linked Markdown task that still contains `#tw`.

When sync runs and the linked Taskwarrior task matches the visible task-line
fields.

Then the bridge rewrites the Markdown line to remove `#tw`.

---

## Scenario: Taskwarrior `silverbullet=1` Exports One Task

Given a pending Taskwarrior task with `silverbullet=1` and no existing Markdown
Taskking link.

When sync runs.

Then the bridge appends the task to `TODO.md`, adds its Taskking link, replaces
the Taskwarrior command value with a backlink like `TODO@0`, records sync state,
and succeeds.

Then an immediate second sync is a no-op.

---

## Scenario: Linked Description Edit From Silverbullet

Given a linked task whose last synced description was `Buy milk`.

When the Markdown line changes to `Buy oat milk` and Taskwarrior still has
`Buy milk`.

Then sync updates Taskwarrior's description to `Buy oat milk`, preserves
Taskwarrior-owned fields, rewrites Markdown only as needed, records state, and
succeeds.

---

## Scenario: Linked Description Edit From Taskwarrior

Given a linked task whose last synced description was `Buy milk`.

When Taskwarrior changes the description to `Buy oat milk` and Markdown still
has `Buy milk`.

Then sync rewrites the Markdown task line to `Buy oat milk`, keeps the Taskking
link, records state, and succeeds.

---

## Scenario: Linked Completion Edit

Given a linked task whose last synced checkbox/status is pending.

When only Taskwarrior marks the task completed.

Then sync checks the Markdown checkbox.

When only Silverbullet checks the Markdown checkbox.

Then sync marks the Taskwarrior task completed.

---

## Scenario: Same Field Conflict

Given a linked task with a last-synced baseline.

When Silverbullet and Taskwarrior both edit the same synced field differently
since that baseline.

Then sync refuses to write Markdown or Taskwarrior, reports a structured sync
conflict with `error_kind: "sync_conflict"`, and includes one issue with both
Silverbullet and Taskwarrior sources, the task UUID, conflicting field,
Silverbullet value, Taskwarrior value, and last synced value.

---

## Scenario: Different Field Edits Merge

Given a linked task with a last-synced baseline.

When Silverbullet changes one synced field and Taskwarrior changes a different
synced field.

Then sync accepts both changes, copies each changed field to the other side,
records the merged state, and succeeds.

---

## Scenario: Silverbullet Metadata Is Description Text

Given a Markdown task containing text such as `[due:tomorrow]`,
`[scheduled:2026-05-10]`, `[[Projects/Admin]]`, or `#home`.

When the task imports to Taskwarrior or syncs as a linked task.

Then those fragments remain ordinary description text. The bridge does not parse
them into Taskwarrior due dates, scheduled dates, projects, or tags.

---

## Scenario: Taskwarrior Fields Stay Backend-Owned

Given a linked Taskwarrior task with backend fields such as tags, due,
scheduled, project, wait, priority, annotations, or recurrence.

When sync updates the linked description or completion.

Then the bridge preserves Taskwarrior-owned fields and does not render them into
Markdown.

---

## Scenario: Backlink Refresh After Markdown Move

Given a linked Markdown task with a valid Taskking link.

When the task line moves to a different page or character position.

Then sync refreshes the Taskwarrior `silverbullet` UDA to the current
`Page@CharacterIndex` value and does not otherwise rewrite the task.

Then an immediate second sync is a no-op.

---

## Scenario: Removed Markdown Line Is Inert

Given a previously linked Markdown task.

When the Markdown task line is removed from Silverbullet.

Then sync does not delete or modify the Taskwarrior task merely because the line
is missing.

After a successful sync, the missing task naturally drops out of
`.sbtw_state.json`.

---

## Scenario: Explicit Taskwarrior Unlink

Given a previously linked Taskwarrior task.

When the Taskwarrior `silverbullet` UDA is set to `0`.

Then sync treats that as an explicit unlink command, drops the linked task from
sync state, and does not export it to Markdown.

---

## Scenario: Completed Or Deleted Task With Stale Backlink

Given a completed or deleted Taskwarrior task whose `silverbullet` UDA still
looks like a backlink but whose Markdown task no longer exists.

When sync runs.

Then the bridge sets `silverbullet=0` for that terminal task and continues.

---

## Scenario: Pending Task With Broken Backlink

Given a pending Taskwarrior task whose `silverbullet` UDA looks like a backlink
or invalid bridge value but no matching Markdown Taskking link exists.

When sync runs.

Then sync refuses loudly with `error_kind: "broken_silverbullet_link"` and a
Taskwarrior source. The user must either restore/fix the Markdown link or set
`silverbullet=0` to unlink the Taskwarrior task.

---

## Scenario: Invalid Markdown Task

Given a Markdown checkbox task with invalid bridge metadata, such as duplicate
Taskking links on the same line.

When sync runs.

Then sync refuses before creating or updating Taskwarrior tasks, preserves the
original Markdown line, and reports `error_kind: "invalid_markdown"` with a
Silverbullet source, description, and typed invalid reason.

---

## Scenario: Duplicate Markdown UUID

Given two or more Markdown tasks with the same valid Taskking UUID.

When sync runs.

Then sync refuses before writing either side and reports every Markdown location
that uses the duplicate UUID in `error_kind: "duplicate_markdown_uuid"` issues.

---

## Scenario: Linked Markdown Task Missing In Taskwarrior

Given a Markdown task with a valid Taskking link for a UUID that is unavailable
in Taskwarrior.

When sync runs.

Then sync refuses before writing either side and reports the Markdown page,
line, character, UUID, description, and Taskwarrior source in
`error_kind: "unavailable_in_taskwarrior"` issues.

---

## Scenario: Concurrent Markdown Edit

Given the bridge has read a Markdown document and planned a write.

When the file changes on disk before the bridge writes its update.

Then sync refuses, does not overwrite the external edit, and does not record a
successful sync state.

---

## Scenario: Sync Lock Already Held

Given another bridge sync holds the Silverbullet space lock.

When sync starts.

Then sync refuses before scanning/writing Markdown or syncing the backend.

---

## Scenario: Backend Failure Before Markdown Write

Given the bridge plans backend changes and Markdown changes.

When backend create, update, or server sync fails before Markdown is written.

Then sync refuses, leaves Markdown unchanged, and does not record successful
sync state.

---

## Scenario: Backend Failure After Markdown Write

Given the bridge has written Markdown and then needs to refresh the Taskwarrior
`silverbullet` backlink.

When that post-write backend operation or server sync fails.

Then sync refuses and does not record successful sync state. A retry must not
duplicate the Markdown export/import.

---

## Scenario: Successful Sync State

Given sync completes successfully.

When `.sbtw_state.json` is written.

Then it records only currently linked Markdown tasks that also exist in
Taskwarrior, using the current visible synced fields as the last-synced
baseline.

---

## Scenario: Idempotence

Given any successful sync that changed Markdown or Taskwarrior.

When sync runs again immediately with no user changes.

Then the second sync must be a no-op.
