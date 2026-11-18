```yaml
lens: board-scope
outcome: reported
```

Task: board-scope lens review of the decision record against the board, the peers and the entry's goal points.

**Goal points of the entry (issue #139)**
- 1a. A widened assertion, tolerance or limit names its measured cause, or it is a finding: **met** by D1 and D2.
- 1b. A test whose limits changed still checks what its name and doc say: **met** by D1.
- 1c. A claim of agreement or improvement carries before and after figures in the change description: **met** by D1 and D7. D1 also accepts commit messages, which goes past the entry's text without contradicting it.
- 1d. Generic across hubs, in every recipe: **met**. D1 puts the checks in the shared rubric, and the record names tapestry, darn and sampler.
- 1e. Domain rules belong in the repository's PATTERN: **met**. Scope Out leaves them there.
- 2. Drift flags every widened or removed assertion in an existing test and requires a measured cause: **met** by D3.
- 3. Large PR-Review findings, by count or severity, send the run back to Code-Review / Code-Burler, and the miss is recorded: **met** by D4, D5 and D6.

**Entries, peers and parent commits checked**
- `drift-judge-and-line-range` (peer, darn, running Darn-Write): **no conflict**. It changes the judge's snapshot check and line-range matching. D3 adds a request list and a judgement key in the same package, which is file overlap only. The record already lists it under Open risks, which is enough.
- `twill-fixer-fork-writes` (board T8, Code-Burler-Fix fork brief): **no conflict**. D2 adds a trailer fragment to the Code-Burler fixer's brief, and T8 changes that fixer's write contract. This is text overlap in burler stencils only.
- `orch-ignores-deny-python` (peer, landing): **none**. It touches shuttle and shellguard only.
- `scratch-symlink-two-paths` (peer): **none**. It is a stencil-wide rule about `.scratch` and touches no file this record names.
- `shuttle-start-idle-and-prompt` and `leftover-strands-and-runs`: **none**. They concern seat start and gate re-prompts, not PR-Review routing.
- `borrowed-orchestrator-ideas` / `orchestrator-survey-ideas` (project and note): **none** can be seen from the board briefs. Neither claims review-routing work.
- Parent commits: `lym loom peers` listed none (`parent_commits: []`).

**Decision notes**
- No `Decision for code-review-result-checks:` note is on the board list, so there was none to read.

**Findings**

- **LOW**, D2 (class `scope`): D2 adds a commit-trailer rule to the Code-Burler fixer's brief, while board entry `twill-fixer-fork-writes` (T8) is changing what that fixer fork may do.
  - Evidence: T8's body proposes either telling the fork it is read-only and where to hand its fixes, or running the fixer as a writing session.
  - If T8 lands the read-only form, the fork that writes the loosening is not the one that commits it, so the trailer instruction belongs with whoever commits.
  - The fix is to word D2's rule for "the session that commits the fix", not for the fork.

No BLOCKING or MEDIUM board contradictions.

Tool calls made: none. My context already held `lym board list`, `lym board get` for this task, `drift-judge-and-line-range` and `twill-fixer-fork-writes`, and `lym loom peers`, together with the peer record's head.
