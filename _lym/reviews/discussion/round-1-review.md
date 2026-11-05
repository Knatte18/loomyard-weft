---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:83
    summary: >-
      D3 changes the meaning of a hub's drift.scope value partial (and of the removed skip list) with no configreg.Reinterpretation, breaking PATTERN-config-reinterpretation; D15's reinterpretation flags only compare without diff and the skip_file placeholder.
    fix: >-
      In D15's reinterpretation list, add a bullet: the reinterpretation flags a hub base whose drift.scope is partial (with its skip list), saying that partial is gone, that the base now runs as full, and that the way forward is to set scope to full or off with lym config drift and drop skip.
      In D3, state that the drift config loader admits a stored partial and reads it as full, so a base not yet edited still loads.
      In D15's acceptance criterion, add that the reinterpretation flags a base holding scope partial.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:84
    summary: >-
      D3 says what a run whose scope record already says off or full does when this lands, but not one whose record says partial with a skip list; its baseline lacks the skipped cases, so each diffs as an unexpected added case and the run stops at Drift-Verify with no stated way forward.
    fix: >-
      In D3, after the sentence on a recorded off or full, add one rule for a recorded partial.
      For example: a recorded partial is read as full; the diff leaves out the cases its skip list names, because the baseline holds none of them; and the verdict states the old partial scope and the cases left out.
      Extend the in-flight Open risks line to name this case and its outcome, and add a D3 acceptance criterion for a recorded partial.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:173
    summary: >-
      D8 puts the drift entry on Merriam's run-level gate and justifies having no fixer because the writer fixes its own regressions, but Master never edits code and clears a run-level gate failure only through a fixer fork or fix-verify, driven by the verify gate's prompt file.
    fix: >-
      Rewrite D8's tapestry sentences and its no-cold-fixer sentence.
      Keep the no-fixer premise for Darn-Write only, where the writer edits code.
      For tapestry, state the fact that Master edits no code and answers a run-level gate failure only through a fixer fork or fix-verify.
      Then either state that a failed drift entry at Master's gate is cleared through those same paths, so its findings reach the fixer's prompt, or leave the tapestry placement to the plan under the operator's decision 3, with that fact recorded.
      Either way, adjust D16's Webster stencil paragraph to say who acts on the entry's findings.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:160
    summary: >-
      A real regression introduced after the writer, by Parent-Align's merge fixer or by Code-Burler-Fix before Drift-Recheck, stops Drift-Verify or Drift-Recheck with no way forward that fixes code; amend would accept it, defer applies only to a missing baseline, and neither row routes back.
    fix: >-
      In D7, add the way forward an unexpected-cell Stuck names, which fixes code rather than accepting the drift.
      For example: the Stuck reason names the parent's route of a goto to the writer row, or a code fixer the stop chain spawns, stated with its bound.
      Add the matching D7 acceptance criterion.
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:264
    summary: >-
      D15 says run_settings carrying the retired skip or verify_max_runs is refused at seeding, but both keys are CategoryNone, and configengine admits and drops a CategoryNone retired key in the local layer, so no refusal happens.
    fix: >-
      In D15, either state that skip and verify_max_runs are declared retired under a refusing category, so their RetiredWayForward text fires at seeding, or replace the refusal claim with the true behaviour, that the run loads and drops them.
      Correct D15's acceptance criterion to match.
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:190
    summary: >-
      In command mode, the judgement entry can check only the cases the judge chose to list, so a judge that omits a drifted case, or writes no ruling, ends Drift-Verify Done; D2's bound does not state this.
    fix: >-
      Add this to D2's Bound sentence: in command mode, code cannot tell which cases drifted, so the judge's ruling covers only the cases it reads from the log, and a judge that lists none passes.
      Alternatively, add a check that bounds it, such as refusing an empty command-mode judgement after a failing compare, and add the matching D9 acceptance criterion.
  - id: F7
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:171
    summary: >-
      D6 has the drift gate entry and lym drift check run the seal check first in either diff mode, while D8 and D2 say both pass in command mode, and the D6 and D8 acceptance criteria pin opposite results for a broken seal at the gate entry in command mode.
    fix: >-
      In D8, change the pass rule to say that in command mode the entry and lym drift check run the seal check (D6), report any seal failure as a finding, and otherwise pass and report the compare's exit and log path.
      In D2, state whether they run the compare to get that exit, and make D2, D8 and the D8 acceptance criterion agree.
  - id: F8
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:139
    summary: >-
      On re-entry, D6 seals any existing expected.yaml that validates without spawning the agent, but an empty expectation is valid, so a stub or partial file left by an expect session that died is sealed as final.
    fix: >-
      In D6's re-entry sentence, state that Drift-Expect seals an existing valid file without spawning only when the expect session ended normally, recorded as today's drift producers record an open agent session; an interrupted session is probed or respawned first.
      Add the matching D6 acceptance criterion.
  - id: F9
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:145
    summary: >-
      D6 names lym drift amend --restore as the way forward for every failed seal check, but restore clears only an edited file; it cannot clear a seal commit whose Warp-SHA differs from the baseline head, or a run past Drift-Expect with an unsealed expected.yaml, so the refusal names a way forward that does not reach it (PATTERN-refusal-way-forward).
    fix: >-
      In D6, name a way forward for each seal-check failure.
      An edited file names amend --restore.
      A Warp-SHA mismatch, and a file with no seal commit on a run past Drift-Expect, each name their own way forward, such as defer, or the parent's removal of the file so that the empty-expectation path applies.
      Correct the D11 acceptance criterion that --restore makes a failed seal check pass, so it covers only an edited file.
  - id: F10
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:191
    summary: >-
      D9's guard snapshots the tracked tree and stops on a tracked change outside judgement.yaml, but a judge that commits leaves a clean tree, and a new untracked file is no tracked change, so the guard behind D4's claim that Drift-Verify spawns no agent that edits code can be bypassed.
    fix: >-
      In D9, make the snapshot cover HEAD and the untracked paths as well as the tracked tree: a moved HEAD, or a new untracked path outside judgement.yaml, is also a Stuck.
      Extend the D9 acceptance criterion to cover a judge that commits.
  - id: F11
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:142
    summary: >-
      The empty-expectation path in D6 depends on a run past Drift-Expect, but the record never says how code tells that; the obvious test, a Drift-Expect entry in the run's history, is never true for the in-flight runs the path exists for.
    fix: >-
      In D6's first seal-check bullet, state the test.
      For example: the run's current row, read from the run status the drift deps already read, comes after Drift-Expect in the recipe's row order.
      Also state what lym drift check answers before that point, with no seal and no file.
  - id: F12
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:219
    summary: >-
      D11's bound says an amendment reaches no agent of the run, yet D9 hands the judge an amended why. In command mode the judge alone does containment, so the record does not say whether an amendment reaches the judge there, and the caller check D11 copies guards only a mistaken call.
    fix: >-
      Rewrite D11's Bound.
      An amendment reaches the judge as a covering entry, in lines mode and in command mode alike, where the judge reads the sealed expectation and every amendment beside the compare's log.
      It reaches no writer or expect agent.
      Its caller check is bounded by the strand-name check and the shell guard that refuses setting lym's strand variables.
      Extend D2 to name the amendments among what the command-mode judge reads.
  - id: F13
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:166
    summary: >-
      D7's rationale has a gate key run the drift check, but a hub gate key holds only test tiers and the closed lints comments and pattern. D8 leaves out that fact from the support log's Turn 2, and the Open risks line on Webster's gate seam repeats D8's hand-off to the plan and says confirms the run-level entry that D8 already settles.
    fix: >-
      In D8's plan hand-off sentence, add the fact that the card and batch gate keys run only test tiers and the closed lints comments and pattern, so running the drift check from one needs a new mechanism.
      Cut the gate-key clause from D7's rationale, keeping the sentence on serialising concurrent checks.
      Delete the Open risks line on Webster's gate seam.
  - id: F14
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:249
    summary: >-
      D14 removes Drift-Verify's verify gate entry, but the record never says what becomes of the hub gate key drift-verify, which the Drift-Verify rows' gate_key, the gateslot template and goReadGates still name.
    fix: >-
      In D14, state the fate of the drift-verify gate key.
      For example: it is removed from the gateslot template and goReadGates, and the rows' gate_key and gates config with it.
      Or it is kept as the gate slot key D7's capture runs under.
  - id: F15
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:236
    summary: >-
      A run already past Drift-Recheck when this lands reaches landing with a scope.json that holds only the old compare, acceptance and waiver fields; D13 renders only the new check fields, so its pull request loses the old compare result and the parent's rulings.
    fix: >-
      In D13, add that a scope record with no new check result but with the old compare or acceptance fields renders those fields, or renders a line saying the check ran under the old Drift.
      Add a line to the in-flight Open risks entry.
  - id: F16
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:104
    summary: >-
      D4 defines a passing check only as every cell expected and ruled matches, which has no meaning in command mode, so whether Drift-Recheck at the head of a failing compare whose cases were all ruled matches is Done at once is undefined.
    fix: >-
      In D4, add the command-mode meaning of a passing check.
      For example: the compare exited 0, or every case was ruled matches, and state whether the second case lets Drift-Recheck skip at the same head, consistent with D2's rule that each failing compare is judged again.
  - id: F17
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:325
    summary: >-
      The D6 acceptance criterion lists an edited file with a hand-edited record as a seal-check failure, but D6 keeps no seal fields in any record; the clause is left from the rejected record-based seal.
    fix: >-
      In the D6 acceptance criterion, replace the clause about an edited file with a hand-edited record by one about an edited file whose seal commit was left intact, or delete the clause.
  - id: F18
    severity: LOW
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:242
    summary: >-
      The judge's open_ranges list, saying what it checked under each open range, is written and gate-checked but read by nothing; D10 stores only tuples, and D13 renders each open range with the expectation's why, so the operator's decision 2 never reaches the pull request.
    fix: >-
      In D10, store each open_ranges sentence beside the judged tuples, or in the check result D4 records.
      Change D13's open-range bullet to render each open range with its why and the judge's sentence on what it checked.
      Add both to the D13 acceptance criterion.
  - id: F19
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:225
    summary: >-
      A baseline directory lost between Drift-Await and Drift-Expect stops Drift-Expect, which needs the baseline for its identity listing, but D12 admits defer only at a Stuck writer, Drift-Verify or Drift-Recheck row.
    fix: >-
      In D12, add Drift-Expect to the rows where defer is admitted for a missing baseline, and state that Drift-Expect stops naming lym drift defer.
      Extend the D12 acceptance criterion.
  - id: F20
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:46
    summary: >-
      The key#n and #n identity syntax has no escape, so a key that holds a hash sign, such as C#, or a key shaped like x#2, makes an identity ambiguous in expected.yaml and in the cell listing.
    fix: >-
      In D1, state how an identity is written when a key holds a hash sign: for example, the occurrence suffix is always the last hash sign followed by digits, and a key matching that shape is refused at listing time with a way forward.
      Or choose an occurrence separator a key pattern cannot capture.
      Add a D1 acceptance criterion.
  - id: F21
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:133
    summary: >-
      D5 lets an open range such as 12- cover every unkeyed line from 12 on, a gap in containment by code, but states its bound only as D9 and D13 listing each one; it does not say what the range lets through.
    fix: >-
      Add a Bound line to D5.
      An open range moves every unkeyed cell from its start to the end of the case out of containment by code, to the judge.
      It is bounded by the judge ruling each covered cell, the judgement entry requiring the range to be listed, and the verdict listing it.
  - id: F22
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md:150
    summary: >-
      D6's rationale sentence on keeping the seal in the commit rather than in a record, and D12's clause that recapturing at the branch point is not offered, re-argue alternatives the support log's Rejected alternatives already holds.
    fix: >-
      Delete D6's rationale sentence that begins Keeping the seal in the commit.
      In D12's rationale, cut the clause about recapturing at the branch point, ending the sentence after so its loss must have a way forward.
      Neither carries a requirement: D6's Decision text and D12's Decision text already state the mechanism.
