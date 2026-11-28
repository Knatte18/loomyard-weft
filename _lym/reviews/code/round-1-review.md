---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/runprofile/reinterpret_test.go:94
    summary: TestReinterpretJacquardBlocks has no row for a defaults entry supplied through a merge key, on either the set or the drop path, none for the flag-only return, and none for a profile holding a jacquard block but no tapestry block
    fix: "Add three rows to the tests table of TestReinterpretJacquardBlocks in internal/runprofile/reinterpret_test.go, after the merge-key row. (1) name \"a defaults entry supplied through a merge key is flagged when the default would be set\", existing \"<<: {defaults: {tapestry: solid}}\\nsolid:\\n  tapestry:\\n    tapestry: {plan: {max_bounces: 2}}\\n\", wantJacquard {\"solid\": {\"tapestry\": {\"plan\": {\"max_bounces\": 2}}}}, wantRewrote [\"profiles.solid.jacquard: set\"], wantFlagged [\"profiles.defaults.jacquard: the default is supplied through a merge key\"], wantDefault empty. (2) name \"a defaults entry supplied through a merge key is flagged when the default would be dropped, and the file comes back unchanged\", existing \"<<: {defaults: {jacquard: solid}}\\nsolid:\\n  darn:\\n    darn: {writer: opus}\\n\", wantFlagged [\"profiles.defaults.jacquard: the default is supplied through a merge key\"], no wantRewrote, wantDefault \"solid\"; add a wantUnchanged bool field to the struct and, when set, assert string(got) == tt.existing. (3) name \"a jacquard block on a profile holding no tapestry block is left as it is\", existing \"solo:\\n  jacquard:\\n    tapestry: {plan: {max_bounces: 1}}\\n\", wantJacquard {\"solo\": {\"tapestry\": {\"plan\": {\"max_bounces\": 1}}}}, no lines, wantUnchanged true."
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/runprofile/reinterpret.go:134
    summary: Replacing an anchored jacquard node that another profile's jacquard block aliases leaves a dangling alias, so the rewritten profiles.yaml does not parse, and applyReinterpretations writes it with no load check
    fix: "In ReinterpretJacquardBlocks, before the block.Encode and setMappingValue(profileNode, jacquardRecipe, &block) at reinterpret.go:130-134, read old := mappingValue(profileNode, jacquardRecipe); when old is non-nil and it or any node under it carries a non-empty Anchor (walk old.Content recursively and check yaml.Node.Anchor), append a flagged line instead of replacing, and continue: \"%s: profile %s's %s block carries a YAML anchor another entry may alias, so it is left as it is; way forward: set the block by hand with `lym config profiles --set %s.%s='<json object>'`\" with blockKey, name, jacquardRecipe, name, jacquardRecipe. Add a row to TestReinterpretJacquardBlocks: existing \"solid:\\n  tapestry:\\n    tapestry: {plan: {max_bounces: 5}}\\n  jacquard: &j\\n    tapestry: {plan: {max_bounces: 2}}\\nwide:\\n  tapestry:\\n    tapestry: {plan: {max_bounces: 2}}\\n  jacquard: *j\\n\", wantFlagged [\"profiles.solid.jacquard: profile solid's jacquard block carries a YAML anchor\"], and the existing parse of the output then proves it still parses."
  - id: F3
    severity: NIT
    class: design
    origin: lens:code-correctness
    location: internal/runprofile/reinterpret.go:183
    summary: When the defaults mapping holds jacquard both as its own key and through a merge key, the drop removes only the own key and reports the default dropped while the merged value still supplies it
    fix: "In reinterpretJacquardDefault, immediately before `if !removeMappingKey(defaultsNode, jacquardRecipe) {` at reinterpret.go:183, add: `if defaultsNode != nil && mappingValue(defaultsNode, \"<<\") != nil { return nil, []string{mergeFlag} }`, so a defaults mapping holding a merge key is flagged for a hand edit and never half-dropped. Cover it with a row whose existing is \"defaults: {<<: {jacquard: solid}, jacquard: solid}\\nsolid:\\n  darn:\\n    darn: {writer: opus}\\n\" and wantFlagged [\"profiles.defaults.jacquard: the default is supplied through a merge key\"]."
  - id: F4
    severity: NIT
    class: consistency
    origin: lens:code-tests
    location: internal/runprofile/runprofile_test.go:124
    summary: The new Default(jacquard) assertion in TestTemplateLadder repeats what TestLoad's no-file row already pins, that the template's defaults.jacquard is solid
    fix: "Delete the three-line `if got, err := profiles.Default(\"jacquard\"); got != \"solid\" || err != nil { ... }` block in TestTemplateLadder (internal/runprofile/runprofile_test.go:124-126), and the doc-comment line \"jacquard's default is solid.\" above TestTemplateLadder. TestLoad's row \"no file reads the template's ladder, docs and defaults\" keeps the jacquard default pinned through its wantDefaults map."
