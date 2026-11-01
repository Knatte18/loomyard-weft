```yaml
lens: forced-edits
outcome: reported
```

Lens: forced-edits — what each card's change forces outside the members it names.

**Findings**

- **BLOCKING** — Card 4 / Card 7 (D9 guard): `internal/fabricengine/livestate_manifest_test.go:28` has a comment spelling `_board`.
  It sits in the file's leading block, before `type Kind` at :33.
  Card 4 lists no member of this file, and card 7 exempts only `livestate_doc_test.go`.
  So card 7's `TestEnforcement_BoltDirNameDeclaredOnce` fails on it.
  Fix: list the file's members (or the member whose doc holds :28) on card 4.

- **MEDIUM** — Card 9 (D6, "the move's result carries it too, `Fabric.Pull`'s result"): `lym fabric pull` builds its output map field by field.
  At `internal/fabriccli/weft_verbs.go:476-477` it copies `result.WarpAdvanced` and `result.NewWarpHEAD` by hand.
  The new `PullResult.SeamWarning` therefore never reaches the foreground pull's printed result.
  No card lists the enclosing `fabriccli` member that builds that map.

- **LOW** — Card 8: `internal/fabricengine#TestConfigTemplate` is edited to assert a third key.
  Its doc comment and its `//testtiming:keep` reason still say "both keys" (`internal/fabricengine/template_test.go`, reason line above `func TestConfigTemplate`).
  The card's Intent does not say to update them, so they go stale.

- **LOW** — Card 3, rubric "Member glyphs on an `Edit` card": every `**Edit:**` target is a non-test `.go` file path.
  The Intent names specific symbols, for example `weftEndedByOrigin`'s hand join in `pull.go` and `codebinding.go`'s re-point refusal message.
  List those as member glyphs: `internal/fabricengine#Fabric.weftEndedByOrigin` and the `codebinding.go` member that builds the message.

- **LOW** — Card 2 (D9 "a comment names the Bolt checkout"): `docs/shared-libs/lymcwd.md` describes lymcwd's private `_board` copy, the `boardDirName` that card 2 deletes.
  No card updates that doc.
  The card 7 guard does not scan `docs/`, so this is stale prose only.

**Calls made**

1. Grep for `_board` across Go files, for the old replay-seam names, for `appendPromptText` callers and for set-aside tag readers. My context lacked the full caller and file set for the D9 sweep and the D7 seam.
2. Grep for the enclosing members of ambiguous `_board` hits, plus hubforge's clone path and possible name clashes with the new types and fields. This was to check that cards 4 and 6 cover each hit and that card 9's `NewHub` → `CloneAndWire` premise holds.
3. Grep for the remaining header-level `_board` hits, for wiring tests on fixtures without config, and for the code that prints `PullResult`. This was to settle which header hits no card covers, and to find who prints `PullResult` for card 9.