---

### [BLOCKING] D3 changes the meaning of `drift.scope: partial` with no reinterpretation

**Location:** `_lym/discussion/decision-record.md:83`, `_lym/discussion/decision-record.md:266`

**Issue:** Today `partial` with a `skip` list is a valid hub value (`internal/driftengine/config.go:190-200`). D3 reads it as `full` and warns once per run, and D15's reinterpretations cover only a `compare` set while `diff` is unset, and the `{skip_file}` placeholder. `PATTERN-config-reinterpretation` requires a binary that changes a hub config value's meaning to rewrite it or flag it with a way forward, behind the board marker. A per-run Warn does not meet that. D15's kept value set for `scope` also leaves out `partial`, so a strict loader would refuse it before D3's Warn could run. Found by the interactions and bound-breaker lenses.

### [MEDIUM] A run whose recorded scope is `partial` when this lands is unbounded

**Location:** `_lym/discussion/decision-record.md:84`, `_lym/discussion/decision-record.md:316`

**Issue:** This is the focus directive's D3/D4 question, settled as a defect. `settleConfiguredScope` keeps a recorded scope (`internal/driftengine/baseline.go:139-145`). A run that recorded `partial` took a baseline capture with no files for its skipped tests (`internal/driftengine/capture.go:39-40`). D3 defines only a recorded `off` or `full`. Under D1, every skipped test diffs as an `added case`, and with no seal the expectation is empty, so the run stops at Drift-Verify. Neither D4's resume rationale nor the in-flight Open risks line names this case. Raised by the existing-paths, interactions, walkthrough and bound-breaker lenses.

