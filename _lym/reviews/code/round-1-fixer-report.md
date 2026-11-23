---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
---

## Changes

- F1: `errorEnvelopeBodyReason` in `internal/loomshed/recordwrite.go` now gives two routes.
  A beam project writes the text with `lym board upsert --body-file <path>` and runs `lym loom resume`.
  Any other run is ended with `lym batten cancel <slug> --remove`, gets its text written, is released with `lym board set-status`, and starts again with `lym batten run <slug>`.
  The start verb and the release step were confirmed against `internal/battencli/startguard.go:473` and `arm.go:868`.
  The doc comment no longer claims every run holds the entry.
  The test's want list names every step.
- F2: `ErrorEnvelopeLine` skips closed fenced code blocks, pairing a fence with a closer of its own character and at least its length.
  A fence left open to the end of the text hides nothing.
  Rows cover a fenced envelope, a bare envelope after a fence, nested and mixed fences, and an unclosed fence.
  `Body_QuotedErrorEnvelopeInAFence` in the Record-Write test pins that such a body reaches Done.
- F3: two `get` rows, `1abc` and `my task`, assert the `invalid json:` error.
- F4: the board skill teaches the bare slug and shows `lym board get my-task`.
  The `RemoveCommand` comment in `internal/boardengine/board.go` no longer says the verb takes no bare slug.
- F5: the upsert flag line and merge's `--body-file` help name the envelope refusal, and the three texts say a fenced envelope passes.
  The `applyBodyFile` doc comment says the same.

Gate repair: the doc comment of `fenceRun` in `internal/output/output.go` was one line of 120 or more characters.
It is now two sentences on two lines, and nothing else changed.

Second gate repair: `gofmt` flagged `internal/loomshed/recordwrite_test.go`, whose added `Body_` row was misaligned.
`lym loom fmt` realigned it, and nothing else changed.

Third gate repair: the upsert long help passed the 2000-byte cap with the F5 line, so that line is now a short clause on the `--body-file` flag line.
It names the envelope refusal and no longer says a fenced envelope passes; merge's help and the `applyBodyFile` comment still do.

Fixed differently: none beyond the above; the F1 route gained the release step the review's fix did not name.

## Self-check

- fix-code-correctness, LOW: the fence toggle ignored fence character and length, and an unclosed fence hid later lines. Fixed (F2 commit).
- fix-consistency, LOW: `doc.go` and `recordwrite.go:136` checked, no change needed. Not a defect.
- fix-consistency, LOW: the `--body-file` docs did not say a fenced envelope passes. Fixed.
- fix-consistency, LOW: no Record-Write test that a fenced envelope reaches Done. Fixed.
- fix-proposal, MEDIUM: the non-beam route lacked the release step. Fixed.
- fix-proposal, LOW: fence toggle pairing and unclosed fences. Duplicate of the first finding, fixed.
- fix-affected-set, LOW: the `RemoveCommand` comment in `internal/boardengine/board.go` denied the bare slug. Fixed.

## Disputed

None.

## Deferred

None.

## Rulings

None.
