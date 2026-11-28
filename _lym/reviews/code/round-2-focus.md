---
round: 2
focus:
  - "internal/dobbyengine/starts.go (firstPassExited, the PassExitOriginWait case, and startReady), internal/dobbyengine/catchupwait.go (oweCatchUp and the success path), internal/battencli/wire.go and catchup.go (the pass's own catch-up of its slug) and internal/beamcli/dobbydeps.go (which pairs the dobby's catch-up covers): for every way a first pass can exit as an origin wait, including a failure only its own slug meets and a dobby with no PullBolt wired, is the retry bounded by a backoff and reported past a bound?"
  - "internal/dobbyengine/pushsummary.go (summarisePushes) and view.go: when only a beam dobby's catch-up waits on origin and the prime's does not, who is told, past what bound? Do internal/dobbyengine/doc.go and _lym/landing/summary.md state what the code does, including what the summary notice names and the heading it is posted under?"
  - "internal/battencli/catchup.go (catchUpFailure, catchUpRefusal) and drive.go (passExitCode), with their readers in internal/dobbyengine (passExited, firstPassExited): which catch-up failures are treated as a transient origin wait, and does _lym/discussion/decision-record.md license each one losing its refused-start notice and backoff?"
  - "internal/dobbyengine tests (pushsummary_test.go, loop_catchup_test.go and the start refusal and backoff tests): does each new behaviour have a test that fails without it, including the fallback from Since to At, the restart of Since when the failure reason changes, and the origin-only filter of the push summary?"
---
