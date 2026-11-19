```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. Scope: for each output, key, message, file shape or behaviour the diff changes, find what calls, reads, asserts or describes it, and report each member the diff leaves wrong.

**Affected set, per change**

- **`ValidateDescription` now refuses a non-empty body with no valid Merge danger section.**
  - Callers: `internal/loomcli/validate.go:121`, `internal/landingshed/describe.go:32` (Describe and darn gate), `internal/landingshed/beamdescribe.go:140` (`lym beam describe`), `internal/beamcli/deps.go:460` (pawl ready check). All are covered: the gate, verb, beam-describe and ready-check behaviour follow from them, and the summary names each one.
  - Fixtures standing for a valid description are updated: `describe_test.go:333`, `parity_test.go:770`, `validate_test.go:727`, `beamdescribe_test.go:145`, `deps_integration_test.go:117`, `loomrecipe/fixture_test.go:316`.
  - Fixtures left without the section, all covered because none depends on passing validation:
    - `parity_test.go:784` is the directory-error case and is replaced by a directory.
    - `prreview_test.go:748`, `fixaccept_test.go:18` and `prreview_wait_test.go:141` feed Publish, Finalize and PR-Review, which only parse (no call to `ValidateDescription` among the callers).
    - The `websterengine` summary fixtures belong to a different producer.
  - Docs:
    - `docs/overview.md:1048,1291` list no checks, so nothing goes stale.
    - `pattern/PATTERN-gate-slot.md:60` names the function only.
    - The spec section and the darn and Describe stencils are updated. Covered.
- **The beam draft now ends with Merge danger placeholders that validation refuses.**
  - `landingshed.DraftBeamDescription` and the `beamDraft` docs are updated, and `contracts/orchrole/role-beams.md:49-50` is updated. Covered.
  - Not covered:
    - The `beam describe` help, `internal/beamcli/cli.go:232-234`, lists what the draft holds without the section.
    - The package doc, `internal/beamcli/doc.go:137-141`, lists the same and also omits the section.
    - The draft envelope's `way_forward`, `internal/beamcli/describe.go:47`, says "edit the draft into the beam's end result and its member pull requests" and never says to call the door and blast radius, which the record step now refuses until both are filled in.
- **New shipped heddle entry `lym-glossary` aimed at `orch-prompt`.**
  - `heddleengine/doc.go`, `pattern/PATTERN-heddle-instructions.md:13`, `store_test.go:191` (shipped pin) and the `render_test.go` passage and `Show` expectations are updated.
  - `HEDDLE.md` comes from `RenderOverview`, which the reach report shows reached and which needs no hand edit.
  - No doc lists the shipped slugs (search empty). Covered.
- **New `glossary` marker in `loom-template-discussion` (and, through it, the chair stencil).**
  - `DiscussionTable` fills it as a required marker, the stencil's header comment declares it, and `envkit`'s fake table names only the stencil. Covered.
- **Code-Review rubric bullets (Merge danger, tautological test, mutation evidence).** These are stencil prose only. No other rubric shares this Test economy list, so nothing else is affected.
- **`docs/reference/weaving-glossary.md`** points to `glossary.md`. Covered.
- **Commit messages:** the three content commits claim only what the diff carries, and no `Pattern-Departure:` or `Loosened-Check:` trailer is needed or present.

**Findings**

1. **LOW · consistency** · `internal/beamcli/cli.go:232-234` (also `internal/beamcli/doc.go:137-141` and `internal/beamcli/describe.go:47`)
   - Summary: the `beam describe --draft` help, the beamcli package doc and the draft envelope's `way_forward` still describe the draft as title, brief and commits only. None of them says the draft now ends with Merge danger placeholders that recording refuses until the orch calls door and blast radius.
   - Evidence: the draft now appends `beamMergeDangerDraft` (`internal/landingshed/beamdescribe.go`, diff piece 009, the `beamDraft` hunk), and `role-beams.md:49-50` already says the draft carries both placeholders.
   - Fix: add the placeholder sentence to the help text and the doc.go paragraph, and have `way_forward` name calling the Merge danger door and blast radius.

Tool calls:
1. The spec's validation list, description fixtures across tests, and docs naming validate-description. The context lacked fixtures and docs outside the diff.
2. The `docs/overview.md` summaryparser and validate-description lines, the `parity_test.go` fixture context, any doc listing shipped slugs, and draft mentions in beamcli `describe.go`. Needed to judge which readers go stale.
3. Every `--draft` mention in beamcli, orchrole and overview, plus `role-beams.md:30-50`. Needed to find each description of the draft's contents.
4. `beamcli/cli.go:225-250` and `beamcli/doc.go:125-142`. These hold the draft-content text the third call located.
