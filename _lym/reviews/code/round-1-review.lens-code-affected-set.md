```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens. For each change the diff makes, find what calls, reads, asserts or describes it, and flag any of those the diff leaves wrong.

**Affected set per change**

- **`Topology.Remove` gains a fifth argument, `rerun`.**
  - The one production caller, `pairteardown/teardown.go:247`, is updated.
  - Every test call site in fabricengine, battencli, ideengine and the pairteardown guard snippet is updated.
  - The guard test's constant and doc comment are updated.
  - Not covered: `pattern/PATTERN-pair-remote.md:100` still describes the guard as flagging a "four-argument `Remove` call". Finding 1.
- **`pairteardown.Request.Rerun`.**
  - Set by `battencli/cancel.go:783`.
  - Left empty by `fabriccli/remove.go:87,96`, `beamcli/dobbydeps.go:455`, `battencli/beamfork.go:180` and `battencli/wire.go:449`. Each therefore gets the `lym fabric remove` fallback.
  - The fields are documented in the code; the pairteardown doc.go names no Request field list.
  - Covered.
- **Held teardown lock refusal names the caller's command.**
  - No test or stencil asserts the old "re-run `lym fabric remove <slug>`" text through a cancel. My grep over `*.go` and `*.md` found the string only in `remove.go:275`.
  - `contracts/stencils/shed/shed-template-driver-guide.md:25` is about `lym fabric remove` itself, so it is still correct.
  - Covered.
- **`rerunWayForward` and `recordsArchiveWayForward` wording, plus the `Rerun` doc on the job and record fields.**
  - Callers are `failedRemoteSteps` (`remoteteardown.go:268-276`) and `pendingTeardownWayForward` (`pendingteardown.go:160`).
  - The `lym fabric status` and `prune` help text (`fabriccli/weft_verbs.go:144-148`, `fabriccli/fabric.go:327-331`) says only "the re-run", so it is still correct.
  - `tools/sandbox/SANDBOX-FABRIC-SUITE.md:964-966` describes a `lym fabric remove` record, which keeps the fallback wording. Still correct.
  - `docs/research/single-repo-undriven-cases.md:22,45` is a dated run report under the measurement-report exception. No finding.
  - `unknownKindWayForward` is unchanged and still correct for a job with no readable kind.
  - Covered, apart from Finding 2.
- **fabricengine `doc.go` paragraph.** Finding 2.
- **Commit messages.**
  - `5f8aafb3f` carries `Root-Cause:` naming the defect the diff removes: `Topology.Remove` writes its job with no `Rerun`.
  - `9e7f06e00` only re-breaks comment lines.
  - No `Pattern-Departure:` or `Loosened-Check:` trailer is needed.
  - Covered.
- **Declared PATTERN changes.** The directive declares none, so there is nothing to check.

**Findings**

1. **MEDIUM**, class `consistency`, at `pattern/PATTERN-pair-remote.md:100`.
   The Pair teardown background still says the enforcement test "flags a four-argument `Remove` call". After the diff, the guard flags a five-argument call: `removeArgCount = 5` and the doc comment at `internal/pairteardown/teardown_enforcement_test.go:17,39-42`. The rule document describing the guard is now wrong.
   Fix: change "four-argument" to "five-argument" in the background file.
   Evidence: `pattern/PATTERN-pair-remote.md:100`, and diff piece 005, lines 32-42.

2. **BLOCKING**, class `consistency`, at `internal/fabricengine/doc.go:242`.
   The added sentence "A command such as `lym batten cancel <slug> --remove` finishes even once nothing of the pair remains" states how another package's command behaves. PATTERN-code-conventions says a package's `doc.go` documents only that package.
   The claim is also untrue in one reachable state: when only the records branch on origin remains, `refuseNothingLeft` answers `nothingLeft`, because it never reads the origin records branch (`internal/battencli/cancel.go:735-769`). The code-correctness lens owns that behaviour.
   Fix: keep the paragraph to what fabricengine does, that a job's `Rerun` names its caller's re-run and an empty one falls back to `lym fabric remove <slug> --remote`, and drop the claim about the cancel.
   Evidence: diff piece 002, lines 80-83, and `pattern/PATTERN-code-conventions.md:3,7`.

**Tool calls**

1. Grepped `*.go` and `*.md` for the four-argument guard wording and for `lym fabric remove … --remote` / `re-run \`lym fabric remove`, and read the `fabric status`/`fabric prune` help text. The manifest lacked the docs, stencils and help text that describe the changed outputs.
2. Read `pattern/PATTERN-pair-remote.md:92-106`, `docs/research/single-repo-undriven-cases.md:15-50`, and grepped the pairteardown doc.go. The manifest had read the pattern file only up to line 80, had not read the research doc, and had not read the pairteardown package doc.
