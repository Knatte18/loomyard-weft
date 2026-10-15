# Card 3 — Check a present relevant list is well-formed

**Create:**
- `plan:internal/discussionparser#CheckRelevant` -> `func CheckRelevant(relevantPath string) ([]Finding, error)`
- `plan:internal/discussionparser#TestCheckRelevant` -> `func TestCheckRelevant(t *testing.T)`

**Edit:**
- `internal/discussionparser#Validate`
- `internal/discussionparser#TestAppendDecision_AppendsAtEndOfDecisionsSection`
- `internal/discussionparser/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/discussionparser#writeFixture`
- `internal/discussionparser#allSectionsContent`

**Intent:**
Add `CheckRelevant` to `internal/discussionparser`, in its own file, per D3: the format check of `relevant.md`, separate from `Validate`, stdlib-only and taking the told path.

- `os.ReadFile(relevantPath)` failing with not-exist returns nil findings and a nil error: a missing list is never a finding.
  Any other read error, such as the path being a directory, is returned as the error with nil findings, as `Validate` does.
- The content is split on `"\n"` and each line right-trimmed of `" \t\r"`; blank lines are skipped.
- The first non-blank line may be an H1 (`# ` followed by text); any other non-blank line that does not start with `- ` at column 0 is a finding, one per line, naming its 1-based line number: a second H1, an H2, a paragraph, a `*` bullet, and an indented (nested) bullet all land here.
- A top-level `- ` line is an entry.
  It is a finding when the text after `- ` does not open with a backtick, or the backtick has no closing backtick, or the backticked path is empty.
  It is a finding when the path starts with `/`, `\` or `~`, or with an ASCII letter followed by `:`, or when splitting the path on both `/` and `\` yields a segment equal to `..`, whatever the host OS (the check never calls `filepath`'s OS-dependent functions on the path).
  A segment such as `..foo` is not `..` and passes.
  It is a finding when the line holds no ` — ` (space, em dash, space) after the path, or when the text after the first ` — ` is blank.
  Member names between the path and ` — ` are not checked.
- After the scan, a list with no entry line is one finding.
- A duplicate path is not a finding, and no named path or member is checked for existence.

Each finding is a `Finding` with `Path` set to `relevantPath`.
New check-name constants, in `validate.go`'s style, distinguish the four kinds: no entry, a stray line, a malformed path, a missing reason.
Each `Detail` names the line number and what is wrong.

`Validate` is unchanged in code, so `loomshed.ValidateDarnDiscussion` and `AppendDecision`, and with it `lyx loom decision add`, never read `relevant.md`.
Its doc comment and `validate.go`'s file header stop calling it the discussion gate's complete check set, and name `CheckRelevant` as the loom gate's other check.
`doc.go` adds the relevant list to the formats the package reads, and names loomengine's `DiscussionRelevant` among the accessors that declare where `_lyx/discussion/` is.

`TestCheckRelevant` is one table-driven test, each row writing its content to `relevant.md` in `t.TempDir()` (or nothing, or a directory) and asserting the exact check names, in order, and the error:
- well-formed, no findings: an H1 and two entries; an entry with members (`` `internal/x/y.go` `PlanTable`, `Config.Validate` — why ``); a duplicate path; blank lines between entries; a `..foo` segment; an entry with no title line.
- missing file: nil findings, nil error.
- path is a directory: a non-nil error, nil findings.
- an empty file, and a file holding only an H1: the no-entry finding.
- a stray line each: a paragraph line, an indented `  - nested` bullet under an entry, an H2, a second H1 after an entry, and a bare `#` or `# ` line with no title text.
- a malformed path each: a bare `- ` line, no backtick, an unclosed backtick, an empty path, a leading `/`, a leading `\`, a leading `~`, `C:/x`, `a/../b`, `a\..\b`.
- a missing reason each: no ` — `, and ` — ` followed by only spaces.

`TestAppendDecision_AppendsAtEndOfDecisionsSection`'s table gains a `relevant string` field, written as `relevant.md` beside the record when non-empty, and one row, "a malformed relevant list beside the record is ignored": the "followed by further sections" record with a `relevant.md` holding one stray paragraph line appends cleanly with no finding, so `AppendDecision`, and with it `lyx loom decision add`, ignores the list.
`Validate` ignoring it on a darn run is pinned at the verb by card 4's `Darn_MalformedRelevantIgnored` row.

`TestCheckRelevant` is a new top-level test in `internal/discussionparser`, whose test-budget row is full, so its row in `cmd/lyx/testdata/test-budget.yaml` goes up by one.

**ImpactSummary:** a new standalone exported check no caller uses yet; `Validate` changes only in comments, so its callers in `loomshed`, `loomcli` and `AppendDecision` behave as before.

**Commit:** 3: discussionparser: check a present relevant list is well-formed
