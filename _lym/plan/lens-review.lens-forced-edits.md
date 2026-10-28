```yaml
lens: forced-edits
outcome: reported
```

Forced-edits lens review of the creel-agents plan (cards 1–16).

Findings:

1. MEDIUM — cards 7 and 9, `TestSweep_RemovedHolders` rows: the existing sweep-wait predicate still claims the chaser's waits, and no card narrows it.
   - `isSweepWait` is `d &gt; 8*time.Second` at `internal/creelengine/deliver_test.go:25`.
   - `env.sweepStep` and `env.sweepLast` pick the sweep's wait through that predicate (`deliver_test.go:47-58`), and so do `rig.sweepStep` and `rig.sweepLast` (`internal/creelengine/server_test.go:274-283`).
   - Card 7's chaser waits `agedPersistInterval`, which is 60 s while running. Card 7 only adds `isChaseWait`, defined as longer than 50 s.
   - Card 9's new rows remove a holder that read a marked message, so the chaser is running, and those rows drive `env.sweepStep`/`env.sweepLast`, both under the card's Uses. The 60 s chaser wait matches `isSweepWait`, so the step can advance or count the chaser's timer instead of the sweep's.
   - Fix: card 7 also lists `internal/creelengine_test#isSweepWait` and narrows it to `8s &lt; d &lt;= 50s`, the sweep ceiling. `internal/creelcli#isSweepWait` (`cli_test.go:49`) gets the same change if any card-10 row drives a sweep.

2. LOW — card 5, `internal/creelengine/doc.go` is under Extends, but the card makes existing text wrong.
   - The Mail section states the line form `[creel] #&lt;id&gt; from "&lt;sender&gt;": &lt;body&gt;` (`doc.go:77`, `:84`), and the Tool-boundary section says "Each message is one MessageLine, the line a notice is typed as" (`:118`).
   - Card 5 changes `MessageLine`'s form for a marked message, so this is a rewrite of what is already there. It belongs under Edit, per the rubric's "Extends on a ref the card rewrites".

3. LOW — card 7, `internal/creelengine/doc.go` is again under Extends while the card falsifies existing sentences.
   - Retention "carries a carried record for each unread message, queued inject and live hold" (`doc.go:67`). After card 7 it also carries read, marked, unanswered messages.
   - The Log section's event list (`doc.go:236-242`) also changes.
   - These need Edit, not Extends.

4. LOW — card 7, `internal/creelengine#Scheduler.Stop` is not listed.
   - Card 7 says the chase clock's `chase-aged` record is journaled "when the scheduler stops".
   - `Scheduler.Stop`'s doc comment (`deliver.go`, "persists the TTL each queued inject has consumed") then no longer describes what it does.

5. LOW — card 11, `internal/reportengine/doc.go` is under Extends only.
   - Its opening says the package computes "the read-only report behind `lym report convergence`" and that "every figure … [is] read from git objects".
   - `ComputeReplies` reads creel log entries, not git, so the opening needs rewording. That is an Edit, not an addition.

6. LOW — card 2, the existing `postCmd` refusals are left stale.
   - They read "the forms are: post &lt;receiver&gt; &lt;text&gt; and post --reply &lt;id&gt; &lt;text&gt;" (`internal/creelcli/cli.go`, `postCmd`).
   - Card 2 adds the `--parent` form and only writes a new refusal with the three-form list. The two existing messages keep listing two forms and never name `--parent`.

Calls made:
- grep for `PostWait`/`--wait-reply` test rows, "To answer it" pins, peer-messaging text, SendMessage in `cmd`/`tools`, shuttle-verb mentions, the trust phrase, and enclosing functions for comment sites. My context lacked these call sites outside the files already read.
- Two `lym quarry resolve` calls (the first's output was mis-filtered) to confirm several card glyphs and the comment ownership at `run.go` and `command.go`. Quarry had not been run on these.
- grep for `clientSeam` literals, `ListAgents` sites, parent-directive pins, the PATTERN `(test)` marker, `server_test.go` wait classifiers, and the `cli_test.go:524-545` row. My context lacked the seam literal sites and the classifier use.
- Read the `deliver_test.go:1464-1495` and `server_test.go` waiting-post rows, to check whether existing rows read a now-marked post. My context lacked these rows.
- Read `cmd/lym/sandbox_coverage_test.go` and the `tools/sandbox` listing, to check whether verb removals or additions force a suite edit. My context lacked the coverage rule.
