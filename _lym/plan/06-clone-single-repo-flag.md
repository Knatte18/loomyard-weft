# Card 6 — clone-single-repo-flag

**Edit:**
- `internal/fabriccli#Command`
- `internal/fabriccli#runCloneWithReset` -> `func runCloneWithReset(ctx context.Context, out io.Writer, args []string, reset bool, subpath string, forceBootstrap bool, into string, shortname string, singleRepo bool) int`
- `internal/fabriccli#CloneAndWire`
- `internal/fabriccli_test#TestCommand_RefusesBeforeAnyHubWork`

**Extends:**
- `internal/fabriccli/doc.go`

**Uses:**
- `plan:internal/fabricengine#PushCloneBoard`
- `plan:internal/fabricengine#AbandonSeededClone`
- `internal/fabricengine#CloneOptions`

**Intent:**
Wire D3's clone into the CLI.

`Command`'s clone subcommand gains `--single-repo` (bool, default false) and passes it to `runCloneWithReset`, which sets `CloneOptions.SingleRepo`.
The usage string gains `[--single-repo]`, and the Long help gains a paragraph:
one URL is the code repository; `--single-repo` seeds a one-repository hub whose records and board live under origin's `refs/lym/`;
a plain one-URL clone of an origin holding that board joins it; the flag is refused for an origin that already holds one, a records repository, a second URL, and an origin left by the old `lym-bolt` layout.
Bound: the flag reaches only `CloneHub`'s refusals and seed path (card 5), which write only under `refs/lym/` on an origin with no board.

`CloneAndWire` replaces its board push, `b.PushRecorded(SyncOptions{}, rec)`, with `fabricengine.PushCloneBoard(res, rec)` at the same step:
after the Bolt commit of `.lym-warp`, `.lym-shortname` and the hub-wide config, before the junction wiring.
It reads no layout; the verb pushes a two-repository clone's and a join's board as before and a `--single-repo` result's seed (D3).
Its doc comment names the step.
A seed push that fails has already removed the hub, so the error returns with nothing left to reconcile.
Every failure between `CloneHub` and `PushCloneBoard` (the hub-wide reconcile, the sparse-marker write, the Bolt commit) returns through `fabricengine.AbandonSeededClone(res, rec, err)`,
which removes a `--single-repo` hub and names the same clone again as the way forward, so `lym fabric reconcile` never pushes an unseeded board unleased;
a two-repository clone's failure there returns unchanged, its hub left for `lym fabric reconcile` as today.

`TestCommand_RefusesBeforeAnyHubWork` gains a row: `clone --single-repo <url> <url>` fails before any hub work, with an envelope naming the one-URL way forward.
The other refusals and the seed path are tested over `hubforge` in card 7.
`doc.go` gains the flag and the push step.

**ImpactSummary:** One flag and one call swap in fabriccli; the clone verb is the only flag reader and `CloneAndWire`'s callers (`runCloneWithReset`, `CloneAndWireFromLocalBares`, hubforge) see the same board push for every two-repository clone.
