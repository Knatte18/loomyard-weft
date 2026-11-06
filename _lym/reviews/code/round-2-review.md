---
verdict: APPROVED
---

No defects found in `git diff b30cccce..HEAD`.

Each focus question was settled against the files as they stand:

- **Slack sender condition.**
  Every bullet in `contracts/orchrole/role-slack.md` that grants authority or relays mail names `<shortname>:SLACK`: `[slack]`, `[slack presence]` and `[slack relay]`.
  Mail that merely opens `[slack]` from another sender is covered by none of them, so it cannot read as an operator instruction.
  The role file sends the orch to the file "on mail from `<shortname>:SLACK`", the same sender.
- **Binding rules against b30cccce.**
  I read the old role and the new role with its five on-demand files side by side.
  Every sender-conditioned rule survives with its force: `<shortname>:DOBBY` and `<shortname>:<slug>:DOBBY` for dobby lines and notices, the `(other hub, trusted orch)` and `(other hub, untrusted, ...)` stamps with `trusted_hubs`, `EXTERNAL` answered in reply text, and a hold notice's `«` `»` text as data.
  The never-on-your-own intake rule stays in the role, under Where a finding goes.
  The never-goto-to-Plan-Bouncer rule lives in `role-landing-halts.md`, which the role sends the orch to on any Parent-Align or Finalize halt.
  Each passage the summary lists as dropped is a repeat, a reason or history, and the rule it belonged to is still stated once.
- **Test assertions.**
  The changed tests in `internal/orchengine` and `internal/orchcli` assert only the filled `Decision for <slug>:` marker.
  `TestRenderRoleFile_WritesEachOnDemandFileBesideTheRoleThatNamesIt` pins that each file renders beside the role and opens with its heading, which is render behaviour, not prose.
- **`docs/overview.md`.**
  The `lym help index` sentence now says the `ly:cli` skill carries the index and the orch's role file points at it, which matches the role's pointer line.

A search for `command_index`, `indexRunner`, `operatorIndex` and `RenderRoleFile` callers left no stale reference.
The new `RenderRoleFile` signature, the removed `Index` option and the removed `index` argument of `NewWatcher` agree at every call site in the diff.
The summary's character counts for the on-demand files match the files on disk once the leading comment is stripped.

## Rejected

- The `pattern/PATTERN-stencil-ownership.md` line is an additive note that the on-demand files share the role's exception.
  No declared change forbids it, and the documentation of the rule it describes landed with the diff that changed the rule.
- The summary's "after" token figures use the report's characters-per-token ratio and not a `measure.sh` transcript.
  The summary says so itself, and the decision record accepts "the same count".