### [MEDIUM] D8's no-fixer premise is false for tapestry's Master

**Location:** `_lym/discussion/decision-record.md:173-174`

**Issue:** The Webster row's `gates` list does reach Master's run-level gate (`internal/websterengine/runlevel.go:790`). But Master's stencil says "You never edit code yourself" and "Do NOT localize or fix the failure yourself", and a gate failure goes to a fixer fork or `fix-verify` driven by the verify gate's prompt file (`contracts/stencils/webster/webster-template-master.md:20`, `:192-205`). So "the writer, who holds the context, fixes its own regressions" does not hold where D8 places the entry for tapestry.

### [MEDIUM] A regression introduced after the writer has no way forward that fixes code

**Location:** `_lym/discussion/decision-record.md:100`, `:108`, `:160-161`, `:219`, `:225`

**Issue:** Parent-Align's merge fixer and Code-Burler-Fix both edit code after the writer. An unexpected cell they cause stops Drift-Verify or Drift-Recheck. No exit fixes it: Drift-Verify spawns no code-editing agent, Drift-Recheck loses its route back, `amend` would accept the regression, and `defer` applies only to a missing baseline.

### [MEDIUM] Retired `skip` and `verify_max_runs` are not refused at seeding

**Location:** `_lym/discussion/decision-record.md:264`

