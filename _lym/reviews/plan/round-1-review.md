---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: handler
    location: _lym/plan/10-gate-detached-runner.md (detachRun.writeLease, detachRun.expireLease); _lym/plan/11-gate-capped-front.md (key directory pointer)
    summary: The lease rewrite and the current-run pointer are Go-written JSON records rewritten by fronts of any binary, but the cards plan a hand-rolled locked write instead of internal/state, against PATTERN-tolerant-records
    fix: >-
      In 10-gate-detached-runner.md's Run directory paragraph, say that writeLease and expireLease
      update lease.json through `state.UpdateJSON(<lease path>, <lease path>.lock, mutate)`,
      writeLease's mutate keeping the later of the stored and the new deadline and expireLease's writing now less a second,
      in place of "under the run's lock file", and add `internal/state#UpdateJSON` under **Uses:**.
      In 11-gate-capped-front.md's Front paragraph, say that the key directory names the current run id
      in a JSON record written through `state.WriteJSON`, and add `internal/state#WriteJSON` under **Uses:**.
      Both cite PATTERN-tolerant-records.
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/11-gate-capped-front.md (Edit list, Texts paragraph); internal/gatecli/cli_test.go:59-66
    summary: Card 11 drops the slot-wait cap from the test verb's help, but the help subtest of TestGateTest_RefusesBeforeResolvingTheHub still asserts the old text, and no card edits it, so card 11 fails its own tier0 gate
    fix: >-
      In 11-gate-capped-front.md, add `internal/gatecli#TestGateTest_RefusesBeforeResolvingTheHub` to the **Edit:** list,
      and in the Tests paragraph say that its subtest "help names the shared keep-cache-warm cap as the slot wait" is removed,
      since the help no longer names a slot-wait cap, together with that file's then unused `internal/shuttleengine` import.
  - id: F3
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/10-gate-detached-runner.md (TestDetachStore, TestStartReading)
    summary: Card 10 plans tier0 tests of unexported helpers whose behaviour the plan's public-surface tests already cover, and its reused-pid row duplicates card 11's TestJoinable row
    fix: >-
      In 10-gate-detached-runner.md, drop `plan:internal/gatecli#TestDetachStore` and `plan:internal/gatecli#TestStartReading` from **Create:**
      and their two Tests bullets.
      TestDetachStore's rows are covered by TestGateRunner_Refusals' claimed row (a second claim fails),
      TestGateRunner's abandoned rows (the exit file's mark), TestGateTest_DetachedRun's supersede and start-failure rows (expireLease)
      and its 1s-cap-joins-6s-cap row (writeLease keeps the later deadline).
      In 11-gate-capped-front.md, TestJoinable's reused-pid row takes TestStartReading's Windows cases:
      it drives joinable with a reader built from `startReading("windows", ...)` over injected creation times,
      so one row pins the Unix-nanosecond reading, the false answer when CreationTime fails, and two creation times telling a reused pid apart.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/14-agent-gate-calls.md (Intent); contracts/specs/loom-plan-spec.md:320 and :595; internal/loomshed/plancontext.go:147
    summary: Once only an agent's capped front answers 75, the spec's every-site-runs-wait sentence and the verify-gate-no-wait rationale in the spec and in the plan check's message become false, and card 14 rewrites only the spec's card and batch gate sentences
    fix: >-
      In 14-agent-gate-calls.md, add `internal/loomshed#verifyLineFindings` under **Edit:** and `//internal/loomshed/doc.go` under **Uses:**.
      Its Intent also rewrites loom-plan-spec.md's gate-keys sentence (line 320): a Go site runs `lym gate test --wait --gate <key>`,
      and an agent's card, batch and `webster-fix` calls carry no `--wait`.
      It rewrites the `verify-gate-no-wait` reason (line 595) and the Detail of the `verifyGateNoWaitCheck` finding in verifyLineFindings to the reason that holds:
      Go re-runs the line outside an agent hook, and a gate call without `--wait` is a slot-held line,
      run inside one slot under the card-verify runner's timeout instead of taking its tiers' own slots.
  - id: F5
    severity: LOW
    class: design
    origin: handler
    location: _lym/plan/11-gate-capped-front.md (Choice (D2) paragraph)
    summary: Card 11 removes the slot-wait cap from every in-process call, the operator's and a zero-cap one, without stating what now bounds that wait
    fix: >-
      In 11-gate-capped-front.md's Choice (D2) paragraph, add a Bound sentence:
      an in-process call now waits for its slots without a cap, ended only when a holder releases its slots,
      each holder's run being bounded by its tier's `timeout_min`, or by the caller's own interrupt, which cancels the wait and exits 1 as today.
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/13-card-verify-capped.md (runner's card-verify branch)
    summary: The runner re-reads the plan and handles a failed read or card match, but not a matched card whose Verify line was removed in the same window, whose answer is then unspecified
    fix: >-
      In 13-card-verify-capped.md's runner paragraph, add: when the matched card has no `**Verify:**` line in the runner's read,
      the runner writes the in-process ok envelope (`card <id> has no **Verify:** line; nothing to run`) to `stdout` and an unmarked exit `{Code: 0}`,
      so the front replays it as the in-process verb answers.
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/11-gate-capped-front.md (TestGateTest_DetachedRun row for helperLaunchEnv=none)
    summary: The none row asserts the runner's log on stderr, but under none no runner starts and nothing writes runner.log
    fix: >-
      In 11-gate-capped-front.md's Tests paragraph, say that helperLaunch under `none` writes a fixed line,
      such as `gate helper: the runner was never started`, to the run's `runner.log` before it returns the ended process's pid,
      and the `none` row asserts that line on the front's stderr.
  - id: F8
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/04-await-resolved-cap.md (Edit list, Intent); internal/shuttlecli/await.go:1-2 and :17-18
    summary: Card 4 makes lym shuttle await read claude.yaml and shuttle.yaml inside a worktree, which falsifies await.go's file header and the awaitVerb comment, and neither is a target
    fix: >-
      In 04-await-resolved-cap.md, replace `internal/shuttlecli#shuttleCLI.awaitCmd` under **Edit:** with `//internal/shuttlecli/await.go`.
      The Intent adds: the file header no longer says the verb reads no file's content, and says it reads the keep-warm config inside a worktree;
      awaitVerb's comment says Command's PersistentPreRunE still skips resolution for it,
      since the verb resolves only its own config dirs when its directory lies in a git worktree and needs no hub.
  - id: F9
    severity: NIT
    class: consistency
    origin: handler
    location: _lym/plan/08-runproc-gate-role.md (Extends); _lym/plan/09-run-records-call-id.md (Extends)
    summary: Cards 8 and 9 list a doc under Extends while their Intent rewrites an existing sentence of it, the runproc role list and the gateslot run-record sentence
    fix: >-
      In 08-runproc-gate-role.md, change the **Extends:** label over `//internal/runproc/doc.go` to **Edit:**.
      In 09-run-records-call-id.md, move `//internal/gateslot/doc.go` from **Extends:** to **Edit:** and drop the empty Extends group.
---

### [BLOCKING] Lease and current-run pointer rewritten outside internal/state

**Location:** `_lym/plan/10-gate-detached-runner.md` (`detachRun.writeLease`, `detachRun.expireLease`); `_lym/plan/11-gate-capped-front.md` (the key directory's pointer to the current run id)

**Issue:** `PATTERN-tolerant-records` binds every Go-written JSON record: never refuse an unknown key, and rewrite through `internal/state`.
`lease.json` is read and rewritten by every front that joins, starts or supersedes a run.
Those fronts can be different `lym` binaries, such as an agent's session before and after the hub's binary is rebuilt.
Card 10 plans that rewrite as its own "write the deadline only when it is later ... under the run's lock file" and never names `internal/state`.
`state.UpdateJSON` (`internal/state/state.go:148`) is the read-modify-write under a lock that the pattern asks for.
Card 11's "points the key directory at the new id" rewrites a record each time a run starts and leaves its form open.
The decision record's Constraint names the pattern only for the run-record fields, so the cards have to carry it for the new records themselves.

### [MEDIUM] Card 11 breaks the help subtest it does not list

**Location:** `_lym/plan/11-gate-capped-front.md`; `internal/gatecli/cli_test.go:59-66`

**Issue:** Card 11's Texts paragraph rewrites the test verb's help, so `Without --wait it waits for the slots for up to %s` goes away.
The subtest "help names the shared keep-cache-warm cap as the slot wait" of `TestGateTest_RefusesBeforeResolvingTheHub` asserts `"for up to " + shuttleengine.DefaultAwaitCap.String()`, and no card edits it.
The card's own tier0 gate therefore fails.
The repo-invariants and affected-set lenses both found this, rated BLOCKING and MEDIUM.
It is kept at MEDIUM: it is a forced edit a one-line target addition fixes, and the plan's design is unaffected.

### [MEDIUM] Card 10 tests unexported helpers its surface tests already cover

**Location:** `_lym/plan/10-gate-detached-runner.md` (`TestDetachStore`, `TestStartReading`)

**Issue:** PATTERN-test-speed's Test economy section asks for a test at the public surface, with an unexported helper tested through the surface that uses it.
`TestDetachStore` tests `writeLease`, `expireLease`, `leaseHolds`, `claim` and `writeExit` directly.
The plan's own surface rows reach each of those:
- `TestGateRunner_Refusals`' claimed row (a second claim fails)
- `TestGateRunner`'s rows with exits `{1, true}` and `{143, true}`
- `TestGateTest_DetachedRun`'s supersede rows and the start-failure rows that leave the lease passed
- its row where a 1s-cap call joins a 6s-cap call's run, which shows the later lease kept

`TestStartReading`'s reused-pid case ("two different creation times give different readings") is the behaviour card 11's `TestJoinable` row "the same pid with a different reading, the Windows reused-pid case through a creation-time reader" pins again.
That gives two tests for one behaviour.

### [LOW] The spec and the plan check still give the old reason for `--wait`

**Location:** `_lym/plan/14-agent-gate-calls.md`; `contracts/specs/loom-plan-spec.md:320`, `:594-596`; `internal/loomshed/plancontext.go:143-147`

**Issue:** After cards 11 and 14, three texts are false:
- The spec's gate-keys sentence, "every site runs `lym gate test --wait --gate <key>`". The agent sites now run without `--wait`.
- The `verify-gate-no-wait` rationale, "a call without it exits 75 on a busy slot". Only an agent's capped front answers 75.
- The plan check's Detail, "a busy gate slot fails the line instead of waiting". An in-process call waits uncapped, and Go runs a Verify line without the strand.

Card 14 already targets the spec, but its Intent names only the card and batch gate sentences.
The affected-set lens found this, and the repo-invariants lens found the `verify-gate-no-wait` part; the two are merged here.

### [LOW] No bound stated for the uncapped in-process slot wait

**Location:** `_lym/plan/11-gate-capped-front.md`, Choice (D2)

**Issue:** The rubric's attack-surface rule asks a card that removes or downgrades a guard to state what it now lets through and what bounds it.
Card 11 deletes `answerSlotBusy` and the slot-wait cap for every in-process call, saying only "the change D2 states".
It gives no bound for how long an operator call, or a call whose resolved cap is zero, can now block.

### [LOW] The runner's card-verify branch leaves a removed Verify line unspecified

**Location:** `_lym/plan/13-card-verify-capped.md`

**Issue:** The runner re-reads the plan and names only two answers: a failed plan read and a failed card match.
A card that still matches but whose `**Verify:**` line was removed between the front's read and the runner's lands in neither.
That is the same edit window the late-launch row exercises.
The in-process verb answers ok with "has no **Verify:** line" (`internal/gatecli/cardverify.go:95-97`).

### [LOW] The `none` launch row asserts a log nothing writes

**Location:** `_lym/plan/11-gate-capped-front.md`, `TestGateTest_DetachedRun`

**Issue:** Under `helperLaunchEnv=none`, `helperLaunch` returns the pid of an already-ended process, and no runner ever writes `runner.log`.
The row's "exits 1 with the runner's log on stderr" can then only see an empty or missing log, so it pins nothing.

### [LOW] Card 4 makes await.go's own docs false

**Location:** `_lym/plan/04-await-resolved-cap.md`; `internal/shuttlecli/await.go:1-2`, `:17-18`

**Issue:** The file header says the verb "only stats the told paths ... and reads no file's content".
The `awaitVerb` comment says the verb "needs no hub, repository, config or engine".
Card 4 makes the verb load claude.yaml and shuttle.yaml and build the engine set inside a worktree, through `shuttleengines.KeepWarm`.
Neither the header nor the constant is a target.
A file target replaces the `awaitCmd` member, and with every proposed fix overlaid together, `lym loom validate-plan --overlay` reports no new blocking finding.

### [NIT] Extends on doc sentences the cards rewrite

**Location:** `_lym/plan/08-runproc-gate-role.md`; `_lym/plan/09-run-records-call-id.md`

**Issue:** The runproc doc lists the roles inside one sentence, "(`RoleDriver`, … `RoleCapture` or `RoleSession`)".
Adding `RoleGate` there rewrites that sentence.
Card 9 has the gateslot doc's run-record sentence "name the call id and the abandoned mark", which also changes existing text.
`Extends` is for adding without changing what is already there.

## Rejected

- Card 11 records `tree.json` in the front before the spawn, where D1 says the runner records HEAD's tree and the dirty paths at its start. No behaviour differs: a tier reads files only when it runs, possibly after a slot wait, so neither place is closer to the content tested. The front's write under the lock also leaves no run without a recorded tree.
- Card 11 does not say when a joining front releases the key directory's lock. The start path releases it once `process.json` exists, and by analogy a joining front releases it after its lease write, since `process.json` already exists. Not a defect.
- Card 15 renders the darn and drift stencils' gate calls without `--wait`. This is licensed by D7 ("every gate call rendered into a prompt or note an agent runs"), so it is no scope creep.
- `launchRunner`'s refusal under `testing.Testing()` has no reaching test. It is a fail-loud wiring guard for the Constraint that tests never re-exec `os.Executable()`, as the guard-reach lens concluded.
- `cmd/testtiming/resources.go` keeps `DefaultAwaitCap`, and no card touches it, as the decision record's Scope settles.
