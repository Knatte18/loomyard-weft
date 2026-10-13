# Card 4 — `lyx loom peers` gives each run's position in the embedded loom recipe

**Create:**
- `plan:internal/loomcli#recipeRow` -> `func recipeRow(status *string, rows []string) int`

**Edit:**
- `internal/loomcli#peerLine`
- `internal/loomcli#peersDeps`
- `internal/loomcli#peersVerb`
- `internal/loomcli#hubPeersDeps`
- `internal/loomcli#loomCLI.peersCmd`
- `internal/loomcli#peersFakeTasks`
- `internal/loomcli#peersFakeDeps`
- `internal/loomcli#TestPeersVerb`
- `internal/loomcli#TestHubPeersDeps_OverRealHub`
- `internal/loomcli/doc.go`

**Uses:**
- `internal/loomrecipe#RowNames`

**Intent:**
D5's Go side: the alignment agent reads the landing order from `lyx loom peers`, served from the loom recipe the running binary embeds, so a hub with no loomyard checkout has the same source and no stencil restates the rows.

`recipeRow` lives in `internal/loomcli/peers.go`: it takes a board status and the recipe's row names in file order, and returns the 1-based position of the status's producer half (`boardengine.RunStatusProducer`) among them, and 0 when the status is no run status or its producer names no row.
Every peer is read against the loom rows, darn runs included, so a darn run at a row name it shares with loom (Merge-In, Publish, PR-Gate) takes that row's loom position and one at a darn-only row such as Darn-Write reads 0.

`peerLine` gains `Row int` with JSON key `row`.
`peersDeps` gains `loomRows []string`, the loom recipe's row names in file order.
`hubPeersDeps` fills it from `loomrecipe.RowNames(shedrun.RecipeLoom)`, the embedded `recipes.LoomRecipe`; a parse error there is a defect of the binary itself, so it logs one `logger.Warn` and leaves the list empty, and every row then reads 0, an unknown order, rather than refusing the verb.
`peersVerb` sets each line's `Row` from the board task's status, never from the peer's pair, so a peer whose pair is unreadable still gets the board's row.
From a task pair it also adds `own_row`, the caller's own position by the same rule from the caller's board task, 0 when the caller has no board task; from the prime the key is absent, as `parent_commits` is.
The two fields are read-only: they reveal only recipe positions of producers the peers' and the caller's board statuses already name.

`loomCLI.peersCmd`'s Long help names `row` and `own_row`, and `internal/loomcli/doc.go`'s `# Peers` paragraph lists `row` among each line's keys and `own_row` beside `parent_commits`; no verb is added, so `docs/overview.md` is unchanged.

Tests:
- `peersFakeTasks` gains a darn-shaped peer at `blocked · Darn-Write` and one at `awaiting · Publish`.
- `peersFakeDeps` sets `loomRows` to `loomrecipe.RowNames(shedrun.RecipeLoom)`, the embedded rows, so the unit rows run on the real recipe with no file read.
- `TestPeersVerb`'s task-pair subtest asserts `row` for each peer: the 1-based index in those rows of `Plan-Write`, `PR-Gate` and `Publish` for the blocked, awaiting and Publish peers, 0 for `Webster-Write` (no loom row) and for `Darn-Write`, and `own_row` equal to `Plan-Write`'s index for the caller `me`; its wanted slug list gains the two new peers.
  The prime subtest derives its peer count from `runStatusTasks(peersFakeTasks())` instead of the literal four, and asserts `own_row` absent.
- `TestHubPeersDeps_OverRealHub`, over a hubforge hub that holds no loomyard checkout, runs `peersVerb` over the real `hubPeersDeps(callerLocation)` and asserts the `peer` line's `row` and the list's `own_row` equal `Plan-Write`'s 1-based index in `loomrecipe.RowNames(shedrun.RecipeLoom)`, both non-zero, since every seeded task holds `running · Plan-Write`.

**ImpactSummary:** One output line type and the list verb of `lyx loom peers` gain two read-only fields; `peers show`, the board and every other verb are unchanged, and the only reader is the alignment stencil card 5 edits.

**Commit:** 4: Give lyx loom peers each run's recipe row and the caller's own
