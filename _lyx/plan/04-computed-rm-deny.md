# Card 4 — computed-rm-deny

**Create:**
- `internal/shuttleengine/claudeengine#steerComputedRmDeny`
- `internal/shuttleengine/claudeengine#noticeComputedRmDeny`
- `internal/shuttleengine/claudeengine#rmCommandPattern`
- `internal/shuttleengine/claudeengine#shellExpansionPattern`
- `internal/shuttleengine/claudeengine#TestComputedRmDenyHook_DeniesRmOnComputedPathsOnly`

**Edit:**
- `internal/shuttleengine/claudeengine#standingDenies`
- `internal/shuttleengine/claudeengine#TestBuildSettings_DenyToggleMatrix`
- `internal/shuttleengine/claudeengine#TestBuildSettings_AgentAndBashHooks`
- `internal/shuttleengine/claudeengine#TestBuildDenyNotice_SentenceContent`
- `internal/shuttleengine/claudeengine/doc.go`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shuttleengine/claudeengine#commandPositionPattern`

**Intent:**
D3: a standing PreToolUse(Bash) deny refuses `rm` on a computed path before Claude Code raises its "statically-unresolvable target" dialog, which shows even in bypass mode and stalls an unwatched run.

`rmCommandPattern` is `commandPositionPattern` followed by `rm` as a whole word, the word end written as `pythonCommandPattern` writes its own.
`shellExpansionPattern` matches a command substitution, `$(` or a backtick, or a shell variable expansion, `$` followed by a letter, a digit, `_`, `{`, `@` or `*`, which takes in the positional parameters, `$@` and `$*`; the head shows the dollar half, and the backtick joins it as an alternative.
Neither holds a single quote.

The new `standingDenies` row, after card 3's in-place row, matches `Bash` and reads the payload once, as the webster fork guard does: `in=$(cat)`, then a grep for `rmCommandPattern` and a grep for `shellExpansionPattern` over `"$in"`, both required, then the deny JSON, ending `; true`.
So it denies a Bash command with `rm` in command position whose payload also holds an expansion anywhere.
It carries `steerComputedRmDeny` and `noticeComputedRmDeny`, and is installed in every run with no config key.
The steer and notice name the literal-path way forward and not the scratch place, which stays the edit directive's alone, and hold no quote or backslash.
Its row comment states the bound: a guardrail, not a barrier.
It falsely denies an `rm` of a literal path in a command that uses `$` elsewhere, a `grep` alternation naming `rm`, and a matching spelling in another payload field such as the description.
It lets through a removal behind `find -delete`, `xargs rm` with no `$`, `bash -c` or a script, and one through the special parameters it does not list (`$$`, `$?`, `$#`, `$!`).

`TestComputedRmDenyHook_DeniesRmOnComputedPathsOnly`, in `hookcmd_integration_test.go` in the same table shape as card 3's test, pins the deny JSON for `rm -rf "$(cat x)"`, `rm $DIR/f`, `rm $1`, `rm ${D}/f` and an `rm` whose path is a backtick substitution such as `` rm `cat list` ``, and empty output and exit 0 for `rm -f .scratch/a` and `rm a b`.

`TestBuildSettings_DenyToggleMatrix`, `TestBuildSettings_AgentAndBashHooks` and `TestBuildDenyNotice_SentenceContent` take the computed-rm row exactly as card 3 has them take the in-place row: counted among the always-installed denies, set apart from the fork guard, and its notice asserted in every run mode.

`claudeengine`'s `doc.go` gains a paragraph on the computed-rm row: what it denies, why a deny and not a prompt, that every run installs it, and its bound.
`docs/overview.md`'s shuttle row names the computed-rm deny beside the in-place one.
`cmd/lyx/testdata/test-budget.yaml` raises `internal/shuttleengine/claudeengine` by one more for the new test.

**ImpactSummary:** Adds one more always-installed Bash PreToolUse hook and notice sentence to every spawned session, the orch included; the settings and notice tests and the claudeengine budget row follow.
