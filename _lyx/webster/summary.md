# lyx shuttle open chooses the permission mode

Batch 01 (`01-open-spec-permission-mode`) finished with `status: done` at head `691ed4380b91b4a6bca3efd191bcd33fef550228`.

It covered the plan's three cards, one commit each:
- `OpenSpec` gains `PermissionMode`, passed blind through `Runner.Open` to `OpenLaunch` and logged on the opened-session line.
- `Claude.OpenLaunch` validates the mode, and `buildOpenCmd` adds the skip flag to the fresh and resume lines.
- `lyx shuttle open --permission-mode` fills `OpenSpec.PermissionMode`, with help text and the `docs/overview.md` row.

The digest listed no deviations and no warnings, so the scope guard found no files changed outside the batch's targets.
The plan-level verify gate runs after this turn ends.
