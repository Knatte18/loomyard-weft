---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md (Scope In last bullet, D7, Open risks second bullet, the forkaudit acceptance criterion)
    summary: D7 assumes tools/forkaudit arrives at Merge-In before Plan-Write and is held by a board depends_on, but Merge-In runs after Webster, the entry's depends_on is empty and a run-held entry refuses a depends_on change, so D7 can never be planned in this run and the hold exists only in prose
    fix: Take D7 out of this task. In decision-record.md, delete D7 and the Scope In bullet that teaches tools/forkaudit the per-card layout, and add a Scope Out bullet saying the forkaudit card arm (marker row, card trace value, span source from card-pack results) is a follow-up board task whose depends_on names both glyph-audit-tool and kickstart-per-card, filed by the orch, since a task that depends on both can only start once both have landed. Carry D7's three bullets and its rationale into that follow-up task's body as written in this record, not into this record. In Open risks, delete the sentences about the depends_on hold and the Plan-Write stop, and keep the sentence that per-card forks classify as file, with a trace disagreement, until the follow-up lands. Delete the acceptance criterion that starts tools/forkaudit classes a per-card fork as card. In support-log.md, add a Rejected alternatives bullet stating that D7 in this task was dropped because Merge-In runs after Webster in contracts/recipes/loom-recipe.yaml, so glyph-audit-tool cannot reach this run's Plan-Write, and the run lock refuses a depends_on change on a running entry.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md (D3, Bound paragraph)
    summary: D3's Bound says card-pack writes nothing but one Info line to the worktree's log, but D3 itself also logs a Warn on quarry failure, and internal/logger's durable sink creates a new per-process trace file with a header under .lyx/logs and the exit sweep may delete older trace files there
    fix: Replace D3's Bound sentence with - Bound - it writes only through internal/logger's durable sink, as every lyx verb that logs does - the process's own trace file under the anchor's .lyx/logs with its header, the webster card pack Info line and, on a quarry failure, the Warn, followed by the exit retention sweep over that directory; it mutates no webster, plan or tracked state and holds no lock, so it adds no route around any gate. Keep the rest of D3 unchanged.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md (D4, sandbox paragraph sentence)
    summary: D4 places the new sandbox marker after lyx quarry resolve and fills it with a bare , lyx loom card-pack, which renders an ungrammatical list without the list's backticks, and in the same sentence defers the exact text to the plan
    fix: In D4, replace the sandbox sentence with - The sandbox paragraph of webster-body-implementer.md gains one new optional marker directly after `lyx loom lint-pattern`, before the word and, filled with a comma, a space and `lyx loom card-pack` in backticks only under the per-card layout, so that list reads lint-comments, lint-pattern, card-pack and quarry resolve, and empty in the file, inline, pack-off and recovery prompts, so their bytes stay unchanged. Drop the parenthetical that defers the exact text and punctuation to the plan.
---

### [BLOCKING] D7 cannot reach this run's Plan-Write, and its hold is prose only

**Location:** `_lyx/discussion/decision-record.md`, Scope In's last bullet, D7, Open risks' second bullet and the forkaudit acceptance criterion.

**Issue:** Scope In takes `tools/forkaudit` "from task `glyph-audit-tool` once it has landed on the parent and been taken in at Merge-In", and Open risks says D7 depends on that tool landing "before this run's Plan-Write".
In `contracts/recipes/loom-recipe.yaml`, Merge-In sits after Webster and Describe (`Describe` has `on_done: Merge-In`), so a parent commit landed after this pair branched reaches the pair only after the code is written.
`tools/forkaudit/` exists neither on this branch nor on `main` (`git ls-tree -d main tools/` lists no `forkaudit`), and `glyph-audit-tool` is still at `running · Plan-Burler-Review`.
At this run's Plan-Write the tool is therefore certainly absent, and the record's own fallback, stop and report, always fires.

