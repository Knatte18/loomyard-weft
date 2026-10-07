---
round: 5
focus:
  - "Go comment sentences the diff rewrote in part (a clause replaced with new words) in `internal/shuttleengine/attach.go`, `internal/shuttleengine/attach_test.go`, `internal/treadleengine/engine_test.go`, `internal/shuttlecli/smoke_guardrail_test.go` and `internal/shuttleengine/claudeengine/settings.go`: does each such sentence now break only at a colon, semicolon, dash, or a comma before a coordinating conjunction that starts a clause with its own subject and verb, or between list items, never mid-phrase, as `PATTERN-comment-line-breaks` requires?"
  - "Every other Go comment sentence in the diff that spans more than one line and whose text the diff changed beyond swapping or deleting single words: is any line break placed inside a phrase, and is the sentence untouched-legacy wrap exempt only where no word of the sentence changed?"
---

The first entry names the sites that were flagged and re-flowed; the second asks for a sweep of sentences with the same defect shape elsewhere in the diff.
