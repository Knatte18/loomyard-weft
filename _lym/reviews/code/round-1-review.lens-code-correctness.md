```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran the old and new rendered hooks on concrete commands and checked the regex branches step by step.

**Branches run**
- `commandSeparatorPattern` and `topLevelReachPattern`: same behaviour as the old inline group (the heredoc branch is unchanged). Ran `cat <<'EOF' | bash -c 'rm $d'\nfoo\nEOF`: denied by computed-rm. Correct.
- `commandWordPrefixPattern` keywords: `if true; then rm $x; fi` is denied; `echo do rm $x` is allowed in both old and new. Correct, and a keyword after a plain command word adds no deny.
- `shellDashCPattern`:
  - `bash -lc`, `bash -e -c` and `/bin/sh -c` after `&&` are denied, as the test rows expect. `time bash -c 'rm $x'` is denied too.
  - `ssh -c` is allowed.
  - `bash -o pipefail -c 'rm $D'` passes. That is a miss, within the guardrail standard. Correct.
- `singleQuotedScriptUnitPattern`:
  - The Seen loop `sh -c 'd=$(mktemp -d); for i in 1 2; do mkdir "$d/x" & rm -rf "$d/x"; done'` was allowed before and is denied now (computed-rm). Correct.
  - `sh -c 'echo $(rm $x)'` is denied. Correct.
- `doubleQuotedScriptUnitPattern`: `sh -c "cd x; LYM_GATE_SLOT=1 lym y"` is denied. `bash -c "echo \"a;b\"; rm $D"` passes: a JSON-escaped `\"` inside a double-quoted script ends the scan. That is a miss of the already documented misread kind. Correct within the guardrail.
- `shellScriptReachPattern` stopping at the script's closing quote: it holds for command position. The argument-word patterns that come after it do not stop there (finding 1).
- `redirectWritePattern` script branch:
  - `bash -lc 'cd x && sed -i s/a/b/ f.go'` is denied.
  - `bash -c 'echo "a > b.md"'` is allowed, so a quoted `>` inside the script is no match.
  - `sh -c 'cat x 2>&1 > out.log'` is allowed (fd duplication and a non-source target).
  - `bash -c 'ls' && ls > 'x y.go'` is allowed, so a quoted target is still no match. Correct.
- `rawGoCommandPattern` fourth alternative: `sh -c "cd x; LYM_GATE_SLOT=1 lym y"` is denied. `sh -c 'printf %s "$LYM_STRAND_NAME"'` passes, as the doc's "outside a quoted string of the script" says. Correct.
- `rawGoCommandPattern` with `'` added to the word-end class: `go test ./x -run 'Foo all'` was allowed before and is denied now (finding 2).
- Cost: a script command of about 9,000 characters takes 32 ms on the new hooks against 26 ms on the old. No backtracking blowup.

**Findings**

1. MEDIUM, design. `internal/shellguard/rows.go:53-56` (`shellScriptReachPattern`), with the argument-word rows built on `commandPositionPattern`.
   - Once a row reaches a command word inside a single- or double-quoted script, its `shellArgumentPattern` words read the script's closing quote as the opening of a quoted argument. They then run past the script's end, across `&&` and `;`, into later commands.
   - The diff brings in new false denies for legitimate commands. Each of these was allowed by the old hooks and is denied by the new ones:
     - `bash -c 'claude --version' && echo 'run -p'` → headless-claude
     - `bash -c "claude --version" && echo "ls -p"` → headless-claude
     - `bash -c 'sed -n 1p x' ; bash -c 'grep -i foo y'` → shell-edit
     - `bash -c 'lym adopt b' && echo 'x --take-hold'` → adopt-take-hold
   - The row comments in this diff list no such false deny.
   - Evidence: `rows.go:168` (`shellArgumentPattern` admits `'[^']*'` and `\\"…\\"` as part of a word), `rows.go:73` (`commandPositionPattern`), and the probe output.

2. LOW, design. `internal/shellguard/rows.go:407` (`rawGoCommandPattern`, first alternative): the new `'` in the word-end class.
   - At top level, `go test ./x -run 'Foo all'` changed from allowed to denied.
   - The existing comment (rows.go:405) already admits false denies inside a quoted `-run` argument. Still, the widening reaches top-level commands and not only scripts.
   - Evidence: the probe output and diff line 107.

**Tool calls**
- One Bash call ran the new hooks from `settings-golden.json` and the old ones from `git show 7346d6a8:` over the probe commands, with a timing check. My context held the patterns but no execution of them.
