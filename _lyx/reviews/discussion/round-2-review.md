---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D1 (last sentence of the Decision)
    summary: D1 says the Plan host passes its own reviewed files for the new relaunch value, but the Plan host sets WriterFork and never renders twill-relaunch.md, so the clause directs a needless change Scope does not list
    fix: In D1, replace "and the Plan host passes its own reviewed files for the new value." with "the new marker is declared only in `twill-relaunch.md`, so the Plan host, which sets `WriterFork` and renders no relaunch part, supplies no value for it." Leave Scope In as it is.
  - id: F2
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D3 (Decision and Bound)
    summary: D3's path rule does not say which separators delimit a `..` segment, so a backslash-separated escape such as a\..\..\x passes on a reading that splits on / only, and the Bound claims containment for the whole entry while the rule covers only its backticked path
    fix: In D3's Decision, change "or has a `..` segment" to "or has a `..` segment between `/` or `\` separators". In D3's Bound, change "the path rule keeps an entry from pointing Plan outside the worktree" to "the path rule keeps an entry's backticked path inside the worktree; the reason text is prose and unchecked".
---

### [MEDIUM] D1 gives the Plan host a relaunch value it never renders

**Location:** `_lyx/discussion/decision-record.md` D1, last sentence of the Decision.

**Issue:** `internal/loomengine/twill.go` `planLensDirective` sets `WriterFork`, and `twilldirective.Directive` renders `twill-relaunch.md` only in the `else` branch where `WriterFork` is nil (`internal/twilldirective/twilldirective.go:108-125`).
`fillPart` reports an unfilled marker only for the stencil it fills (`internal/stencil/stencil.go:53-62`), so a marker declared in `twill-relaunch.md` alone never needs a Plan-host value.
D1's clause "the Plan host passes its own reviewed files for the new value" therefore rests on a false premise.
It also directs a change to `planLensDirective` that Scope In does not list, since Scope names only "the Discussion host's twill values".
A plan writer following D1 adds a needless Plan-host edit, or makes the value required at the top-level directive fill, where every host must then supply it.

On the focus question itself: D1 states the mechanism explicitly (a new host-supplied `twilldirective.Values` field rendered by `twill-relaunch.md` in place of "the reviewed files", `ReviewedFiles` unchanged), and Scope In names the files that mechanism changes: `twill.go`'s Discussion values, `internal/twilldirective` and `twill-relaunch.md`.
The only defect is the Plan-host clause above.

### [LOW] D3's `..` rule leaves the separator open and its Bound overclaims

**Location:** `_lyx/discussion/decision-record.md` D3, Decision and Bound.

**Issue:** D3 rejects a path that starts with `/`, a backslash, `~` or a drive letter and colon "whatever the host OS", which covers the absolute forms on every OS.
"Has a `..` segment" does not say what delimits a segment.
On Linux a backslash is a filename character, so a check that splits on `/` alone passes `docs\..\..\x`, which a Windows reader resolves outside the worktree.
The host-independent intent needs the separator set stated, or the Bound's containment claim holds only on one OS.
The Bound also says the rule keeps "an entry" from pointing outside the worktree, but the rule constrains only the opening backticked path;
the reason text after ` — ` is unchecked prose.

On the second half of the focus question: D3 does say what happens when the writer skips `relevant.md`, and the claim holds.
The Discussion-Write lead is the chair seat (`internal/loomengine/discussiontable.go:123-133`, no `Name`), so `seatSpec` sets `QuietHold` false (`internal/seatengine/prompt.go:172`), and `notifyHeld` notifies the parent unless the run is interactive (`internal/shuttleengine/hold.go:138`).
A turn end without every output file is held up to the caller's timeout (`internal/shuttleengine/doc.go:62-75`).

## Rejected

- D2, D4 and D6 subject clarity against `contracts/specs/loom-discussion-spec.md`: each clause has an unambiguous subject.
  D2's spec clauses are about the file and the Discussion writer by name; D4's replacement wording names the decision record, `relevant.md` and Plan explicitly, which also removes the ambiguous "it never reads" at spec line 10.
- Overlap between `## Notes for the plan writer` and `relevant.md`: D6 rewords the optional section (spec line 20, today "a non-exhaustive head start for the plan writer") to design hints and keeps the head-start wording only in the `relevant.md` section, so the two are not both described as the plan writer's head start.
- The spec's status banner and title still say "the two files a design step writes": not raised, since updating the spec's own cross-references is enumeration the rubric leaves to `Plan-Write`.
- `internal/loomengine/config.go:147` also says "sole input from `_lyx/discussion/`" and is absent from Scope: not raised, under the same rubric exclusion.
