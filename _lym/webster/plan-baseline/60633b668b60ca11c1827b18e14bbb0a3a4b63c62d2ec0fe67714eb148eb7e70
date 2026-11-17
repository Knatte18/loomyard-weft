# Card 1 — Fabric verbs for a working-tree id and a diff of two ids

**Create:**
- `plan:internal/fabricengine#Fabric.WorktreeTreeID` -> `func (f *Fabric) WorktreeTreeID() (string, error)`
- `plan:internal/fabricengine#Fabric.ChangedPathsBetween` -> `func (f *Fabric) ChangedPathsBetween(fromTree, toTree string) ([]string, error)`
- `plan:internal/fabricengine_test#TestFabric_WorktreeTreeID` -> `func TestFabric_WorktreeTreeID(t *testing.T)`

**Extends:**
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/fabricengine#Fabric.ReadTaskPatch`
- `internal/fabricengine#Open`

**Intent:**
D5 needs two general fabric verbs, shaped for no consumer (`PATTERN-fabric-boundary`).

`WorktreeTreeID` hashes what `git add -A` would stage in the code checkout's working tree into a git tree object and returns the tree's id.
It builds the tree in a throwaway index: a temp directory from `lymtemp.MkdirTemp`, `GIT_INDEX_FILE` pointing into it, `git read-tree HEAD` to seed it, as `newIntegrateBuild` seeds its temporary index, then `git add -A` and `git write-tree`, all through `gitexec.RunWithEnv` in the checkout.
The checkout's own `info/exclude` and ignore files apply, so the excluded and ignored untracked paths stay out, as they do for `ReadTaskPatch`'s untracked set;
the seed keeps every tracked file staged whatever the ignore rules say, as `git add -A` against the real index does.
The real index, HEAD and every ref are untouched, and the temp directory is removed before it returns.
Uncommitted edits count, so a fix that is committed and one left uncommitted with the same bytes give the same id.

`ChangedPathsBetween` runs `git diff-tree -r --name-only --no-renames <fromTree> <toTree>` in the checkout and returns the changed paths, checkout-relative with forward slashes, sorted.
A tree id git no longer holds is a returned error; callers read it as a missing cache.

The doc.go paragraph that lists the read-only verbs gains both.

`TestFabric_WorktreeTreeID` is tier1 (git), one table over a hermetic fixture built as `TestFabric_ReadTaskPatch` builds its fabric.
Rows: an unchanged tree gives the same id twice; an uncommitted edit to a tracked file changes the id and `ChangedPathsBetween` lists exactly that path; committing that edit leaves the id equal to the uncommitted one; a new untracked, non-ignored file changes the id and is listed; an untracked ignored file leaves the id unchanged; an edit to a tracked file that an `info/exclude` pattern matches changes the id and `ChangedPathsBetween` lists it; after the calls `git status --porcelain` and `git rev-parse HEAD` read as before, so the index and refs are untouched; an unknown tree id makes `ChangedPathsBetween` return an error.

**ImpactSummary:** Two additive read-only verbs on `*Fabric`; no existing verb or caller changes.
