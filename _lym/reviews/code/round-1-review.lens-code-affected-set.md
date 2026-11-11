```yaml
lens: code-affected-set
outcome: reported
```

**Affected-set list: lens code-affected-set**

No finding at MEDIUM or above. The set the diff touches is covered, apart from one LOW item.

- **`s` shortcut (`shortcuts.yaml`)**
  - Set found: `shortcut_test.go` (`TestMount_File`), `cmd/lym/shortcuts_test.go`, `docs/overview.md` (the shortcuts bullet; the directory tree line only names `lym s`), `docs/getting-started.md`, `adoptcli/doc.go` and `shortcutcli/doc.go` (mentions `lym s` generically).
  - A repo-wide grep for `lym s` and `lym beam start, then` in md, go, yaml and tmpl files found no other text describing the old two-step shortcut. Covered.
- **`runAdopt` signature (`discardFlag bool` → `callFlags`) and `resolvePrime` → `resolveCall`**
  - Set found: the only caller of `runAdopt` is `adoptcli/cli.go:84`; `driveWorker` is updated.
  - Other `resolvePrime` hits are separate functions in `reportcli`, `orchcli` and `fabricengine`. Covered.
- **New flags `--here` and `--archives`**
  - Set found: the command help in `cli.go`, `adoptcli/doc.go` (here-call and archives sections), `docs/overview.md` (new bullets), `adoptrecord/doc.go` (`TakesUp`) and `fabricengine/doc.go` (`ReadOwnerArchives`).
  - `orchcli/start.go:58` and `orchcli/doc.go:44` say only `lym adopt` sets the no-orch flag; that stays true because the `s` shortcut calls `lym adopt`. Covered.
- **End envelope key `archives`**
  - `adoptcli/doc.go` (envelope paragraph) and the `TestAdoptFields` row cover it. A grep for other docs listing envelope fields (`prime_beam_warning`, `check_warnings`) found none outside `adoptcli`. Covered.
- **`remoteArchiveTags` refactored into `remoteTagsUnder`**
  - Its other caller keeps its wrapped error message.
  - `remoteTagsUnder` returns the raw `ls-remote` error, which `ReadOwnerArchives` wraps. Covered.
- **PATTERN entries**
  - **`PATTERN-fabric-boundary` and `PATTERN-gitrepo-client-boundary`:** git runs only in `fabricengine`. `adoptcli` calls `Bolt.Pull`, `Topology.ReadOwnerArchives` and `hubgeom.ParentOf`. The `git -C … log` text in `archives.go` is a string for the operator, not a git run. The fetch goes through `repo.FetchWithArgs` in `gitrepo`. Covered.
  - **`PATTERN-owner-machine`:** the parent comes from `hubgeom.ParentOf`. Covered.
  - **`PATTERN-refusal-way-forward`:** this entry covers webster, shed and loom refusals only, so the adopt refusals fall outside it. They name the way forward anyway.
  - **Declared PATTERN changes:** the directive lists none, and `PATTERN.md` and `pattern/` are unchanged in the diff. Covered.
- **Commit messages:** neither commit message has a `Pattern-Departure:` trailer, and none is needed.

**Findings**

1. **LOW, consistency:** `internal/adoptcli/doc.go:4` still reads "They take the flags `--keep-local`, `--drop-local`, `--no-orch` and `--discard`." The diff adds `--here` and `--archives` to the same package doc. The first line was reworded to name the two new verbs, but this flag list was left unchanged and is now incomplete. Fix: add `--here` and `--archives` to that sentence (evidence: `doc.go:1-4` in the diff).

**Tool calls**
1. Grep for `resolvePrime`, `runAdopt(`, `lym s` and the old shortcut text across md, go, yaml and tmpl files: I needed the other callers and docs that describe `s`, which the manifest lacked.
2. Grep for `Pattern-Departure`, `adopt --here` and `adopt --archives`, plus a `pattern/` listing and a diff stat of `PATTERN.md` and `pattern/`: to confirm no PATTERN change landed or was declared.
3. Grep for `prime_beam_warning`, `check_warnings`, `--no-orch` and `keep-local` outside `adoptcli`: to find other docs or fixtures that list envelope fields or flags.
