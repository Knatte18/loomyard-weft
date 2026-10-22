# Card 15 — contracts-csharp

**Prosa:**
- `contracts/specs/loom-plan-spec.md`
- `contracts/stencils/loom/loom-template-plan.md`
- `contracts/stencils/loom/loom-rubric-plan-review.md`

**Intent:** Teach the planner, Plan-Review and the plan format the C# alphabet (D2, D8).

- `contracts/specs/loom-plan-spec.md` describes: the `language:` row accepting `csharp`, set when the code the cards change is C#, read from the files' extension; the alphabet each ref takes under `csharp` and that a ref both alphabets parse is C#; that a C# member glyph is copied from a quarry answer with its parameter list; that a `.cs` file is written as a plain path; a C# `plan:` handle's `<namespace>#<owner.member>` shape with a C# head; the C# re-sign bounds of D7, `resign-head-mismatch`'s C# way forward, `resign-overload-collision` and `resign-not-done`; that a later card names a re-signed C# member by its old glyph; and `plan-language-mismatch`, `rename-namespace-pair`, the `plan-language-unrecognized` row's three legal values and the checks `csharp` skips, in the validation-checks section.
- `contracts/stencils/loom/loom-template-plan.md` says `language:` is `go`, `csharp` or `none`, that the planner sets `csharp` for a C# plan, and points at the plan spec for the C# rules rather than restating them (`PATTERN-producer-pointer-rule`).
- `contracts/stencils/loom/loom-rubric-plan-review.md`'s "Member glyphs on an `Edit` card" rule is worded for both languages: an `Edit` card whose targets are only source file paths (`.go` or `.cs`) of non-test files while its `Intent` names specific symbols.

Every file keeps one sentence per line.
