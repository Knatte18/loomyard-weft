```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced where the records store's remote refs are written, read and parsed, and checked that D1–D4's namespace mapping holds on those paths.

**Findings**

1. **BLOCKING — D2/D4: `gitrepo` pushes by URL with explicit `refs/heads/` destinations, outside D2's fabric-only translation.**
   - Evidence: in `internal/gitrepo/push.go`, `r.runChecked("push", dest.url, source+":refs/heads/"+branch)`; the force-with-lease paths spell `--force-with-lease=refs/heads/<branch>:<sha>` and `sha+":refs/heads/"+branch`.
   - A push to a URL, not to a remote name, never applies `remote.origin.push`, so D1's mapping is bypassed. Records branches would land in `refs/heads/` on origin, which is exactly the clutter D1 exists to avoid.
   - D2 says the translation covers "every ref name fabric spells" and "lives in fabricengine". The record is silent on `gitrepo`, which builds these refspecs itself.

2. **BLOCKING — D2: `pushdestination.go` writes upstream config with an untranslated name.**
   - Evidence: `branch.<b>.merge` is set to `refs/heads/<b>`.
   - In the records store, the upstream must name `refs/lym/heads/<b>`. Otherwise `@{u}` and ahead/behind reads point at a code-branch name on the shared origin, or at nothing.
   - This is another writer of the state D2 changes, and the record does not mention it.

3. **MEDIUM — D2/D4: `ls-remote` flags and output parsing defeat a translation applied only to the names fabric spells.**
   - Evidence: `remoteleftover.go` and `cleanupremotewarp.go` run `ls-remote --tags origin <pattern>` and strip `refs/tags/` from the output. `archive.go`, `remoteleftover.go`, `recordsprobe.go`, `pushclassified.go` and `originreconcile.go` run `ls-remote --heads`.
   - `--heads` and `--tags` filter to `refs/heads/` and `refs/tags/`, so they return nothing under `refs/lym/`. The prefix-strip parsing also needs the reverse mapping.
   - The archive-coverage check and the leftover-branch replacement would then read "absent". That can wrongly refuse, or replace a branch whose tag does exist.
   - D2 needs to say that translation covers flags and parsed output both ways, not only spelled names.

4. **MEDIUM — D1/D2: the records store's origin also holds every code branch under `refs/heads/`.**
   - Evidence: in `originreconcile.go`, `args := []string{"ls-remote", "--heads", originRemote}`. Run in the records store, an untranslated call lists code branches as if they were records branches.
   - In a two-repository hub, the weft origin's `refs/heads/` held only records. The one-repository layout breaks that invariant, so an untranslated listing misreads rather than fails.
   - The record should state the invariant that no records-store path reads origin's `refs/heads/`.

5. **MEDIUM — D4: `gitrepo/pushretry.go` matches push result lines only against `refs/heads/` and `refs/tags/`.**
   - Evidence: `namesRef` checks `name == "refs/heads/"+ref || name == "refs/tags/"+ref`. With a mapped destination of `refs/lym/heads/x`, the result line goes unmatched.
   - The retry and rejection classification would then misread the outcome of a records push. The record is silent.

6. **LOW — D4: the remote-teardown way-forward hints survive.**
   - `remoteteardown.go` tells the operator to run `git -C <weft> push origin refs/tags/<tag>`. A bare source pushed to the remote name goes through the configured push refspec, so it maps correctly (the git 2.53.0 probe showed this mapping).

7. **LOW — D4: `cleanupremotewarp.go` runs `ls-remote --symref origin HEAD`.**
   - If this runs in the records store of a one-repository hub, it reads the code repository's default branch. Fabric must not take that as the records store's main branch. This needs checking against the call site.

**Traced, survives**
- D3 clone wiring: the Bolt push through the `CloneAndWire` walk survives, since a push to the remote name is mapped.
- D5: removing `statuscommit`'s protected and stalled seams survives. No two-repository path sets them.
- D7: the reverted merge-state probe survives, since every hub runs the two-repository local paths.
- `ownerarchive.go`'s fetch of `refs/tags/<tag>` is a name fabric spells, so D2's translation covers it.
- No production code runs `remote set-url` or rewrites `remote.origin.fetch`, so nothing undoes D2's refspec, the layout's only record.

**Tool calls**
- One grep for remote-config writers and spellings of `refs/tags/` and `--tags`. My context did not show how fabric spells tag refs.
- One grep for `ls-remote` callers and `gitrepo` push refspec and output parsing. My context had no view of `gitrepo`'s push internals.