**Issue:** Both keys are `CategoryNone` (`internal/driftengine/config.go:121-125`). `internal/configengine/doc.go` states that a retired key's unit is judged by its old category alone, so `CategoryNone` admits it and only a refusing category ends with the `RetiredWayForward` text. As written, a `run_settings` entry carrying `drift.skip` loads and is dropped, with no refusal.

### [MEDIUM] In command mode the judge alone picks which cases it rules on

**Location:** `_lym/discussion/decision-record.md:68-74`, `:188-193`

**Issue:** The compare is opaque, so the `judgement` entry's "every case has a valid ruling" can only mean every case the judge listed. A judge that omits a drifted case, or writes nothing, passes. D2's bound names only the limit of the log's first deviating line.

### [MEDIUM] Seal check in command mode contradicts the command-mode pass

**Location:** `_lym/discussion/decision-record.md:71`, `:140`, `:171`, `:325`, `:327`

**Issue:** D6 runs the seal check first at the gate entry and `lym drift check` in either mode, and a failed seal check is a finding. D8 says both pass in `command` mode. The D6 and D8 acceptance criteria pin opposite results for a broken seal there.

### [MEDIUM] Re-entry seals a file an interrupted expect session left

**Location:** `_lym/discussion/decision-record.md:139`

**Issue:** An empty expectation is valid (D5), so a stub `expected.yaml` written before a pane death validates and is sealed as final. Today's drift producers track an open agent session and probe it on re-entry for this case (`internal/driftengine/verify.go:125-140`).

