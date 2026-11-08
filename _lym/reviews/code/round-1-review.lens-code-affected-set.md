```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens, judging what the commit's wording change forces outside `plugins/ly/skills/board-policy/SKILL.md`.

**Per change, the set found:**
- New bullet, "a note or bundle weighed for a tapestry run is proposed as a `motif` task" (`plugins/ly/skills/board-policy/SKILL.md:44-48`).
  - `contracts/orchrole/orch-role.md:44-49` names the inner run as "a tapestry, motif, darn or sampler, as the entry's `recipe` says". That describes what a run is, not how a recipe is chosen, so it doesn't conflict. Covered.
  - `plugins/ly/skills/board/SKILL.md:63-71` lists recipe names and says the Sketch is broken down by "triage". It names no choice rule. It is slightly stale beside the new "promoting the Sketch runs raddle" wording, but it doesn't contradict it. Not a finding.
  - `contracts/stencils/raddle/raddle-template-agent.md:20,47` and `contracts/specs/raddle-proposal-spec.md` hold raddle's own shape choice. The Goal says to leave it unchanged, and the diff leaves it unchanged. Covered.
  - Other shipped text that sends a large change straight to tapestry: my grep found none in the plugin skills, `contracts/` or the stencils. The hits in `README.md:262`, `docs/overview.md` and the glossary only define recipes. Covered.
- Reworded "split an entry" bullet (`plugins/ly/skills/board-policy/SKILL.md:84-85`).
  - The nearby line, "Never propose one note alone as a loom task" (`:41-43`), sits next to the new bullet without contradicting it. A tapestry-sized note proposed as a motif task is itself a loom run, which may read as tension. It is not a conflict.
  - `docs/overview.md:1576` says "The Board task's `recipe` must be `tapestry`, `darn` or empty". This is code-level documentation, not shipped instructions, so it is out of the Goal's scope.
- Repository rules: no `PATTERN` entry declares changes for this diff, and none governs skill prose. Nothing to land.
- Commit message: it claims the proposal as a `motif` task, raddle's recipe choice standing, and "A Sketch is split only through raddle, never by hand". The diff bears out all three.

**Findings**

1. **LOW / scope** — `plugins/ly/skills/board-policy/SKILL.md:84-85`
   - The reworded bullet adds "A Sketch is split ... never by hand", a rule the Goal didn't ask for. The Goal asks only that tapestry-sized notes run as a Motif and that raddle's choice stands.
   - The rule is plausibly implied by "raddle then splits its Sketch", and it also matches `plugins/ly/skills/board/SKILL.md:67-68`. The risk is that it forbids a legitimate operator-approved hand split.
   - Evidence: the decision record `## Goal`, and the diff hunk at `:81`.

2. **LOW / consistency** — `plugins/ly/skills/board-policy/SKILL.md:47-48` against `:49-50`
   - "never change the recipe of a task raddle filed" is a new restriction beyond the Goal's "that choice stands". It may overlap the operator-names-darn rule at `:50`: if the operator names darn for a raddle-filed task, the two lines disagree.
   - Fix: scope it to "you never change it yourself; the operator may name darn".

**Tool calls**
1. Grep for other shipped text that routes a large change straight to tapestry. My context only had the `tapestry` grep for `plugins` and `contracts`.
2. Grep for the matching lines in `README.md`, `docs/overview.md`, the board skill, `runprofile/template.yaml` and the glossary. I needed to confirm that those hits only describe recipes.
