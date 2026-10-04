# Card 9 — constraints-repoint-internal-a-l

**Edit:**
- `internal/batcher/config.go`
- `internal/battencli/testmain_test.go`
- `internal/battenrecipe/doc.go`
- `internal/battenshed/doc.go`
- `internal/boardcli/testmain_test.go`
- `internal/boardengine/boardtest/testmain_test.go`
- `internal/boardengine/testmain_test.go`
- `internal/buildinfo/doc.go`
- `internal/burlercli/cli_test.go`
- `internal/burlercli/run.go`
- `internal/burlercli/testmain_test.go`
- `internal/burlerengine/doc.go`
- `internal/burlerengine/prompt.go`
- `internal/burlerengine/template_test.go`
- `internal/burlerengine/template.yaml`
- `internal/burlerengine/testmain_test.go`
- `internal/cliwire/bannedecl_enforcement_test.go`
- `internal/cliwire/callerset_enforcement_test.go`
- `internal/configcli/testmain_test.go`
- `internal/configengine/config.go`
- `internal/fabriccli/testmain_test.go`
- `internal/fabricengine/ancestors.go`
- `internal/fabricengine/destroy.go`
- `internal/fabricengine/doc.go`
- `internal/fabricengine/merge.go`
- `internal/fabricengine/mergestage_integration_test.go`
- `internal/fabricengine/testmain_test.go`
- `internal/fabricengine/weftgit_exclude_test.go`
- `internal/fabricengine/weftwiring.go`
- `internal/gitexec/gitexec.go`
- `internal/gitexec/testmain_test.go`
- `internal/githubclient/doc.go`
- `internal/gitkit/callerset_enforcement_test.go`
- `internal/gitkit/doc.go`
- `internal/gitkit/reexecguard.go`
- `internal/gitrepo/doc.go`
- `internal/gitrepo/noforceadd_test.go`
- `internal/gitrepo/remote.go`
- `internal/gitrepo/testmain_test.go`
- `internal/hubforge/doc.go`
- `internal/hubforge/testmain_test.go`
- `internal/hubgeom/testmain_integration_test.go`
- `internal/idecli/testmain_test.go`
- `internal/ideengine/testmain_test.go`
- `internal/loggerconfig/config.go`
- `internal/logger/sink_callsite_integration_test.go`
- `internal/loomcli/testmain_test.go`
- `internal/loomshed/doc.go`
- `internal/lyxcwd/docslink_test.go`
- `internal/lyxcwd/enforcement_test.go`
- `internal/lyxcwd/leaf_enforcement_test.go`
- `internal/lyxcwd/testmain_test.go`

**Intent:** Repoint every `CONSTRAINTS.md` (and bare `CONSTRAINTS`) citation in the listed `internal/` packages, `batcher` through `lyxcwd`, to its PATTERN entry name (D7).
The rule and the "use the `entry-map` table only" bound are card 8's: comments, doc.go text, YAML comments and failure-message strings name `PATTERN-<name>`, and test function names stay unchanged.
`internal/burlerengine/prompt.go`'s "CONSTRAINTS-fit" lens wording becomes "PATTERN-fit", and `internal/burlerengine/template.yaml`'s "CONSTRAINTS.md entries" becomes "PATTERN entries".
Grep each reworded string that a test asserts (for example the burler prompt tests), and update those assertions in this card.
`internal/lyxcwd/docslink_test.go` changes only its citation comment here.
Its `../CONSTRAINTS.md#package-naming` allowlist entry is resolved by card 13, which repoints the link and deletes the file.

**ImpactSummary:** Comment, doc and message rewording across the listed packages; one burler prompt lens phrase changes, with its asserting test.
