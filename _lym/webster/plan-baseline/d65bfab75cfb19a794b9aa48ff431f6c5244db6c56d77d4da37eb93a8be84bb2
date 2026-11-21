# Card 8 — docs-and-pattern

**Create:**
- `pattern/PATTERN-single-repo-hub.md`

**Edit:**
- `docs/getting-started.md`
- `PATTERN.md`
- `pattern/PATTERN-code-conventions.md`

**Extends:**
- `README.md`

**Intent:**
Describe the one-repository hub as the earlier cards build it (D9), restore its PATTERN entry, and declare the `PATTERN-code-conventions` change, each with the text `## PATTERN changes` declares.

`docs/getting-started.md`'s "Build and make a hub" section today says a hub needs two repositories.
It is rewritten to offer both: the two-URL clone, and `lym fabric clone --single-repo --shortname <short> <project-url>` for one repository,
with a plain `lym fabric clone <project-url>` joining such a hub from another machine.
It lists D9's hazards:
- a `git push --mirror` from either store deletes the other's refs on origin;
- a `git clone --mirror` carries the records;
- a bare `git push` in the records store pushes every local tag into the namespace;
- branch protection and rulesets do not cover `refs/lym/`;
- `--single-repo` over the code repository of an existing two-repository hub starts a separate hub.

`README.md` gains a short paragraph, beside its hub description, on the one-repository layout: records and board under origin's `refs/lym/`, which GitHub's branch and tag lists do not show, with a pointer to `docs/getting-started.md` for the hazards.

`PATTERN.md` gains this entry in its "Fabric and git" section:

```
- `PATTERN-single-repo-hub` — Running a hub as one repository: a code store and a records store of one origin, records and the board under origin's `refs/lym/`, mapped by the records store's refspecs; only fabric reads the layout. — [background](pattern/PATTERN-single-repo-hub.md)
```

`pattern/PATTERN-single-repo-hub.md` is its background, rewritten to D1 to D4:
the layout (two local stores of one origin, the namespace, the fixed board branch `main`, `lym.primeRecordsBranch`);
the layout as the records store's fetch refspec, read and cached only by fabric's `originNamespaceAt`;
the namespace passed to every records-store remote operation in `gitrepo` and fabric, with no bare push;
the clone's check, refusals and leased, ordered seed push; and the pair lifecycle running the two-repository paths.
Each rule names the test that holds it from cards 2, 4, 5, 6 and 7; its rules carry no `(test)` marker, since the entry line carries none.

`PATTERN.md`'s `PATTERN-code-conventions` line, beside the restored entry, takes the summary `## PATTERN changes` declares:

```
- `PATTERN-code-conventions` — Writing code: a package's `doc.go` documents only that package; a name or path that can go stale is one named constant; no language, engine or code branch is hard-coded, and a branch lym owns in its records namespace is one named constant in fabric. — [background](pattern/PATTERN-code-conventions.md)
```

`pattern/PATTERN-code-conventions.md` takes the same summary line, and its "No hard-coded choice" section's **Branch** bullet is rewritten:
the rule governs code branches, read from git or from lym's own records;
a branch lym itself owns inside its records namespace (the one-repository board branch `main` under `refs/lym/heads/`, and the f06acf39 layout's `lym-bolt` that the clone's refusal names)
is a named constant defined once in fabric, never a literal at a use site, since neither is read from git by design and only fabric knows the hub layout.

The one-repository paragraphs D9 drops from `PATTERN-fabric-boundary`, `PATTERN-lym-files`, `PATTERN-pair-remote`, `PATTERN-cwd-resolution` and `PATTERN-run-fsm`,
and the records-on-code-branch text of `docs/overview.md` and `internal/fabricengine/doc.go`, were removed by 13cebe3d; none is left, so no card edits them.
No doc or PATTERN text in the tree describes records on a code branch or `lym-bolt`.

**ImpactSummary:** Docs and PATTERN only: one rewritten section, one added paragraph, one restored overview entry within the headroom and its new background file, and one rewritten PATTERN rule with its overview line; no code reads them.
