# Claude account per worktree: all eight batches done

Every batch reported `status: done`, and no batch needed a recovery.
Batches 01 and 03 were done before this session; this session ran the other six.
No digest listed a `departures` entry or a `renames` entry.

## Batches run this session

- **04-claude-account-store**: the account file, usage records, pick and resolution; pick at Worktree-Create and adopt through `AccountKeeper`; launches export or unset `CLAUDE_CONFIG_DIR` and record the account.
  Head `7fdad554`.
  Scope-guard deviations: the account and keeper files under `internal/shuttleengine` and `claudeengine`, `internal/battencli/accountpick.go`, and tests in `cmd/lym`, `adoptcli` and `shuttleengines`.
- **07-known-config-dirs**: readers of Claude's config directory read every known directory.
  Head `8c0ef122`.
  Scope-guard deviations: `claudeengine` files (`claudeengine.go`, `manifest.go`, `resume.go`, `scope.go`, `transcriptread.go`) and `tools/tokencount/main.go`.
- **08-resume-carries-transcript**: a resume carries the transcript into its account; `lym claude account set|show` and the status line verb; the status line in settings of listed accounts.
  Head `25622f1e`.
  Scope-guard deviations: two `cmd/lym` guard tests, `internal/claudecli/cli.go`, `claudeengine` account verbs and resume files, and `shuttleengine` engine and fake files.
- **11-reed-hold-and-relaunch-ops**: reed's hold and relaunch ops under the same GUID; one running predicate for a relaunching run record.
  Head `059c8dc8`.
  Scope-guard deviations: `reedengine/lifecycle.go`, `shuttleengine` attach, reed, run, rundir, sessionread and stop files with their tests, and the shuttlefake.
- **13-account-switch-on-limit**: records `blocked_until` on a limit and switches the entry; the in-run relaunch through the stop verb's relaunch mode; a relaunch bound on attach; a refused copied resume starts fresh.
  Head `559df40b`.
  Scope-guard deviations: `accountrelaunch.go`, `accountswitch.go`, `apierrorretry.go`, `hold.go`, `loadturn.go`, `run.go` and `stop.go`, with their tests, and two `cmd/lym` and completion-signal guard tests.
- **17-orch-switch-and-fresh**: the orch watcher switches the orch's account and restarts it, fresh on a refused resume; the accounts reference doc.
  Head `52e7447a`.
  Scope-guard deviations: `orchcli/start.go`, `orchengine/restart.go` and `watcher.go`, with their tests.

## Scope guard

Every deviation listed is a file the batch touched outside its card's member targets.
The digests carry no warnings.
