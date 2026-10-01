# Webster-Review round 2 — fixer report

Every finding in `round-2-review.md` is fixed, one commit each.
After the last fix, `go build ./...`, `go vet ./...`, `go vet -tags smoke ./...`, `go test ./...` and `go test -tags integration ./...` all pass, and the round's overlay probe classes every shape the review named as the review asks.

| Finding | Commit | Change |
|---|---|---|
| F1 | `118704089` | `grepOpensPager` counts a long option whose name before any `=` is a prefix of `--open-files-in-pager` at least as long as `--op`, and its doc says so; new rows: `--op=`, `--ope=` and bare `--op` are correctness, `--or` and `--only-matching` stay policy. |
| F2 | `eb4047254` | New `steersPrograms` names `PATH`, `HOME`, `XDG_CONFIG_HOME`, `BASH_ENV`, `ENV`, every `LD_` name and every `GIT_` name but `GIT_PAGER`; `readOnlySegment` returns mutating on such an assignment (including the `+=` form) whether it prefixes a program, comes through `env`, or stands as a bare segment. New correctness rows for `GIT_EXTERNAL_DIFF`, `GIT_CONFIG_COUNT/KEY/VALUE`, `env GIT_EXTERNAL_DIFF`, `HOME`, `PATH` (prefixed, bare, `+=`) and `LD_PRELOAD`, and new policy rows for `GIT_PAGER=cat git log` and `f=…; cat $f`; card 44's `env LC_ALL=C grep` row keeps its policy class. |
| F3 | `5fa58d31c` | `skipOptions` also drops the next word when a short cluster's first value letter ends the cluster (new `clusterEndsInValue`), and `time`'s output check counts an `o` anywhere in a single-dash cluster. New correctness rows for `/usr/bin/time -vo`, `/usr/bin/time -vf`, `sudo -nu`, `env -iu` and `xargs -tI`, and policy rows for `sudo -nu root cat`, `xargs -0I{} cat {}`, `/usr/bin/time -v` and `/usr/bin/time --format=%e`. |
| F4 | `226eb3711` | Every Go comment the review listed breaks after its semicolon, across `battencli`, `loomshed`, `shedengine`, `shedverbs`, `webstercli` and `websterengine`. |
| F5 | `a543bf061` | The Shed Verb-Set and Refusal Way-Forward lines in `CONSTRAINTS.md` and the out-of-scope sentence in `refusal-spec.md` break at their semicolons and sentence end. |

## Deferred

Nothing deferred.
