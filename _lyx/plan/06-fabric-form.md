# Card 6 — fabric-form

**Create:**
- `internal/fabriccli/doc.go`

**Edit:**
- `internal/fabriccli/fabric.go`
- `internal/fabriccli#addMergeVerbs`
- `internal/fabriccli#addWeftVerbs`
- `internal/fabriccli/help_test.go`

**Uses:**
- `plan:internal/clihelp#AudienceAnnotation`
- `plan:internal/clihelp#AudienceOperator`
- `plan:internal/clihelp#AudienceRole`
- `plan:internal/clihelp#AudienceInternal`

**Intent:** Bring `fabric` to the D3, D6, D8 and D9 form.
`Use` rewrites, flags leaving for `Flags:`: `clone <weft-url> [<warp-url>]`, `remove <slug>`, `prune`, `cleanup`, `merge [<branch>]` (with `--continue` and `--abort` explained in `Long` as the two flag-only forms); `checkout [branch]` becomes `checkout [<branch>]`; `merge-stage <path>...`, `merge-in <branch>`, `diff <since-warp-sha>`, `add <slug>` and `shortname [<shortname>]` already fit.
The group's `Short` ("unified warp↔weft git-coordination") is reworded to say when to reach for it, in lower case, as D8 names `fabric` among the group `Short`s to rewrite.
`Long` texts over the cap are trimmed to what the verb does, when to use it, its arguments and flags and one or two examples: `cleanup`, `remove` and `merge-in` today; their design rationale and refusal narratives move into the new `internal/fabriccli/doc.go`, which also takes over the package comment now at the top of `fabric.go`, so the package has one package doc.
Each trimmed page's refusals are checked to still name their way forward in the refusal message itself.
Audiences: every fabric verb is `operator` except `merge-stage`, which is `role` (the operator's named example: the Publish and Finalize producers' conflict-resolution step).
`help_test.go`'s `TestCommand_VerbDeclarations` gains an assertion that every verb carries an audience from the closed set, which is the card's own coverage of the annotation; the `Short` and `Use` rewrites are covered by card 16's tree walk.
Any test asserting an old `Use` string, found by grepping `internal/fabriccli` for `[--force]`, `[--apply]`, `--continue |` and `[--shortname`, joins the card.

**ImpactSummary:** The whole fabric command file plus the merge and weft verb builders change help text and annotations, and a package doc file appears; no runtime behavior changes.
