# Card 2 — orch-rules-verb

**Create:**
- `internal/orchcli#rulesBolt`
- `internal/orchcli#boardRulesBolt`
- `internal/orchcli#boardRulesBolt.Push`
- `internal/orchcli#boardRulesBolt.PullThenCommitWritten`
- `internal/orchcli#orchCLI.rulesCmd`
- `internal/orchcli#writeHubRules`
- `internal/orchcli#fakeRulesBolt`
- `internal/orchcli#fakeRulesBolt.Push`
- `internal/orchcli#fakeRulesBolt.PullThenCommitWritten`
- `internal/orchcli#TestWriteHubRules`
- `internal/orchcli#TestOrchIntegration_RulesCommitsOnBolt`

**Edit:**
- `internal/orchcli#Command`
- `internal/orchcli#TestOrchIntegration_Refusals`
- `//internal/orchcli/doc.go`
- `//internal/orchengine/doc.go`
- `//docs/overview.md`

**Uses:**
- `internal/orchcli#rulesRel`
- `internal/fabricengine#NewBolt`
- `internal/fabricengine#Bolt.Push`
- `internal/fabricengine#Bolt.PullThenCommitWritten`
- `internal/fabricengine#EnvSyncOptions`
- `internal/fabricengine#NewMutations`
- `internal/fabricengine#BoardDir`
- `internal/gitrepo#ErrPullRebaseFailed`
- `internal/fsx#AtomicWriteBytes`
- `internal/output#ErrFields`
- `internal/orchcli#refusalCase`
- `internal/hubforge#CopyHub`

**Intent:** Add D3's verb, `lyx orch rules --from <file>`, which replaces the hub-wide rules file on Bolt with `<file>`'s content and commits and pushes it.

`rulesCmd` builds the `rules` leaf with the operator audience annotation, `Args: cobra.NoArgs` and a string flag `--from`.
Its `Long` says the verb replaces the whole hub-wide rules file, `<board>/` plus `filepath.ToSlash(rulesRel())`, so a draft starts as a copy of the current file at that path, and that an empty file clears the rules; the path is built with `fmt.Sprintf` from `rulesRel`, so no string literal spells `_lyx`.
Its `RunE` reads the cwd through `lyxcwd.CwdFrom(cmd.Context())` and calls `writeHubRules(out, cwd, from, c.paths.HubRulesPath, boardRulesBolt{bolt: fabricengine.NewBolt(fabricengine.BoardDir(c.location.HubPath))})`, aborting with its exit code through `clihelp.Abort` when non-zero, as the other verbs do.
`Command` registers `c.rulesCmd()` beside the other verbs.
The verb runs under the `orch` group's existing `PersistentPreRunE` unchanged, so it reaches `RunE` only after the drivable-worktree check and `refuseNonPrime` pass and the engine config loads; see the overview's Shared Decisions.

`rulesBolt` is the seam over Bolt with the two calls the verb makes, `Push() error` and `PullThenCommitWritten(write fabricengine.BoltWrite) (fabricengine.BoltWriteResult, error)`.
`boardRulesBolt` wraps a `*fabricengine.Bolt`: `Push` calls `bolt.Push(fabricengine.EnvSyncOptions())`, and `PullThenCommitWritten` calls `bolt.PullThenCommitWritten([]fabricengine.BoltWrite{write}, fabricengine.NewMutations(""))`.

`writeHubRules` runs these steps, each refusal exit 1.
Every envelope after step 1, refusal or success, carries D3's three fields: `path` (the hub-wide rules path), `committed` and `unpushed`; refusals go through `output.ErrFields`.
1. An empty `from` is refused as `lyx orch rules needs --from <file>`; any other `from` is joined onto `cwd` when relative and read.
   A read error is refused through `output.Err` naming `--from`, the resolved file and the error; nothing else runs, so no lock is taken and Bolt is not touched.
2. `bolt.Push()`; an error is refused with the push error, `committed: false` and `unpushed: true`, saying nothing was written and that a re-run once Bolt pushes is the way forward.
   When the error wraps `gitrepo.ErrPullRebaseFailed`, the recovery rebase conflicted with an upstream change and a bare re-run fails the same way, so the refusal names the board's diverged recovery in place of the re-run: run `git pull --rebase` in the board dir (`rulesPath` with `rulesRel()` trimmed from its end) and resolve the conflict, then `lyx board sync`, then re-run `lyx orch rules --from <file>`.
