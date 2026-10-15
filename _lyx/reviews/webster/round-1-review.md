---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: PATTERN.md:74
    summary: The PATTERN-shuttle-provider-seam entry departs from the text card 14 gives and drops the provider-specifics-containment half of the rule
    fix: "In PATTERN.md replace the entry line with the one card 14 specifies, which reads: - `PATTERN-shuttle-provider-seam` — Referencing a provider, its specifics live only under `internal/shuttleengine/<engine>`, and only `internal/shuttleengines` constructs or imports a concrete engine. (test) — [background](pattern/PATTERN-shuttle-provider-seam.md). Use the colon after Referencing a provider exactly as card 14 writes it. The line is 264 characters, under the 320-character cap in internal/pattern/check.go."
---

### [LOW] PATTERN entry for the provider seam departs from card 14 and loses half the rule

**Location:** PATTERN.md:74 (entry `PATTERN-shuttle-provider-seam`)
**Issue:** Card 14 gives the exact replacement entry: "Referencing a provider: its specifics live only under `internal/shuttleengine/<engine>`, and only `internal/shuttleengines` constructs or imports a concrete engine. (test)".
The commit has "Provider code: `internal/shuttleengine/<engine>`, imported only by `internal/shuttleengines`. (test)".
The shorter line states only the import half.
It no longer says that a provider's specifics live only under the engine's directory, which `pattern/PATTERN-shuttle-provider-seam.md` still states in its opening line.
The planned text fits the cap, so nothing forced the cut.
`PATTERN.md` is the overview inlined in agent prompts, so the dropped half is the part those agents never see.

## Verified, no finding

- Card 13: `ModelSwitchSequence` has no remaining Go caller, fake, mock, template comment or `doc.go` reference.
  The one remaining mention is `docs/research/codex-feasibility.md`, a dated study (2026-10-13) that records a past state and is not claiming to stay current.
- Cards 6-9: every production `NewRunner` and `NewDetachedRunner` call site takes its set from `shuttleengines.New`.
  The only production construction of a concrete engine outside `claudeengine` is in `internal/shuttleengines`.
  Within `shuttleengine`, `Run` and `Start` use the run's own engine, and reconstruct resolves it from the record.
- Card 10: `checkRequirementsAtStart` and `checkRequirementsAtAttach` name the engine, each unmet requirement and the meeting engines.
  Both add their way forward and wrap `ErrRequirementsUnmet`.
  An attach of a finished run skips the check as the card specifies.
- Cards 11 and 12: `AuditEngineFrom`, the `build` forks guard, `finalize`'s `ForkAuditor` assertion and `resumeContextCmd`'s `ResumeContextRenderer` assertion each handle the absent-capability path with a named error.
  `resumeContextCmd` resolves the engine before it renders the pointer, where the card renders first.
  That keeps the no-delivery-mark outcome the card requires, and a code comment gives the reason.
- Card 14: the module-wide scan and its table rows match the card, `pattern/PATTERN-shuttle-provider-seam.md` and the `doc.go` and `engine.go` comments are updated, and `PATTERN.md` changes in the same commit (see F1 for its wording).
- Test budget: `TestTestBudget_PackagesStayWithinBudget` passes with the edited rows.
  No new test duplicates one that already covers its behavior.
- Docs: `internal/shuttleengines/doc.go` exists, and `docs/overview.md` carries the new package in its tree and in the shuttle paragraph.
- `lyx gate test` over the touched shuttle, webster, orch, tender, loom and `cmd/lyx` packages passes.
