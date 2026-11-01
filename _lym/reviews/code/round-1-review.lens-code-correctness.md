```yaml
lens: code-correctness
outcome: reported
```

**Task:** code-correctness lens on the new `shell-edit` row (patterns, texts, row wiring), by hand-tracing concrete inputs. I ran no code, because writing a probe was ruled out for a read-only fork.

**Branches and cases**

| Branch | Case | Result |
|---|---|---|
| `inPlaceEditCommandPattern` sed | `sed -i 's/a/b/' f`, `-i.bak`, `-Ei`, `"s/a/b/" -i f`, `--in-place`, `cd x && sed -i …` | correct, denied |
| sed false positives | `sed -n 1p f \| grep -i x`, `sed -n '1,20p' f.go`, `grep -rn 'sed -i' internal`, `sed 's/a/b/' f.go`, `s/-i/x/` in an unquoted word | correct, pass. `shellArgumentPattern` stops at `\|;&()<>` and a backslash, and quoted text is consumed whole. |
| perl | `perl -pi -e …`, `-0777 -i -pe`, `-i` after other args | correct, denied. `-Mstrict -ne` passes because `M` is outside the cluster class. |
| perl false positive | `perl script.pl -i foo` | denied, although `-i` is the script's own argument (trivial, rare) |
| tee | `tee -a docs/a.md`, a pipe into `tee x.go`; `tee .scratch/gate.log`, `tee /dev/null` | correct |
| `redirectWritePattern` | `echo x > docs/a.md`, `>>f.go` with no space, `cd x\nprintf y > go.mod`, `cat > f.go <<EOF`, `cat <<'EOF' >> docs/x.md` | correct, denied |
| redirect passes | `> /dev/null 2>&1`, `> .scratch/board.json`, `"x > y.go"` in quotes, heredoc body, `f.go.orig` | correct, pass. `2>&1` cannot match because `&` is outside the path class and the target needs `.ext`. |
| redirect with quoted target | `> "f.go"` | passes, as documented |
| `sourceFilePattern` word end | `x.go.orig` | passes, correct |
| Texts (`editToolsRule`, steers, notices) | forbidden-character check at `init`, one notice carrying the rule | correct. The text has no quote or backslash. Only the `shell-edit` notice carries the rule, so the system prompt always states it even when `deny_python` is off. |
| Row wiring | `SwitchNone`, placed in `rowsAfterRawGo` | correct. A command matching two rows gets whichever row's steer comes first, and both steers carry the same edit rule. |

**Findings**

1. **LOW, design** — `internal/shellguard/rows.go:110-115` (inherits `rows.go:76`). The commonest looping spellings of `sed -i` pass the guard:
   - `for f in …; do sed -i …`, `then sed -i`, and `find … -exec sed -i`.
   - `xargs -0 sed -i` or `xargs -I{} sed -i`. The wrapper list takes `xargs` only when `sed` follows it directly, so any `xargs` flag defeats it.

   Reason: `do`, `then`, `{` and `-exec` are not command-position separators, and `xargs` flags are not skipped. These are the shapes the board entry's 74-edit evidence most plausibly uses. The row comment at `rows.go:253-256` lists bypasses but not these, so the "guardrail, not barrier" claim is incomplete. Fix: either add them to the row comment and the doc comment at `claudeengine/doc.go:265`, or extend the prefix with `(do|then|else)[[:space:]]+` and `xargs` flags.

2. **LOW, design** — `internal/shellguard/rows.go:106-108`. `sourceFilePattern` says a captured-output target such as `.json`, `.txt` or `.log` passes. But output captured to `.yaml`, `.md`, `.sh` or `.sum` is refused, for example `lym config show > .scratch/c.yaml`. The edit directive also permits scratch files under `.scratch/`, and a scratch `.sh` or `.md` written by heredoc or redirect is denied. The deny's steer says to use Write, so it recovers rather than stalls. Fix: either exempt a target under `.scratch/` in `redirectWritePattern` and the `tee` branch, or state in the comment and directive that scratch files with these extensions also go through Write.

I found no wrong result, error-path or inconsistent-state defect in `checkTexts`, the text constants or the row wiring.

**Tool calls (3 commands in 1 Bash call)**
- `sed -n 56,80p rows.go`: the full `commandPositionPattern`, because the diff showed only its uses.
- `grep -n "sed -i\|perl -i" ../fabricengine/refscanner_test.go`: line 175 is an unrelated scanner case that expects `false`, so it is not affected.
- `grep -n "Forms" *.go`: the `Row.Forms` contract at `rows.go:37`, which the diff did not show.
