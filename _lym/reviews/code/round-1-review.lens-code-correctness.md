```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the `projectFacts` change in `internal/boardengine/render.go`.

**Per-function list**

- `projectFacts`, project with a body (`ship`, `Body: "Ship design."`, prefix `design-`): `slugCell` returns ``[`ship`](design-ship.md)``. `renderDesigns` writes `design-ship.md` for the same entry. Correct.
- `projectFacts`, project with an empty body (`tidy`, nested `inner`): `slugCell` takes the `Body != ""` false branch and returns the plain ``​`tidy` ``. `renderDesigns` skips the entry on `Body == ""`. Correct.
- `projectFacts`, `b.out.DesignPrefix` empty (zero `Outputs`): the link becomes `](ship.md)` and the written file is `ship.md`, because both use the same prefix value. They match, so correct.
- `projectFacts`, `b.out.DesignPrefix` set for both README.md and project files: it is read from `b.out`, one `Outputs` for the whole render. Both files get the same prefix. The design files and the project files all sit at the render root, so the relative link resolves from either. Correct.
- `projectFacts`, slug containing a character that needs escaping: `slugCell` does not escape it, but the same function already builds the task and note slug cells. The diff adds no new risk here.
- `projectFacts`, root project (Prime): `fileHeader` and the README heading call `projectFacts` only for real project entries. The diff does not change that, and I found no path where the root is passed with a body that has no design file. Correct.
- `renderDesigns`: not changed by the diff. Its `Body == ""` skip and `slugCell`'s `Body != ""` test are the same condition on the same field, so a dangling link or an unlinked readable body cannot occur.

I found no defects.

**Tool calls**
1. Read `render.go` lines 610-621: the `slugCell` body was not in my context, only its grep line, so I needed it to trace the branch.
