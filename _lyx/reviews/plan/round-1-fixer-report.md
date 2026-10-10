# Plan round 1: fixer report

Every finding was validated against the code and the decision record, and every one was fixed.
`lyx loom validate-plan` reports `ok: true` after the edits, with informational `create-new-unit` notes only.

## Fixed

### F1: persistent flags make `burler` and `webster` invocable

Confirmed: `internal/burlercli/cli.go:157-159` and `internal/webstercli/cli.go:280-284` declare visible persistent flags on `GroupRunE` groups, and cobra's `LocalFlags()` includes a command's own persistent flags.
The other groups with persistent flags (`board`'s `--board-path`, fabric's `--weft-path`/`--warp-path`) hide them, so only these two were affected.

- Card 1: `Invocable` and the leaf flag run now read `LocalNonPersistentFlags` minus hidden ones, and the card states that a group's own persistent flags never make it invocable, naming the `burler` and `webster` cases, while `config`'s `--print`/`--set` still qualify it.
  `index_test.go` gains a row: a group whose only flags are its own persistent ones renders in group form, with those flags in no line.
- Card 16: the audience subtests gain a passing row for a `GroupRunE` group whose only flags are persistent, which is not invocable and carries none.
- Cards 4 and 8 state that the `burler` and `webster` groups carry no audience, and why.

### F2: `fabric clone` is over the `Long` cap

Confirmed by measuring every command in the installed tree with `lyx <path> --help --json | jq -r .long | wc -c`: `fabric clone` is 4,187 bytes.
The pages over 2,000 bytes are fabric `clone`, `cleanup`, `remove` and `merge-in`, the `loom` group, loom `resume` and `start` (and the root alias), the `webster` group, and webster `reset` and `validate`.
The three `status` verbs under `batten`, `loom` and `shed` answer `--help --json` with plain text; their whole plain help is under 2,000 bytes.

- Card 6 adds `clone` to its trim set.
- Cards 3 to 9 now define the trim set by measurement before editing (card 9 after its own edits), with the command to run, and record the set as it stood when the plan was written.
  Cards 3, 5 and 9 previously said nothing about D9, so each gains that sentence; card 3 also notes that the `status` verbs' plain help is measured, since their JSON help does not parse.

### F3: the "Watcher index binary" decision is neither carried nor reconciled

Confirmed: the record's added decision ("renders the index from its own recorded path", declining launch-path plumbing, and pointing at an Open risks entry that does not exist) contradicts Scope "In", D4, the `PATTERN-pane-binary-resolution` constraint and the acceptance criteria, all of which require the strand-recorded path.

- `00-overview.md` gains the Shared Decision `watcher-index-binary-reading`, applying to cards 13 and 14.
  It reads "its own recorded path" as the path recorded on the orch strand, and gives the reason: four sections of the record against one.
  It also states that recording the path at every launch and relaunch removes the mismatch the added decision accepted.
  Cards 13 and 14 stand unchanged.

### F4: cards 6 and 8 duplicate card 16's audience check

- Card 6 drops the `TestCommand_VerbDeclarations` audience assertion and `internal/fabriccli/help_test.go` from its `Edit:` list, which only that assertion justified, since the test asserts no `Use` string.
- Card 8 drops the audience assertion from `TestCommand_LongStringsHaveNoStaleBatchLanguage` and `internal/webstercli/cli_test.go` from its `Edit:` list; that test walks `Long` text for forbidden words and passes over the trimmed text unchanged.
- Both cards now say, as cards 4, 5, 7 and 9 do, that card 16's tree walk covers the form, audience annotations included.

### F5: how a runnable group prints

- Card 1 now states the rule: a runnable group prints in leaf form when its own annotation matches, in group form when only a descendant prints, and not at all otherwise, so a printed child never nests under a missing parent line.
  It cites D2's group rule, read with D3's runnable group counting as a leaf.
- `index_test.go` gains the row: a runnable group outside the audience renders in group form above a child that is in it.

## Disputed

None.

## Deferred

None.

## Constraint conflicts

The decision record contradicts itself on the watcher's index binary (F3) and on `startAutoReload`'s failure handling: D4 and the acceptance criteria say it goes idle and never fails a tick, while the added "Missing-binary bound at auto-reload" decision keeps the tick error.
Card 14 already follows the added decision on the second point and cites it, and the review raised no finding there, so it is unchanged.
