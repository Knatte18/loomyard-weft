# The gate fixer loads only the language skills, not prose or code-quality

`gate-fixer-no-prose-skills` · Note · bug, gate, loom

## Seen

The gate fixer, the cold session lym spawns after a failed gate, loads only the hub's language skills, `scribe:golang-comments` and `scribe:golang-testing` in this hub.
`internal/loomcli/gatefixer.go` and `internal/gatecli/detachfixer.go` pass `languageconfig.AppendSkills(nil, ...)`, so it gets no `scribe:prose` or `scribe:code-quality`.
Every other code role starts from `scribe:prose`, `scribe:code-quality` and `scribe:testing`: the darn writer (`darnSkills`), the Webster roles (`roleSkills`), the Burler seats, the merge fixer and the conflict session.
In the transcripts of `creel-cross-hub-build`, `python-guard-gives-no-reason` and `drift-halts-bundle`, gate fixers loaded the two Go skills and nothing else, and edited Go files (2 to 20 edits a session).

## Idea

Give the gate fixer the same own list as the other code roles.
