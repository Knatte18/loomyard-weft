```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals that the fixer carried out (F1's wiring, F2, F3, F4).

1. LOW, card 1, the new `internal/runproc/doc.go` Intent bullet: "or the strand role of a fixer that `internal/gatefixer` declares (`gate-fixer`, `verify-fixer`)".
   Summary: the proposal makes a lower-layer package doc list a higher package's roles. When `gatefixer` adds a fixer role, runproc's doc goes stale and nothing flags it.
   Evidence: every current `Register` caller passes a runproc constant (`RoleSession`, `RoleBatten`, `RoleLoop`, `RolePush`, `RoleCapture`, `RoleSync`, `RoleDobby`, `RoleGate`), so runproc's doc never had to name an outside package before. Naming the group ("a fixer role `internal/gatefixer` declares") would be enough, and the parenthesis duplicates card 1's `Roles()`.

2. LOW, card 7, the new `contracts/specs/shed-recipe-spec.md` Intent bullet: "a `&lt;module&gt;.&lt;key&gt;` setting holding a model-spec".
   Summary: the spec text the proposal prescribes leaves out that an empty setting value turns the fixer off. That fact is the contract a recipe author needs, and the same card states it.
   Evidence: card 7's `splitGateFixer` bullet says "An empty setting value returns a nil request: the fixer is off and the row behaves exactly as today", and its refusal says "set the key to a model-spec … or `""` to turn the fixer off".

F1's wiring (wrapper in `specFor`, `ConsumeTackler`, `loomConsumeAttempt`; verdict-check guards left bare) and F3's comment list: I built no case where the change is wrong. `EndRole` skips the caller and its ancestors, the fixer panes are reed panes rather than descendants of the driver, and a halted run has no wrap waiting on the fixer, so ending it under the run or status lock blocks nothing.

Tool calls:
- grep of `runproc.ChildRecord`/`SelfRecord` callers outside runproc, to test whether runproc's doc already covers roles that aren't its own constants. My context didn't have the full caller list.
