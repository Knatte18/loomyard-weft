```yaml
lens: fix-claims
outcome: reported
```

Task: check that the claims the fix added or rewrote hold against the code at HEAD (fix-claims lens).

**Finding 1 — LOW**
- **Changed text:** card 3 (`03-fabric-takeover-writer.md`), the Intent sentence that begins "The commit, push and retry of the parent's records branch in steps 4 and 5 hold the parent slug's teardown lock too, taken the same way, so a pair's remote steps hold its slug's lock (`PATTERN-pair-remote`)."
- **Summary:** "taken the same way" means exclusive, but the pattern cited for this asks for a shared lock on a pair push outside a removal. So the sentence names the pattern for a lock mode the pattern does not prescribe.
- **Evidence:** `pattern/PATTERN-pair-remote.md` says "under its teardown lock, shared for a pair push outside a removal and else exclusive". The push of the parent's records branch is not a removal. The only exclusive precedent is `HandOverTips` (`internal/fabricengine/handover.go:71`), and that lock is on the run's own slug, not on a parent's. Either name the shared mode for the parent's push, or say why the rebase-and-retry needs exclusive.

The other claims I checked hold:
- `HandOverTips` takes the lock exclusive through `lock.AcquireWriteLockWithin(..., StatusPushLockWait)` (`handover.go:71`).
- `ArchiveOwnerTips` takes each pair's own lock through `archiveOwnerPair` (`ownerarchive.go:75`), and `archiveOwnerSides` is the helper that runs under a lock already held (`ownerarchive.go:89`).
- The only production caller of `NewWatch` is `orchcli.UnderAdoptWatch` (`internal/orchcli/watch.go:91`). The only production caller of `Record.Covers` is `Watch.covers` (`internal/adoptrecord/watch.go:65`); orchcli calls `Watch.Covers`.
- The sentence "WriteHold is the one writer of the holds" is in `boardengine/doc.go:239`, and the hold field list sits under `# Machine registry and holds` (`doc.go:210-216`).
- The guard step runs after a `--card` scope (`internal/gatecli/doc.go:317-322`).
- `testOnlyCodeAllowed` holds the `internal/runfsm.Order` precedent, and stale entries fail through `RequireNoStale` (`cmd/lym/testonlycode_linux_test.go:59,96`).
- `hubGreenDeps(prime, subject)` is the current signature (`internal/adoptcli/green.go:63`).
- The replay result carries `Lost` (`internal/fabricengine/boltreplay.go:46-49`).

Tool calls:
1. Read `PATTERN-pair-remote` for its lock rule, `orchcli/watch.go:85-100`, the `NewWatch` callers and the `Lost` field in `boltreplay.go`: my context had none of the pattern text, the orchcli caller body or the replay fields.
2. Read `PATTERN-pair-remote`'s lock-mode lines and `HandOverTips`'s lock code (`handover.go:60-80`): my context had only grep hits for the lock call, not which mode the pattern requires.
