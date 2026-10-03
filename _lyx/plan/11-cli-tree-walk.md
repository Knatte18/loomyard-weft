# Card 11 — cli-tree-walk

**Edit:**
- `cmd/lyx/helptree_test.go`
- `cmd/lyx/unknown_subcommand_test.go`
- `CONSTRAINTS.md`

**Delete:**
- `cmd/lyx/longlist_test.go`
- `cmd/lyx/drift_test.go`

**Create:**
- `cmd/lyx/clitree_test.go`

**Intent:** Replace the hand-kept help tree with one walk of `newRoot()` (D5).
`clitree_test.go` walks every command under `newRoot()`, skipping cobra's `help` and `completion` subtrees, and asserts: every command has a non-empty `Short`;
for every group (a command with non-hidden children), a bare invocation through the `run()` seam exits 0, emits no `"ok":false` envelope, and lists every non-hidden child;
`<group> bogus` exits 1 with a JSON `ok:false` envelope (through `testkit/envelope`);
and `root.Long` names every mounted module.
The walk runs from a `t.TempDir()` cwd that is not a git repository, so it also covers what the per-CLI `NoGitRepoNeeded` variants asserted.
A group whose bare or `bogus` invocation legitimately differs is a finding to report, not a case to special-case silently.
It replaces `helptree_test.go`'s module and subcommand lists, `unknown_subcommand_test.go`'s hard-coded groups, `longlist_test.go` and `drift_test.go`;
`TestHelpTree_OrchStartListsAdopt` and `TestUpdateCommandRemoved` stay, since each pins one behaviour rather than a list.
`registration_test.go` stays the independent mount oracle and is not touched.
Before deleting anything, check its comments: a test citing a finding or crucible round keeps its assertion in the walk or stays (D21).

What this lets through, bounded per D5: the derived listing no longer notices a verb being deleted, which that verb's own behaviour tests must catch, and `go test` of one CLI package no longer checks `Short`, which `go test ./cmd/lyx` does and the plan-level verify runs.

`CONSTRAINTS.md`'s CLI / Cobra Invariant gains an enforcement line: one walk of `newRoot()` in `cmd/lyx` enforces `Short`, bare-group listing and unknown-subcommand refusal for every command, and `cmd/lyx/registration_test.go` is the oracle that every `Command()` package is mounted;
per-CLI copies are no longer an obligation.
Deleted test names go in the commit body for the count accounting.

**ImpactSummary:** `cmd/lyx` tests only plus one `CONSTRAINTS.md` line; the help-tree and drift tests fold into one derived walk.