### [MEDIUM] `--restore` is named for seal failures it cannot clear

**Location:** `_lym/discussion/decision-record.md:145`, `:330`

**Issue:** `--restore` writes the sealed copy back, which clears only an edited file. Two failures stay: a seal commit whose `Warp-SHA` differs from the baseline head, and a run past Drift-Expect with an `expected.yaml` and no seal commit. For both, the named way forward does not reach the fault, against `PATTERN-refusal-way-forward`. The rule makes a refusal's way forward a requirement. Here the record names a way forward, but the wrong one, which is a design gap rather than a missing way forward, so it stays MEDIUM.

### [MEDIUM] The judge's tree snapshot misses a commit or an untracked file

**Location:** `_lym/discussion/decision-record.md:191`, `:100`

**Issue:** A judge session that commits leaves the tracked tree clean against its new HEAD, and a new untracked file is no tracked change. Agent sessions can commit in a worktree (`fe2663d3`). The guard that backs D4's claim that no agent edits code is then bypassed.

### [MEDIUM] How code tells a run is past Drift-Expect is unstated

**Location:** `_lym/discussion/decision-record.md:142`, `:316`

**Issue:** This is the focus directive's D6 question, settled as a defect. The empty-expectation path, and the in-flight Open risks line, turn on "a run past Drift-Expect". The drift deps already read the run status (`internal/loomcli/driftwiring.go:60-66`), which carries `CurrentProducer` and `History` (`internal/shedengine/status.go:136-161`). A run in flight when this lands has no Drift-Expect entry in its history, so a history test never holds for the runs the path is for. The test must be the current row's position in the recipe's row order, and the record names neither the source nor the rule. The existing-paths lens raised the same gap.

### [MEDIUM] D11's bound overstates and the command-mode reach of amendments is undefined

**Location:** `_lym/discussion/decision-record.md:216-219`, `:68`, `:184`

**Issue:** D9 hands the judge an amended entry's `why`, so an amendment does reach an agent of the run. In `command` mode, D2's judge reads "the compare's log and the expectation", and nothing says whether amendments are part of that. If they are not, `amend` is no way forward for a `command`-mode Stuck. The caller check D11 copies from `lym loom decision add` guards only a mistaken call (`internal/loomcli/decision.go:8-11`). Merged from interactions 4 and 5 and bound-breaker 9.

### [LOW] Webster gate seam: hand-off is legitimate, but D7 and Open risks overstate it

**Location:** `_lym/discussion/decision-record.md:166`, `:173`, `:314`

**Issue:** This is the focus directive's D8/Open risks question. The verdict on the hand-off itself is that it is no defect. The operator's decision 3 hands to the plan where tapestry's fix loop lives, after reading Webster's gate seam. Leaving the card and batch gate key choice to the plan is a choice the operator assigned, not a deferred research question. The fact under that choice is settled, though: a hub gate key holds only test tiers and the closed lints `comments` and `pattern` (`internal/gateslot/config.go:424-433`, `internal/gateslot/template.yaml:14-22`). D7's rationale presumes a gate key that runs the check, D8 leaves out the Turn 2 fact, and the Open risks line repeats the hand-off and says the plan "confirms" a run-level entry that D8 already settles. Bound-breaker 10 raised the D7 part.

### [LOW] The `drift-verify` gate key is left orphaned

**Location:** `_lym/discussion/decision-record.md:249`

**Issue:** The Drift-Verify rows' `gate_key: drift-verify` (`contracts/recipes/tapestry-recipe.yaml`, Drift-Verify row), `goReadGates` (`internal/gateslot/tiers.go:113`) and the gateslot template all name this key. D14 removes the entry it served without deciding what becomes of the key.

