## Interview

A darn run holds no interview.

## Rejected alternatives

- Checking the empty branch in PR-Review and Finalize beside the resolver: two deciders, which `PATTERN-pull-request-mode` forbids.
- Reading GitHub's 422 message "No commits between" to detect the empty branch: the board entry requires git to decide.
- Reusing `ParentHoldsTaskWork`: it also holds after a squash landing with task commits present, which is not "no commits beyond the parent".
- Comparing against the parent's upstream instead of the local parent: Finalize merges into the local parent, and refuses one ahead of or diverged from its upstream.
- Making `githubclient.IsTransient` refuse token sentinels: changes every consumer's retry policy, wider than this task.

## Review rounds

_No rounds yet._

## Question ledger

- Does an empty branch skip the pull request under a required parent? Self-pick: yes; the board entry says such a run needs no pull request, with no exception for required.
- Does a rejected environment token count as an authentication failure for the halt text? Self-pick: yes, it is a 401.
