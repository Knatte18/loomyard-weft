---
round: 2
focus:
  - "internal/battencli/overview.go overviewCmd: for which worktrees (prime, beam, task pair, any other) is a dobby reader attached, and does a task pair's frame carry no dobby section, as the decision record's acceptance criterion requires?"
  - "internal/battencli/dobbyreads.go lockHeld and dobbyengine.Live: does a pane poll contend with the dobby lock or a driven run's driver lock, so that a starting dobby or a hand-run lyx batten run (battenDriveRefusal in internal/battencli/drive.go) can be refused, and do the doc comments in dobbyreads.go and battencli/cli.go state the true locking behaviour?"
  - "internal/battencli/dobbyreads.go DobbyTasks: is the behaviour of open dependencies listed by slug, pruned dependencies skipped, and parent and owner read for every task with an empty status covered by a test that would fail on a break?"
  - "internal/dobbyengine/starts_test.go and updateStart: does a test fail if the Refusal term in the delete condition or the clear on a claim is dropped?"
  - "internal/dobbyengine/view.go BuildView versus passes.go drivePasses and runsInFlight: does the view's slot count match the scheduler's for a done run whose driver lock is held while a pass is in flight?"
  - "internal/dobbyengine/passes_test.go TestLog_OneLinePerStartAndPass: is every name passKind.String can write pinned by an assertion?"
  - "internal/battencli/overview.go overviewCmd doc comment and Short text: do they agree with the Long text, cli.go and docs/overview.md on which runs strands the pane belongs to?"
---
