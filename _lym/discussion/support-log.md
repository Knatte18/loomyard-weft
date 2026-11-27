# Support log: self-healing-runs

## Interview

The session ran autonomously, so no operator answered; each turn below is the writer's own pick, recorded in the question ledger.

### Turn 1 — the board and the study

- Read the entry, the board, and `push-retry-not-halt`'s decision record, the touching entry at Parent-Align.
- Read `docs/research/autonomy/stop-classes-2026-11-27.md` whole, the role text `contracts/orchrole/role-autonomy.md` and the discussion spec.
- No `Decision for self-healing-runs:` note stands on the board.
- `push-retry-not-halt` settles the network, GitHub API, push lock, failed-push, catch-up and landing network classes, so this task names them in its table and changes none of them.

### Turn 2 — exploration

Three explore subagents answered:

- Autonomy is one boolean per-task key, `beam.autonomy`, read through `dobbyengine`'s lookup and `hubgeom.Autonomy`; the orch classifies every halt, and lym classifies none.
- The orch reaches Slack by its turn reply or a creel post to `<shortname>:SLACK`; the dobby's wait reminders already post as a program.
- The driver's rule table is `decideDrive`; the tackler takes unnamed stops; the arbiter refuses cause `budget`; bounce budget exhausted is handed back.
- The quiet notice skips `OperatorWaitRows`, live gates, verify and parent-lock waits, and does not read a held turn awaiting a creel reply; `creelengine.Store.AwaitsReply` exists and is wired only into loom, burler and webster.
- `lym loom resume` refuses `driver_not_parked` with no wait.
- Sessions run with permissions skipped; no mid-run permission dialog is detected.
- The relay drops the mail of an orch whose slack button is off, and mentions no one.

### Turn 3 — the three open points and the level

- Open point 1, auto-pick: lym's retries count toward the third halt at a row, each stop once, waits and accepts excluded (D3).
- Open point 2, auto-pick: only a severe halt mentions the operator; split ready, merge ready and held lines stay quiet (D8).
- Open point 3, auto-pick: the unattended orch holds a run with an open BLOCKING finding the fixer did not defer, and never accepts it (D6).
- The level, auto-pick: `beam.autonomy` gains `unattended` beside the booleans, which keep their meaning (D1).

### Turn 4 — lens review

Every lens returned a report: interactions, existing-paths, bound-breaker, walkthrough, board-scope and leanness. Duplicates are merged.

- interactions 1, bound-breaker B1, walkthrough 1 (BLOCKING), a held run chased into a severe post: fixed; a pause covering the run settles the halt in D10, criterion added.
- bound-breaker B2 (BLOCKING), the arbiter accepting deferred BLOCKING findings unattended: fixed; D6's arbiter accepts only under `false` and `true` and hands on under `unattended`.
- walkthrough 2 (BLOCKING), arbiter-settled escalations making a hard-cap the third halt: fixed; D3 counts no escalation the arbiter settles.
- interactions 2, bound-breaker M1, walkthrough 6, existing-paths 3, resume blocking on a waiting driver: fixed; D5 answers at once for a driver that keeps a wait record.
- interactions 3, bound-breaker table, walkthrough 3, the third stop with two verdicts: fixed; D2 states the verdict order and the board-claim bound is one.
- interactions 4 and bound-breaker hibernate LOW, the pause wait in the push summary and over hibernate: fixed; the driver no longer waits on a pause, the dobby resumes on the lift (D4).
- interactions 5, walkthrough 11, bound-breaker D1, existing-paths 1 and 2, the level check and older binaries: fixed; D1 names every check path, reads a refused value as `false` with one notice, and bounds older binaries.
- interactions 6, escalation counting: fixed with walkthrough 2.
- interactions 7, bound-breaker M2 and M3, walkthrough 7, 8, 9 and 10, existing-paths 5 and 6 (D8), board-scope 4, the post's path, order, sender and the dobby's input: fixed; the stop record carries the verdict, the dobby posts under its program name after-success, its state record keeps the post and the chase.
- bound-breaker M4, the `AwaitsReply` skip hiding an unread reply: fixed; D11 skips only an open question and names an unread reply.
- bound-breaker M5 and existing-paths 4, the wake reset and per-key counts: fixed; D3 counts lym's bound per class at the row over the history.
- walkthrough 4, the dedup key: fixed; a settle record names the halt entry it settled.
- walkthrough 5, the arbiter accepting a red gate: fixed; D6 accepts only a green round.
- walkthrough 12, a severe halt notifying twice: fixed; D8 mentions once per slug within the reminder interval.
- existing-paths 6 (D11), the held notice in runners with no reply probe: fixed; every runner reads the open question.
- existing-paths D5 note, batten's resume blocking a tick: fixed; D5 leaves batten's path unchanged.
- bound-breaker LOW D8, no configured operator: fixed; D12 warns of it too.
- bound-breaker LOW D11, the Raddle-Write timing gap: fixed; the skip reads the proposal and a live agent at each tick.
- board-scope 1, held board lock at start: fixed; D2 and D4 name it as the board-claim class.
- board-scope 2, hand-backs outside the table: fixed; D2 says every hand-back to the orch carries `settle: orch`.
- board-scope 3, `worktree-create-transient-net`: fixed; Open risks names it for `depends_on`.
- leanness 1, the network row: fixed; folded into Scope, Out, with its severe case kept in D2's operator-only class.
- leanness 2, D9 restating D6: fixed; D9 points to D6.
- leanness 3, rationale paraphrases: rejected; each is one sentence of why, which the rationale carries.
- leanness 4, D13's rationale: fixed; trimmed to the operator leaving the name open.
- leanness 5, assumptions restating decisions: fixed for the name; rejected for the bounds, an assumption the operator may revise.
- leanness 6, the Open risks design statement: fixed; moved into D3.
- leanness 7, the notes' study hint: rejected; emphasis only, as the lens says.
- leanness 8, the Goal's opening: rejected; it states the task's purpose the rest rests on.
- leanness 9, the listed stopped classes: fixed; Scope, Out cites the study's table.

