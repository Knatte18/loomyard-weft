```yaml
lens: code-affected-set
outcome: reported
```

**Task:** the code-affected-set lens. For each change the diff makes, this lists what reads or asserts it outside the touched lines, and whether the diff covers it.

**Per change, the set found**
- **`parentGuard.ensure(ctx, producer, front)` signature.** Production callers are `finalize.go:238` and `parentalign.go:245`, and both are updated. The tests in `parentguard_test.go` are updated. Covered.
- **`parentlock.Waiter.Front` and `Ticket.Front` (new JSON key, `omitempty`).**
  - Ticket readers are `liveTickets`, `tryAcquire`, `List`/`Entry.Waiters`, and `keepTicket`.
  - A ticket file written by an older binary decodes with `Front` false. That is a safe default, and the record is tolerant of unknown keys.
  - `doc.go` for `parentlock` and `landingshed` is updated. The `TryAcquire` and `Store.Covered` comments are updated, and `Store.Covered` still says "oldest live ticket", which is stale only in wording.
  - Covered.
- **`parentlock` imports `internal/fswatch`.**
  - `seam_enforcement_test.go` allowlist updated.
  - `pattern/PATTERN-leaf-packages.md` has no `parentlock` entry, so nothing to update there.
  - `internal/fswatch`'s own allowlist is unchanged.
  - `cmd/lym/daemonwakeups_test.go:32` already lists `acquire.go`, and the diff adds no sub-second literal.
  - `pattern/PATTERN-daemon-wakeups.md:13` is updated.
  - Covered.
- **`FixerAttempts.Judged` (new key).** Only Finalize writes it, in `letThrough` and `rebaseAfterParentMerge`. `AcceptFixReview` and the other writers round-trip the whole struct, so they keep it. Parent-Align uses a separate per-row file. Covered.
- **Finalize frees the lock during the fixer and fix review.**
  - Doc surfaces updated: `docs/overview.md:604`, `landingshed/doc.go` (lines 49–55 and the Bound line), and the comments in `finalize.go`.
  - `template.yaml:20` (`parent_lock_idle_release_min`, "bar a run at Finalize … which keeps it"): the idle-release hooks in `loomcli/looplock.go` act only on records the loop owns. A freed lock has no record, so the text stays true.
  - `PATTERN-parent-lock` (`PATTERN.md:33`) says landing happens only in Finalize under the lock. Landing still does, so the text holds, and there is no `pattern/` background file for it.
  - `orchcli/doc.go:79` is generic and still true.
  - `overview.md:603` still says Finalize "releases the lock on every exit". That still holds, and the new line follows it.
  - Covered, except the finding below.
- **`rebaseAfterParentMerge` and the `Baseline == head` shortcut in `judgeFixerChange`.** The only other reader of `Baseline` is `endSpent` (`finalize.go:897`), which uses it for the "fixer commits" text. After a rebase it names the post-merge range. That fits the stated Bound. Covered.
- **PATTERN declared changes.** The directive declares none. The `PATTERN-daemon-wakeups.md` edit is in the same diff as the code change. No `Pattern-Departure:` trailers are present or needed.

**Finding**

- **LOW, `decision`, `internal/loomcli/landingdeps.go:223, 543` and `internal/pauserecord/hold.go:286, 445`.**
  - Summary: the verify pause-hold exempts a run that holds the parent lock (`HoldsParentLock` → `SlugEntryExempt` and `UnderParentLock`). The diff makes the fixer's own `lym gate test` calls run with the lock freed, so a hub pause covering the run now holds those gate calls. Under the lock they were exempt.
  - Behaviour: a covered returning run also yields the queue and waits for the resume. That is consistent with pause semantics, but neither the code nor any doc says it.
  - Fix, if the exemption is wanted: have the parent decide this. Otherwise add one sentence to `landingshed/doc.go` saying a pause now holds the fixer's gate calls, because the run no longer holds the lock.
  - Evidence: `holdsParentLock` reads `store.HeldBy(slug)`, which is false once `freeLock` has released the record.

**Tool calls (4)**
1. Grep across docs, contracts, internal and pattern for text that ties the parent lock to the fixer or verify. My context had no such search.
2. Grep and read of `template.yaml`, `orchcli/doc.go`, `loomcli` lock mentions, and the `fswatch` and `leaf-packages` allowlists. I needed to check the leaf and idle-release surfaces.
3. Read `holdsParentLock` and its consumers. I needed to see what depends on the run holding the lock.
4. Grep of `docs/*.md`, `doc.go` and `PATTERN.md` for other statements about Finalize and the lock. This was a completeness check on the docs.
