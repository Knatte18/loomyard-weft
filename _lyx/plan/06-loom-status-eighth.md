# Card 6 — loom-status-eighth

**Edit:**
- `internal/loomcli#statusStrandShareDen`
- `internal/loomcli#TestStatusStrandAddSpec`
- `internal/loomcli/doc.go`

**Intent:**
D6: the loom status strand takes at most an eighth of the free height.

- `statusStrandShareDen` becomes 8, so the status strand's add spec declares `ShareNum` 1, `ShareDen` 8 and `MinRows` 3; `statusStrandShareNum` and `statusStrandMinRows` are unchanged.
- The doc comment of the const block holding `statusStrandShareDen` says "at most an eighth of the free height" instead of "a quarter".
- `TestStatusStrandAddSpec`'s expected size becomes `render.Size{Kind: render.SizeContent, ShareNum: 1, ShareDen: 8, MinRows: 3}`, so it fails on the old share.
- `internal/loomcli/doc.go`'s status-strand sentence says "at an eighth share" instead of "at a quarter share".

The checklist's format and its fit are unchanged.
A live status strand from an older build is replaced at the next loom bootstrap, as today, so the new size takes effect then.

**ImpactSummary:** One constant read only by `statusStrandAddSpec`; its one test assertion and two comments move with it, and no other strand's size changes.

**Commit:** 6: loom: the status strand takes at most an eighth of the free height
