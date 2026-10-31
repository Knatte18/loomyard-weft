```yaml
lens: code-correctness
outcome: reported
```

Task: run the code-correctness lens over the diff's changed functions and error paths.

**Per-function list (case run, verdict)**

- `Config.CreditBlock`, credit off, with engines run: returns `""`. Correct.
- `Config.CreditBlock`, `Credit` on, nil `enginesRun`, LoomYard text set: heading, then ` - LoomYard…`. Correct.
- `Config.CreditBlock`, heading set, LoomYard blank, no engines: no item lines, so `""`. The heading is not left alone at the end of the message. Correct per D2.
- `Config.CreditBlock`, heading empty or whitespace: the block is the item lines only, opening with ` - `. Correct per D2, which only says "trimmed heading". The block still starts with a space, so `LandingMessage` yields `body\n\n - LoomYard…`.
- `Config.CreditBlock`, `["alpha","alpha","beta"]`: sorted and compacted, so each engine appears once, in name order. Correct.
- `Config.CreditBlock`, nil `CreditEngines` map or an engine with no entry: lookup on a nil map returns `""`, so no line. Correct.
- `Config.CreditBlock`, engine text with an inner newline: the second line gets no ` - ` prefix. This needs a hand-edited multi-line value and is not a defect.
- `Deps.creditBlock`, `EnginesRun == nil`: no engines, no error. Correct.
- `Deps.creditBlock`, `EnginesRun` returns an error: the error is wrapped and returned. Finalize adds `landingshed: Finalize:` and PR-Review adds its name, so there is no double prefix and nothing is swallowed. Correct.
- `bodyWithCredit`, empty credit: the trimmed body. Correct.
- `bodyWithCredit`, empty body with credit: the credit alone, with no leading blank line. Correct.
- `bodyWithCredit`, both present: `body\n\ncredit`. Correct.
- `bodyWithCredit` differs from the old PR path: Create and Edit used to get the untrimmed `summary.Body` (`"\nChange body\n"`) and now get the trimmed body, which matches D4. Correct.
- `Finalize.Call`, `EnginesRun` fails: this happens in step 1a2, before any parent-side mutation, and returns an error. The doc on `Deps.EnginesRun` still matches. Correct.
- `Finalize.Call`, `LandingMessage(credit)`:
  - With `credit == ""` it returns `CommitMessage()` alone, so the message ends in the body.
  - Otherwise it adds a blank line and then the block.
  - Correct for D2 and D3, since `Lym-Task` is gone.
- `Finalize.Call`, GitHub merge message: `bodyWithCredit(summary.Body, credit)` uses the same `credit` value as the commit message. Correct.
- `PRReview.openPullRequest`, credit read failure: the read happens after the push and the description read, and before any `FindOpen`, `Create` or `Edit`. It returns an error, with no PR state touched and no partial record written. Correct. A push without a PR is the same state a description-read failure already leaves, and a retry redoes the push.
- `PRReview.openPullRequest`, `Create` (no PR open) and `Edit` (`editTitleAndBody` true): both receive the composed `body`. The only other `Create` or `Edit` calls in non-test landingshed code are these two, so no path sets a body without the block. Correct.
- `PRReview.openPullRequest`, record-gated branch where `Edit` is skipped (`editTitleAndBody` false): this follows the pre-existing logic, which the diff does not change.
- `MigrateConfig`, document with `co_authored_by`, `trailer_text` and `engine_trailers` and no renamed key: `RenameKeys` finds nothing to rename, so it returns `existing` byte-identical and no lines. Nothing moves into `credit_engines`. A second pass is unchanged. Correct.
- `MigrateConfig`, document with `require_pr_to_base` or `merge_fix_attempts`: the rename behaves as before, and a destination that is already set drops the old key. Correct.
- `MigrateConfig`, unparsable or non-mapping root:
  - `RenameKeys` calls `parseMappingRoot(existing, "config")` and returns the error.
  - `MigrateConfig` marks it `ErrInvalid`.
  - `TestMigrateConfig_InvalidDocument` (`Unparsable`, `NonMappingRoot`) still lines up with this.
  - I did not read `parseMappingRoot`, so its error on those two inputs is inferred from the call shape and the existing test.
  - Correct as far as I can tell.
- `MigrateConfig`, empty document: `root == nil`, so it returns `existing`. Correct.
- `ConfigSpec`, `Keys` and `RetiredKeys`: the four credit keys are cross-pair. The three retired keys are cross-pair, so a hub file holding them loads and reconcile prunes them. Correct.
- `ConfigOpenMaps`: returns `credit_engines`, so it stays an open map. Correct.
- `LoadConfig`, `credit_engines.claud`: fails with the key `credit_engines.claud` in the error. Correct.
- `LoadConfig`, `credit_engines: {}` in a hub file: the map is empty, so no engine lines and the load succeeds. Correct.
- `LoadConfig`, template: `credit: true` and `credit_heading: "Created using"`, with the Claude engine line only. Correct for D1 and D6.
- `ValidateDescription` message change: text only, no logic change. Correct.

**Findings**

None under this lens.

**Tool calls**
1. `grep -A40 'func RenameKeys' internal/yamlengine/*.go`: my context lacked `RenameKeys`, so I checked that it leaves a document with no rename untouched and errors on a bad document.
