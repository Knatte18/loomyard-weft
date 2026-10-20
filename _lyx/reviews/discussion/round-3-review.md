---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:44
    summary: 'The first LoadConfig refusal says a tree_scope_verify set while tree_scope holds no tree, and set is undefined; the shipped template and the off switch need an empty value accepted'
    fix: 'In D1, reword the first refusal to a non-empty tree_scope_verify while tree_scope holds no tree, and reword set to non-empty in the editor sentences (the last tree removed while tree_scope_verify is set, and the editor refusal text). Add an acceptance criterion that both keys empty, as the template ships them, load with tree scope off, and that lyx config darn --set tree_scope_verify= is accepted when tree_scope holds no tree. Reword the criterion that LoadConfig refuses a tree_scope_verify with no tree to say non-empty.'
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:54
    summary: 'The editor refusal keeps the edit at the staging copy and says to re-open and fix the key, but re-opening rewrites the staging copy from the hub file and editHubWide removes the staging copy on every error except a changed hub file'
    fix: 'In D1, state two things about the editor refusal. First, editHubWide keeps the staging copy for the new load refusal, which needs its own sentinel error checked the way errHubFileChanged is, because today it removes the staging copy for every other write error. Second, the way forward says to re-apply the edit from the staging copy with the key fixed, as the changed-hub-file message does, never to re-open and fix, because every open of the editor overwrites the staging copy with the hub file bytes. Keep the last-tree clause that names lyx config darn --set tree_scope_verify= first.'
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:46
    summary: 'The editor message says to fix the key named, but no Decision requires the malformed-key and duplicate-key refusals to name the offending key, and the editor way forward is wrong for a --set refusal where the bad key was rolled back'
    fix: 'In D1, state that the malformed tree key refusal names the offending key as written, and the duplicate refusal names both keys. State that the editor way forward applies where the file already holds the key (run start and the editor save), and that a --set refusal, whose write is rolled back, points at --set with a valid key instead.'
  - id: F4
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:63
    summary: 'D2 does not say whether the both-sides verbatim read replaces fabricengine.ChangedPaths or sits beside it, and ChangedPaths feeds loom landing overlap record and peers output, which the last acceptance criterion says stay unchanged'
    fix: 'In D2, state that the darn classification uses a new read beside ChangedPaths, run with -z and --no-renames, and that ChangedPaths and its other readers keep their current output; or, if ChangedPaths itself changes, say so and amend the loom acceptance criterion to name the overlap record and peers changed_paths as changed.'
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:118
    summary: 'D5 says the content kinds name a file and line and gives fix a file that does not parse for the parse kind, but a test file that cannot be read is also returned as that kind with no line'
    fix: 'In D5, say the parse kind names the file and the line only when the parser reports one, and that an unreadable test file arrives as that kind with the operating system error, its way forward being to make the file readable. Keep the unreadable package directory and failed enumeration clauses as they are.'
---

### [MEDIUM] The first refusal's condition says "set", which the record never defines

**Location:** `_lyx/discussion/decision-record.md:44`, also `:52`, `:54` and `:168`.
**Issue:** D1 states each refusal's condition for the second one only, as "a non-empty `tree_scope_verify` without `{targets}`".
The first refusal reads "a `tree_scope_verify` set while `tree_scope` holds no tree", and the editor sentences say "while `tree_scope_verify` is set".
The template ships `tree_scope_verify` as a present key holding an empty string, and the record says it ships both keys empty and that `--set tree_scope_verify=` turns tree scope off, after which the last tree can be removed.
Read as "key present", the first refusal rejects the shipped template and the off state with no trees, so every hub's `darn.yaml` would fail to load.
Only "non-empty" is defined, in "Tree scope is on only when both are non-empty".
The acceptance criteria pin the accepted direction "trees with no `tree_scope_verify`" but nothing pins both keys empty, so a test could pass either reading.
Verdict on the focus question: the second refusal states its own condition, the first does not, and the removal order for the last tree is stated.

### [MEDIUM] The editor refusal promises a kept edit that re-opening destroys

**Location:** `_lyx/discussion/decision-record.md:54`; code `internal/configcli/hubwide.go` `editHubWide`.
**Issue:** The record says the editor refusal "keeps the edit at the staging copy as a changed hub file does" and tells the operator to "re-open `lyx config darn` and fix the key named".
Two parts of that are false against the code.
`editHubWide` removes the staging copy after every `copyStagedToHub` error except `errHubFileChanged`, so a load refusal as written would delete the edit; the changed-hub-file case is the only one that keeps it.
And every editor open writes the hub file's bytes over the staging copy before the editor starts, so a re-open after a kept refusal discards the kept edit; the existing changed-file message therefore says "re-apply it" and does not say re-open.
The plan writer reading only the record would implement the check in `copyStagedToHub` and ship a message whose way forward loses the operator's edit.
Rollback of the hub file itself is true for both paths: `setHubWide` restores through `rollBack` after `newLoadFailure`, and the editor path does so once D1's check is added.

### [LOW] The editor message points at a named key that no refusal is required to name

**Location:** `_lyx/discussion/decision-record.md:46`, `:54`.
**Issue:** The editor refusal says to "fix the key named", and "names the load error".
D1 requires the key refusals to name the editor as their way forward, and a test to reach each, but never that the refusal names which tree key is malformed or duplicated.
Without that the editor message names no key, and the before-and-after comparison in `newLoadFailure`, which matches on the error text, cannot tell a save that swaps one bad key for another from a no-op.
Separately, the way forward "the editor, since `--set` cannot remove a key" is right where the file already holds the key, and wrong for a `--set` write, which `setHubWide` rolls back so the key is never written.

### [LOW] D2 leaves open whether the shared changed-paths read changes

**Location:** `_lyx/discussion/decision-record.md:63`, `:176`.
**Issue:** D2 says `fabricengine.ChangedPaths` "lists only a rename's new path, so the read used here must report both sides" and report paths verbatim.
`ChangedPaths` also serves the Merge-In overlap record in `internal/landingshed/mergein.go`, which loom runs through too, and the peers `changed_paths` line in `internal/loomcli/peers.go`.
The smallest edit to `ChangedPaths` changes their output, against the acceptance criterion that loom runs keep their landing behaviour.
The choice between a new read beside it and a change to it is left to the plan writer without a stated bound.
This raises the decision, not the call-site list.

### [LOW] D5's parse kind also covers an unreadable test file

**Location:** `_lyx/discussion/decision-record.md:118`-`119`; code `internal/impactset/guards.go` `guardNamesInFile`.
**Issue:** `guardNamesInFile` wraps every `parser.ParseFile` error in a `GuardScanError`, and `ParseFile` returns a file read failure as is, with no position.
So a test file that cannot be read arrives as the parse kind, naming the file but no line, and "fix a file that does not parse" is the wrong way forward for it.
The record's sentence that only the three content kinds name a file and line is true of the marker kinds, and of the parse kind only when the parser reports a position.
The misplaced marker and the file no `integration` run compiles do name a line, as D5 says, and the unreadable package directory and the failed enumeration are stated correctly.
