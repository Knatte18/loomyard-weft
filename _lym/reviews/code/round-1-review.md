---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/shellguard/rows.go:41-42
    summary: The steer text editToolsRule still names awk, cat and sed -n for reading, which the parent decision of 2026-11-01 removed from every steer
    fix: In internal/shellguard/rows.go change editToolsRule so it names Read for reading a file (whole or a span through offset and limit) and grep for searching, and no longer names awk, cat or sed -n; keep it free of quotes and backslashes. Update the sentences that restate the old wording to match, namely the shell-edit and python row comments, internal/shellguard/doc.go, internal/shuttleengine/claudeengine/doc.go and the shuttle entry in docs/overview.md (which says the python deny steer names sed -n among the readers, now removed). Regenerate internal/shuttleengine/claudeengine/testdata/settings-golden.json so the steers carry the new text.
    origin: handler
  - id: F2
    severity: MEDIUM
    class: design
    location: contracts/stencils/edit/edit-directive.md:3-11
    summary: The edit directive still tells agents to search and glance with grep, awk, cat or sed -n and never says it overrides the harness bypass-mode reminder that permits shell edits
    fix: In contracts/stencils/edit/edit-directive.md state that in lym sessions this directive overrides any harness text that permits reading with cat, head or sed -n or changing files with sed, heredocs or short scripts instead of Read, Edit and Write. Replace the line naming grep, awk, cat or sed -n with one naming Read for reading a file and grep for searching. Keep the phrases in editRulePhrases (contracts/stencils/stencils.go) pinned to the new wording, and mirror the change in internal/editdirective/doc.go.
    origin: handler
  - id: F3
    severity: LOW
    class: design
    location: internal/shellguard/rows.go:110-115
    summary: Looped spellings of sed -i pass the guard, namely after do or then, behind find -exec, and behind xargs with a flag, and the row comment does not list them
    fix: Either add these spellings to the guardrail-not-barrier sentence in the shell-edit row comment (rows.go) and in internal/shuttleengine/claudeengine/doc.go, or extend commandPositionPattern's use here to accept do and then as command-position prefixes and to skip xargs flags before sed, perl or tee, with deny rows in rows_integration_test.go.
    origin: lens:code-correctness
  - id: F4
    severity: LOW
    class: consistency
    location: internal/shellguard/rows.go:106-108
    summary: sourceFilePattern lets .json, .txt and .log capture targets pass but refuses output captured into a scratch .yaml, .md, .sh or .sum file, which the directive otherwise allows under .scratch
    fix: Either exempt a target under .scratch/ in redirectWritePattern and the tee branch of inPlaceEditCommandPattern, or state in the sourceFilePattern comment and in edit-directive.md that scratch files with these extensions are written with Write.
    origin: lens:code-correctness
  - id: F5
    severity: LOW
    class: scope
    location: internal/shellguard/rows_integration_test.go:154-178
    summary: Three pattern branches of the shell-edit row have no case, namely the >| clobber redirect, a redirect into a quoted target, and every sourceFilePattern extension but go and md
    fix: In TestRows_PatternsDenyAndAllow add a deny row for echo x >| f.go, a pass row for echo x > "f.go", and deny rows covering the other extensions of sourceFilePattern (one per extension, as table rows of the same test).
    origin: lens:code-tests
  - id: F6
    severity: LOW
    class: consistency
    location: internal/shuttleengine/claudeengine/hookcmd_integration_test.go:237-241
    summary: The hook-command integration test has rows for machine-sleep and computed-rm but none for shell-edit, whose long pattern with single quotes is rendered into the hook command
    fix: Add a deny row for sed -i s/a/b/ f and an allow row for sed -n 1p f for the shell-edit row beside the machine-sleep rows in hookcmd_integration_test.go.
    origin: lens:code-affected-set
---

### [BLOCKING] Steer still names awk, cat and sed -n

**Location:** internal/shellguard/rows.go:41-42
**Issue:** The decision record's `Added after Discussion (parent, 2026-11-01)` entry wins over the board entry and says the steering sentence, `steerPythonDeny` and any steer the new guard prints, no longer names `sed -n` or `awk` for reading, and names Read for reading a file and grep for searching. `editToolsRule` ends with "search and glance at files you do not change with grep, awk, cat or sed -n", and both steers and the shell-edit notice embed it. The golden settings file and four doc sites repeat it. The board entry's own Idea line keeping `sed -n` is superseded by this entry.

### [MEDIUM] Edit directive does not override the harness text

**Location:** contracts/stencils/edit/edit-directive.md:3-11
**Issue:** The same decision says the instruction lym gives about editing states that, in lym sessions, it overrides the harness bypass-mode reminder. The rewritten directive says "never by shell" and lists `sed -n` among the readers, but it does not name the override, and it keeps the reader list the decision removed from steers. The rest of the directive (`lym loom fmt`, `.scratch/`, own-worktree rule) is intact against the board entry.

### [LOW] Looped sed -i spellings pass

**Location:** internal/shellguard/rows.go:110-115
**Issue:** `do`, `then`, `find -exec` and `xargs` with a flag are not command positions for the pattern, so the loop forms of `sed -i` the board entry's evidence most plausibly shows are not refused, and the row comment's list of bypasses omits them.

### [LOW] Scratch captures into source extensions are refused

**Location:** internal/shellguard/rows.go:106-108
**Issue:** The comment says captured output passes; that holds only for extensions outside the list, so a capture into a scratch `.yaml`, `.md` or `.sh` is denied. The steer recovers (use Write), so a run is not stalled.

### [LOW] Untested pattern branches

**Location:** internal/shellguard/rows_integration_test.go:154-178
**Issue:** Deletion of the `>|` branch, of the quoted-target rule, or of any extension but `go` and `md` leaves every case green.

### [LOW] No hook-command case for shell-edit

**Location:** internal/shuttleengine/claudeengine/hookcmd_integration_test.go:237-241
**Issue:** Only the pattern test runs the shell-edit pattern; quoting damage when it is rendered into the hook command would not be seen.

## Rejected

- Perl script argument `-i` falsely denied (`perl script.pl -i foo`): the lens called it trivial and rare, and a false deny recovers through the steer.
- `cmd/lym/norecoverydeny_test.go` self-check row changed to `sed -n` is trivial coverage: the row stays valid against the changed scan and costs nothing.
- Extension list of `sourceFilePattern` as a reading of "tracked source file": sound against the tracked files (go, md, yaml, sh, mod, sum, cs are covered; the uncovered .txt, .tsv, .jq and .cmd are mostly research data), so not a finding.
- Row order and `Forms`: the lens traced that both steers carry the same rule, so order cannot change which instruction an agent gets.
- Commit message: its claims match the diff and no `Pattern-Departure:` trailer is needed, since no PATTERN entry forbids the diff's way; the Co-Authored-By line names a different model than this session's attribution, which is not a finding.
