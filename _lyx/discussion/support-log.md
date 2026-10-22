# Support log — quarry-csharp

## Interview

Autonomous run: no operator answers; every question below was settled by the writer.

### Turn 1 — scope and what changed in quarry

Read the board entry, related entries (`enclose-checks`, `kickstart-pack-remove`, `lenses-language-sets`, `tsa-quarry-audit`, `dotnet-scoped-verify`) and quarry commit `ca716c2` and `docs/glyph.md`.
Quarry `v0.4.0`: a C# glyph's unit is a namespace, not a path; only the Go unit alphabet spells a path, so a `.cs` file's self glyph is a Go-alphabet path self glyph; resolve tries every alphabet and reports cross-alphabet ambiguity; `Declaration.Lang` is required.
LoomYard parses every plan ref with one plan-wide language, `glyph.Go`, from the `language:` key.

### Turn 2 — how "alphabet per file, by extension" maps onto LoomYard

Question: can each ref pick its alphabet by file extension?
Recommended: no for member glyphs, which name a namespace and carry no file; the plan names its alphabet (`language: csharp`), the planner picks it by the files' extension, and a guard checks the pick by extension (D2, D4).
Path entries always use the Go unit alphabet, per quarry.

### Turn 3 — refs both alphabets parse

Question: under `csharp`, which reading does `Norce.Models#Pump` or `README.md#` get?
Recommended: C#; the path checks then skip a slash-free self glyph, and resolve, which tries both alphabets, checks existence (D3).

### Turn 4 — Go-only checks, handles, answer shapes

Go-toolchain checks run only under `go` (D5); C# handles pass `Owner` to `Name` (D7); cross-alphabet ambiguity and new Kinds are read (D6).

### Turn 5 — lens review

Every lens returned a report.

- walkthrough, interactions — BLOCKING: handle member split by last `.` breaks C# members with parameter lists — fixed: D3 helper parses handle bodies, D7 states it.
- bound-breaker — BLOCKING: C# re-sign changing a parameter type always fails `resign-head-mismatch` — fixed: D7 accepts an id differing only in parameters; done and drift expect the minted id.
- bound-breaker, walkthrough — BLOCKING/MEDIUM: D4 blocks Go plans that edit `.cs` fixtures — fixed: fires only when no source target of the plan's own language is named; bound stated.
- goal-scope, interactions — MEDIUM: D4 misses a C# plan with no path targets — fixed: `glyph-malformed` under `go` names `language: csharp` when the C# alphabet parses the ref.
- interactions, bound-breaker — MEDIUM: listing reader keyed on D3's parse misreads `README.md#` — fixed: D6 reads whichever key the answer carries.
- bound-breaker — MEDIUM: D3's bound misses file-rename pairs and redundancy — fixed: both-self-glyph pairs stay file pairs; redundancy listed in the bound.
- existing-paths — MEDIUM: batcher estimate and target dirs undercount C# cards — fixed: stated in D3's bound and Open risks.
- walkthrough — MEDIUM: C# Rename derivation fails on generic arity; constructors — fixed in D7.
- goal-scope — MEDIUM: departure from "per file, by extension" undeclared — fixed: Scope states the reading.
- board-fit, goal-scope — MEDIUM/LOW: kick-start criterion vs `kickstart-pack-remove` — fixed: criterion holds while the pack exists.
- board-fit — MEDIUM: `enclose-checks` body states a stale pin — rejected: its own run reads the pin from `go.mod`; the stale sentence changes no decision there.
- board-fit — MEDIUM: relation to `lenses-language-sets`'s hub language key — rejected: that key is not designed yet; a later hub key can default `language:` without undoing D2 or D4.
- existing-paths — LOW: `Plan.GlyphLanguage` contract — fixed: D3 reduces it to the glyph-enabled answer.
- interactions, walkthrough — LOW: `Lang` source for Create handles — fixed in D1.
- interactions, bound-breaker — LOW: D4 misses root `.go` under `csharp` — fixed: D4 reads the Go-alphabet reading of path entries.
- goal-scope — LOW: Plan-Review rubric change vague — fixed: D8 names the `Edit` Go-path rule.
- board-fit — LOW: C# `rename_tools` dependency — fixed: D7 names it.
- walkthrough — LOW: C# fixtures in Go-hub listings, `.cs` scan cost — fixed: Open risks.
- leanness — Kind value list perishable — fixed; D4 rationale repeat — fixed; `lyx quarry` criterion tests quarry — fixed (cut); `enclose-checks` note repeats Scope — fixed (cut); Go-unchanged constraint repeats Goal — fixed (cut); open risk on the round trip — fixed (says what LoomYard does).
- leanness — Auto-mode assumptions repeat Decisions — rejected: the spec requires the section, and it is where the auto-picks are named.
- leanness — PATTERN citations in rationales — fixed for D1, D3, D4.
- board-fit — LOW: `tsa-quarry-audit` states the old pin — rejected: an audit note; no decision there depends on it.
- board-fit — LOW: `dotnet-verify-protocol` not named out — fixed in Scope.
- goal-scope — LOW: D4 not asked for — rejected: it is the extension check the entry names.

