```yaml
lens: leanness
outcome: reported
```

Task: leanness review of decision-record.md and support-log.md, looking for content to move or cut.

**MEDIUM — `## Open risks`: the only entry is a settled fact, not a risk.**
Evidence: the line says D3 "rests on dobby giving a covered run no full pass, which `internal/dobbyengine/passes.go` states and its tests pin, and on the batten first pass refusing a covered run with exit code 78". That is evidence for D3, not an open question for the parent. D3's first bullet already cites `passes.go` and `runnotice.go`.
Fix: move the exit-code-78 evidence into D3's rationale and leave Open risks empty (or "none").

**LOW — D4, the closing paragraph of the bound repeats the Notes for the plan writer.**
Evidence: D4 says "The release logic is adopt's own resume moved out of adopt, so `lym beam go` and `lym adopt` share one function for the runs and one for the report." The first Notes bullet says the resume code moves out of adopt and asks the plan writer to place the shared functions. Where the shared function lives is a plan hint, not a bound.
Fix: cut the D4 sentence. The "share one function" requirement should stay somewhere, so fold it into the Notes bullet.

**LOW — D3, the "Adopt resumes no run" bullet restates D4.**
Evidence: this bullet names `battencli.StartFirstPass` and the Finalize-child `lym loom resume`. D4 names both again as what `lym beam go` does.
Fix: shorten it to "Adopt resumes no run; the resumes move to D4."

**LOW — D1 rationale, the second sentence is interview narrative.**
Evidence: "The operator's hand sequence on 2026-10-31 (`lym fabric pull`, `lym s`, `lym adopt`) needs neither of the first two: see D2 and the beam steps adopt already runs." This is incident history, and its claim about `fabric pull` is D2's.
Fix: move it to the support log's Turn 1.

**LOW — Scope Out, the last bullet repeats D6.**
Evidence: "Holding a released task that has no run yet: see D6." Auto-mode assumptions repeat it a third time.
Fix: one of the three can go. Scope Out is the likeliest cut.

**LOW — D2, the last sentence adds nothing a plan writer acts on.**
Evidence: "A hand `lym fabric pull` before `lym adopt` stays harmless and is no longer needed."
Fix: cut it.

**LOW — Constraints, two bullets restate rules without adding anything specific to this task.**
Evidence: the `PATTERN-cli-cobra` bullet ("a subcommand of the existing `lym beam` subtree") and the `PATTERN-spawn-observability` bullet are restatements; the plan writer gets PATTERN in full anyway.
Fix: these could go. Keep the `owner-machine`, `dobby-orch-pair`, `tolerant-records` and `refusal-way-forward` bullets, which say how this design meets each rule.

**MEDIUM — Outside my lens, worth passing to the interactions or walkthrough lens: D4 never checks that the beam's dobby is running.**
Evidence: the Constraints say "`lym beam go` starts none, since adopt's beam steps already did". D4 then starts each run through `battencli.StartFirstPass`, which is "driven on by its parent's dobby". If that dobby stopped between the adopt and the go-ahead (tmux killed, reboot), the first pass runs once and nothing drives the run on. D4 states no bound for this case and doesn't name `lym beam start` as the way forward.

The support log needs no relocation: its Turn 1 list of what was read overlaps `relevant.md`, but the support log is the right home for it.

Tool calls: none. My context already held both files, the rubric and the exploration.
