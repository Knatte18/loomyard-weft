# Card 15 — fabricengine-untag

**Edit:**
- `internal/fabricengine#`

**Intent:**
Run the untag procedure (Shared Decisions, `untag-procedure`) on `internal/fabricengine`'s `integration` files.

fabricengine is lyx's sole git-coordination module (`PATTERN-fabric-git`), so most of its `integration` tests pin real git behavior and stay tagged.
The ones to move are those whose assertion is about fabricengine's own vocabulary, error text, record shapes, path computation or gate decisions and that build a hub only to reach that code: for example the merge-vocabulary and merge-error tests, the correspondence-index (`corrindex`) round trips, and gate request-shape checks, which the committed report already shows as covered by other tests.
A moved test runs against an in-memory or temp-dir fixture; where the engine reaches git or the filesystem link layer only through a package-level value, add the seam as a struct field or option with production passing today's value (D6).
The destruction chokepoint (`destroy.go`, `PATTERN-fabric-destruction-chokepoint`) and the `os.Root` write containment (`PATTERN-fabric-write-containment`) keep their order and their checks; a seam never bypasses them.

Then merge scenario families (Shared Decisions, `scenario-merge`).
The package's cost is the `hubforge` hub or pair each integration test builds; group tests that build the same fixture (one hub, one pair, one adopted pair) and differ only in the verb sequence and checks after it.
A family whose tests each need a pristine hub, such as one that destroys or rolls back what it built, stays separate.
The commit message names each merged family and how many fixture builds it removed.
A test that touches process-global state (its comment names it) carries that comment onto its scenario.

`TestMain` keeps `tmuxkit.Main` and `gitkit.HermeticGitEnv()`.

**ImpactSummary:** Test files of one package, plus a seam field in its production code only where a moved test needs one; production passes today's value, and no fabric verb changes behavior.

**Verify:**
go vet -tags tmux ./internal/fabricengine/
go vet -tags llm ./internal/fabricengine/
