# Support log — adopt-robust

## Interview

Autonomous run: no operator answers. Every pick below is an auto-pick, recorded in the question ledger.

### Turn 1 — exploration

Three exploration subagents read the adopt verb, the pause and owner paths of tender, dobby and the orch watcher, and the `lym s` shortcut, detach and archive code.
Findings that shaped the design:

- Adopt runs in-process today and records no progress; only the adopt check is detached.
- The adopt hold is a `pauserecord` entry of caller `adopt-hold`; the green end lifts it, brings up dobby and orch, starts first passes and resumes owner-halted children.
  That is the path that started every adopted run on both dates.
- `lym tender pause` finds the hold entry of the same scope and answers `already_paused`.
- The orch watcher stops the orch strand when the local board reads a foreign owner; the dobby exits on its probe.
- `lym s` is a static shortcut; `runArgvs` stops at the first non-zero exit.
- Archives exist as `archive/<slug>/<sha12>` tags; there is no archive-branch concept.

### Turn 2 — design picks

Each board finding mapped to a decision: detach and record (D1, D2), step order (D3), suppression during the adopt (D4), the pause (D5), owner before deploy (D6), orch on request (D7), green check (D8), status (D9), entry point (D10), old owner archive (D11, D12).

### Turn 3 — lens review

Six forks reviewed the record; every fork returned a report. Duplicates are merged under the first lens that raised them.

- interactions/walkthrough/bound-breaker BLOCKING, D12 read the archives at `reconcile`, before the old owner can archive: fixed; D12 is now `lym adopt --archives`, a repeatable read the end envelope names.
- interactions/existing-paths MEDIUM, D4 did not suppress D11's archive and owner pause on the adopting machine: fixed; D4 now covers D11's steps and D11 runs only when the subject is not under adopt.
- interactions/walkthrough/bound-breaker MEDIUM, an unrelated unfinished record made `adopt --here` refuse and blocked `lym s`: fixed; `--here` reads the owner after a Bolt pull and, owned here, ends without touching the record.
- interactions MEDIUM, a caller whose worker lost the lock printed another adopt's result: fixed; D1 matches its worker by pid and refuses on other subjects.
- interactions MEDIUM, resume flag merge ambiguous: fixed; the resume keeps the record's flags and adds the call's.
- interactions LOW, a tender resume could not start a pass under an interrupted record: fixed; the suppression now ends at the subject's owner write.
- interactions LOW, a fix task filed under an interrupted record: fixed by the same narrowing.
- interactions LOW, the D4 criterion contradicted D11: fixed; the criterion names D11.
- existing-paths MEDIUM, a `done` record with a held-back project ended the suppression while still foreign-owned: fixed; such a record ends `stopped` and D4 keys on the subject's owner write.
- existing-paths MEDIUM, the check-subject rule and the run report read `holds.json`: fixed; the check starts for each subject whose owner write stood, and the report reads the live pause entry.
- existing-paths LOW, an older check can lift a legacy hold once: fixed; stated as a D5 bound.
- existing-paths LOW, nothing brings up a dobby whose beam step failed: fixed; D7 states the warning is the only way back.
- walkthrough MEDIUM, the old owner's Bolt may never learn the new owner: fixed after research; no periodic Bolt pull exists (`internal/fabricengine/hubpull.go` is reached only from `lym fabric pull`), so D11 adds a periodic Bolt pull in the dobby.
- walkthrough/bound-breaker BLOCKING, today's settled-runs lift and the live-here exception let a run go on with no operator resume: fixed; D3 drops the lift and D5 pauses live-here runs too.
- walkthrough/bound-breaker MEDIUM, a dead `--here` record idled an owned subject: fixed; `--here` writes no record when owned here, and D4 needs a foreign owner.
- walkthrough MEDIUM, D11's retry had no mechanism to start no pass: fixed; D11 owner-pauses the runs before it archives.
- walkthrough LOW, the record did not keep which owner writes stood: fixed; D2 lists them.
- walkthrough LOW, an `--all` resume computed another set: fixed; `--all` matches an `--all` record.
- walkthrough LOW, the caller could not tell a fresh record from an old one: fixed by the pid match in D1.
- bound-breaker MEDIUM, a stale record on the losing machine disabled its owner exit and archive: fixed; the suppression ends at the subject's owner write, and the bound before it is stated.
- bound-breaker MEDIUM, D11's archive push took no teardown lock: fixed; it runs under the slug's teardown lock.
- bound-breaker LOW, record written before the reconcile not stated: fixed in D2.
- bound-breaker LOW, D6 claim false for live-here runs: fixed by D5 pausing them.
- bound-breaker LOW, task starts under a failed archive: fixed; D11 names batten's owner refusal.
- bound-breaker LOW, `--keep-local` ignored on a resume: fixed; it restarts at `reconcile`.
- leanness MEDIUM, the open risk restated a made choice: fixed; Open risks reads None.
- leanness LOW, Scope In repeated the decisions: fixed; one line with ids.
- leanness LOW, constraints restating PATTERN rules the decisions apply: fixed; cut.
- leanness LOW, D3's last paragraph repeated D5: fixed; cut.
- leanness LOW, assumption bullets duplicating decisions: fixed; the tags reason moved into D11.
- leanness LOW, the plan-writer note on `HoldCaller` repeated D5: fixed; cut.
- leanness LOW, D11 and D12 rationale overlap: fixed; D12's rationale now states the timing.
- board-scope LOW, tags in place of archive branches: rejected; the record states the deviation and its reason in D11.
- board-scope LOW, the adopt pause differs from an ordinary pause: rejected; D5 states the board's operator-only lift and why the dobby runs no continue pass.

