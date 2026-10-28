---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: internal/creelcli/cli.go:76
    summary: operator help for lym creel and lym creel post still states the old rule that only an unknown or hookless engine gets the typed notice
    fix: In internal/creelcli/cli.go reword the `creel` Long text (lines 76-77) and the `post` Long text (lines 315-317) from "A strand whose engine is unknown or has no hooks, a hand-added one included, gets a typed notice or reminder" to "Any other strand, a hand-added one or a session launched without those hooks included, gets a typed notice or reminder", matching docs/overview.md.
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: handler
    location: internal/creelengine/verbs.go:105
    summary: the post --interrupt refusal for a receiver without the hook mark says it gets its mail by notice only when its engine is unknown or hookless and by lym creel read otherwise, which is false after the change
    fix: In internal/creelengine/verbs.go:105 change the tail of the way-forward text to "or a post without --interrupt, which the receiver gets by its typed notice or reminder" and drop the clause about its engine and lym creel read. Tests in verbs_test.go:395 assert only the substrings "runs no creel hook delivery", "inject --interrupt <name>" and "post without --interrupt", so they still pass.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/creelcli/smoke_tmux_test.go:76
    summary: the doc comment of TestSmokeCreelFakeProviderPane lost its opening line and now has two overlapping sentences about the strand
    fix: In internal/creelcli/smoke_tmux_test.go replace lines 76-77 with the single line "// TestSmokeCreelFakeProviderPane runs the real daemon on the real socket over a hub whose one strand is the fake provider." followed by "// The strand carries the provider tag `script` and no `CreelHooks` mark, so it is a pane-mail strand.", then keep the existing "A post is acknowledged..." line.
  - id: F4
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/shuttleengine/session_test.go:598
    summary: no row pins the submitted-state branches that the doc comment states, namely an unreadable box, the sent text itself, or the read cap after an empty read
    fix: Add rows to the checkLandingStartConfirmation table with afterPaste scripts {"",true},{"",false} and {"",true},{"/compact",true} (the sent text the call types), both wantWait with no key sent; a row that makes the read cap bind before the window closes is optional.
---

### [MEDIUM] Operator help states the old pane-mail rule

**Location:** internal/creelcli/cli.go:76 and :315
**Issue:** After D3 every strand not marked `CreelHooks` gets the notice and reminder. The `lym creel` and `lym creel post` help still limits that to a strand whose engine is unknown or has no hooks, and docs/overview.md was corrected in the same diff.

### [MEDIUM] Interrupt refusal message names the removed distinction

**Location:** internal/creelengine/verbs.go:105
**Issue:** The refusal for a non-hook receiver says the receiver gets a post by notice when its engine is unknown or has no hooks and by `lym creel read` otherwise. A Claude strand without `CreelHooks` now gets the notice, so the message misdirects the operator about how the post arrives.

### [LOW] Smoke test doc comment corrupted

**Location:** internal/creelcli/smoke_tmux_test.go:76
**Issue:** The edit replaced the first doc line instead of the second. The comment no longer begins with the symbol name and carries two overlapping sentences.

### [LOW] Submitted-state branches have no test row

**Location:** internal/shuttleengine/session_test.go:598
**Issue:** The new rows cover empty then foreign text and foreign text before any empty read. An unreadable box and the sent text reappearing after an empty read are behaviors the `confirmPasted` doc comment states, and moving the `!submitted` guard to cover only the foreign-text case would restore the Enter and `ErrSentTextInDraft` paths with every test still green.

## Rejected

- Lens code-correctness, LOW: an empty read followed by the sent text sends no Enter. D2 states that after the empty read any box text sends no key and that the operator's draft is never submitted; the behavior is the decided one.
- Lens code-affected-set F3, LOW: `probe` doc says "pane-mail strand" without redefining it. The `Scheduler` and package docs define the term, so the comment is correct as it stands.
