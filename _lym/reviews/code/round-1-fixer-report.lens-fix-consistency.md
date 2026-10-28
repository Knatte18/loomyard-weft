```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix commits `48a277c2..11367dda` for agreement with their surroundings (lens fix-consistency).

**Findings**

- **LOW**, `internal/fabricengine/doc.go`, "The hub workspace file" section, the added sentence "A beam's file splices the beam's own `.vscode/settings.json` when it exists, and the prime's otherwise."
  - The F3 fix added a third case to the code: a beam whose settings cannot be read is logged at Warn and gets the prime's settings.
  - The module doc says "otherwise", which reads as "no file", and it never mentions the fallback.
  - Evidence: `hubworkspace.go` now has `logger.Warn(... "beam", beam.slug ...)` followed by `beamSettings = settings` in the `beamWorkspaceSettings` error branch.
  - The same module doc and the `RegenerateHubWorkspace` comment list which failures return an error. A reader there cannot tell that an unreadable beam settings file is not one of them.
  - A short clause fixes it: "...and the prime's when it has none or cannot be read, with a Warn."

- **LOW**, `internal/fabricengine/hubworkspace.go`, the `beamWorkspaceSettings` doc comment: "An unreadable anchor or settings file is an error, which the caller answers by splicing primeSettings."
  - The comment now describes what one caller does with the error.
  - That is a consistency risk for a function documented as a standalone contract, and the point is already carried by the Warn branch in the code.
  - Suggestion: restore the original sentence and keep the fallback in the caller's inline context.

No inconsistency found in the other changes:
- F4's output-only wording in `UpsertPayloadKeys` and the `cli.go` payload help agrees with `boardengine/doc.go:109` ("`beam_color` is no upsert key"). The `owner_machine` twin at `boardengine/doc.go:118` is a different key and still holds.
- The F7 `beamcli/doc.go` step 2 text agrees with the `create.go` condition `!entry.Beam || entry.Owner != here || entry.BeamColor == ""`.

**Tool calls**
1. `git diff d2007e9a -- internal/fabricengine/hubworkspace.go` plus a grep for `owner_machine` in docs: my context lacked the sed-modified `hubworkspace.go` text, and I needed to find twin statements about upsert keys.
