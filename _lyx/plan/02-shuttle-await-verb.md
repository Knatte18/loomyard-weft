# Card 2 — shuttle-await-verb

**Create:**
- `plan:internal/shuttlecli#awaitVerb` -> `const awaitVerb = "await"`
- `plan:internal/shuttlecli#defaultAwaitCap` -> `const defaultAwaitCap = 250 * time.Second`
- `plan:internal/shuttlecli#shuttleCLI.awaitCmd` -> `func (c *shuttleCLI) awaitCmd() *cobra.Command`

**Edit:**
- `internal/shuttlecli#Command`
- `internal/shuttlecli/cli_test.go`

**Prosa:**
- `docs/overview.md`

**Uses:**
- `plan:internal/shuttleengine#AwaitAny`
- `internal/burlercli#burlerCLI.awaitReviewCmd`

**Intent:**
Add `lyx shuttle await <path>... [--cap <duration>]`, the general keep-cache-warm wait of decision record D9, in a new file `internal/shuttlecli/await.go`, shaped after `burlerCLI.awaitReviewCmd`.

`awaitCmd` takes one or more paths (`cobra.MinimumNArgs(1)`), resolves a relative one against the seam cwd through `lyxcwd.CwdFrom` (`PATTERN-cwd-resolution`), and calls `AwaitAny` with the cap.
It prints ok with `ready: true`, `path` (the absolute path that was met) and `watched` (every absolute path it watched) when one appears;
when the cap elapses first it prints ok with `ready: false`, `watched` and `message: "none of the watched paths exists yet; call this command again"`, exiting 0 either way.
A stat error other than not-exist, and a cancelled context, are JSON errors with exit 1.
The flag `--cap` defaults to `defaultAwaitCap`, under the 5-minute prompt-cache lifetime;
the help text says the caller runs it with a Bash tool timeout above the cap (300000 ms) and re-runs it in a turn that does nothing else, and that a marker is a path like any other: the condition is "this file appears", whoever writes it.
The `Long` text names no Twill, Discussion or loom file.
It carries `Short`, the role audience annotation and JSON errors (`PATTERN-cli-cobra`), and is read-only.

`Command` adds the verb to the subtree and skips the group's `PersistentPreRunE` resolution for `awaitVerb` as it does for the bare group, so the verb needs no hub, git repository, config or engine and runs the same in a fork, a chair or a writer.
Its header comment lists the new verb.

New edge and its bound: the verb lets any agent wait on any file, and it only stats the told paths and ends each call by its cap; it writes nothing and reads no file's content.

`cli_test.go` gains one table-driven test, `TestAwaitVerb`, run from a temp directory outside any git repository through `RunCLIIn`: an existing path answers `ready: true` at once naming it under `path` and listing it under `watched`; with two paths of which only the second exists, `path` names the second; an absent path with a short cap answers `ready: false`, exit 0, with `watched` and `message`; a relative path resolves against the seam cwd; a path under a regular file is a JSON error with exit 1; no argument is an error; and the directory holds nothing new afterwards.
A live-poll case waits out no interval here; card 1's engine test pins it.

`docs/overview.md`'s shuttle module line and its CLI line extend the verb list with `await` and say what it does.

**ImpactSummary:** `Command` gains one verb and one pre-run skip; no other caller changes.
