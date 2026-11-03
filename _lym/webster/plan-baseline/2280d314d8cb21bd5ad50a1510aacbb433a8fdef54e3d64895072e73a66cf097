# Card 14 — tackler-remote-check

**Extends:**
- `contracts/stencils/shed/shed-template-tackler.md`

**Intent:** Decision record D5: the tackler checks the remote before judging a commit or branch absent from it.

Add a rule to the tackler stencil, under `## Reading order` after its numbered list, as its own short paragraph:
a verdict that rests on a commit or branch being absent from a remote first reads the remote tip with `git ls-remote <remote> <ref>`;
a branch is absent only when `ls-remote` answers no line for its ref;
the commit the stop names is present when it equals the tip, or when it is an ancestor of the tip by `git merge-base --is-ancestor` where the tip is known locally;
otherwise it is unsettled, and never read as absent.
Both commands are read-only, inside the stencil's `## What you never do` rule against git commands that change anything.

No test pins the new prose: see Shared Decisions, `d5-no-prose-test`.

**ImpactSummary:** One paragraph of static prose in one stencil the tackler reads at call time through `internal/stencilstore`; no marker is added, so `tacklerPrompt` and its tests are untouched.

**Commit:** `14: The tackler reads the remote tip before judging a commit absent`