3. `bolt.PullThenCommitWritten` with one `BoltWrite` whose `Message` is `orch: replace the hub-wide rules` and whose `Write` sets a `wrote` flag, calls `fsx.AtomicWriteBytes(rulesPath, data)` (which creates `_lyx/orch/` when absent, writing a temporary file beside the rules file and renaming it) and returns `[]string{rulesRel()}`, the one path committed, which is Bolt-relative because `rulesPath` is `BoardDir` joined with `rulesRel()`.
   An error with `wrote` false came before `Write` ran, a board or push lock or a git read inside the pull: it is refused as `<err>; <rulesPath> unchanged, nothing written`, with `committed: false` and `unpushed: false`.
   A failure inside `Write` is recorded by the closure; the call's error is then refused as `<err>; <rulesPath> unchanged`, with `committed: false` and `unpushed: false`, since the rename never ran and the file keeps its prior content.
   Any other error, with `wrote` true and no write failure, is a failed commit after the write: it is refused with `committed: false` and `unpushed: false`, naming the file as written but uncommitted and `lyx orch rules --from <file>` as the way forward, since that run commits the one path under the verb's own message.
   A result with a non-empty `Skipped` is refused with `SkipDetail` as the way forward, `committed: false` and `unpushed: false`; nothing was written.