## Rejected alternatives

- Per-ref alphabet by trying every alphabet like quarry, with no plan key: a ref both alphabets parse (every C# type glyph, every root file) has no single reading, and LoomYard's path checks need one.
- Deciding an overlapping self glyph by disk existence: makes parsing read the disk and ties a ref's meaning to the worktree state at check time.
- Remembering which refs came from path canonicalization: carries a language beside every ref string through rewrite and binding, for little gain over D3's bound.
- A hub-wide default language in `language.yaml`: a new hub key the board does not ask for; the planner sees the files and D4 catches a wrong pick.
- Deriving an absent `language:` from the plan's file extensions: changes the meaning of every existing plan without the key.
- A mixed Go and C# plan with both alphabets on: no hub needs it; `language: none` covers the case.

## Review rounds

### Round 1

Verdict BLOCKING; every finding fixed, see `_lyx/reviews/discussion/round-1-fixer-report.md`.

- F1–F3: `isInterfaceMethod` goes Go-only, the C# re-sign's minted id is rewritten plan-wide at record-batch, and C# Rename derivation parses the new handle and replaces the declared name token (D7).
- F4–F8: D4 gains a resolve half; namespace pairs are refused at the plan gate; D3's bound and `ScopeGuard` are completed; the Go path reading is one planparser export; the re-sign guard states its bounds.
- F9–F15: explicit interface owners, the listing rule cut, D1's re-sign `Lang`, one partitioning rule in D6, D5's verify-package bound, the PATTERN background file in Scope, and the Scope paragraph and an Open risk cut.

## Question ledger

- Q1 How does a plan pick its alphabet? — auto-pick: `language: csharp`, chosen by file extension, checked by D4. Resolved.
- Q2 Reading of a ref both alphabets parse under `csharp`? — auto-pick: C#. Resolved.
- Q3 Path canonicalization alphabet under `csharp`? — Go unit alphabet, per quarry's rule (not a pick). Resolved.
- Q4 Is `plan-language-mismatch` blocking? — auto-pick: blocking, with the way forward. Resolved.
- Q5 Go-toolchain checks under `csharp`? — auto-pick: skipped, no C# counterpart in this task. Resolved.
- Q6 Mixed Go and C# plans? — auto-pick: out; `language: none`. Resolved.
- Q7 Does the raise wire Enclose? — no; `enclose-checks` owns it. Resolved.
- Q8 How is a C# signature change planned? — auto-pick: a re-sign whose minted id may differ only in parameters. Resolved.
- Q9 Does D4 refuse a Go plan editing `.cs` fixtures? — auto-pick: no, only a plan with no source target of its own language. Resolved.
