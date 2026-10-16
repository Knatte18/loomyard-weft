# Support log — webster-fork-stencil-fixes

## Interview

Autonomous mode: no operator answered; every pick below is an auto-pick recorded in the question ledger.

Turn 1 — read the board entry, the related entries (`discussion-lands-invariant`, `review-inline-code`, `weaver`, `burler-fixer-fork`, `plan-writer-finding-classes`), the edit directive, the five pattern directives, the webster stencils, the plan spec's Rename passages, the Webster-Review rubric, `languageconfig` and the webster digest and pack config.

Turn 2 — rename tool: the latest commit put `verify_tiers` into `language.yaml`, so "the hub's language setup" maps onto a sibling key, `rename_tools`. The tool is required, so a plan gate check catches an empty list. `gopls` is on this machine and serves as the Go example.

Turn 3 — departures: the Master runs no git, so Go has to carry commit trailers into the digest; `Digest.Deviations` is the model. The verify-fix fork's `fix:` commits cannot be read by the Master either, so its reply stays the source, named as an exception.

Turn 4 — declared PATTERN changes: the judge never opens artifacts and in-session forks read only the master's context, so a sentence pointing at the plan reaches neither. Rendering the declared list into the directive reaches every role; `internal/pattern` is a leaf, so callers pass the entries in.

Turn 5 — inline pack: with the inline layout gone, `kickstart_pack_file` has nothing to choose; `configengine` already supports `RetiredKeys`.

Turn 6 — lens review; every fork returned a report.

- leanness: Auto-mode assumptions restate decisions — fixed, cut to ID pointers.
- leanness: "operator decision" rationales are provenance only — rejected; they tell the plan writer a pick is not its to reverse, and each keeps its substantive half.
- leanness: literal gopls syntax in D1.2 — fixed, cut to `gopls rename -w`.
- leanness: Scope In restates decisions — rejected; it is the in/out boundary the Out list pairs with.
- leanness: generic Constraints bullets — fixed, dropped documentation-lifecycle, test-economy and no-denied-recovery.
- leanness: gopls-specific open risk — fixed, generalized.
- leanness: test-budget note restates CLAUDE.md — fixed, cut.
- leanness, premises, pattern-fit: the new plan checks' home unstated — fixed, D1.7 and D5.3 place them in the shared `loomshed` plan-validation function, all modes, outside `planparser`, naming `PATTERN-gate-self-check-parity`; D5.1 names `discussionparser.Validate`.
- goal-scope: hub cannot rename until `rename_tools` is set — fixed, Open risks.
- goal-scope: D1.7 unlicensed — fixed, rationale names the operator's "required" ruling.
- goal-scope: departure rule silent on declared new text — fixed, D2.1.
- goal-scope, premises, scenarios: verify-fix departures reach no digest — fixed, D2.8's reply names them.
- premises, scenarios: `none` → `FAILED` contradicts the closed `FAILED` list; standalone runs exhaust recoveries — fixed, D1.8 replaces the fallback with a `begin-batch`/`recover-batch` refusal and a master rung.
- premises: trailer read behind webster's `Git` seam — fixed, Constraints and Notes.
- premises: `planparser` imports `pattern`, so the signature change reaches it — rejected; call-site enumeration is the plan's.
- board-fit: `discussion-lands-invariant` unnamed — fixed, Scope Out.
- board-fit: overlap with `dotnet-scoped-verify` — rejected; no conflict, and `language.yaml` is already the shared home.
- board-fit: precedent for `weaver` — rejected; nothing for this task to decide.
- board-fit: textual collision with `reed-add-parentless-placement` — rejected; a landing merge, not a design question.
- pattern-fit: trailer and section shapes restated across stencils — fixed, D2.1 and D5.1 name the owning spec, D2.2 points at it.
- pattern-fit: directive banners and one-marker doc and test — fixed, D5.4.
- pattern-fit: standalone and told path for the record — fixed, D5.4.
- pattern-fit: `PATTERN-fabric-git` read-only exemption — fixed, Constraints.
- pattern-fit, scenarios: shell guard can refuse a configured tool; prose-only bound — fixed, D1.4 and Open risks.
- scenarios: merge-in trailers attributed to the batch — fixed, D2.3 first-parent non-merge range and a criterion.
- scenarios: resumed Master loses departures — fixed, D2.3 progress trail and a criterion.
- scenarios: D2.6 weakens a guard without a bound — fixed, bound stated.
- scenarios: malformed section at render time — fixed, D5.4 refusal with way forward and a criterion.
- scenarios: PATTERN change after Discussion has no path — fixed, D5.2 `decision add` form.
- scenarios: invalid trailer content — fixed, carried verbatim, Webster-Review judges.

## Rejected alternatives

- Rename tool hard-coded per language in the stencil: operator decision against it.
- Hand renames by Edit when `rename_tools` is empty: the operator made the tool required.
- A new report field for departures: operator decision for the commit message.
- The Master reading `fix:` commits itself: it runs no git.
- A Go-side list of verify-fix commits for the summary: more plumbing than naming the one exception.
- A required `## PATTERN changes` section with a literal `None`: breaks every in-flight decision record.
- Directives pointing at the plan's section instead of rendering it: the judge and the in-session forks never read it.
- Admitting `discussionparser` into `internal/pattern`: changes `PATTERN-leaf-packages` for no gain over passing entries in.
- Keeping `kickstart_pack_file` with a new meaning: nothing left to choose.
- The fork reporting `FAILED` on a symbol pair under an empty `rename_tools`: contradicts the closed `FAILED` list and burns both recoveries in a standalone run.
- A new PATTERN entry for the declaration rule: it governs a task's documents, not code structure, and lives in the specs.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Where does the rename tool come from? Auto-pick: a `rename_tools` key in `language.yaml`, shaped like `verify_tiers`.
- Q2 What happens with no tool configured? Auto-pick: plan gate `rename-tool-missing` and a `begin-batch`/`recover-batch` refusal (revised after lens review; the fork's `none` fallback was dropped).
- Q3 Trailer shape for a departure? Auto-pick: `Pattern-Departure: <entry> — <text>`.
- Q4 Status of a batch gate failure the cards did not cause? Auto-pick: `OK`, left to the plan-level verify gate.
- Q5 How does the summary get the verify-fix commits? Auto-pick: from the fixer's reply, named as the one exception.
- Q6 How does the master self-identification rule key? Auto-pick: on "You are an implementer fork".
- Q7 What does `kickstart_pack_file` become? Auto-pick: retired.
- Q8 Is the decision record's PATTERN section required? Auto-pick: optional.
- Q9 How do directives learn the declared changes? Auto-pick: rendered by `internal/pattern` from entries the caller passes.
- Q10 Does `CLAUDE.md` get the same treatment? Auto-pick: no; no directive makes its rules gating.
- Q11 Does this task change a PATTERN entry? Auto-pick: no.
- Q12 How is a PATTERN change decided after Discussion declared? Auto-pick: a PATTERN-change form of `lyx loom decision add`.
- Q13 What does a directive caller do with a record that fails to parse? Auto-pick: refuse the spawn, naming `lyx loom validate-discussion`.
- Q14 Which commits does the trailer read cover? Auto-pick: the batch's own first-parent non-merge commits.
