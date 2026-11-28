---
round: 2
focus:
  - "internal/gitexec/transport.go, namesMissingLocalPath and missingRepositoryPattern: which strings git quotes in 'does not appear to be a git repository' and 'repository ... does not exist' (a configured remote's path, an unconfigured remote name such as origin, a file:// URL, a relative path, a path holding an apostrophe, a dangling symlink), and does IsTransportFailure return the right verdict for each, for push, fetch, pull, clone and ls-remote?"
  - "internal/gitexec/transport_test.go: does each row and real-git case for the missing-local-path classification fail when the behaviour it names is removed from the code?"
  - "The IsTransportFailure doc comment in internal/gitexec/transport.go and the package doc in internal/gitexec/doc.go: does each statement about unconfigured remotes, relative paths, removed or unmounted directories and mount points match what the code does?"
  - "_lym/landing/summary.md: do the listed callers whose behaviour changes with IsTransportFailure, and the Merge danger blast radius, cover what the diff can break, including internal/shedtransient and the retry, probe-error and owed records push paths in fabric?"
---

The round number of this file is the round it targets.
