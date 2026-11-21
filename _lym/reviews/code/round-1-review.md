---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: internal/fabricengine/remoteleftover.go:52
    origin: lens:code-affected-set
    summary: "The leftover refusal for a weft branch tells the operator to run git push origin --delete with the short branch name, which cannot reach refs/lym/heads/ in a one-repository hub."
    fix: "In internal/fabricengine/remoteleftover.go, make ErrRemoteLeftover carry the full origin ref name of the leftover weft branch, set where probeWeftLeftover builds the weft-kind error from originNamespaceAt(weftRoot).Branch(weftBranch). Format the leftoverDivergedAdoptedWeft message's push --delete command with that full ref instead of the short e.Branch. Leave the warp-branch message as it is. Add a one-repository row to the add_remoteleftover_integration_test.go case that asserts this message."
  - id: F2
    severity: MEDIUM
    class: consistency
    location: internal/fabricengine/remoteteardown.go:199
    origin: lens:code-affected-set
    summary: "The records-side manual archive-tag command pushes refs/tags/<tag> with no destination, which bypasses the records store's push refspec and writes a records tag into the code origin's refs/tags/ in a one-repository hub."
    fix: "In failedRemoteSteps in internal/fabricengine/remoteteardown.go, build the records-side command (the weftRoot branch at line 199) as git -C <weftRoot> push origin refs/tags/<tag>:<ns.Tag(tag)>, with ns from originNamespaceAt(weftRoot). When that read errors, omit the manual tag push, as the empty-weftRoot case already does. Leave the code-store command at line 186 unchanged. Update any test that asserts the records-side text."
  - id: F3
    severity: LOW
    class: design
    location: internal/fabricengine/destroy.go:1789
    origin: lens:code-correctness
    summary: "A failed git config write of the board's upstream after both seed pushes landed returns a bare error with a leftover hub, and the later unseeded-board refusal then names a way forward (--single-repo with --reset) that origin now refuses."
    fix: "In the upstream-config loop at the end of pushCloneSeed, wrap the error with the right way forward: origin now holds the seeded hub, so run a plain lym fabric clone <url> to join it, or set branch.main.remote and branch.main.merge in the records store by hand."
  - id: F4
    severity: LOW
    class: design
    location: internal/gitrepo/pushretry_test.go:66
    origin: lens:code-tests
    summary: "No row pins the namespaced tag path of the push-refusal retry: pushArchiveTag sends refs/tags/<t> to refs/lym/tags/<t> in a one-repository hub, and every namespaced row uses a branch."
    fix: "In TestRetryablePushRefusal_ClassifiesByTheReasonOnThePushedRefsLine, add a row for ref refs/tags/archive/s/abc against the line ' ! [remote rejected] refs/tags/archive/s/abc -> refs/lym/tags/archive/s/abc (failed)' that wants retryable under the namespaced RefNamespace and not retryable under the identity namespace."
  - id: F5
    severity: NIT
    class: consistency
    location: internal/gitrepo/doc.go:284
    origin: lens:code-affected-set
    summary: "The bullet saying git.PlainOpenWithOptions with EnableDotGitCommonDir is mandatory is stale, since goGit no longer calls it and resolves the common dir itself."
    fix: "Reword the bullet at internal/gitrepo/doc.go:284 to say goGit must resolve a linked worktree's common dir, and that the convenience open call is not used for that reason."
---

### [MEDIUM] Leftover refusal names a delete command that cannot reach the namespace

**Location:** internal/fabricengine/remoteleftover.go:52
**Issue:** D4 states that a short name cannot reach `refs/lym/heads/`, so teardown deletes by the full namespaced name.
The diff makes every delete, listing and fetch in this file namespaced, but `ErrRemoteLeftover.Error()` still prints `git push origin --delete <short>` for the weft branch.
In a one-repository hub that command either does nothing useful or targets the wrong ref, so the way forward the refusal names does not work.
The message is a string with no namespace in reach, so the full ref must be carried in the error at construction.
No test asserts this text, so nothing blocks the change.

### [MEDIUM] Records-side manual tag push lands in the code origin's tag list

**Location:** internal/fabricengine/remoteteardown.go:199
**Issue:** The way forward for a failed records archive says to run `git -C <weftRoot> push origin refs/tags/<tag>`.
A push with an explicit source and no destination bypasses `remote.origin.push`, as D1 and D2 record, so in a one-repository hub it writes `refs/tags/archive/...` on the code origin.
That is the clutter the namespace exists to avoid, and the next remote cleanup then reads the tag from the wrong place.
The code-store command at line 186 is right as written.

### [LOW] Seed's last step can fail with no teardown and a wrong follow-up way forward

**Location:** internal/fabricengine/destroy.go:1789
**Issue:** After the records branch and the board both landed, `pushCloneSeed` sets `branch.main.remote` and `branch.main.merge`.
A failure there returns a bare error, leaves the hub in place and leaves the board without an upstream.
`refuseUnseededBoard` then refuses every later board push and names `--single-repo ... --reset`, which origin now rejects because it holds the board.
A fresh `git init` store makes the write fail only on a broken disk, so this is LOW.

### [LOW] Namespaced tag path of the push-refusal retry has no row

**Location:** internal/gitrepo/pushretry_test.go:66
**Issue:** `namesRef` maps a namespaced destination back through `LocalRef`.
The tag case, source `refs/tags/<t>` and destination `refs/lym/tags/<t>`, is the one `pushArchiveTag` sends.
`TestRefNamespace` covers the mapping alone, and the retry rows cover branches alone, so a break in the tag composition passes every test.

### [NIT] Stale "mandatory" bullet in the gitrepo package doc

**Location:** internal/gitrepo/doc.go:284
**Issue:** The bullet still states that `git.PlainOpenWithOptions` with `EnableDotGitCommonDir: true` is mandatory.
The diff edited the sentence under it so that `goGit` resolves the common dir itself, which leaves the bullet's lead line describing a call the code no longer makes.

## Rejected

- Peeled `^{}` handling untested with annotated tags (lens:code-tests, second finding): fabric creates archive tags with a plain `git tag`, so they are lightweight, and the peel logic is unchanged and covered under the identity namespace; the new namespaced mapping in front of it is covered by `TestRefNamespace`.
- A records-store remote `HEAD` read answering `main` (lens:code-scope, open point): `remoteDefaultBranch` has two callers, `addadopt.go:68` and `cleanupremotewarp.go:87`, and both pass the code store's worktree; no records-store path reads origin's `HEAD`, so there is nothing to translate.
- `originNamespaceAt` answering identity for a directory with no `.git` entry (lens:code-correctness, noted as not a finding): D2 requires exactly this for a missing store, and every caller passes a worktree root.
- `--single-repo` with `--reset` clean path has no row (lens:code-tests, in its list): the join test covers `--reset` with another and a missing subpath, and the seed's `--reset` shares `checkResetTarget` and `resetHub` with the two-repository clone, so no behaviour the diff adds is uncovered.
