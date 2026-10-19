# Run lyx on Ubuntu 24.04's toolchain (git 2.43, tmux 3.4): audit, fix and a minimum-version check

`ubuntu-2404-toolchain` · Note · enhancement, fabric, reed

Imported from [issue #747](https://github.com/Knatte18/loomyard/issues/747).

The operator's work laptop runs Ubuntu 24.04.5 LTS with its packaged git 2.43.0 and tmux 3.4, and lyx has to work there; see the host report #745.
The other development machine has newer git and tmux, so it never shows these faults, and only this host can prove a fix.

Known faults on this toolchain:
- #743: `git cherry-pick --empty=drop` does not exist in git 2.43 (used by `gitrepo.Repo.CherryPick`, called from fabricengine's `dropSeedCommits` for Bolt), and two Bolt rebase tests fail with seed-commit conflicts.
- #738: tmux 3.4 has no separate current-session format in `#{S:}`, so reed's session row draws every session twice with a stray comma.
- #744 may belong here too; not yet known.

The test suite catches only what it exercises: tmux formats are set but never checked as rendered, and git or tmux flags no test runs go unseen.

Proposal, as one task:
1. Audit every git and tmux invocation in the code and note the version each flag or format feature needs.
2. Fix what fails on git 2.43 and tmux 3.4 today, plus what the audit finds, without dropping behaviour on newer versions.
3. Add a minimum-version check for git and tmux at start-up with a clear error, so a host below the floor fails plainly instead of in odd places.
4. Prove it with `lyx gate test ./...` and `lyx gate test --tags integration ./...` green on this host.

It needs #739 fixed first, since no test package can run on this host until then.

## From issue #745

Imported from [issue #745](https://github.com/Knatte18/loomyard/issues/745).

Environment report from a second operator machine where lyx cannot run its own test suite and runs started on another PC were moved over.
It collects the background for #737–#744 in one place, with the open #739 fix inline so it can be applied on a machine whose suite is green.

## The host

| Item | Value |
|---|---|
| OS | Ubuntu 24.04.5 LTS, kernel 7.0.0-38-generic |
| User | Active Directory domain user `hanf`, uid 1856216086 (10 digits), gid 1856200513 `domain users`, home `/home/AD.NORCERESEARCH.NO/hanf` |
| `TMPDIR` | unset, so lyx's temp root is `/tmp/lyx-1856216086` |
| git | 2.43.0 (Ubuntu package) |
| tmux | 3.4 |
| go | 1.26.9 linux/amd64, gcc 13.3.0 |
| Claude Code | 2.1.303 |
| Terminal / IDE / gh | konsole 23.08.5, VS Code 1.142.0, gh 2.45.0 |
| lyx | built 2026-10-19 12:26 from `c03af73d`, which equals `prod` and `origin/main` |

The operator's other machine runs Kubuntu with a local user, most likely uid 1000, and newer git and tmux; there the suite is green.
Three host differences explain every failure seen: the 10-digit uid, git 2.43 and tmux 3.4.

## What happened, in order

1. A fresh orch started on this host saw `hub-workspaces` (`running · Plan-Write`) and `implementer-seam` (`running · Merge-In`) as running, with no pair, branch or session here: they were started on the Kubuntu machine. Nothing said which machine owns them (#741).
2. `lyx fabric add <slug>` adopted both pairs from origin and `lyx batten run <slug> --window` resumed both at `Run-Shed` with no ownership check (#741). No terminal window opened, because `Open-Window` was recorded done on the first machine (#742).
3. The reed status bar drew every session twice with a stray comma (#738, tmux 3.4 has no separate current-session format in `#{S:}`) and sorted sessions by name, so the prime `loomyard` was not leftmost (#740).
4. `implementer-seam`'s Merge-In verify failed in every package before any test ran: the tmux socket path below the lyx run directory is 108 bytes against the 107-byte limit, since the uid appears twice (#739).
5. A fix for #739 (below) makes the suite start; `lyx gate test ./...` then fails only in `internal/creelengine` (#744), and `lyx gate test --tags integration ./...` fails in `internal/gitrepo` and `internal/fabricengine` on git 2.43 (#743).
6. `lyx tender pause hub-workspaces` reported the Plan-Write strand's background fork (`fork0 writes the plan`) under `background`, and the strand stayed `busy` after the pause; the board note `pause-background-work` already covers this.
7. Both runs are paused on this host with nothing pushed from it; they continue on the Kubuntu machine.

## Socket path arithmetic for #739

```
/tmp/lyx-<uid>/r<run>/tmux-<uid>/<server key, up to maxKeyBytes = 61>
```

| uid | run dir name | longest path |
|---|---|---|
| 1856216086 | `r859730815` (os.MkdirTemp, 10 chars) | 108 bytes, refused |
| 1856216086 | `r` + 6 hex (fix below) | 105 bytes |
| 4294967295 | `r` + 6 hex | 105 bytes |
| 1000 | `r859730815` | 96 bytes |

The fix leaves a 2-byte margin; a longer key or one more path level breaks it again, so a shorter tmux socket directory may be the sturdier fix.

## Open #739 fix, untested on a green host

On this host `lyx gate test ./internal/lyxtemp ./internal/testkit/tmuxkit ./internal/durfmt` is green with it, and `go vet` and `gofmt` are clean.
It is not pushed, since the full suite cannot go green here (#743, #744).

```diff
diff --git a/internal/lyxtemp/doc.go b/internal/lyxtemp/doc.go
index 5c8ece60..c278ac16 100644
--- a/internal/lyxtemp/doc.go
+++ b/internal/lyxtemp/doc.go
@@ -4,8 +4,8 @@
 // It is created with mode 0700 and refused unless it is a real directory, which off Windows also means mode exactly 0700 and an owner equal to the user.
 // Nothing falls back to the bare system temp directory: a refused root is an error, and an error path creates nothing.
 //
-// A run directory is `os.MkdirTemp(root, "r")` holding a lock file `run.lock`, which its process holds until it removes the directory.
-// The names stay short so a tmux socket path below a run directory stays under the `sun_path` limit.
+// A run directory is `r` plus six random hex digits, created 0700 under the root and holding a lock file `run.lock`, which its process holds until it removes the directory.
+// The names stay short so a tmux socket path below a run directory stays under the `sun_path` limit for a uid of any length.
 // A process whose system temp directory is itself a run directory under a lyx root takes that root as its own, so a child works beside its parent and never inside it.
 //
 // Dir, MkdirTemp, CreateTemp and Remove are the process API: Dir creates the process's run directory on first use, MkdirTemp and CreateTemp create under it, and Remove deletes it.
diff --git a/internal/lyxtemp/run.go b/internal/lyxtemp/run.go
index ab837291..e81979c8 100644
--- a/internal/lyxtemp/run.go
+++ b/internal/lyxtemp/run.go
@@ -5,6 +5,8 @@ package lyxtemp
 import (
 	"errors"
 	"fmt"
+	"io/fs"
+	"math/rand/v2"
 	"os"
 	"path/filepath"
 
@@ -17,6 +19,13 @@ const (
 
 	// lockFileName is the file in a run directory whose lock its owner holds.
 	lockFileName = "run.lock"
+
+	// runSuffixHexDigits is the length of the random hex suffix of a run directory's name.
+	// Six digits keep a tmux socket path below a run directory under the 107-byte `sun_path` limit for a uid of any length.
+	runSuffixHexDigits = 6
+
+	// runNameAttempts bounds how many taken random names one run directory's creation tolerates before it fails.
+	runNameAttempts = 10000
 )
 
 // run is a run directory and the lock its owner holds on it.
@@ -30,9 +39,9 @@ type run struct {
 // The lock is a blocking one, so a sweeper's momentary hold delays the owner rather than failing it.
 func (r *Root) newRun() (*run, error) {
 	r.Sweep(true)
-	dir, err := os.MkdirTemp(r.path, runPrefix)
+	dir, err := createRunDirectory(r.path)
 	if err != nil {
-		return nil, fmt.Errorf("create run directory: %w", err)
+		return nil, err
 	}
 	held, err := lock.AcquireWriteLock(filepath.Join(dir, lockFileName))
 	if err != nil {
@@ -41,6 +50,23 @@ func (r *Root) newRun() (*run, error) {
 	return &run{dir: dir, lock: held}, nil
 }
 
+// createRunDirectory creates a new directory with mode 0700 under rootPath, named runPrefix plus runSuffixHexDigits random hex digits, and returns its path.
+// A name already taken is retried with a fresh one.
+func createRunDirectory(rootPath string) (string, error) {
+	for range runNameAttempts {
+		name := fmt.Sprintf("%s%0*x", runPrefix, runSuffixHexDigits, rand.Uint32N(1<<(4*runSuffixHexDigits)))
+		dir := filepath.Join(rootPath, name)
+		err := os.Mkdir(dir, 0o700)
+		if err == nil {
+			return dir, nil
+		}
+		if !errors.Is(err, fs.ErrExist) {
+			return "", fmt.Errorf("create run directory: %w", err)
+		}
+	}
+	return "", fmt.Errorf("create run directory under %q: %d random names were all taken", rootPath, runNameAttempts)
+}
+
 // remove releases the lock, then deletes the directory, since Windows cannot delete a locked file.
 func (u *run) remove() error {
 	releaseErr := u.lock.Release()
diff --git a/internal/lyxtemp/run_test.go b/internal/lyxtemp/run_test.go
index 289334db..7d743e06 100644
--- a/internal/lyxtemp/run_test.go
+++ b/internal/lyxtemp/run_test.go
@@ -23,6 +23,9 @@ func TestNewRun_HoldsItsLockUntilRemoved(t *testing.T) {
 	if filepath.Dir(created.dir) != root.Path() || !strings.HasPrefix(filepath.Base(created.dir), runPrefix) {
 		t.Fatalf("run directory = %q, want a %q-prefixed child of %q", created.dir, runPrefix, root.Path())
 	}
+	if got, want := len(filepath.Base(created.dir)), len(runPrefix)+runSuffixHexDigits; got != want {
+		t.Errorf("run directory name %q is %d bytes; want %d", filepath.Base(created.dir), got, want)
+	}
 	probe, acquired, err := lock.TryAcquireWriteLock(filepath.Join(created.dir, lockFileName))
 	if err != nil {
 		t.Fatal(err)
diff --git a/internal/testkit/tmuxkit/tmuxkit_test.go b/internal/testkit/tmuxkit/tmuxkit_test.go
index 111e7670..b77b0a0a 100644
--- a/internal/testkit/tmuxkit/tmuxkit_test.go
+++ b/internal/testkit/tmuxkit/tmuxkit_test.go
@@ -23,6 +23,7 @@ func TestCheckSocketPath(t *testing.T) {
 		wantErr bool
 	}{
 		{"short directory", "/tmp/lyx123", 1000, false},
+		{"run directory of the widest uid", "/tmp/lyx-4294967295/rffffff", 4294967295, false},
 		{"directory at the limit", "/tmp/" + strings.Repeat("d", 107-len("/tmp/")-len("/tmux-1000/")-maxKeyBytes), 1000, false},
 		{"directory one byte over", "/tmp/" + strings.Repeat("d", 107-len("/tmp/")-len("/tmux-1000/")-maxKeyBytes+1), 1000, true},
 		{"longer uid widens the path", "/tmp/" + strings.Repeat("d", 107-len("/tmp/")-len("/tmux-1000/")-maxKeyBytes), 1000000, true},
diff --git a/pattern/PATTERN-lyx-temp-root.md b/pattern/PATTERN-lyx-temp-root.md
index 160489a5..22b57e44 100644
--- a/pattern/PATTERN-lyx-temp-root.md
+++ b/pattern/PATTERN-lyx-temp-root.md
@@ -4,7 +4,7 @@ A temp file or directory lyx or its tests create lives in the process run direct
 
 - The root is `<system temp>/lyx-<uid>` off Windows and `<system temp>\lyx` on Windows, where the system temp directory is already per user.
   It is created 0700 and refused unless it is a real directory owned by the user; a refused root creates nothing.
-- Each process has one run directory below the root, `os.MkdirTemp(root, "r")`, holding a lock file its process holds until it removes the directory.
+- Each process has one run directory below the root, `r` plus six random hex digits, holding a lock file its process holds until it removes the directory.
   `lyxtemp.MkdirTemp` and `lyxtemp.CreateTemp` create below it.
 - A process whose system temp directory is already a run directory under a lyx root takes that root as its own and makes its run directory beside the parent's, so nesting never pushes a tmux socket path past the `sun_path` limit.
   No code hands a child a `TMPDIR` that is not a run directory.
```

## Still unknown

- Whether the `internal/fabricengine` merge-lock failure (`worktree dirty`, want `ErrForeignMergeState`) is git 2.43 or a separate fault.
- Whether #744 fails on the Kubuntu machine too, or only here.
- Which git version lyx needs at minimum: `cherry-pick --empty` arrived after 2.43.