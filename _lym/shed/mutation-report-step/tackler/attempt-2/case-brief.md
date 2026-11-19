# Case brief: Worktree-Create stop for run `mutation-report-step`

## 1. The question the next judge must answer

Why does `battencli` keep refusing to claim the board entry of `mutation-report-step` with "a held board lock deferred the push of the commit, and the board is still ahead of origin", after attempt 1 resumed with `goto: Worktree-Create` and the board sync was already run by the driver? Is the board commit left unpushed in `_bolt` safe to push by a re-run, or does it need a board sync by someone who can make that call?

## 2. The facts, each with a pointer

- The stop envelope is `_lym/shed/mutation-report-step/drive-reports/drive-envelope-20261119-102902-9aca.json`: `state: blocked`, `current_producer: Worktree-Create`, `history_length: 3`, `interrupt_policy: handback`, reason as in the task.
- The stop's trace copy `.lym/shed/mutation-report-step/steps/a223821f5cd36e91.trace.log`:
  - line 6 (10:27:57): `fabricengine: push lock busy: .../loomyard-weft/.weft/fabric.push.lock still held after 30s, held by pid 3750437`.
  - lines 35-36 (10:28:31, 10:28:40): `gitrepo: push refused` for `main` twice: "cannot lock ref 'refs/heads/main': is at 73fee242 but expected 8ba9c1f8", then "is at f5037a6c but expected 73fee242". The remote `main` moved between attempts.
  - line 37 (10:28:46): `battenshed: producer stuck` with the same board reason.
  - line 40 (10:28:46): detached board sync, `commit_created` in loomyard-weft, `9744e0d04baf10a89415caf94392c2a0dafdfe84`.
  - lines 41-42: `main-weft` push refused twice with `(failed)`.
  - line 43 (10:29:02): `branch_pushed` target `main-weft`.
  - line 44: `shed: step done`, outcome `stuck`.
- The driver log `.lym/logs/trace-20261119T092552Z-96e51db72c94d52d-3737402.log`:
  - line 4 (10:26:43): battencli warns the task worktree `/home/knatte/Code/loomyard-LYMHUB/mutation-report-step` is not present, though durable status says it was created.
  - line 15 (10:29:02): attempt 2 tackler spawned for this stop.
- Attempt 1: `_lym/shed/mutation-report-step/tackler/attempt-1/verdict.yaml` returned `resume` with `goto: Worktree-Create`, on the reasoning that the board commit `4212594a` was already on origin. `attempt-1/attempt.json` records `history_length: 1`, so attempt 1 was for an earlier stop key. The same board reason returned on re-step at 10:29:02, so the resume returned unchanged.
- Current read-only state:
  - `git -C /home/knatte/Code/loomyard-LYMHUB/_bolt status -sb`: `main...origin/main [ahead 1]`.
  - `git -C .../_bolt ls-remote origin main`: `c44f3d193055ee7342191f2e45304f6c2a341c26`, which is the local `origin/main`.
  - `git -C .../_bolt rev-parse HEAD`: `262c5f17269bfd490db5b062013c128ff1461be5` (`board sync`), local only and not on origin.
  - `git -C .../loomyard-weft ls-remote origin main-weft`: `9744e0d04baf10a89415caf94392c2a0dafdfe84`, equal to the local `main-weft` tip. The records commit the stop names is present on origin.
  - `git -C .../loomyard-weft ls-remote origin main`: `0886f9ff35c8d4b755841e5aa7dd8626d6162622`.
  - `.weft/fabric.push.lock` is a 0-byte file dated Oct 27; `ps -p 3750437` shows no such process.

## 3. What you ruled out, and how

- **A plain resume again.** The rule for a stop already resumed that returned unchanged is hand-back or escalate. The board reason is the same as attempt 1's, so this stop returned unchanged.
- **The records commit as the cause.** `9744e0d` is on origin `main-weft`, equal to the local tip (read-only `ls-remote` and `rev-parse`). The records side has landed.
- **A stale push lock.** The lock holder pid 3750437 is not running now. The lock file is empty.
- **A hand-back.** The run records no parent (`prompt.md` says so), so a question has nowhere to go. The cause also needs a judgment on whether the local-only board commit may be pushed while `origin/main` keeps moving.
- **A commit absent from a remote.** Not the basis of this verdict. The unpushed board commit `262c5f17` is local-only, and I read it from `_bolt` directly, not from a fetch.

## 4. Hypothesis (unverified)

Hypothesis: the board's `main` push fails because other writers keep moving `origin/main` between the board sync's read and its push (the two `remote rejected ... expected` lines in the trace). The local board commit `262c5f17` then stays unpushed, so `battencli` keeps reporting the board as ahead of origin. A later board sync or re-run may push it, but nothing I read shows that the push would succeed, or who should run `lym board sync` against a moving `main`.
