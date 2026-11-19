```yaml
lens: fix-proposal
outcome: reported
```

Task: test whether the F14 fix proposal holds up, by building concrete cases where it is wrong or breaks other parts of the artifact.

**F14: proposal fixed as proposed (D1, D3, new acceptance criterion)**

- MEDIUM. The changed D3 text has `--single-repo` make the records store "by `git init` with D1's refspecs and a fetch, never by the two-repository weft clone", and then seed it "as a fresh two-repository bind seeds its weft".
  Problem: the fetch pulls in the leftover records branch that D3 says must never be adopted, and the reused fresh-bind seeding adopts it anyway.
  So a rerun with another `--subpath` fails, and the existing criterion says that rerun succeeds.
  Evidence:
  - A failed earlier attempt leaves `refs/lym/heads/<prime>-weft` on origin, and D3's own text names this case.
  - Under D1's fetch refspec, the new fetch maps that branch to `refs/remotes/origin/<prime>-weft`.
  - The fresh-bind seeding runs `suffixWeftPrimaryBranch`. It checks `refs/remotes/origin/<suffixed>` and, when that ref exists, runs `checkout -b <suffixed> origin/<suffixed>` (`internal/fabricengine/clone.go:515-523`). That is the adopt the unchanged D3 text forbids: "creates the prime's records branch fresh, never from `origin/<prime-branch>-weft`".
  - `CloneHub` then reads `.lym-anchor` from the records store's root and takes the adopt path, `adoptRecordedAnchor`, which refuses a different `--subpath` (`clone.go` anchor block after `ensureBoardWorktree`).
  - The criterion "a rerun of `--single-repo`, with the same or another `--subpath`, replaces the leftover records branch and succeeds" therefore fails.
  - Before this round the store was seeded "as a fresh two-repository bind", which fetched nothing from the namespace, so this round's change introduced the conflict.
  - For `--single-repo`, the fetch brings only refs D3 says to ignore: the board is absent by the precondition, and the leases take the tip the `ls-remote` observed.
  - Suggested direction: drop the fetch from the `--single-repo` path, or state that its seed skips the origin adopt in `suffixWeftPrimaryBranch` and in the anchor read.

No other case I built breaks the proposal:
- Other readers of the board worktree's branch (`originreconcile.go:734`, `hubpull.go:79`, `anchormarker.go:187`) treat it as the board, never as the prime, so the fixed name `main` does not affect them.
- `bornWeftPrimaryBranch` already handles the unborn HEAD a `git init` store starts with (`clone.go:533-547`).

Tool calls:
1. Grep for readers of `BoltDir` and the board's branch. The context showed only `primaryWeftBranch`, and I needed to know whether other code derives the prime from the board's branch.
2. Read the board-branch readers in `originreconcile.go`, `anchormarker.go` and `hubpull.go`. The context lacked their bodies.
3. Read `suffixWeftPrimaryBranch` and `bornWeftPrimaryBranch` in `clone.go`. The context showed only the call site, not the adopt logic a `git init` store's fetch would trigger.
