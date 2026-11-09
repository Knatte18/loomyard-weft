```yaml
lens: board-scope
outcome: reported
```

Task: check the machine-name-config decision record against the board, its peer runs and any parent commits.

**Findings**

1. LOW: Scope, D5 and D6. The record's scope is wider than the entry's own sizing.
   - Evidence: the entry says "Every caller already takes the name through `machinename.Name`, so the change sits in that package and its config loader."
   - The record goes further. It adds a top-level `lym machine` module (`internal/machinecli`), a new `boardengine.OwnerRequest` field, a change to `battencli.DobbyLandings`, a new `standalonestate` resolver and a `tmuxkit` env change.
   - The entry's open question about migrating owners already written under the old name requires work beyond that package, so this is no contradiction. A reviewer may still want the widening named as following from that open question.
   - Class: scope.
2. LOW: Constraints and Scope. The pair teardown seam is not covered by name.
   - Evidence: the entry's "What the name reaches" list asks for each consumer to be checked one by one, including "the pair teardown seam that compares machines".
   - The record covers owners (D6), the address rule (D4) and Slack (Constraints). It never says that pair teardown reads owners through the same `OwnedElsewhere` equality and so follows the sweep. The exploration did establish this through `pairteardown/originseams.go`.
   - Class: consistency.

**Goal points of the entry**

- A machine-wide config file that lym reads gives the machine's name: met (D1, D3).
- `osl-1046` reads `osl46` and `neve` keeps `neve`: met (D3, D4 and the acceptance criteria).
- A machine with no value keeps the hostname rule: met (D3).
- The task runs before the Slack channel-name work: met, since the record changes no Slack code and states why a rename is safe there.
- Owners written under the old name are not orphaned: met (D6, D7).
- The cross-machine sender address and the Slack channel names: met (D4, Constraints).
- The pair teardown seam: met in substance but not stated (finding 2).
- Where the file lives (open question): answered (D1).
- How a machine already written into owners migrates (open question): answered (D5, D6).

**Entries, peers and parent commits checked**

- `slack-machine-bridge` (F3): no conflict. D4 excludes `:` from names, which fits the `<machine>::` address format, and the record puts the bridge out of scope.
- Peer `terminal-titles-hub-project`: it also edits `pattern/PATTERN-leaf-packages.md` (adding `consoletitle`). That is a shared bookkeeping overlap, so no finding.
- Peer `testmap-verb`: no conflict. Its paths include `internal/hubgeom/hubgeom.go` and `internal/lymdirs/dirs.go`, which the record does not touch.
- Peer `raddle-perch-mode`: no conflict. Its only `internal/testkit` change is to `envkit`, while D8 changes `tmuxkit`.
- Parent commits: `lym loom peers` lists none.

**Decision notes**

The board shows no `Decision for machine-name-config:` note, and the record has no `Added after Discussion` entries.

**Tool calls**

- `lym loom peers`: my context had the board list and this task's entry, but not the peer runs or parent commits.
