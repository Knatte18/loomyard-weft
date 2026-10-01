# Card 64 — refusal-spec-round-four

**Prosa:**
- `contracts/specs/refusal-spec.md`
- `contracts/specs/webster-spec.md`

**Intent:**
Bring the webster section of the refusal table in line with cards 55–63, which PR #321 round 4 asks for alongside the code, and fix finding G's spec items.
Each edited row's Way forward cell quotes the clause its message now carries, as the file's own rule requires.
Keep semantic line breaks;
table cells stay on one line.

Above the table:

- The audit sentence adds a fork write under webster's run directory other than the fork's own report, and a parent write under the run's `.lyx` state directory, to the correctness list.
- A new sentence where the audit rows are defined: the audit sees writes made through the Write, Edit and NotebookEdit tools only, so a write made through Bash is caught only when its command references the fabric repo;
  the audit catches slips, not an adversary.
- A sentence stating the evidence rule: a correctness halt clears only on evidence that HEAD and every suspect path match what the run recorded, never on an acknowledgement.

Rows:

- Card 55:
  - new `recorded commit missing` row: accept-audit or `run --fresh` finds a batch commit the run recorded missing from the repository; correctness halt; fetch the task branch from the machine that ran those batches with git, then re-run the verb.
  - `rebaseline card set changed` and `fresh over a differing suspect path` name the start commit picked by ancestry, with no wording change.
- Card 56: new `accept-audit: HEAD past last batch head` row: HEAD carries a commit past the last batch head other than a clean parent merge; correctness halt; move HEAD back to the head with git, then re-run `lyx webster accept-audit`.
- Card 57:
  - new `restore-plan: copy missing` row: a plan file to restore has no stored copy; correctness halt; reset the branch to the run's start commit with git and run `lyx webster run --fresh`.
  - `no run in progress` and `fabric sync failed` add `restore-plan` to their verb lists.
- Card 58:
  - `accept-audit: suspect path differs` covers paths outside the plan directory, and a new `accept-audit: plan path differs` row names `lyx webster restore-plan`, or `lyx webster rebaseline --card NN` for a card no batch has begun, then re-running `lyx webster accept-audit`;
  - `run-exit audit correctness`, `pending audit findings` and the run-exit clauses of the `audit: parent-write` and `audit: fork-plan-write` rows add the plan-path clause;
  - new `fresh: HEAD not at the start commit` row: `run --fresh` finds HEAD is not the run's start commit while it would drop findings; correctness halt; reset the branch to that commit with git, then re-run `lyx webster run --fresh`;
  - new `fresh over a differing plan path` row: a pending finding's plan path differs from the plan the run recorded; correctness halt; `lyx webster restore-plan`, or `lyx webster rebaseline --card NN` for a card no batch has begun, then re-run `lyx webster run --fresh`.
- Card 59:
  - new `plan changed before the verb's own rewrite` row: validate, record-batch or recover-batch finds the plan changed since the run recorded it, or a begun batch's card changed since it began; correctness halt; the `plan fingerprint mismatch` way forward;
  - `plan fingerprint mismatch` adds `or restore the plan the run recorded with lyx webster restore-plan`.
- Card 60:
  - `audit: fabric-reference` and `audit: parent-write, tracked, _lyx, scratch or another worktree` at record-batch: the batch fails, and when a finding has no path or a path recovery cannot check, `recover-batch` refuses toward the `--fresh` route instead of recovering;
  - new `recover-batch: findings recovery cannot check` row: correctness halt; reset the branch to the run's start commit with git and run `lyx webster run --fresh`.
- Card 61: `recovery suspect path not cleared` says the flagged content is looked for anywhere in the recovery head's tree, and adds the plan-path clause for a plan suspect path.
- Card 62: new `audit: fork-state-write` row: a fork's transcript wrote under webster's run directory other than its own report; correctness halt; at record-batch the batch fails, even over a record already terminal, and recovery refuses toward the `--fresh` route since the path cannot be checked;
  at run exit, the pathless-or-uncheckable `--fresh` route.
- Card 63: the `audit: parent-write, tracked, _lyx, scratch or another worktree` row's name and trigger add the `.lyx` state directory, and `audit: parent-write, elsewhere`'s trigger excludes it.
- Finding G's run-exit item: `audit: parent-write, elsewhere`'s way forward becomes: warn and record once;
  at record-batch after the card verify commands re-run and pass, a failing verify making the finding correctness;
  at run exit with no re-run, the integration stage covering a plan with a `## verify:` section.

`webster-spec.md`'s ownership sentence lists `plan-baseline/`, the content store behind the recorded plan hashes, among what webster owns under `_lyx/webster/`.