The hold the record names does not exist either.
`lyx board list` shows `kickstart-per-card` with `depends_on: []`.
The entry is run-held (`running · Discussion-Burler-Review`), and `internal/boardengine`'s run lock refuses every `depends_on` change on a held entry but the prune case (`scopeChanged` in `runlock.go`, and the run-lock section of `doc.go`), so the orch cannot set it now.
Even if it were set, `depends_on` gates only which unclaimed task can start (`readySlugs` in `internal/battencli/land.go`) and no shed or loom row reads it mid-run.
What is left is the Plan-Write stop, which is an instruction in Open risks with no board or shed mechanism behind it.

### [MEDIUM] D3's Bound misstates what `card-pack` writes

**Location:** `_lyx/discussion/decision-record.md`, D3, the Bound paragraph.

**Issue:** The Bound says the verb "writes nothing but its one `webster card pack` Info trace line to the worktree's log".
D3 also says a quarry failure logs a Warn, so the Bound contradicts its own Decision.
`internal/logger`'s durable sink (`armDurableSinkLocked` in `sink.go`) arms on the first Info record, creates `.lyx/logs` if absent, opens a new `trace-<ts>-<traceid>-<pid>.log` file per process and writes a header line before any record.
At exit, `notifyExitAndSweep` in `cmd/lyx/exitsweep.go` runs `logger.Sweep` over that directory, which deletes trace groups past the retention bounds.
The substance of the bound holds: none of this touches webster, plan or tracked state, and every logging lyx verb does the same.
The literal claim is false, and the rubric's attack-surface rule asks the Decision to state what the verb can do and what bounds it.
A plan writer taking the sentence at face value could pin a test on "exactly one line written", which the sink's header line would break.

### [LOW] The sandbox-list marker renders an ungrammatical list, and D4 both pins and defers its text

**Location:** `_lyx/discussion/decision-record.md`, D4, the sentence about the sandbox paragraph.

**Issue:** The always-allowed sentence in `contracts/stencils/webster/webster-body-implementer.md` reads "`lyx loom lint-comments`, `lyx loom lint-pattern` and `lyx quarry resolve` are always allowed from any strand".
A marker after `lyx quarry resolve` filled with `, lyx loom card-pack` renders "… `lyx loom lint-pattern` and `lyx quarry resolve`, lyx loom card-pack are always allowed", which breaks the list's grammar and drops the backticks every other entry carries.
D4 states that fill text and in the same breath defers "exact text and punctuation" to the plan, so the record both pins a wrong form and leaves it open.
The substance the focus asks about holds: `stencil.FillOptional` already exempts named markers in `RenderForkPrompt` and `RenderRecoveryPrompt`, so an optional marker shared by `composeForkTemplate` and `composeRecoveryTemplate` keeps the file, inline, pack-off and recovery bytes unchanged when empty.

## Rejected

- Focus 1, whether the record gives the plan writer a way to make `lyx loom card-pack` reachable while the other prompts stay byte-unchanged: it does, through D4's optional sandbox marker, which the existing `FillOptional` mechanism supports in both the fork and the recovery render. Only its placement and fill text are wrong, filed as F3.
- Focus 2, whether D7's `card` row can match a file-layout prompt: the file-layout prompt's free-text markers are card paths, derived `lyx gate test` card gates, the configured batch gate and a previous-batch digest of status, SHA and deviation paths, none of which can carry `lyx loom card-pack`. Not a defect on its own; it falls away with F1, which moves D7 to a follow-up task.
- Focus 3's "is the instruction enforceable from the board and shed": it is not, filed as F1.
- Focus 4's question about D3's Bound: filed as F2.
- D3's premises were checked and hold: `loomcli` imports `websterengine`, `websterengine` imports neither `planglyph` nor `quarry/quarry` outside tests, `clihelp.AudienceRole` exists, and `planglyph.KickstartPack` takes any card list.
