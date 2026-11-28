---
round: 1
ledger:
  - key: unconfigured-remote-name-reads-unreachable
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: change-description-omits-affected-callers
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: dangling-symlink-counts-as-existing
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: unmounted-directory-claim-overstated
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: apostrophe-breaks-path-capture
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: path-rows-cannot-fail
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: relative-path-resolved-against-cwd
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised its findings against the new missing-local-path classification in `internal/gitexec` and against the change description.
The gating one is that a bare unconfigured remote name is stat-ed as a relative path and reads as unreachable.
The other `internal/gitexec` findings cover dangling symlinks, apostrophes in paths, relative-path resolution,
the doc claim about unmounted directories, and test rows that cannot fail.
The change description names fewer callers and a narrower blast radius than the diff changes.
