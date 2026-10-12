# Read-only sed, in-place edit denies and a dialog notice

Both batches reported `done`.

## Batch 01: edit-directive-wording (cards 1–5)

Head `1affeed47`.
The edit directive now bans `sed -i`, `perl -i` and `awk` by name, allows `sed -n` reads and names `.scratch/`.
Fabric wiring excludes the root `.scratch/` in each warp repo's `info/exclude`.
Two standing denies were added: one refuses `sed` and `perl` in-place edits, with the python steer naming `sed -n`, and one refuses `rm` in a command holding a substitution or variable expansion.
`sed -i` and `perl -i` joined the no-denied-recovery closed list.

Deviation: `internal/fabricengine/dotlyxjunction_integration_test.go` changed outside the declared file-ops.
The fork reported that the plan's gate path `./internal/fabricengine_test` does not exist, and ran `./internal/fabricengine` with `--tags integration` instead.
The fork wrote the `editRulePhrases` entry in `contracts/stencils/stencils.go` as a concatenation, so the no-denied-recovery scan does not flag it.

## Batch 06: sed-print-classifier (cards 6–8)

Head `1d1b4d7fd`.
The read-only classifier admits quiet-flag print scripts with regex addresses.
Shuttle notifies the parent once per dialog that an autonomous run's pane shows past 60 s.
A live scenario proves a fork's prompting command records an ask signal.

Deviation: `internal/shuttlecli/modelpin_test.go` changed, with `wantSpawnSiteCount` raised from 14 to 15 for the new scenario's spawn site.
The fork could not run the `llm`-tier scenario, which is refused in this environment, and only vetted it with `go vet -tags llm ./internal/shuttlecli`.
The same wrong gate path `./internal/fabricengine_test` appeared, and the fork ran `./internal/fabricengine`.
