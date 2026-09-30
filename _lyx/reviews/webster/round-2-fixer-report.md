# Webster-Review round 2 — fixer report

Every finding in `round-2-review.md` is fixed, one commit per finding.
Every commit this round changes comment lines only: `git diff -U0 e29b05465..HEAD` holds no added or removed line outside a `//` comment.
`go build ./...`, `go vet -tags smoke ./...` and `go test -tags integration ./...` pass after the last commit.

## Fixed

- **F1 (LOW)**: `1b39ce2a9`.
  The `NewPlanDirRotator` result sentence no longer breaks after its colon, and the break before "so the new session knows" stays.
- **F2 (LOW)**: `0dd472445`.
  The `seedPriorPlanStencils` doc is one line.
  The inline comment in `MissingStencilWithPriorFilesIsAnErrorAndMovesNothing` breaks only before "so the failed attempt".
- **F3 (LOW)**: `2432b1125`.
  The `PrepareFreshSpawnErrorNeitherArchivesNorSpawns` doc breaks only before "and nothing downstream of it runs", and the `PrepareFreshSpawnAmendment` doc is one line.
- **F4 (LOW)**: `44dbec91e`.
  The `TestPlanWriteEntry_CallAppendsPriorPlanBlock` doc is one line.
- **F5 (LOW)**: `b8095022d`.
  The `TestPrepare_AppendSystemPromptMatchesDenyNotice` doc is one line.
- **F6 (LOW)**: `24ce95ae2`.
  The edited `seedBouncerStencils` first sentence is one line.
  "shedadapters.NewBouncer", which had shared the sentence's last line, now starts the next legacy line, so no line holds one word, and the later sentences keep their legacy wrap.
- **F7 (LOW)**: `14701cf74`.
  The `fork_mode_real_notice_pwsh` inline comment is one line.
- **F8 (NIT)**: `c94bd0b42`.
  The long new sentences in `command.go` (`buildLaunchCmd` doc), `settings.go` (`noticeAgentDeny`), `priorplan.go` and `planwrite.go` (two rotator doc sentences) now break at their independent-clause boundaries.

## Deferred

None.