4. `bolt.Push()` again; an error is refused with `committed` (the result's `Committed`) and `unpushed: true`, naming the commit as made but not pushed and the next run of the verb as what pushes it.
   An error wrapping `gitrepo.ErrPullRebaseFailed` names step 2's diverged recovery in place of the next run.
   D3 settles only a recovery rebase over a non-conflicting upstream change, so this conflict wording in steps 2 and 4 is the plan's call on the case D3 leaves open.
5. Otherwise `output.Ok` with `path`, `committed` (the result's `Committed`, false for unchanged content) and `unpushed: false`.
An empty or whitespace-only `<file>` takes the same path and commits an empty rules file; the role render then shows the hub-wide placeholder card 1 gave it.

What the verb lets through, and its bound: it publishes whatever `<file>` holds to Bolt and Bolt's remote, but it reads only the file `--from` names and commits only `rulesRel()` under Bolt, never another Bolt path; a write killed between the temporary file's creation and its rename can leave that temporary file untracked beside the rules file, where the next whole-repo `Bolt.Commit` sweeps it in.
It runs only with the prime's worktree as its working directory, under every orch verb's prime-only refusal, which checks the directory and not the caller; what bounds the written text is the role preamble card 1 wrote, under which the rules add to the role and override none of it.

`internal/orchcli/doc.go`: the verb list in the package comment's first paragraph gains `rules`, and a new `# The orch rules` section says `lyx orch rules --from <file>` replaces and commits the hub-wide file on Bolt with a pull first and a push before and after, and the own file is committed as any weft file is, through `lyx fabric commit`.
`internal/orchengine/doc.go`'s role-file paragraph, which card 1 rewrote, gains one sentence: the file at `Paths.HubRulesPath` is written only through `lyx orch rules`, and the one at `Paths.OwnRulesPath` is committed through `lyx fabric commit`.
The orch module line of `docs/overview.md`, whose verb list reads `lyx orch start|status|refresh|distill|stop|cleanup`, gains `rules`, giving `lyx orch start|status|refresh|distill|stop|cleanup|rules`.

Tests:
- `TestWriteHubRules` is table-driven over `fakeRulesBolt` and a `t.TempDir()` whose `board/_lyx/orch/rules.md` path (joined through `rulesRel`) is the rules path; each row seeds an optional prior file, a `--from` value and fake failures, and asserts the exit code, the envelope's `error` substring or `path`/`committed`/`unpushed` fields, the rules file's bytes afterwards (or its absence) and the fake's call log.
  `fakeRulesBolt` records each call and keeps an `unpushed` count of commits not yet pushed, which a row may seed.
  `Push` returns the next error from a queue (nil when empty) and zeroes `unpushed` when it returns nil.
  A row may also seed an `upstreamMoved` flag, standing for an upstream that moved past Bolt.
  `PullThenCommitWritten` returns its configured pre-write error without running `write`, or its configured skip with no write, or a `diverged` skip with no write when `upstreamMoved` is set and `unpushed` is above 0, as the pull-first would skip such a Bolt, or runs `write.Write()` and returns the write's error, then a configured commit error, and otherwise reports `Committed` true, raising `unpushed`, only when the file's bytes differ from the bytes before the write.
  Every row calls `writeHubRules` once and asserts the fake's `unpushed` afterwards beside the envelope's three fields.
  Rows: a new file under an absent `_lyx/orch/` is written and committed with `path`, `committed: true`, `unpushed: false` and the fake's `unpushed` 0; an empty draft writes an empty file and commits; a relative `--from` resolves against `cwd`; an empty `--from` is refused naming `--from` with no fake call; a missing `--from` file is refused naming `--from` and the file, with no fake call and the prior file unchanged; a first push failure is refused with the push error, `unpushed: true`, no write call and the file unchanged; a first push whose error wraps `gitrepo.ErrPullRebaseFailed` is refused naming `git pull --rebase` and `lyx board sync`, with no write call and the file unchanged; a pre-write error is refused naming the file as unchanged and nothing written, with the prior content kept; a pull skip is refused naming the `SkipDetail` with the file unchanged; a failed write (the rules directory made read-only, skipped on Windows and as root, as `TestWatcher_NoteReplyGate`'s read-only row does) is refused naming the file as unchanged, with the prior content kept; a failed commit leaves the new content and is refused naming the re-run `lyx orch rules --from`; a failed last push leaves the new content with `committed: true`, `unpushed: true`, the fake's `unpushed` 1 and exit 1; a last push whose error wraps `gitrepo.ErrPullRebaseFailed` leaves the new content with `committed: true`, `unpushed: true` and the fake's `unpushed` 1 and is refused naming `git pull --rebase` and `lyx board sync`; and a re-run after a failed push, seeded with the prior file already holding the draft's content, the fake's `unpushed` at 1 and `upstreamMoved` set, makes no new commit and exits 0 with `committed: false`, `unpushed: false` and the fake's `unpushed` 0.
  A dropped first push fails that last row on its exit code and fields, through the fake's `diverged` skip, as well as on its call log; a dropped second push fails the new-file row on the fake's `unpushed` 0 and every committing row's call log.
- `TestOrchIntegration_RulesCommitsOnBolt`, in a new `//go:build integration` file over `hubforge.CopyHub(t, hubforge.Shape{Anchor: "."})`, clears `FABRIC_SKIP_GIT` and `FABRIC_SKIP_PUSH` with `t.Setenv` and so does not call `t.Parallel`; a comment at the test names that process-global state, as `PATTERN-test-speed` requires.
  From the prime anchor it writes a draft into `t.TempDir()` and runs `RunCLIIn(anchor, &out, []string{"rules", "--from", draft})` with the draft's absolute path on a Bolt with no `_lyx/orch/`: exit 0, the board's rules file holds the draft, the board's commit count rose by one, `git show --name-only --format= HEAD` in the board dir lists only `_lyx/orch/rules.md` (built with `filepath.ToSlash(rulesRel())`), and `HEAD` equals `@{u}`.
  The same call again exits 0 with `committed: false` and leaves `HEAD` unchanged, pinning that real git commits nothing for unchanged content.
- `TestOrchIntegration_Refusals`' prime-only step adds `rules` to its verb list, so `lyx orch rules` from the task worktree and the records sibling is refused by the prime-only refusal before any `.lyx/orch` is created.
  The header comment of its file, which lists the prime-only verbs, gains `rules`; the file has no other target here because `edit-test-file` refuses a `_test.go` path and the test's own glyph is listed.

**ImpactSummary:** Adds one verb to the `orch` tree through `Command`; no existing verb or caller changes, and the refusal test, both orch packages' `doc.go` and the orch module line of `docs/overview.md` gain the verb.
