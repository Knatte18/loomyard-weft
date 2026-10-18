# Card 2 — gitrepo-local-branch-mark

**Create:**
- `plan:internal/gitrepo#LocalBranchKey` -> `const LocalBranchKey = "lyxlocal"`
- `plan:internal/gitrepo#ErrLocalBranch` -> `var ErrLocalBranch error`
- `plan:internal/gitrepo#Repo.MarkBranchLocal` -> `func (r *Repo) MarkBranchLocal(branch string) error`
- `plan:internal/gitrepo#Repo.ClearBranchLocalMark` -> `func (r *Repo) ClearBranchLocalMark(branch string) (cleared bool, err error)`
- `plan:internal/gitrepo#Repo.BranchMarkedLocal` -> `func (r *Repo) BranchMarkedLocal(branch string) (bool, error)`

**Edit:**
- `internal/gitrepo#Repo.pushPlainWithRetry`
- `internal/gitrepo#Repo.PushRebaseFree`
- `internal/gitrepo#Repo.PushCoalesced`
- `internal/gitrepo_test#TestPush`

**Uses:**
- `internal/gitrepo#readGoGit`
- `internal/gitrepo#Repo.CurrentBranch`

**Intent:** Give gitrepo the local-only branch mark D3 rests on, and make every push primitive refuse to publish a marked branch.

The mark is the per-branch git config key `branch.<branch>.lyxlocal`, a bool, `LocalBranchKey` naming its last segment.
It lives in the branch's own config section, so `git branch -D` drops it with the branch and the mark lives and dies with the branch.
`MarkBranchLocal` writes it to `true` and `ClearBranchLocalMark` unsets it, reporting whether a mark was there; both run git through the repo's checked runner (PATTERN-gitexec-checked-call).
`BranchMarkedLocal` reads it in-process through `readGoGit` over the repository config's raw `branch` section (PATTERN-gogit-read-helper, PATTERN-gitrepo-client-boundary): an absent key, an absent branch section and a value other than true read false, and only an unreadable config is an error.
Fabric owns the key: only fabricengine calls `MarkBranchLocal` and `ClearBranchLocalMark`, which `lyx fabric add --local` and a plain add's adopt use (card 4).

`ErrLocalBranch` is the sentinel, worded "gitrepo: branch is local-only, never pushed", that a push of a marked branch returns.
`pushPlainWithRetry` reads `CurrentBranch` and, when that branch is marked, returns `ErrLocalBranch` without running git push, so `Push`, `PushRebaseFree` and `PushCoalesced` all refuse through it; `Push`'s rebase retry and `PushRebaseFree`'s rejection mapping never fire on it, since it is no `*gitexec.GitError`, and `errors.Is(err, ErrLocalBranch)` holds on every returned wrap.
`PushCoalesced` takes its lock only after the same check, so a marked branch leaves no lock residue.
A mark that cannot be read is returned as the push's error rather than pushing: the direction that publishes nothing.
What this lets through: nothing more than before; it only removes pushes.
What bounds it: only a branch carrying the key is skipped, and `DeleteRemoteBranch`, `DeleteRemoteBranchLeased` and `UpdateRemoteBranchLeased` are untouched, since D3 guards fabric's two remote writes outside the push primitives at their fabric callers (cards 5 and 6).

`TestPush` gains a step: on clone A, `MarkBranchLocal` on its branch, a new commit, then `Push`, `PushRebaseFree` and `PushCoalesced` each return an error satisfying `errors.Is(err, gitrepo.ErrLocalBranch)` and the bare remote's tip is unchanged; `BranchMarkedLocal` reads true; `ClearBranchLocalMark` reports cleared true, a second call false; then `PushRebaseFree` lands the commit on the remote; `BranchMarkedLocal` of a branch never marked reads false with a nil error; a branch whose key is set to `false` by hand reads false; and with clone A's `.git/config` overwritten by text git cannot parse, `BranchMarkedLocal` returns an error and `PushRebaseFree` returns that error with the remote tip unchanged.

**ImpactSummary:** Every gitrepo push of a marked branch is refused; no branch carries the key until card 4, so no caller's behaviour changes in this card.

**Commit:** 2: gitrepo: local-only branch mark and the push refusal
