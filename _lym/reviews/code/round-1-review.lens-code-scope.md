```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens. I map each change in the diff to the board entry's Goal points and flag any change or Goal point that has no counterpart.

**Decisions** (the board entry's `## Goal`; the record has no `Added after Discussion` entries)
- G1: Record-Write refuses, or fails the run, when the Goal body is a CLI error envelope or otherwise not the board body.
- G2: The upstream step that fed Record-Write the error is traced.

**Per change**
- C1, `ErrorEnvelopeLine` (`internal/output/output.go`, its file header, `TestErrorEnvelopeLine`): traces to G1. It is the detector the refusal needs.
- C2, Record-Write's Stuck branch (`internal/loomshed/recordwrite.go`): traces to G1. This covers `errorEnvelopeBodyReason`, the envelope branch in `Call` and its doc comment, the `internal/loomshed/doc.go` line and `TestRecordWrite_Call/ErrorEnvelopeBodyIsStuckNamingTheEnvelope`. The `seam_enforcement_test.go` allowlist entry for `internal/output` is forced by C2's import.
- C3, the `--body-file` refusal: traces to G2. This covers `applyBodyFile` (`internal/boardcli/bodyfile.go`), its `TestApplyBodyFile` row and the `internal/boardcli/doc.go` line. The trace found that the envelope entered the board through `upsert --body-file`, so guarding that write is the traced step's fix.
- C4, bare-slug lookup: traces to G2. This covers `resolveLookup` and `isBareSlug`, the `get my-task` help example, `payloadLegend`, the `internal/boardcli/doc.go` line and the `TestCLI` `get_by_bare_slug` row. The trace is in commit `cecf8b2b9`'s message and in `_lym/landing/summary.md`.
  - The trace holds up. `lym board get &lt;slug&gt;` is taught in `contracts/orchrole/role-beams.md:73`. A bare slug failed as JSON, and the reported envelope's `invalid character 'h'` matches the first letter of the `halt-…` slug.
  - The widening to every lookup verb (set-status, rename, promote, set-deps, and `get --body`/`--record`) is forced, because they all share `resolveLookup`.

**Per decision**
- G1: built by C1 and C2. The "or otherwise not the board body" clause builds nothing beyond envelope detection. Record-Write renders `BoardRecord(body)` from the board entry itself, so the rendered Goal is the board body by construction. The reported failure, an envelope in place of the text, is covered. I read no concrete non-envelope case into the Goal, so I raise no finding.
- G2: built by C3 and C4, and by the trace in the commit message and summary.

**Findings:** none.

Tool calls: none. The manifest already held the diff, the commit log, the summary, the decision record, and the `role-beams.md:73` grep hit.
