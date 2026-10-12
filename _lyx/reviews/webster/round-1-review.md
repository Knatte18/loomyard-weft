---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/shortcutcli/shortcut.go:196
    summary: A false-valued help or json word such as --json=false is dropped, while doc.go says a json word prints the shortcut's help
    fix: In internal/shortcutcli/doc.go's Words list, change the help or json bullet to say that only a word which raises help (-h, --help, --json, --json=true, a shorthand run holding h) prints the shortcut's help, and that a false-valued one (--json=false, --help=false) is dropped and the argvs run. Add one row to TestShortcut_Run in internal/shortcutcli/shortcut_test.go with args prime and --json=false, expecting both runs (orch start then reed attach) and exit 0, so the dropped word is pinned.
---

### [LOW] A false-valued help or json word is dropped, which the package doc does not say

**Location:** internal/shortcutcli/shortcut.go:196
**Issue:** `routeWords` sets `Help` only when the mirror's `help` or `json` value is true, so `prime --json=false` and `prime --help=false` parse as global words, join neither `Global` nor `Trailing`, and the shortcut runs both argvs.
That is a sensible outcome, since the real root runs normally under `--json=false`.
But `doc.go` lists `--json=<bool>` among the words that print help and run nothing, and no `TestShortcut_Run` row covers a false value.
The code and the doc disagree on that case, and the behavior is unpinned.

## Scope notes

All six commits were read against their cards.
The suites of `internal/shortcutcli`, `internal/battencli`, `internal/seatengine`, `internal/reedengine`, `internal/loomcli` and `cmd/lyx` pass under `lyx gate test`.
The `tmux`-tier long-paste test could not be run here, so it was read only: it fills a second strand with `stty -icanon -echo; cat`, pastes a 70 001-byte line that starts with `-` and ends with `;`, and compares the file's content.

- Card 1: the record is read once per poll before the slug loop, a read error logs one Debug line and falls back to the empty record, and the state cell changes only for `running`, `paused` and `starting`.
  Rows are added to `TestRenderOverview` and frames to `TestOverviewSource_ReadsThePrimesRunsFromDisk`; no new top-level test.
- Card 2: the session reading precedes the up-line check, the four held or dead names skip the advisor, and an erroring reading falls through to the up-line clause.
  `TestEngine_Run_ChairHoldDeferral` gains the asking, idle-stalled, idle-done and dead rows, a dead row with a landed up line, and the unreadable row; the existing up-line row gained an `act`.
- Card 3: `load-buffer -b <buffer> -` and the paste share one invocation through `runStdin`, the text is never an argv element or logged, and the failed-invocation `delete-buffer` stays.
  The existing table tests were extended, with no duplicate test.
- Cards 4 and 5: each routed argv reaches the intended command, unknown flags and bad-valued globals fall to the last argv, `--` and everything after it is trailing, and the module guards skip `shortcutcli.GroupID`.
  `TestShortcuts_FileAndRootListing` bounds that group to the file's shortcuts.
- Card 6: the constant, its test, and both comments move together.
- Test economy: the new top-level tests are the two in `internal/shortcutcli` and the one in `cmd/lyx`, matching the budget rows `internal/shortcutcli: 2` and `cmd/lyx: 79`.
