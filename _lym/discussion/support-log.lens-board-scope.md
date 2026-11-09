```yaml
lens: board-scope
outcome: reported
```

Task: review the adopt-robust decision record for fit with the board.

**Findings**

- LOW: D11 and D12, Auto-mode assumptions. The entry asks for an "archive branch on origin" and says "the new owner sees the archive branches". The record uses `archive/<slug>/<sha12>` tags instead. This changes the mechanism the entry names, and the record says so openly in its Auto-mode assumptions with its reasons: tags already exist, the cleanup already reads them, and a tag push moves no branch the owner rule reserves for the owner. The entry's intent still holds, because the new owner sees the archives through D12. No `Decision for adopt-robust:` note settles this. Evidence: entry body under "The old owner"; D11 step 1; the first bullet of Auto-mode assumptions.
- LOW: D5. The entry asks for "an ordinary operator pause". D5 adds a new `adopt` caller with the dobby's `Held` meaning, so the dobby runs no continue pass under it. An ordinary pause would allow one. The entry's own principle, "only the operator lifts that pause", still holds, and the record gives its reason: a continue pass during a takeover is one of the entry's own findings. So this is no contradiction, only a different wording from the entry, and the record explains it. Evidence: the entry's step 4 and its finding about `--continue-pass`; D5.

**Goal points of the entry**

- Detach and record; the watcher stays up and dobby starts no pass under the subjects while the record exists: met (D1, D2, D4).
- Reconcile from origin with keep-local and drop-local: met (D3 step 1).
- Add the pairs this machine lacks: met (D3 step 2).
- Pause every adopted run with a pause only the operator lifts: met (D5; the wording point is above).
- Write the owner before the deploy: met (D6).
- Deploy when lym is behind: met (D3 step 5).
- Start dobby always, and the orch only on request: `--no-orch`, with no orch when adopt runs from an orch session: met (D7).
- The adopt check: green only reports, red files a fix task: met (D8, Scope).
- Mark the adopt done, and `lym beam status` shows an unfinished adopt with the step it stopped at: met (D2, D9).
- The operator resumes runs with `lym tender resume`: met (D3, D5).
- Entry point `lym s` adopts first, the beam in a beam and root in the prime: met (D10).
- The old owner's dobby archives the tips origin lacks, then stops its runs; the new owner sees the archives: met, with tags instead of branches (D11, D12).
- The entry's findings: `already_paused` is answered by D5. "No pair, so no pause" is answered by D4 and D5 and recorded as out of scope with an open risk. The dobby's continue pass is answered by D4 and D5, an adopt that dies with its session by D1, and the unpushed commits by D11 and D12. All are met.

**Entries, peers and parent commits checked**

- Peer `arras-recipe-perch` (Darn-Write): no conflict; no adopt, dobby or orch paths.
- Peer `testmap-verb`: no conflict. It touches `internal/hubgeom/hubgeom.go` and `internal/lymdirs/dirs.go`, but the overlap is text only.
- Peer `terminal-titles-hub-project` (Finalize): no conflict. It touches `reedcli`, `battencli` and `boardengine` handles, none of which the adopt design uses.
- Parent commits from `lym loom peers`: none listed.
- `machine-restart-shortcut` (F2, note): no conflict. It brings up orchs after a reboot and does not adopt; the shortcut label is shared, but the entry adds no shortcut of its own.
- `machine-name-verb` (T13): no conflict. Its owner sweep across a rename goes through `boardengine`'s owner setter, and the record does not change that setter.
- `reparent-running-task` (F20) and `one-land-notice` (E3): no conflict.

**Decision notes**

- `lym board list` shows no `Decision for adopt-robust:` note, and the record has no `Added after Discussion` entries. There was none to read and none I could not reach.

**Tool calls**

- None. My context already held `lym board list`, `lym board get` for adopt-robust and for the touching entries, and `lym loom peers`.
