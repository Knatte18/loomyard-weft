```yaml
lens: fix-affected-set
outcome: reported
```

Task: fix-affected-set lens, checking that the fixes for F3 and F1 cover every member of their sets.

**MEDIUM: F3 left the weaver twins of the webster stencils without the rules**
- Changed text: the new `## External tools and processes` section in `contracts/stencils/webster/webster-body-implementer.md`, and the matching bullet in `webster-body-verify-fix.md`.
- The weaver stencils are the code-editing counterparts and carry neither rule:
  - `contracts/stencils/weaver/weaver-template-batch.md` has the same job as `webster-body-implementer.md`: implement cards, commit, report.
  - `contracts/stencils/weaver/weaver-body-verify-fix.md` has the same job as `webster-body-verify-fix.md`: fix after a failed plan-level verify, commit `fix:`.
- Evidence: `grep -l "outlives your command"` over `contracts/stencils` lists seven files, with no `weaver/*` entry. The review's rule for F3 was "any other code-editing writer or fixer stencil".
- These agents run under the same autonomous `tmux-script` row, so they get the same steer.

**LOW: other code-editing stencils lack the rules**
- `contracts/stencils/landing/landing-template-conflict.md` has the agent edit files, including files outside the listed paths so the tree builds and its tests pass. It forbids git and has no external-tool rule.
- `contracts/stencils/loom/loom-fragment-arbiter-fix.md` lets the arbiter make and commit a fix of its own.
- Whether either counts as a "writer or fixer stencil" is a judgement call, so I rate them LOW.

**Checked, no finding**
- F1: no old steer text remains outside `_lym`, `.lym`, `.scratch` and `.git`, so the golden and the Go source are fully converted.
- Covered by the cold frame: `gatefix-body-card.md`, `gatefix-body-darn.md`, `gatefix-body-plan.md` and `gatefix-body-discussion.md`. I did not confirm that the last three also join `gatefix-frame-cold.md`.
- `webster-prefix-recovery.md` and `webster-prefix-verify-fix-fork.md` are not separate agent bodies as far as I could tell from their markers. They appear to be prefixes joined to the implementer and verify-fix bodies that now carry the rules.
- `webster-fragment-card-fixer.md` is a paragraph filled into `webster-body-implementer.md`, so the implementer body's new section covers it.
- Left unchecked: F2, F4, F5 and F6 are not set findings. F4 and F5 are decision-class and not the fixer's.

**Calls made**
1. A grep across `contracts/stencils` for files with and without the rule, plus any leftover old steer text. My context held no listing of the stencil directory.
2. A per-file grep of the weaver, landing-conflict, recovery, shed and arbiter-fix stencils to see whether each edits code. The directory listing did not show what they do.
