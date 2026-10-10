---
round: 1
focus:
  - "D4 against its Acceptance criterion: D4 says Windows and `watchdog: off` keep their current unset path, while the criterion says `watchdog: off` issues the unset and Windows issues neither. Settle which is the intended Windows behaviour and whether the two statements agree."
  - "D5's bound says poll mode is transient on POSIX after D4, but D4 installs the signal entry only with `watchdog: on`. Settle what bounds poll-mode wakeups on POSIX with `watchdog: off`, and whether the premise holds."
  - "D7 attack surface: `-f /dev/null` drops every operator tmux setting, and the `default-shell` and `default-command` pins change how every strand pane starts. Settle whether the Decision states what this can now skip or let through, and what bounds an operator whose setup depended on `~/.tmux.conf` beyond the Open risks line."
  - "D3 and D2 attack surface: the daemon overrides the worktree config's tmux binary, and the watchdog verb stops discarding stderr. Settle whether each states what it lets through and what bounds it, and whether D3's claim that all worktrees of a hub must already agree on one binary is a stated fact or an unchecked premise."
  - "D8 premise: the identification of the watchdog's children rests on code reading, and the name-repair pass is called 'the rest'. Settle whether the Decision presents an unmeasured attribution as established, and whether it should say the re-measurement is what confirms it."
  - "Relocation: Decisions D6 and D7 carry 'Refusing is the smaller change' and 'Per-session pins rather than a reed-written config file' rationale that argues against alternatives, which `support-log.md` already records under Rejected alternatives. Apply the completeness-before-leanness test: settle whether any requirement the plan writer needs is only in that narrative before suggesting a move."
  - "Exclusion: Acceptance criteria and Constraints for work that is not this task's behaviour, such as the lint-comments pass, the commit-message line, and the PATTERN-refusal-way-forward remark about modules it does not bind. Settle whether each is a requirement the plan writer needs or belongs elsewhere."
  - "D1 and D6 and D7 acceptance coverage: settle whether each Decision's behaviour has a criterion that would fail if the Decision were not implemented, in particular the refusal reached by a test (D6, D7) and the Debug-absent-at-default half of D1."
---

Focus is set from a first read of both files and the writer stencil.
The stencil tells the writer not to gather exact signatures, `file:line` citations, interface shapes or dependency lists, so none of those appear as missing.
Rejected-alternative absence, a missing plan-writer notes section and call-site enumeration are excluded by the rubric and are not focus items.