## Rejected alternatives

- **A separate key `beam.unattended` beside `beam.autonomy`.** Rejected: it means nothing while autonomy is off, and two keys can disagree; one key with three values cannot.
- **Values `off`, `on`, `unattended`, with the booleans reinterpreted.** Rejected: a reinterpretation walk for a rename buys nothing; the booleans keep their meaning as they are.
- **One level only: make today's autonomy the strict one.** Rejected: the operator asked for a second level beside today's, and under `true` a scope change or cancel is cheap to ask while the operator is around.
- **The orch alone keeps classifying severe halts.** Rejected: an orch that stalls overnight would hide every severe halt; lym classifies the classes the table makes severe and posts them itself (D7).
- **Accept open BLOCKING findings with a board note under `unattended`.** Rejected: it lands a known defect on the parent unattended, which is hard to undo; holding the run costs only that run's time.
- **lym files a board note per open finding at every hard-cap accept.** Rejected: the accept already carries open findings into the record, and one note per finding floods the board; only the arbiter's budget accept past the scope files one note per run (D6).
- **lym answers a known permission dialog itself.** Rejected: approving a permission from a pattern list is a trust decision lym should not make unread; sessions skip permissions, so such a dialog is rare, and the orch reads it.
- **lym runs a measurement an agent asked for.** Rejected for this task: six of 73 questions, each needing lym to parse an agent's question; the orch runs it, as today.
- **Mention the operator for split-ready and merge-ready lines too.** Rejected: they are waits by design that block no other work; a mention would wake the operator for something that can wait until morning.
- **Name the level `overnight`.** Rejected: the level fits any time the operator is away, not only the night.
- **The driver waits through a pause, holding its driver lock.** Rejected after the lens review: it blocks resumes, shows in `push-retry-not-halt`'s summary and keeps a pair busy over a hibernate; the dobby resumes the run once the pause lifts instead (D4).
- **The driver's hand-back posts severe halts to Slack.** Rejected after the lens review: the hand-back notifies only with a recorded orch strand, and D10's later severe post comes from the dobby; one poster, the dobby, keeps one record of what it posted (D7).

## Review rounds

_No rounds yet._

## Question ledger

- Q1, resolved, auto-pick: name and shape of the stricter level. Picked `beam.autonomy: unattended`, booleans unchanged (D1, D13).
- Q2, resolved, auto-pick: do lym's retries count toward the third halt at a row? Yes, each stop once; waits and accepts count nothing (D3).
- Q3, resolved, auto-pick: are a raddle approval and a pull request review imperative or quiet on Slack? Quiet; only a severe halt mentions the operator (D8).
- Q4, resolved, auto-pick: accept open blocking findings with a note under `unattended`, or hold the run? Hold the run, quiet `[operator] held` line (D6).
- Q5, resolved, auto-pick: who posts a severe halt to Slack? lym, for the halts the table makes severe; the orch, for the ones it classifies itself (D7).
- Q6, resolved, auto-pick: where does the table live? A new leaf package, `internal/stopclass`, read by the driver, the notices and tests (D2).
- Q7, resolved, auto-pick: the chase interval. `beam.halt_chase_min`, 10 minutes by default (D10).
- Q8, resolved, auto-pick: the network and push classes. Left to `push-retry-not-halt`, named in the table with that task's rules (Scope, Out).
- Q9, resolved, auto-pick: what does "a finding the fixer deferred" mean? The fixer report's `deferred` disposition (Auto-mode assumptions).