## Rejected alternatives

- Keep the hold and only stop the green end lifting it: rejected, since its name and the dobby's `Held` handling would still describe a self-lifting hold; a new `adopt` caller states the operator-pause meaning, and the legacy value reads the same.
- Make the adopt pause an `EXTERNAL` pause: rejected; the report and `tender` could no longer tell the adopt's pause apart, and an ordinary pause lets the dobby run continue passes, the finding of the dobby's `--continue-pass` during a takeover.
- Detach and return at once: rejected; `lym s` must not start the beam before the adopt ends, and the caller wants the envelope.
- Change `lym beam start` to adopt a foreign-owned worktree itself: rejected; it widens a verb many paths call, while a separate `adopt --here` argv keeps `beam start`'s refusal.
- Decide `--here` from the local Bolt alone: rejected; a stale Bolt that reads this machine as owner would skip the adopt whenever origin answers.
- Make `--here` refuse when origin is unreachable: rejected; every offline `lym s` on an owned-here worktree would stop, and `beam start` judges the local Bolt the same way.
- Archive branches on origin: rejected for tags; a tag push moves no branch the owner rule reserves.
- Reuse the `archive/` tag prefix and teach its readers to skip D11's tags: rejected for an `archive-owner/` prefix; every reader would need the rule, and `--archives` could not tell teardown archives apart.
- Spare the adopt role in the anchor's dobby end: rejected; the anchor would then move the prime under a worker mid-step, while an interrupted worker resumes under D2.
- Let `lym tender pause` accept a slug with no pair: rejected; adopt pauses every run itself and the record keeps the dobby idle before then.
- Keep the resume of owner-halted children at the green end: rejected; it starts work. The run report names the hand step instead.
- Read the old owner's archives in the adopt's own reconcile: rejected; the old owner archives only after the owner write reaches it.
- Show the archives in `lym beam status`: rejected; it would reach origin on every status read.
- One adopt record per subject: rejected; one adopt spans several subjects with one reconcile, and `--here` taking up only a record that lists its own subject removes the conflict that motivated it.
- Keep live-here runs unpaused: rejected; the dobby's owner lift would let such a run go on with no operator resume.
- Archive before pausing on the old owner, as the board orders: rejected; the dobby could start a pass during a failing archive.

## Review rounds

- Round 1, BLOCKING: `--here` now reads the record first and resumes an adopt that died after its owner write (D10);
  D4 covers beams that read their owner from an adopted subject;
  a resume before `owner` re-runs the reconcile (D2); `--discard` takes `adopt.lock`;
  D11's tags take an `archive-owner/` prefix that D12 reads.

## Question ledger

- Q1 How does adopt survive its caller? Auto-pick: a detached worker the caller follows (D1). Resolved.
- Q2 Where does progress live, and how does a re-run resume? Auto-pick: one hub adopt record under the adopt scratch directory, resume on same subjects, `--discard` for a dead one (D2). Resolved.
- Q3 Does the suppression also cover the dobby's exit? Auto-pick: yes, the dobby stays up and idle like the watcher (D4). Resolved.
- Q4 Which pause? Auto-pick: a new `adopt` caller lifted only by hand, `Held` semantics in the dobby (D5). Resolved.
- Q5 Are unstarted released tasks paused too? Auto-pick: yes, as today's holds (D5). Resolved.
- Q6 What does "run from an orch session" skip? Auto-pick: the orch start of every subject (D7). Resolved.
- Q7 How does `lym s` adopt only when needed? Auto-pick: `adopt --here --no-orch` as the shortcut's first argv, judged after a Bolt pull; changed at the lens review from a full reconcile, and at review round 1 to take up first an unfinished record that has not reached `check` for the worktree (D10). Resolved.
- Q8 Archive branches or tags? Auto-pick: tags, under their own `archive-owner/` prefix, changed at review round 1 from the existing `archive/` tags (D11). Resolved.
- Q9 Archive before or after stopping runs? Auto-pick: pause first, changed at the lens review so no pass starts during a failing archive (D11). Resolved.
- Q10 Does a failed archive keep the dobby up? Auto-pick: yes, retrying at each probe with its runs owner-paused (D11). Resolved.
- Q11 Should `lym tender pause` lose its pair refusal? Auto-pick: no; kept out of scope. Resolved.
- Q12 How does the old owner learn the new owner? Researched: only on a Bolt pull, which an idle machine never makes; auto-pick: a periodic Bolt pull in the dobby (D11). Resolved.
- Q13 When does the suppression end? Auto-pick: at the subject's owner write, and only while the subject reads foreign-owned (D4). Resolved.
- Q14 Where does the new owner see the archives? Auto-pick: `lym adopt --archives`, a repeatable read (D12). Resolved.
