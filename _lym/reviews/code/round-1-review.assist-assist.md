---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/shuttleengine/claudeengine/session.go:347
    summary: "The suffix-only border check can hide a nonempty multiline draft and allow a send into it."
    fix: "In isTopBoxRule, exclude caret-bearing draft lines from truncated-label recognition. Extend TestInputBoxText with the multiline draft shown below and require its text to remain visible."
    origin: lens:code-correctness
  - id: F2
    severity: LOW
    class: scope
    location: contracts/stencils/burler/burler-review-format.md:46
    summary: "The prompt requires quoting colon-space values, while the goal requires quoting values containing any colon."
    fix: "Require quoting any value containing a colon in the review-format stencil; retain the colon-space example to explain the YAML parse failure."
    origin: lens:code-scope
---

### [MEDIUM] A draft can become the input box's top border

**Location:** internal/shuttleengine/claudeengine/session.go:347

**Issue:** This capture contains a nonempty draft, including a literal caret on its second line:

```text
────────────────────────────────
❯ explain this… ─
  ❯
────────────────────────────────
```

`inputBoxInterior` selects the last caret-bearing line, then searches upward for a border.
The new suffix branch accepts the first draft line as that border and discards it from the interior.
`InputBoxText` strips the remaining caret and reports an empty box; `IdleSession` also reports idle.
Before this change, the actual top rule preserved the draft text.
`readBox` passes the false empty reading to `judgeReadiness`, bypassing its draft hold once operator activity is quiet.

### [LOW] The quoting instruction is narrower than the requested rule

**Location:** contracts/stencils/burler/burler-review-format.md:46

**Issue:** The goal requires frontmatter values containing `:` to be quoted.
The changed instruction requires that only for `: `.
Although colon-space is the reported YAML failure, the requested prompt rule also covers a colon without a space.

## Focus checks

- Routed Stuck and reroute entries both correctly display their destination.
  The Stuck-oriented comment explains the motivating case without changing the outcome-independent rendering.
- Generated history uses RFC 3339 timestamps and enum outcomes.
  Ordinary older three-field rows remain readable; separators in producer or outcome already broke the old format.
  `reportengine` parses through `haltnote.Parse` and classifies stops without reading history fields.
- The pinned yaml.v3 scanner contains the matched diagnostic.
  It describes a forbidden mapping-value token, rather than uniquely identifying prose with an unquoted colon.
  The added suffix is conditional advice and retains the original error; the quoted-fragment branch still runs first.
- The stencil's escaping instructions and examples produce valid YAML and preserve the quoted-fragment prohibition.
  F2 records the remaining wording gap against the goal.
- The review-base gate already returns the parse error and re-prompts within the same attempt.
  Recognizing the truncated label addresses its existing no-input-box delivery failure.
- F1 records the draft boundary the new input-box test rows do not cover.

## Validation

Reviewed the supplied committed diff against `c5a0ea745f362a36974fda25ba0c3f917f152684..HEAD`.
The four lens reports are saved beside this report.
The attempted tier0 gate for the changed packages could not acquire a slot:
the sandbox makes the hub's gate-holder and run-process directories read-only.
No tests ran in this review; the supplied reach and redundancy reports remain separate evidence.
