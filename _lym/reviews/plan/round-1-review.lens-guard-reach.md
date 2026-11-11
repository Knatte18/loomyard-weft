```yaml
lens: guard-reach
outcome: reported
```

Task: guard-reach lens. For every guard, refusal, branch and test row in cards 1–4, check that a real entry point can reach it.

**Per guard, reachable or not**

Card 1 (`owner-replace-set`):
- `Replace` refused together with `Slugs`, `Prime`, `Claim` or `Takeover`: not reachable from any state a run can produce, and that is by design. The only caller passes none of them, so it is a fail-loud wiring guard in the `refusal-spec.md` sense (contracts/specs/refusal-spec.md:36). No finding.
- Prime selected when its recorded owner is in the set: reachable. `ReplaceSetSelectsAfterThePull` seeds prime `box`.
- Open beam selected by recorded owner after the pull: reachable. `oldbeam` exists at origin only (`pushFromOther`).
- Empty entry in `Replace` selects nothing: reachable. `Replace {"box","","old"}` runs against the unowned `beamer`.
- Done beam kept: reachable (`shipped`).
- Owner outside the set kept: reachable (`farbeam`).
- No match writes nothing: reachable (`ReplaceSetWithNoMatchingOwnerWritesNothing`).
- A replace-selected slug lost to a competing origin write: reachable through the `BeforePush` path, the same one `CompetingWriteLosesToOrigins` takes (ownerset_integration_test.go:453).

Card 3 (`machine-name-show`):
- File does not decode, or holds an invalid name: reachable through the fake `locate` path.
- File does not read: reachable. A directory at the file's path makes `ReadFile` return exists false with an error (machinename.go:55-61).
- Relative variable: reachable through a `locate` error.
- `hostName` error with no file: reachable.

Card 4 (`machine-name-set`), all through `TestName_Set` fakes unless noted:
- Step 1, invalid name: reachable.
- Step 2: the `locate` error, no directory, and a file that exists but does not read are each reachable. A decode error correctly passes on to the repair.
- Registry read failure: reachable through a `hubs` error, before any write.
- Pull error or skip marked checked locally: reachable. At tier 1, an unreachable origin gives `BoltSkipFetchFailed` (bolt.go:186), which counts as a skip, not an error.
- Unreadable board: reachable through a `readOwners` error. In production, an undecodable `board.json` reaches it through `ParseOwners`.
- Collision: reachable at tier 0. At tier 1 an origin-only owner reaches it after the pull.
- Own former name, and the hostname rule's name, pass the collision check: reachable. With `BAD`, `readCurrent`'s valid name is empty and the own names come from `names` and the host.
- Step 5, no write when nothing changes: reachable only at tier 1 (`FailedHubIsSweptByTheReRun`, file bytes unchanged). The `AlreadySetChangesNothing` row in `TestNextFile` reaches the helper, not this write decision.
- Step 7, failed, lost, swept and nothing to sweep: each reachable.
- Step 7, `pushed`: reachable through `unpushed` true.
- `hubOfDir`'s container and plain-directory rows: reachable at tier 0. `t.TempDir` lies under `os.TempDir()` (lymtemp/root.go:36), outside any git repository, so `ResolveWorktree` falls through.
- `hubsToSweep` sweeping a hub once: reachable through the fake `cwdHub`. In production both sides take the hub path from a resolved Location (creelcli/hubdirectory.go:56), so the strings match.

**Findings**

1. LOW, design, card 4 (`_lym/plan/04-machine-name-set.md:68`, `:85`).
   Card 4 adds the `unpushed` seam (`func(boardDir string) (bool, error)`, `Bolt.HasUnpushed`), but no step says what an `unpushed` error does: refuse, mark the hub, or treat it as false. The branch exists, but it has no prescribed outcome and no test row.
   Evidence: `_lym/plan/04-machine-name-set.md:85` ("read `unpushed` (kept for step 7)") and `internal/fabricengine/bolt.go:171` (`HasUnpushed` returns an error).
2. LOW, consistency, card 4 (`_lym/plan/04-machine-name-set.md:98`).
   The way forward for a failed hub forks on "when the error names one". Nothing defines how `sweepHub` tells whether an opaque `SetOwners` error names a way forward, so the row `FailedHubIsReportedAndOthersSweep` reaches a branch with no stated condition.
   Evidence: `_lym/plan/04-machine-name-set.md:98` and `internal/boardengine/ownerset.go:122` and `:278`, where the way forward is embedded in the error text only.

No guarded site recurs unguarded on a path the cards miss.

Tool calls:
- Read `lymtemp.ProcessRoot`'s root path, to judge whether `TestHubOfDir`'s `t.TempDir` could lie inside a git repository. The manifest lacked lymtemp.
- Found the production writers of `hubregistry.Record`, to judge whether `hubsToSweep`'s once-only check compares matching path strings. The manifest lacked them.
