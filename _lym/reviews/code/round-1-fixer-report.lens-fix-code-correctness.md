```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the changed code (F1–F7 change set) through the code-correctness lens.

**No findings** of BLOCKING or MEDIUM severity. One LOW:

- **LOW** — `internal/beamcli/deps.go` `MarkReady` dep, the new `ReadyFor`-then-skip-write branch. Nothing pins that a record already present is committed without being rewritten. The `landWorld` fake models the skip itself (`if !w.ready[slug]` before the append), so it passes whatever the real dep does. The hubforge test calls `markReady` only on an unflagged beam. Take a dep that always rewrites, or one that skips the commit for a flagged beam: every test still passes. F3's premise, a failed commit followed by a rerun, is therefore covered only at the verb level. A tier1 row that flags a beam by writing the record uncommitted, calls `markReady`, and checks the record is committed would pin it.

**Traced and holding:**
- **`landBeam`, refusal then retry.**
  - `Land` refuses on the first tick: the notice posts, `Land` is recorded as `{Refused: true}`, and `lands` is 1.
  - On the second tick the record is `Refused`, so it falls through, `Land` refuses again, `record.Land != nil` returns nil, and no second notice goes out.
  - After a success, `recordLand(false)` overwrites the record and closes the episode.
  - A reading that stops passing clears it.
  - A failed notice post leaves `Land` nil, so the next tick retries.
  - A failed `recordLand` after a post would repost, as before the fix.
- **Parent-owner branch.** A refused episode whose parent later reads as owned elsewhere posts the owner notice and closes the episode with `refused=false`. That is a second, different notice, and acceptable.
- **New dobbyengine test.** With the defect in place the old early return stops tick 2 from calling `Land`, so `lands != 2` fails. With the fix it gives `lands=2` and one notice, then `lands=3` and still one notice, with the episode closed. The reworded refusal-once test no longer pins `lands`, and the new test covers that.
- **Retries actually fire.** The loop re-ticks on every probe interval (`loop.go:432`, `:484`), not only on watch events, so a refusal that waits on Beam-Await is retried.
- **`markReady` always calling `MarkReady`.** The `record` key is now also set for an already-flagged beam. No test asserts its absence, and the doc comment states the new behaviour.
- **F4 rows.**
  - `OwnerRefusal…` hits `judgeLandOwners` before `Ready`.
  - `ReadyRefusal…BeforeWorkRemaining` fails if the work-remaining check moves earlier, because `wantRefusalRead` is false.
  - `markCalls` pins "flagged still commits" and "refusal before `MarkReady` gives 0 calls"; the failing-`MarkReady` row expects 1.
- **F2 test.** With the ready check removed from `beamLandReading` or from `land`, the first `CanLand`/`Land` assertions fail: blockers are empty, so the reading passes and `Land` writes the request. It does not assert that `ReadBeamLandBlockers` was skipped, but the outcome is pinned.
- **F5, F7, F6.** The test row's `want` order matches `pawlRecheckLines` (blocker, beam owner, parent owner, ready). The text edits are inert.

**Tool calls:**
1. `grep` of `internal/dobbyengine/loop.go` for probe, tick and watch. My context did not show whether ticks run on a timer or only on watch events, which decides whether the refusal retry ever fires.
