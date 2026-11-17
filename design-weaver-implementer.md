# Weaver: the cold-start implementer with model escalation, its run directory, reset verb and loom seam

`weaver-implementer` · Task · feature, webster, weaver · running · PR-Review

Decision record: [Sketch](decision-weaver-implementer.md)

Filed from the Sketch `weaver`.

Takes: internal/runplan as implementer-shared-run-plan delivers it: the run-plan state and batch records Weaver's state file embeds, the report contract with journal, the post-batch checks, suspect checks, digest, card gate rendering, strand await and verify-failure record, each driven by the geometry and way-forward text Weaver passes
Delivers: the weaver value of tapestry's implementer row: its engine package, the weaver.yaml config module, its stencils, _lym/weaver/ and .lym/weaver/ named through lymdirs, the lym weaver reset verb, the active-implementer seam every loom reader of implementer state goes through, the implementer-neutral plan-conflict path, and the implementer-spec updates
Exports: internal/weaverengine with its row producer and reset; internal/weavercli with lym weaver reset; the active-implementer seam the D1 guard, plan gate, renumber verbs, decision-notice guard, Describe, the PR rework archive and merge-in read

The part changes these packages:

- internal/weaverengine (about 5000 changed lines)
- internal/weavercli (about 800 changed lines)
- internal/runplan (about 120 changed lines)
- internal/websterengine (about 120 changed lines)
- internal/shedrecipe (about 120 changed lines)
- internal/loomshed (about 300 changed lines)
- internal/loomcli (about 350 changed lines)
- internal/shedadapters (about 100 changed lines)
- internal/landingshed (about 30 changed lines)
- internal/fabriccli (about 50 changed lines)
- internal/configreg (about 10 changed lines)
- internal/lymdirs (about 40 changed lines)
- internal/hubgeom (about 60 changed lines)
- internal/standalonegeom (about 60 changed lines)
- cmd/lym (about 10 changed lines)
- contracts/stencils/weaver (about 250 changed lines)
- contracts/specs (about 30 changed lines)
- contracts/recipes (about 10 changed lines)
