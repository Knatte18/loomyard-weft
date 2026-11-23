---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed-differently
    reason: "The proposed Guard and Admitted hooks need a loomCLI with a location (internal/loomcli/arm.go:478-479, prreviewgoto.go:64-67, settledguard.go:21-30), which a tier0 fixture cannot build cheaply; the review's own fallback applies, so the goto stays bare and the doc comment is narrowed. The implementerSeam half is made as proposed."
---

## Changes

Commit a3f3e60e4, `internal/loomcli/amendduringwebster_test.go`:

- F1: dropped `statePath` and both `os.ReadFile` calls.
  The record is read with `websterengine.LoadState(websterengine.Dir(anchorPath), websterengine.ScratchDir(anchorPath))` before `decisionVerb` and after `shedengine.Goto`.
  The test asserts the second value is non-nil and `reflect.DeepEqual` to the first, with the message `LoadState() after the goto = %+v; want %+v`.
  The doc comment now says the record holds the same batches as before.
- Gate repair: the doc comment's wrapped line, which broke mid-sentence, is rewritten one sentence per line.
- F2: set `f.implementerSeam = weaverengine.ActiveDeps{AnchorRoot: anchorPath}` so the verb's implementer read runs over the planted record.
  The goto stays bare and the doc comment says the record survives `decisionVerb` over its test seams and the engine's goto, not loom's goto hooks.
  Not made: passing `Guard` and `Admitted`, which need a `loomCLI` anchored at `anchorPath`.

`lym loom fmt` and `go vet ./internal/loomcli` are clean; `lym gate test ./internal/loomcli` passed tier0 with 0 failed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, fix-proposal, fix-affected-set: not spawned. My one spawn call carried only fix-code-correctness, and a second pass is not allowed.

## Disputed

None.

## Deferred

None.

## Rulings

None.
