```yaml
lens: fix-proposal
outcome: reported
```

fix-proposal lens: I challenged each review proposal the fixer carried out as proposed. I built the cases below; I name no proposal list because the lens asks for none.

**1. BLOCKING — D5's hold on the board `done` commit (F3, carried out as proposed), and D1's sentence "The board target skips pushing past a commit a landing entry holds (D5)".**
- **Summary:** the hold binds only the board target's pusher. The synchronous Bolt pushes D1 keeps still push the board branch past a held `done` commit, so origin can still read an ended entry before its parent holds the work.
- **Case:** landing A's parent push is owed. The board branch holds A's `done` commit, which nothing has pushed yet. Then `lym adopt` or a beam owner claim runs the board owner write. That write is a pull-first Bolt commit that it pushes at once (`internal/boardengine/ownerset.go:3`). The push sends the whole board branch to origin, A's `done` included, before A's parent.
- **Second path:** `configcli/hubwide.go:163,241` `bolt.Push` does the same.
- **Effect:** the F3 premise "nothing holds the board done back" still holds for every synchronous board push, and D5's promise "origin never reads ... an ended entry whose parent lacks the run's work" does not hold. D1's list of kept synchronous pushes says nothing about the hold.

**2. MEDIUM — F8's definition, "owes a parent push while its ledger entry stands, refused or not", combined with F3/F7's hold.**
- **Summary:** one refused landing push freezes every later board push on the hub until an operator clears the refusal.
- **Case:** A's parent push is refused, and its entry stands marked refused. The board target then pushes nothing past A's held `done`. Every later board commit from any run sits behind it (board writes, other runs' `done`s, `lym board sync`) for as long as the refusal stands.
- **Effect:** one task's refusal stops the board for every run. That goes against the Goal's "only a refusal that retrying cannot fix stops anything", which a reader takes to mean only the refused thing stops. The record neither states nor bounds this.

**3. MEDIUM — D5, F6's rule (carried out as proposed): "Every Finalize pass that finds the local parent holding the task's work while origin's parent lacks it records that landing entry, the resume included".**
- **Summary:** with git or pushes off, origin's parent never holds the work, so every landing writes an entry that never goes.
- **Case:** the pusher starts none because of the skip in `start.go:109-111`, so the entry stands forever. The bar's minutes keep counting, the all-clear never posts, and D13's dobby teardown condition "the landing target owes no parent push" never comes true.
- **Effect:** this conflicts with D13's new pushes-off sentence (F10) and with D11. D5 needs "with git or pushes off no landing entry is written".

**4. MEDIUM — D9's new evaluation text, "runs `git check-ignore --no-index -z --stdin` there ... Only per-directory ignore files from the base count, never `info/exclude` or `core.excludesFile`".**
- **Summary:** a git run in the scratch repository still reads the user's global excludes, so the stated rule sources do not hold.
- **Mechanism:** git reads `core.excludesFile`, and `$XDG_CONFIG_HOME/git/ignore` when that key is unset, in every repository. The step as written does not override it; it would need `-c core.excludesFile=/dev/null` or an equivalent.
- **Effect:** a file the developer's global ignore matches, such as `*.env` or `.idea/`, would be removed although the base's rules do not ignore it. That contradicts D9's bound "a file those rules do not ignore is never touched".
- **Evidence:** no Go file under `internal` sets `core.excludesFile` (grep found none).

**5. LOW — D6 (F13, carried out as proposed): "The step writes no halt note for a failed stop whose transient class is `git-transport` or `github-api`; the driver writes one only when it stops waiting".**
- **Summary:** a step that no Go driver drives, such as `lym loom step` run by hand, which the `goDriver` naming in `sheddriver/drive.go` suggests is one of several drivers, now leaves no halt note for a network stop.
- **Effect:** no driver waits on it or writes the note later, so the stop leaves no friction. The rule should apply only under the Go driver.

The rest hold against the cases I built. F5's driver lock is `shedrun.DriverLock`, which `loomcli/drivewiring.go:74`, `battencli/drive.go:72` and `dobbyreads.go:228` all use. I also checked F2, F4, F9, F10, F12, F19, F20, F21, F23, F24 and F26.

Tool calls:
- One grep of `shedrun.DriverLock` users, any `excludesFile` handling, and the push in `boardengine/ownerset.go`. My context held none of these: I needed them to test F5's lock name, D9's rule sources and whether the owner write pushes the board branch synchronously.
