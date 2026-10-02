# Card 3 — fabric-code-verb

**Create:**
- `internal/fabriccli/code.go`
- `internal/fabriccli/code_integration_test.go`

**Edit:**
- `internal/fabriccli/fabric.go`
- `internal/fabriccli/argsarity_test.go`
- `internal/fabriccli/cli_test.go`
- `cmd/lyx/helptree_test.go`

**Uses:**
- `internal/fabricengine/bolt.go`
- `internal/agentname/agentname.go`

**Intent:**
Add `lyx fabric code [<code>]`, the way a hub bound before this task gets its code.
It runs from any worktree of the hub: it resolves the location through `lyxcwd` and reads the board root through `fabricengine.BoardDir(l.HubPath)`.

With no argument it prints the recorded code (`{"ok":true,"code":"ly"}`), and an uncoded hub refuses naming `lyx fabric code <code>` and the grammar.
With one argument it validates the code with `agentname.ValidateCode`, then:
equal to the record, it is a no-op;
a different record refuses naming the recorded code, since changing it would orphan every name already in use;
no record, it writes it with `fabricengine.WriteCode` and commits and pushes it through `fabricengine.NewBolt(boardDir)`, as `CloneAndWire` commits the anchor.
The writing form is a mutating fabric verb, so its envelope carries `mutations` and `partial` through the package's existing `okWithRecord`/`errWithRecord` helpers (Mutation Record Invariant), recording the file write and the commit.

The command carries a `Short`, takes at most one positional argument (`cobra.MaximumNArgs(1)`), and checks `clihelp.ShouldAbort` first (CLI / Cobra Invariant).
`argsarity_test.go`'s table gains `"code": {"a", "b"}`, and both help-tree listings (`fabriccli`'s own and `cmd/lyx`'s) gain `code`.

`code_integration_test.go`, on a `hubforge` hub: the verb prints the fixture's code;
on a hub whose record is removed and committed away, it records and commits a new code, visible as `.lyx-code` at weft:main's tip;
re-running it with the same code is a no-op with no new commit;
a different code refuses naming the recorded one;
the no-argument form on an uncoded hub refuses naming `lyx fabric code <code>`.

**ImpactSummary:** Adds one verb to the fabric subtree and its two help-tree listings; no existing verb changes.

**Commit:** 3: fabric code: print or record the hub's short code