### [LOW] A run past Drift-Recheck lands with an empty verdict

**Location:** `_lym/discussion/decision-record.md:236-243`

**Issue:** Landing calls `driftengine.Verdict`. For a run already past Drift-Recheck when this lands, `scope.json` holds only the old compare, acceptance and waiver fields, which D13 no longer renders.

### [LOW] A passing check is undefined in command mode

**Location:** `_lym/discussion/decision-record.md:104-105`

**Issue:** D4's skip rule turns on a check that passed, which D4 defines only through cells. Command mode has neither cells nor stored rulings.

### [LOW] D6 acceptance names a hand-edited record

**Location:** `_lym/discussion/decision-record.md:325`

**Issue:** D6 keeps no seal fields in a record (`:138`), so "an edited file with a hand-edited record" tests a mechanism the support log rejected (`_lym/discussion/support-log.md:73`).

### [LOW] The judge's open-range statements reach no reader

**Location:** `_lym/discussion/decision-record.md:187`, `:200`, `:242`

**Issue:** The judge writes and the entry checks one sentence per open range on what it checked. D10 stores only tuples, the next judge overwrites `judgement.yaml`, and D13 renders the expectation's `why` instead. The operator's decision 2 asks that every open range be listed in the judge's verdict.

### [LOW] A baseline lost before Drift-Expect has no admitted way forward

**Location:** `_lym/discussion/decision-record.md:225`

**Issue:** Drift-Expect needs the baseline for its identity listing (D5), but D12 admits `defer` only at the writer, Drift-Verify and Drift-Recheck rows.

### [LOW] The identity syntax is ambiguous for keys holding a hash sign

**Location:** `_lym/discussion/decision-record.md:46-47`, `:122`

**Issue:** With the default patterns, a key such as `C#` is possible, and its second occurrence reads `C##2`. A key `x#2` reads the same as the second occurrence of `x`. Neither D1 nor D5's parser has an escape rule.

### [LOW] D5's open ranges state no bound in the Decision

**Location:** `_lym/discussion/decision-record.md:123`, `:133`

**Issue:** This is the focus directive's attack-surface question. D2's `command` mode states its bound in D2's own Bound line, D3's `run_settings` `scope: off` in D3's Bound line, and D12's `defer` after the writer in D12's Bound line. All three hold. `judge_interactive` is an existing setting renamed, and it adds no skip, so it is no finding. D5's open range does move unkeyed cells out of containment by code, yet D5 says only that D9 and D13 list each one. It does not say what the range lets through.

### [NIT] Rationales re-argue rejected alternatives

**Location:** `_lym/discussion/decision-record.md:150`, `:231`

**Issue:** The relocation focus applied. D6's sentence on keeping the seal out of a record and D12's clause on recapturing at the branch point argue alternatives that the support log's Rejected alternatives already records (`_lym/discussion/support-log.md:73`, `:76`). The completeness-before-leanness test finds no requirement in either: both Decisions state the mechanism. D1's `cs` example and sources, D8's rationale citing `runlevel.go`, D14's rationale and the Notes for the plan writer carry evidence or plan hints, and stay.

## Rejected

- Board rules (focus): no Decision edits this task's own board entry. The baseline re-capture is raised for the parent as a follow-up note, and `sampler-drift-trial` is revisited per the board entry. Neither is filed or promoted as a task. No finding.
- D9 against D4 and D8 (focus): D4's "spawns no agent that edits code" and D9's judge, which writes only `judgement.yaml`, agree, and D13's counts match D4's recorded check result. D8's "writes no tracked record" holds, because D7's scratch directories sit under `.lym/` and `judged.json` is written only by Drift-Verify. The one defect here is the snapshot's reach (F10).
- `judge_interactive` as an unbounded edge: it renames today's `verify_interactive` and skips no check, so the attack-surface rule does not apply.
- Existing-paths, the claim that a Master spawned by the old binary runs without the `drift` entry: the lens itself judged it survivable, because Drift-Verify repeats the check. No finding.
