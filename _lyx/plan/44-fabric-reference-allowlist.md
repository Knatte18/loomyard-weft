# Card 44 — fabric-reference-allowlist

**Edit:**
- `internal/websterengine/audit.go`
- `internal/websterengine/doc.go`
- `internal/websterengine/auditseverity_test.go`
- `internal/websterengine/recordbatch_test.go`
- `internal/websterengine/runlevel_test.go`

**Prosa:**
- `contracts/specs/refusal-spec.md`

**Intent:**
Fix PR #321 round-2 finding A: the fabric-reference classifier fails open.
`mutatingSegment` returns false for any program it does not recognise, so a command that rewrites the fabric sibling through an unlisted writer (`python3 -c "open(...,'w')"`, `dd of=`, `rsync`, `install`, `patch`, `chmod`, `mkdir`, `tar -x`, `unzip`, `awk '{print > f}'`, `bash script.sh`, `eval`, `source`, any `lyx` verb outside read-only `lyx fabric`) classes as policy and only warns.
Invert the per-program rule to an allowlist, so an unknown program is mutating.

In `audit.go`:

- Replace `mutatingSegment` with an unexported `readOnlySegment(words []string) bool`, true only when the simple command is one of these shapes:
  - after leading shell keywords and `NAME=value` assignments, no words at all (a bare assignment);
  - a program in a new `readOnlyTools` set: `cat`, `ls`, `grep`, `head`, `tail`, `wc`, `stat`, and the shell builtins that write nothing without a redirect: `cd`, `pwd`, `echo`, `printf`, `true`, `test`, `[`;
  - `git`, after its global options, with a subcommand in `readOnlyGit` and no `--output` option (`--output` and `--output=<file>` make `git log`, `git show` and `git diff` write a file);
  - `lyx fabric` with a verb in `readOnlyFabric`, or `prune`/`cleanup` without `--apply`;
  - `find` with none of `-delete`, `-fprint`, `-fprint0`, `-fprintf`, `-fls`, and every `-exec`/`-execdir`/`-ok`/`-okdir` command itself read-only;
  - a wrapper in `commandWrappers`, `timeout`, or `xargs` whose wrapped command is read-only, an `xargs` with no command counting as read-only since it runs `echo`;
  - `bash`, `sh`, `zsh` or `dash` with a `-c` script that `mutatingCommand` finds read-only.
  Every other shape is mutating: an unknown program, a program spelled through a variable, `bash`/`sh` without `-c`, `eval`, `source`, `.`, `sed`, `perl`, `awk`, any `lyx` verb outside the read-only fabric ones.
- `mutatingCommand` keeps `splitShell` and its substitution recursion unchanged, and is true when any segment writes or is not `readOnlySegment`.
  A command with no segment and no substitution at all is mutating, so an empty or unparseable command fails closed.
- Delete `mutatingTools` and `editsInPlace`, which the allowlist leaves unused.
- Rewrite the doc comments on `mutatingCommand`, `readOnlySegment` and `ClassifyViolation` to state the allowlist rule: a fabric reference is policy only when every simple command is a known read-only program with no write redirect.

`doc.go`'s audit-findings paragraph replaces the parenthesised mutating list with the same rule: a fabric reference is correctness unless every command in it is a known read-only program with no write redirect.

Tests:

- `auditseverity_test.go`, `TestClassifyViolation_FabricReferenceByCommand`: add correctness rows for `python3 -c "open('<dir>/webster/state.json','w').write('x')"`, `dd if=/dev/zero of=<dir>/webster/state.json`, `rsync -a x <dir>/webster/`, `install x <dir>/webster/state.json`, `patch <dir>/webster/state.json p.diff`, `chmod 000 <dir>/webster/state.json`, `mkdir <dir>/webster/x`, `tar -xf a.tar -C <dir>`, `unzip a.zip -d <dir>`, `awk '{print > "f"}' <dir>/webster/state.json`, `bash <dir>/script.sh`, `eval "rm <dir>/webster/state.json"`, `source <dir>/env.sh`, `lyx webster rebaseline`, `$TOOL <dir>/webster/state.json`, `git -C <dir> log --output=<dir>/x -1`, `find <dir> -fprint <dir>/x`, `sed -n '1,5p' <dir>/webster/state.json` and `perl -MList::Util=sum -ne 'print' <dir>/webster/state.json`.
  The last two rows already exist as policy and move to correctness, since a sed `w` command or a perl `open` can write.
  Add policy rows for `wc -l <dir>/webster/state.json`, `stat <dir>/webster/state.json`, `tail -n 5 <dir>/webster/state.json`, `find <dir> -name '*.yaml' -exec cat {} \;`, `ls <dir> | xargs`, `env LC_ALL=C grep x <dir>/webster/state.json`, `bash -c "cat <dir>/webster/state.json"` and `lyx fabric prune`.
  Every existing row other than the sed and perl ones keeps its class.
- `recordbatch_test.go` and `runlevel_test.go`: the fabric-reference fixtures that ran `lyx FABRICREF sync` stand for a read-only fabric reference, which the allowlist now classes as correctness.
  Change each to `cat FABRICREF/webster/state.json`, so every test that asserts the policy path (one recorded warning, a done batch, a run-exit warning, the card-verify promotion to correctness) still reaches that path.
  `TestRecordBatch_MutatingFabricReferenceFailsBatch` keeps its `git ... checkout` command.
  Add one recordbatch test beside it: a fork whose Bash ran `python3 -c "open('/fabric/sibling/webster/state.json','w').write('{}')"` on an OK report with a passing card verify → the batch is recorded failed with its report archived and the digest names the command, the state the round-2 finding showed passing as a warning.

`refusal-spec.md`: the webster section's audit sentence says a fabric reference is correctness unless every command in it is a known read-only program with no write redirect;
the `audit: fabric-reference, mutating` row's trigger says "ran a fabric-referencing command outside the read-only allowlist", and the read-only row's trigger says "ran a fabric-referencing command made only of known read-only programs with no write redirect".
Neither row's class nor way forward changes.

**ImpactSummary:** Fabric-reference findings from any command outside the read-only allowlist become correctness, failing the batch or demoting run exit; only allowlisted read-only commands stay recorded warnings.