---

The diff makes the change the board entry asks for.
The template's `speed`, `light`, `solid` and `wide` profiles hold jacquard blocks equal to their tapestry blocks minus `loom.discussion`, and `defaults.jacquard` is `solid`.
`ReinterpretJacquardBlocks` sits beside `ReinterpretArrasDefault`, registered after it, and the walk is unchanged.
It derives each profile's jacquard block from its tapestry block and sets or drops `defaults.jacquard`, as D7's two cases ask.
On the shipped template the rewrite is a fixed point, so a hub left as shipped gets no rewrite and no notice.
The change description's claims hold against the diff, and its one-way door with a `profiles` blast radius fits a rewrite of the hub's profiles file.
The kept findings are edge-path gaps, none blocking.

### [LOW] Merge-key, flag-only and tapestry-less rows are missing from the rewrite's table test

**Location:** `internal/runprofile/reinterpret_test.go:94`
**Issue:** `reinterpretJacquardDefault` returns its merge-key flag on two paths, and `ReinterpretJacquardBlocks` returns the file unchanged with flags alone.
A profile holding a jacquard block and no tapestry block is left as it is.
No row reaches any of these, so removing a `mergeFlag` return or the `holdsJacquard[name] = held` bookkeeping fails no test.

### [LOW] An anchored jacquard block that another entry aliases is replaced, leaving a dangling alias

**Location:** `internal/runprofile/reinterpret.go:134`
**Issue:** `setMappingValue` swaps the value node of `solid.jacquard` for a freshly encoded one.
A `wide.jacquard: *j` whose derived block already matches keeps its alias to the anchor that left the tree, and the encoded file no longer parses.
`applyReinterpretations` (`internal/hubreconcile/reinterpret.go:68-73`) writes it without a load check.
Reaching it needs an anchored jacquard block written by hand before the one-time run, so it is unlikely, but the result breaks the hub's profiles file.

### [NIT] A defaults jacquard held both as its own key and through a merge key is reported dropped while it survives

**Location:** `internal/runprofile/reinterpret.go:183`
**Issue:** `removeMappingKey` removes the own key and returns true, so the line reports the default dropped.
The merged value still supplies `defaults.jacquard`, so the line is untrue, and the default can still name a profile with no jacquard block.

### [NIT] Default(jacquard) in TestTemplateLadder duplicates TestLoad's no-file row

**Location:** `internal/runprofile/runprofile_test.go:124`
**Issue:** `TestLoad`'s row "no file reads the template's ladder, docs and defaults" already asserts the template's defaults map, `jacquard: solid` included.
The new assertion pins nothing more.

## Rejected

- code-affected-set F1, profiles keys named bare in comments (`defaults.jacquard`, `defaults.tapestry`): the repository reads a seed-only module's entries as names the operator chooses, not template keys (`internal/configreg/keynames_enforcement_test.go:36`), and the full-name sweep `e2ac87a8a` edited `internal/runprofile/doc.go` and left `defaults.arras` bare, so `PATTERN-config-key-names` does not reach them.
- code-tests MEDIUM on the merge-key flag of `defaults.jacquard`: folded into F1 and lowered to LOW, since only a hand-written merge key reaches it.
- code-tests LOW on the aliased defaults path: an aliased `defaults` entry needs an anchor on another top-level entry, and every other top-level entry is a profile, so no sensible file reaches it.
- code-tests LOW on no test after the real follow: the rewrite reads only bytes, the "tuned shipped profile" row feeds it the post-follow shape, the follow is unchanged, and the configreg test pins the order after the arras entry.
- code-correctness NIT on whole-valued floats re-encoded as ints: no module template holds a float key, so no tapestry block that passes the profiles load check carries one.
- Handler: a starting profile whose tapestry block the operator removed still gains the template's jacquard block through the follow. D7 keeps the follow unchanged and asks the reinterpretation for only its two cases, and the follow reports the unit it added.
- Handler: `loomModule` and `discussionArea` redeclare values `internal/loomengine` holds as unexported constants. Those are not reachable from `runprofile`, and package-local constants for the `loom` module name are the repository's existing practice (`internal/loomcli/raddlewiring.go:44`, `internal/loomcli/parentseatwiring.go:31`).
