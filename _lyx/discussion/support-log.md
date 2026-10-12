# Support log — cleanup-1012

## Interview

Autonomous run: no operator answered; every pick below is the writer's own, recorded in the question ledger.

1. Read the board entry and the rest of the board.
   No other entry touches `tools/tokencount`, `refscanner.go` or the vocabulary scan; `kickstart-pack-measure` and `suite-weight-quiet-rerun` measure with other tools, `intake-bugs-1012` works on plan-gate paths, `lyxtemp` and Merge-In, and `run-window-bugs` on run windows, shuttle and weft push.
   The board holds no `tokencount-full-runs` entry.
2. tokencount: read the bound path.
   The bound is the commit date of a lightweight archive tag naming the previous pair's weft tip, the last records commit before teardown, so a later run's transcripts normally start after it.
   Dropping a session needs that commit date to be later than the next run's first transcript, which needs clock skew across machines.
   No report under `docs/` shows a left-out line, so there is no evidence the skew happens; the off switch is the cheapest guard and is opt-in.
   The shed span clamp (`ShedRecord.Older`) reads the same tag date, so the flag lifts both.
3. Audit readers: read `IsReadOnlyCommand` and its consumers (webster's batch audit through `loomcli` and `webstercli`).
   Globs are bounded in `lyx`, `git` and `sed` words because a glob can expand into a flag-shaped name and those commands have writing flags.
   None of `cat`, `head`, `tail`, `ls`, `wc`, `grep`, `jq`, `cut` has a flag that writes or executes, so their globs and flags are harmless; bounding only globs would guard nothing, and bounding flags would reject everyday reads.
   First draft: document and pin rather than tighten.
   `doc.go` has no paragraph on the classifier today; the git `--` pathspec reason lives only in the function comment.
4. Carve-outs: read both.
   `BareTokenInRootDoc` strips `\bloomyard-weft\b`; the test strips the substring.
   Every root-file occurrence is word-bounded, and the stripped text also feeds only the host-phrase check, which root files are outside, so dropping the test-local strip changes nothing the scan admits.
   Keep the owner-package function.
5. Lens review by forks; every lens returned a report.
   - scenarios, BLOCKING, D2 premise false: a zsh glob qualifier (`cat *(e:'cmd':)`) runs code, and readers skip the paren rule. Fixed: D2 now rejects an unquoted parenthesis in reader segments; Goal, Scope and Acceptance criteria follow.
   - scenarios, MEDIUM, D1 bound names only transcripts, not base, Merriam rows, window and orch charge. Fixed: D1's bound lists every figure the cutoff gates and the doc comment says so.
   - scenarios, leanness and premises, byte-identical criterion impossible because the header prints the generation time. Fixed: the criterion now says the line is absent without the flag.
   - scenarios, LOW, span start falls back to the earliest transcript without `started_at`. Fixed: named in D1's bound.
   - scenarios, LOW, a blocking reader such as `tail -f`. Fixed: one clause in D2's admission rule.
   - goal-scope and premises, MEDIUM, the cited synthetic-tree test for a bare token next to the name does not exist. Fixed: the criterion asks for two rows in `TestFabricVocabularyFailures_Fixtures`.
   - goal-scope, MEDIUM, Goal 2 hides that the brief's bound is not built, and D2 does not say how the flag premise was checked. Fixed: Goal 2 states it, and D2 names the flag sets checked.
   - leanness, MEDIUM, the `doc.go` paragraph would restate every rule in the function comment. Fixed: it carries only the reader rationale, admission rule and git reason, and points at `IsReadOnlyCommand`.
   - goal-scope, LOW, the paragraph documents `lyx` and `sed` rules beyond the entry. Fixed by the same trim.
   - goal-scope, LOW, the flag is added without the measurement the entry asked for. Rejected: the brief asks for the guard and it is opt-in; recorded as an auto-mode assumption.
   - goal-scope, LOW, lifting the `Older` clamp is an addition. Rejected: it reads the same tag date; kept as an auto-mode assumption.
   - board-fit, LOW, no `tokencount-full-runs` board entry exists. Fixed: named in the auto-mode assumption.
   - board-fit, LOW, the duplication check did not name `intake-bugs-1012` and `run-window-bugs`. Fixed: turn 1 names them.
   - pattern-fit, LOW, D3's "the function's doc comment" is ambiguous. Fixed: it names `fabricVocabularyFailures`.
   - pattern-fit, LOW, the line-break constraint should leave neighbouring wrapped text alone. Fixed.
   - pattern-fit, LOW, tie the flag's doc to the same commit. Fixed in Scope.
   - premises, LOW, "can only drop sessions" overstates. Fixed: "chiefly".
   - leanness, LOW, the Open risk about earlier runs' transcripts repeats D1. Fixed: cut.
   - leanness, LOW, the auto-mode bullet on the `Older` clamp restates D1. Rejected: it records a self-made pick the entry did not license, which goal-scope asked to keep visible.
   - leanness, LOW, D1's rationale holds deliberation. Fixed: trimmed to the guard and its trigger.
   - leanness, LOW, the `tools/` constraint restates CLAUDE.md, and the test-file hint belongs in notes. Fixed: cut, hint moved.
   - leanness, LOW, the both-gates criterion is a standing rule. Fixed: cut.

## Rejected alternatives

- **Tolerance window on the tokencount cutoff.**
  Subtracting a fixed slack from the bound would let in a previous run's late-starting sessions on every report, trading a hypothetical drop for a certain overcount, and the slack size has no measured basis.
- **Automatic skew detection in tokencount.**
  No signal in the transcripts separates a skewed tag from a previous run's sessions that continue past its last records commit.
- **Per-reader flag allowlists and glob rejection in `readOnlyCommands`.**
  Turns common reads (`grep --include=*.go`, `cat *.md`) non-read-only, tightening the webster audit with no safety gain, since no listed reader flag writes.
- **Rejecting reader globs only, flags still free.**
  A glob can only produce what a typed flag already could, so it bounds nothing.
- **Documenting reader globs with no code change (the first draft of D2).**
  A zsh glob qualifier in parentheses runs code, so readers need the unquoted-parenthesis rule.
- **Putting reader segments through the shell-word split.**
  The split also rejects `$` and backslashes, turning common quoted regex reads non-read-only.
- **Refusing `-no-cutoff` together with `-calibrate`.**
  A new refusal for an opt-in diagnostic switch; documenting the effect is enough.
- **Keeping the test-local `ReplaceAll` and dropping `BareTokenInRootDoc`.**
  The function is in the owner package, word-bounded and tested; the test-local constant spells the name a second time.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): add the tokencount flag now, or wait for the measurements the board mentions? Picked: add `-no-cutoff` now; the brief asks for a guard and the flag is opt-in.
- Q2 (auto-pick): flag or tolerance window? Picked: flag; see Rejected alternatives.
- Q3 (auto-pick): does the flag lift the shed span clamp too? Picked: yes; it reads the same tag date.
- Q4 (auto-pick): tighten reader globs, or document why they need no bound? Picked first: document and pin; revised after the scenarios lens to reject an unquoted parenthesis and document the rest; see D2.
- Q7 (auto-pick): may `-no-cutoff` combine with `-calibrate`? Picked: yes, with the effect stated in the flag's description.
- Q5 (auto-pick): which carve-out to keep? Picked: `fabricvocab.BareTokenInRootDoc`.
- Q6 (auto-pick): flag name? Picked: `-no-cutoff`, matching the board's word for the bound.
