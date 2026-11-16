---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/boardengine/render.go:883
    summary: runFieldLines adds the board.json key names pull_request and on_land as repeated string literals, against PATTERN-code-conventions' named-constant rule
    fix: 'In internal/boardengine/task.go, declare two unexported constants beside the Task type, pullRequestKey = "pull_request" and onLandKey = "on_land", each with a one-line comment naming it as the board.json key of its field. In render.go runFieldLines, build the lines as pullRequestKey + ": " + t.PullRequest and onLandKey + ": " + t.OnLand. Replace the existing literals of the same keys with the constants too, so each is referenced everywhere, at replay.go:207 and :214 (the Name values), front.go:33, :37 and :57, and the two entries in the key list at store.go:500. The struct tags at task.go:49 and :67 stay literal, since Go tags cannot reference a constant.'
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/boardengine/doc.go:141
    summary: The package doc says the README entry cell names a task's profile, but the profile renders in the Recipe cell and the diff's own entry-cell paragraph leaves it out
    fix: 'In internal/boardengine/doc.go:141, replace "The README entry cell names only a profile the task carries." with "The README''s Recipe cell names only a profile the task carries."'
---

### [BLOCKING] Run-field key names are repeated literals

**Location:** `internal/boardengine/render.go:883`, `internal/boardengine/render.go:886`

**Issue:** `runFieldLines` writes `"pull_request: "` and `"on_land: "`.
The summary describes each line as `<board.json key>: <value>`, so these are the field's on-disk key names, and a rename of either key leaves the README line stale with no failure.
`PATTERN-code-conventions` (Named constants) requires a name that can go stale to be one named constant referenced everywhere, never a repeated literal.
The same keys already repeat as literals at `replay.go:207`, `replay.go:214`, `front.go:33`, `front.go:37`, `front.go:57` and `store.go:500`, and the diff adds two more instead of introducing the constant.
The constraints section makes every listed-constraint violation BLOCKING regardless of size.

### [LOW] Package doc places the profile in the entry cell

**Location:** `internal/boardengine/doc.go:141`

**Issue:** The line reads "The README entry cell names only a profile the task carries."
`recipeCell` renders the profile after ` · ` in the Recipe column, and `entryCell` never renders it.
The diff rewrote the entry-cell paragraph at `doc.go:366-367` to list the cell's run fields and run settings, and its design keeps profile out of the cell, so the two paragraphs now disagree about what the entry cell holds.

## Focus verdicts

- **Field coverage in `runFieldLines`.** Not a defect.
  `notify_on_land` only asks the dobby for a creel notice, and the dobby clears it (`dobbyengine/landnotice.go`), so it does not change how the run goes.
  `isolated` and `depends_on` already decide the row's section and its After cell.
  `owner_machine` is never recorded on a task: `PATTERN-owner-machine` says a task records none and a load drops the key, and `lym board get` reports the owner read through the parent.
  `front` marks lym's own filing and drives no run step that `on_land` does not already show.
  Recipe and profile already show in the Recipe column of the Running, Ready, layer and Draft tables through `recipeCell`, so the README answers the entry's question for both, and bullets would repeat them.
- **`on_land` in `entryCell`.** Shown only when set, and only front tasks set it, so the bullet appears exactly where it is meaningful.
  The values are validated upstream (`pull_request` to on, off or merge-in; `on_land` against the `fronttask` action table), and `codeSpan(cellText(line))` escapes a pipe and fences a backtick the same way it does for run settings.
  The appended slice is fresh per call, so the `append` aliases nothing.
- **`TestRenderReadmeGolden` and `readmeFixture`.** Row T1 pins brief, `pull_request`, `on_land`, sorted settings; row T3 pins fields before settings without a brief; every other row pins the empty case.
  `readmeFixture` has one caller (`render_test.go:342`), so no other golden breaks, and the two new comment lines state what those rows assert.
- **`doc.go` and the commit message.** The rewritten entry-cell paragraph and the commit messages match `render.go`.
  The one stale claim about the entry cell is F2; no spec, stencil or `docs/` page describes the cell's contents.

## Rejected

- Recipe and profile get no bullet, although the board entry names them as examples: rejected as a finding, because the entry asks for fields "the cell omits", its goal is that the operator can see them from the README, and the Recipe column already shows both on every table that lists a runnable task.
- The summary's claim that `owner_machine` shows in the Owner column, which only the Running table has: rejected, because a task never records `owner_machine` and reads its owner from its parent, which the project heading's `Owner` part already shows, so the exclusion holds and the claim misleads no reader about what the diff does.
