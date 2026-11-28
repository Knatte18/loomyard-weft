# Review fork guard halts the run on a push from the fork's own throwaway repo after an ln, with a message that names the wrong rule

`review-fork-guard-ln-push` · Task · bug, burler, stencil

Imported from [issue #296](https://github.com/Knatte18/loomyard/issues/296).

## What happened

A `lym loom step` at Code-Burler-Review halted the run as `failed` (anomaly `producer-hard-failure`, empty transient class) on round 1 attempt 1.
One review fork had experimented with git on a throwaway repository it created under the worktree's `.scratch`.
lym's own output, with paths shortened to `<...>`:

```
shedadapters: Code-Burler-Review (burler-review): round 1 attempt 1: review: burler: fork "<transcript>.jsonl"
ran a git-mutating command ("S=<worktree>/.scratch/exp2; mkdir -p $S; git init -q $S/r;
git -C $S/r commit -q --allow-empty -m c; ln -s /nonexistent-mount/x.git $S/link.git;
git -C $S/r remote add origin $S/link.git; git -C $S/r push origin HEAD 2>&1 | head -1; echo "--- -C relative";
git -C $S/r remote set-url origin ../gone.git; env -C $S git -C r ls-remote origin 2>&1 | head -1;
ls -d $S/gone.git $S/r/../gone.git 2>&1")
— a fork must never run a state-mutating git command outside a repository it created
```

Reading `internal/burlerengine/cluster.go`, the `commit` and `remote` calls run inside the recorded root `$S/r` and pass.
The invocation that counts is the `push`: an earlier `ln` sets `pushRedirected`, and every later push then counts.
This diagnosis comes from reading the code; it was not reproduced by a test.

The run went on about 40 seconds later with a second round-1 review, and the first review file was archived.

## Why it is abnormal

- The message says the fork ran git "outside a repository it created", but the push ran from a repository the fork created.
  The rule that fired is the `ln` rule, which the message never names, so a reader of the halt cannot tell why the command counted.
- A whole cluster review round was discarded and run again, although the push destination did not exist and the target was untouched.
  The `ln` rule is deliberate and documented on `mutatingGitCommands` (the link may lead out); the cost of its false positive is a halted run.
- The halt text for a review round names no way forward, while the fix-round self-check path appends one through `fixViolationWayForward`.
- The fork brief in `contracts/stencils/burler/burler-cluster-fan.md` says never to write any file and to run only git commands that read.
  The fork broke that brief, and the audit allows part of what the brief forbids (throwaway repositories), so the two state different rules.
  A fork is told neither the throwaway allowance nor that an `ln` makes every later push a violation.

## What to do

- Make the violation message name the rule that fired, here "push after `ln`", and append a way forward on the review path.
- Align the fork brief with the audit: either state that no throwaway repositories or links are allowed, or state the allowance with its limits.
- Consider whether a violation that left the target untouched should rerun the round without halting the run as `failed`.

Provenance: task `missing-origin-unreachable`; note `loom-halt.md`.
