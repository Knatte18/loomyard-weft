```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-1 fix set agrees with the decisions, criteria and cross-references around it (fix-consistency lens).

1. **MEDIUM**: Decision 5's heading and first line, and Scope In's Part 2 bullet, contradict the new Decision 3 wiring-failure bullet.
   - Changed text: Decision 3, "`runDobby` writes its line with the error to `Geometry.LogPath` directly, without the `Deps.Log` seam".
   - Its twin was left as it was: Decision 5's heading "`dobby.log` is written only by the dobby's log seam", its first line "The `Deps.Log` seam opens `dobby.log`…", and Scope In "`dobby.log` written only by the dobby's own log seam".
   - Decision 5's new rotation sentence also names "wiring failure" as a writer, so the record now names two writers to `dobby.log` while still saying it has one.

2. **LOW**: Decision 3's heading overclaims now that the `placeDobby` bound is stated.
   - The heading still reads "Every way a dobby ends leaves a line in `dobby.log`".
   - The added bullet says a `placeDobby` refusal's "error envelope in `dobby.out` is its only trace".

3. **LOW**: the first Open risks line is stale against the same bound.
   - It still reads "Decisions 1 to 3 … leave a line for every exit the process itself takes".
   - A `placeDobby` refusal is an exit the process takes itself, and it now leaves no `dobby.log` line, as Decision 3 states.

4. **LOW**: the Question ledger in the support log is stale against Decision 3's new notice.
   - The ledger reads "Orch notices for dobby exits — auto-pick: failed exec and refused re-exec start only; resolved (Decision 3)".
   - Decision 3 now also posts the failed-re-exec notice on a `hubDobbyDeps` wiring failure after a re-exec.
   - Decision 3's first paragraph has the same gap: "A failed exec and a refused re-exec start also post one notice" does not mention the wiring-failure case that the bullet below adds.

No further tool calls were made.
