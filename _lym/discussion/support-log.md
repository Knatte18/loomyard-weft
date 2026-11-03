## Interview

Autonomous run: no operator answered; every pick below is an auto-pick recorded in the question ledger.

### Turn 1 — reading the task and the board

Read the entry `decision-record-on-board`, note `discussed-task-split` and task `shed-sub-run-block`, and `lym loom peers` (one peer, `shed-sub-run-block`, at Worktree-Create, no record yet).
No `Decision for decision-record-on-board:` note exists.
The entry fixes the record's place (a file per entry beside `board.json`, never inside it) and the flag's (a `board.json` field lym sets as it sets run status).

### Turn 2 — exploring the code

- Board: `boardCriticalSection`, the run lock (`checkRunLocks`, `scopeChanged`), `SetRunStatus`, the replay and its design-file rule, the render manifest, `lym board get --body`.
- Loom: `decision add`, `runRecordOf` and the project record, Record-Write, the tapestry, darn and sampler recipes.
- Shed: the Bouncer entry's `commit_seam` and `approve_seam` keys; every approving Bouncer path calls `approveAndCommit`.

### Turn 3 — design settled by auto-picks

Picks Q1 to Q8 in the ledger.
The main call: store through Discussion-Bouncer's approve seam rather than a new row, since every approval path reaches it and the row list `cartoon` copies stays unchanged.

### Turn 4 — lens review

Six forks returned; every report is saved beside this log.
Judged together, duplicates merged:

- interactions, existing-paths, bound-breaker, walkthrough (BLOCKING, D7 selector): fixed. `artifact_paths` also matched every Code-Bouncer; D7 now defines the approval row as the Bouncer a DiscussionWrite row's `on_done` names, the guard also refuses the seam on any other Bouncer, and D8 and D9 use the same rule.
- interactions, existing-paths (MEDIUM, D5 replayed removal leaves an orphan): fixed. The entry check applies only to a commit that writes a record file.
- bound-breaker, interactions (MEDIUM/LOW, D9 caller set undefined without `--by`): fixed. D9 names its callers.
- interactions (LOW, D3 refusal points at a verb that may refuse): fixed. The refusal says the flag is never set by hand.
- interactions (LOW, `design_prefix` could make render own record files): fixed. D1 refuses that prefix at config load.
- existing-paths (LOW, a store on a done entry may be set aside by the run ledger): fixed. D2 refuses a done entry.
- existing-paths, bound-breaker (LOW, older binary leaves orphan files): fixed. Stated under Auto-mode assumptions.
- bound-breaker (LOW, merge re-upserting its removed slug): fixed. D4 judges removal by net change.
- bound-breaker (LOW, D10 message for a flagged entry with no file): fixed in review round 1. An older binary's rename reaches that state, so D10 refuses it, naming the old slug's file and the entry's next Discussion approval as the way forward.
- walkthrough (LOW, D6 overstates carry-over ordering): fixed. D6 says carry-over precedes the seam only on paths that write one.
- leanness (MEDIUM, garbled carry-over assumption): fixed. Stated in D8 as behaviour.
- leanness (LOW, assumptions repeating D1 and D6; constraints that change nothing; OpenHub hint; duplicate constants line; Goal clause repeating Scope; Open risk repeating D7): fixed. Cut or moved to Notes; the Open risk is now only the question for the parent.
- leanness (LOW, Notes placement hint near plan detail): rejected. It is an optional hint, as the lens itself says.
- board-scope (MEDIUM, decision class, who owns the change to tapestry's Discussion-Bouncer row given T1 keeps tapestry unchanged): kept under Open risks for the parent.

## Rejected alternatives

- **A new Go row, Discussion-Store, after Discussion-Bouncer.** Visible and resumable, but it shifts the row list and progress numbers, adds a registry entry, and would have to be copied into `cartoon` too; the approve seam already runs on every approving path and halts and resumes the same way.
- **The record inside `board.json`.** Ruled out by the operator, 2026-11-03.
- **Rendering the record like a design file.** A rendered file is rebuilt from `board.json` and dropped by the render manifest; the record is source data that `board.json` must not hold.
- **`decision add` copying only the appended entries to the board.** The body asks for the same copy as at approval; a whole-record copy keeps one code path and one shape.
- **`decision add` copying whenever the entry is flagged discussed.** A second run of a once-discussed entry would then overwrite the approved record with its unapproved one during its review.
- **No recovery verb, telling the caller to re-run `decision add`.** That appends a duplicate entry.
- **Letting `lym board upsert` set `discussed`.** The entry requires lym itself to write it, and a flag without a record breaks the follow-up's gate.
- **Clearing the flag and record on `lym batten cancel`.** The entry requires a cancelled run to leave both.

## Review rounds

- Round 1: the design and consistency findings are fixed; the round's fixer-report lists each change.

## Question ledger

- Q1 (auto-pick): Where is the record stored at approval? Through `approve_seam: discussion` on Discussion-Bouncer. Rationale: every approving path calls the seam, and no row is added.
- Q2 (auto-pick): The file's name? `decision-<slug>.md` at the Bolt root. Rationale: beside `board.json` as the operator asked, with a prefix no rendered file uses.
- Q3 (auto-pick): Which order, file or flag? File first, then the flag in the same critical section. Rationale: a crash never leaves a flag without a file.
- Q4 (auto-pick): What may `decision add` copy, and when? The whole committed record, only once the run is past its Discussion approval, judged by the recipe's rows. Rationale: keeps an unapproved record off an already discussed entry; darn, sampler and pawl are excluded by their rows.
- Q5 (auto-pick): How does a failed copy recover? A new idempotent verb, `lym loom decision store`. Rationale: re-running `decision add` duplicates an entry.
- Q6 (auto-pick): What happens to the file on rename, removal and cancel? Moved, removed, kept. Rationale: the file belongs to its entry, and the entry requires cancel to keep it.
- Q7 (auto-pick): Does anything clear the flag? No. Rationale: nothing reads it yet; the follow-up that makes it skip Discussion decides how it is cleared.
- Q8 (auto-pick): How is the parallel `cartoon` work kept in step? A recipe guard test that every discussion approval row carries the seam and no other Bouncer does. Rationale: it fails whichever task lands second until its copy carries the key; raised to the parent under Open risks.
- Q9 (auto-pick, after lens review): How is the discussion approval row found? The Bouncer a DiscussionWrite row's `on_done` names. Rationale: `artifact_paths` also matches every Code-Bouncer; the DiscussionWrite engine names Discussion alone.
- Q10 (auto-pick, after lens review): May a done entry be stored? No. Rationale: the run ledger sets aside a board commit whose entries have all ended, so the copy would never reach origin.
- Q11 (auto-pick, after lens review): What if a hub sets `design_prefix` so a design file can take a record file's name? The config load refuses every prefix of `decision-`, the empty one included, and every prefix starting with `decision-`. Rationale: the render would otherwise own and delete record files.
