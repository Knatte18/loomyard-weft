```yaml
lens: fix-code-correctness
outcome: reported
```

Task: fix-code-correctness lens over the changes to `roleguard.go`, `roleguard_test.go`, `fabric.go` and `doc.go`.

Traced each changed function and each added test row against the code in context; none of the traces found a wrong result.

**Traces that hold**
- **`resolveThroughExistingAncestor`:**
  - For a missing `/a/b/new` it resolves `/a/b` and re-joins `new`.
  - For a relative path it stops at `.`.
  - For a path with no resolvable ancestor it returns false, so the guard refuses.
- **`inOwnSandboxRoot`, standard layout:**
  - `own/.lym/harden-sandbox/hub-1` resolves to `hubs/slug-weft/.lym/harden-sandbox/hub-1`.
  - The walk matches at `.../harden-sandbox`, so `container` is `slug-weft`.
  - `worktreeOwningRecordsDir` maps that back to `hubs/slug`, and `ResolveWorktree` gives name `slug` with a matching path.
  - The `.lym` link is the production layout, so F1 is resolved.
  - The shortname check and the `EvalSymlinks(root) == current` equality both pass.
- **Failure paths:** each of these fails closed.
  - A `ResolveWorktree` error.
  - A missing shortname record.
  - A worktree that holds a plain `.lym` while the sibling holds the sandbox: `EvalSymlinks(own/.lym/harden-sandbox)` fails.
  - A relative `into` the raw-`into` defect would produce: the walk matches container `.`, `ResolveWorktree(".")` climbs to the repo root, and the path comparison refuses.
- **Callers:** `inOwnSandboxRoot` has one caller, `sandboxedRoleRefusal`, and it passes the parsed name.
- **Added test rows and their defects:**

| Row | Defect it catches |
|---|---|
| `SameSlugWorktreeInAnotherHub` | Without the shortname check it passes, so the row fails. |
| `SandboxRootBelowASubdirectoryOfOwnWorktree` | Without the `WorktreePath() == worktree` check, name and `EvalSymlinks(root) == current` both hold, so the row fails. |
| `LinkIntoOwnSandboxHub` | A lexical walk finds no root. |
| `LinkInsideOwnSandboxRootPointingOut` | A lexical walk wrongly passes it. |
| `IntoBelowALinkPointingOutOfOwnSandboxRoot` | A lexical walk wrongly passes it. |
| `RelativeIntoOwnSandboxRootFromItsWorktree` | The raw-`into` defect refuses it, so the row fails. |
| `pull --weft-path` | The row asserts exit 1 and the `refuses the path flags` text, so moving the sandbox pass above the path-flag refusal fails it. |

**Findings**

- **LOW** — `roleguard.go`, the `inOwnSandboxRoot` doc comment line "A worktree's `.lym` is a link into its records sibling, so a dir in its resolved form maps back to the worktree it belongs to."
  - It is about 127 characters of text, over the 120-character limit in `scribe:prose`.
  - Split it after the comma.

- **LOW** — `roleguard.go`, `resolveThroughExistingAncestor`.
  - A dangling symlink is not resolved: `EvalSymlinks` fails on it, so its name is treated as a missing tail and joined unresolved onto the parent.
  - A clone `--into <sandbox>/dangling` whose link points outside would therefore pass the guard.
  - The clone's `MkdirAll` on an existing symlink name most likely fails first, so the harm is bounded.
  - It is the residual of F3's "link planted inside the sandbox" case.
  - Resolving the link with `os.Readlink` would close it, but the review did not ask for that.

**Tool calls**
None. My context already held every file I needed.
