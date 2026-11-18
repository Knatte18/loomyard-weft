```yaml
lens: fix-claims
outcome: reported
```

Checked whether every bound, behaviour and fact claim the fix added holds against the code at HEAD.

**LOW**: card 08 (verb help text and refusal 1) and card 12 (PR-Review brief) say the `.scratch/` directory "is git-ignored".
- **Summary:** The claim holds in this worktree, but `.gitignore:23` doesn't make it true, so the fixer-report shouldn't cite that line, as the review's premise did.
- **Evidence:**
  - `git check-ignore -v .scratch` resolves to `loomyard/.git/info/exclude:9: /.scratch`.
  - `.gitignore:23` is `**/.scratch/`, and its trailing slash can't match `.scratch`, because `.scratch` is a symlink (`.scratch -&gt; …-weft/.lym/scratch`).
  - Files written through the link sit outside the worktree, so `git status` never lists them.
  - The card text doesn't depend on the `.gitignore` rule. The risk is only that it gets cited as the reason.

**LOW**: card 10's Intent says "Every text that says PR-Review's Stuck is always a Halt now says …".
- **Summary:** The list it gives isn't every such text, so the sentence is wider than the edits it names.
- **Evidence:**
  - `contracts/recipes/tapestry-recipe.yaml:29-31` lists PR-Review among the rows "and their Stuck halts the whole run". That is the same claim, and the new sentence doesn't name it. The earlier "no edge out" sentence may or may not be meant to cover it.
  - `internal/loomrecipe/testdata/tapestry-rows-before-motif.yaml:29` and `:620` carry the same wording. Those are testdata, so leaving them is defensible, but "Every text" is then not literally true.

The other added claims hold:
- **Card 7:** `lym loom run` builds the shed once (`shedverbs/run.go`). Today the rubric is filled once at build (`entries_burler.go:125`, `:373`). Reading the rubric before `roundProfile` is called in `Call` comes after the round resolution and hand-on exits and before any spawn or archive, so "no reviewer is spawned" holds.
- **Card 3:** the `driftTestEnv` and `TestDriftEntries` rows, and the bound line, match the code.
- **Card 9:** `wake` ends the wait, so the doc-comment claim holds.
- **Card 10:** `loomshed.NameCodeBouncer` exists (`progress_test.go:70`).

Outside my lens: whether `tapestry-rows-before-motif.yaml` needs the `on_reroute` edge too is for the affected-set lens.

Tool calls:
1. `git check-ignore -v` and `info/exclude`, to see which rule makes `.scratch` ignored. My context showed it as ignored but not the rule.
2. The order of profile build, archive and spawn in `BurlerReviewProducer.Call` (lines 298-460), to check the new "returned before the profile is built" claim. My context held only the method's doc comment.
3. A grep for "always a Halt" and "halts the whole run", to test card 10's "Every text" claim. My context had only the grep over recipes and Go/md files, not testdata.
