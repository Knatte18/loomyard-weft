# Card 3 — shed-verbs-audience

**Edit:**
- `internal/shedverbs#VerbText`
- `internal/shedverbs#runCmd`
- `internal/shedverbs#stepCmd`
- `internal/shedverbs#statusCmd`
- `internal/shedverbs#pauseCmd`
- `internal/shedverbs#gotoCmd`
- `internal/shedverbs/goto_test.go`
- `internal/battencli#battenVerbTexts`
- `internal/battencli#Command`
- `internal/shedcli#shedVerbTexts`
- `internal/shedcli#Command`
- `internal/shedcli#newSeedCommand`

**Uses:**
- `internal/clihelp#AudienceAnnotation`
- `internal/clihelp#AudienceOperator`
- `internal/clihelp#AudienceRole`
- `internal/clihelp#AudienceInternal`

**Intent:** Give the shared shed verb bodies their audience per mount (D3) and bring the `batten` and `shed` mounts to the D6 form.
`VerbText` gains an `Audience` field, and each of the five builders sets `Annotations[clihelp.AudienceAnnotation]` to its text's value, so a body mounted under `loom`, `shed` and `batten` takes a different audience at each mount with no second registry; `goto_test.go` adds one assertion that a built verb carries the audience its text names.
`battenVerbTexts` and `shedVerbTexts` set `Audience` on all five verbs (`operator` for both mounts: the orch runs `lyx batten run|status|pause|goto` and `lyx shed goto` by hand, per the orch role file) and rewrite `goto`'s `Use` to `goto [<run-id>]`, since `--to` is not a required flag in cobra's sense and the verb's own refusal already names it; the flag stays in `Flags:` and the index lists it in the leaf's flag run.
`battencli.Command` and `shedcli.Command` annotate nothing on their non-runnable groups and keep their `Short`s, which already start lower-case; `newSeedCommand` carries `operator` and keeps `seed <run-id> --recipe <name>`, whose flag is marked required.
Any test asserting an old `Use` string, found by grepping the three packages for `--to <producer>`, joins the change.
The D9 cap is checked by measurement: the implementer measures every `batten` and `shed` page's `Long` (`lyx <path> --help --json | jq -r .long | wc -c`; the `status` verbs answer `--help --json` with plain text, so their plain `--help` output, an upper bound on their `Long`, is measured instead) and trims every page over 2,000 bytes; when this plan was written none was over.

**ImpactSummary:** One struct field and five annotation writes in shedverbs, two text tables and three command builders in its mounts; help text changes, no verb behavior changes.
