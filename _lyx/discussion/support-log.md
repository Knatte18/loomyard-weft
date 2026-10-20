# Support log — lyxbin timeout test independent of helper start time

## Interview

A darn run holds no interview; the writer read the board entry and the code and made its own calls.

## Rejected alternatives

- A longer timeout on the test: still includes process start, and makes every passing run wait it out.
- Pre-warming the helper with an echo run before the sleeper: relies on the scanner caching a hash between runs, which is not guaranteed.
- Retrying with growing timeouts: bounded but timing-dependent, and slower on every scanned host.
- An exported context-taking `Run` variant: no production or test caller outside this package needs it.

## Review rounds

_No rounds yet._

## Question ledger

- How does the test start the clock after process start? Self-pick: an unexported context body ended by a cancel cause the test sets once the pid file exists.
- How does `Run` keep naming its timeout in the error? Self-pick: `context.WithTimeoutCause` with `ErrTimeout` wrapped with the timeout as the cause.
- How does the test avoid reading a half-written pid file? Self-pick: the sleeper writes by rename.
