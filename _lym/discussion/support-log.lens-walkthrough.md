```yaml
lens: walkthrough
outcome: reported
```

Lens walkthrough: I ran each decision end to end on one concrete case and checked each step's input against what the step before it produces. Three findings are MEDIUM and three are LOW. None is BLOCKING.

**Findings**

1. **MEDIUM, D6 (interaction with the Bouncer's ledger):** skipping the judge on a stalemate round leaves that round with no ledger.
   - Case: rounds 1 and 2 are unchanged, round 2 escalates `stalemate`, and the parent runs `continue`.
   - The judge writes `ledger(round)`, so round 2 has no ledger.
   - Round 3's judge then gets `previous_ledger` "(none)". `droppedKeyRefusal` is skipped, with only a warning (`bouncer.go` ~L1108 and ~L1830).
   - `circlingEvidence` and `recurringKeys` lose round 2's open keys. A finding that is open in rounds 1 to 3 then shows a gap, which weakens Go's circling guard.
   - The escalation stencil also names `budget` for "the soft-cap rule could not read … the previous round's ledger". So a soft cap at round 3 can escalate as `budget` for the wrong reason.
   - The record says nothing about this. The fix is either to have the stalemate path write a ledger itself, or to state why the gap is harmless.

2. **MEDIUM, D3 and D4 (decision-class findings):** an unruled `decision`-class finding from round N cannot stay open in round N+1.
   - D1 and D2 leave it out of the fixer's dispositions and out of the prior-findings block.
   - So it has no `prior` entry that could carry it.
   - If the reviewer re-raises it, the finding has no `carries` key, so it counts as new. D4 then refuses it unless it is BLOCKING and sits on a changed file.
   - End state: a parent-pending question can silently drop out of `findings`. The decision path that reads the accepted review then no longer sees it.
   - The record needs a rule: either carry decision findings through `prior`/`carries`, or exempt them from D4 the way `origin: parent` is exempt.

3. **MEDIUM, D4 (path forms):** the file-level check compares a finding's free-text `location` path with the paths in the changes file, and the record never fixes the form either one uses.
   - Source scope: a git diff gives worktree-relative paths. Those probably match locations like `internal/x/y.go:42`.
   - Overlay scope: Go builds the diff itself from the target files, which may be absolute or `_lym/...` paths. The reviewer's location may use a different form.
   - A legitimate BLOCKING finding on a changed file would then be refused.
   - The record should state one normalization, for example worktree-relative paths with the `:line` suffix stripped.

4. **LOW, D5 (source scope):** the working-tree id must leave out untracked noise the fixer leaves behind, or a no-op fix reads as "changed" and D6 never fires.
   - `fabricengine` names `.scratch` in `gitexclude.go`, so `.scratch` is probably excluded already.
   - Even so, the record does not say that the id covers only files `git add -A` would stage under the warp excludes.
   - It would help to state that bound in D5.

5. **LOW, D4 (amendment first round):** the record makes "the round does not open an amendment" decide full versus incremental, but never names where the review producer reads that fact. The data has no named source.

6. **LOW, D1 (dispute evidence):** `burler-step-fix.md` accepts verify output as dispute evidence, but D1 demands at least one `&lt;path&gt;:&lt;line&gt;` item.
   - A dispute that rests only on verify output then has to cite a made-up location, or it is refused.
   - Either allow a non-location evidence item, or record this narrowing as deliberate.

**Walkthrough list**

- D1, the fixer disputes F2 with `premise false`: the check passes it, and the D2 block gets the rebuttal. Purpose met (path evidence caveat in finding 6).
- D2, round 2 after a round-1 dispute: the block lists F1 to F3 with their dispositions, for both engines. Purpose met.
- D3, the reviewer keeps disputed F2 open with an `answer` and `carries: F2`: the check holds. Purpose met, except for decision-class findings (finding 2).
- D4, a new LOW finding in round 2 is refused, and a new BLOCKING finding on a changed file is accepted. Met for source scope; overlay depends on path form (finding 3).
- D5, a fixer edit left uncommitted changes the id, and a no-op leaves it unchanged. Met, given the exclude bound (finding 4).
- D6, two no-change rounds escalate `stalemate` and `continue` restarts the count. The escalation works, but the ledger gap breaks the later rounds (finding 1).

**Tool calls**

- grep for `.scratch` in `fabricengine` and for `ledgerPath` uses in `bouncer.go`: my context did not have the exclude source or the Bouncer's ledger readers.
- Read of `bouncer.go` around L1098–1125 and L1825–1840: my context did not show how a missing previous ledger degrades.
