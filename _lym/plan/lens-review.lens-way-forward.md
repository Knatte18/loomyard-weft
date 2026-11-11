```yaml
lens: way-forward
outcome: reported
```

Way-forward lens: I traced each refusal, stop and recovery the cards add to the next step it names, and checked whether that step can succeed from the state left behind.

**Findings**

1. `MEDIUM`, Card 4 (D5 collision check, D6 failed hub): a hub whose `readOwners` fails escapes the collision check for good.
   - Step 4 says a read error "fails that hub as in step 7 and the check goes on". The file is then written in step 5, and the failed hub's way forward is the re-run `lym machine name <name>`.
   - On the re-run, `name` is already this machine's current name, so it is "one of this machine's names". The collision check can never fire on that hub again.
   - So the named next step finishes a write that the check never cleared. A board that records `<name>` for another machine then sees this machine's sweep, and every owner comparison is string equality (record, Constraints).
   - D5 allows only a failed pull to fall back to "checked locally"; it says nothing about an unreadable board.
   - Fix: refuse before any write when a hub's owners cannot be read, naming the hub and the error, with a re-run as the way forward. Add a `TestName_Set` row that reaches it through `fakeHubs` returning a `readOwners` error.

2. `LOW`, Card 4 step 7 (D6 failed hub): for a dirty or diverged Bolt, the named re-run cannot succeed alone.
   - `SetOwners` refuses with "the board could not be brought up to date: <detail>" (`internal/boardengine#SetOwners`). The detail's own way forward comes from `dirtyDetail` / `divergedDetail` (`internal/fabricengine/bolt.go:415-420`).
   - Card 4 gives every failed hub the single way forward "`lym machine name <name>` (a re-run)". From a dirty or diverged Bolt that re-run fails again until the detail's step is done.
   - The same holds for the deferred-push refusal, whose own text says "run `lym board sync`, then re-run the verb" (`internal/boardengine#pushOwnerCommit`).
   - Fix: state that the outcome's way forward is the error's own way forward when it names one, then the re-run.

3. `LOW`, Card 4 step 2: a machine file that exists but does not read is refused naming only its path, with no next step.
   - The show (card 3) sends this state to `lym machine name <name>` as its way forward, and card 4 then refuses that same state. The show's way forward therefore leads to a second refusal that names nothing to do.
   - `machinename`'s own way forward for this state is "run `lym machine name <name>`, which rewrites the file, or remove the file" (`internal/machinename#setNameWayForward`). Only "remove the file" progresses from it.
   - It is arguably the propagated-I/O exemption. Still, the show's own way forward points into it, so either path should name removing or fixing the file at its path.

4. `LOW`, Card 4 step 4 (collision refusal for a lost former name): the way forward does not state what follows the hand write.
   - The way forward is writing `{"name": "<name>"}` by hand.
   - After that write, owners left under the other lost names are not swept by any later run; D5 sends them to `lym adopt`.
   - The refusal names only the hand write. An operator who follows it is not told about `lym adopt` for the owners the lost names still hold.
   - Fix: add D5's "an owner left under a lost name … is taken back with `lym adopt`" to that refusal's way forward.

Other refusals checked, each with no finding: invalid name (CheckName's rule), relative variable (FilePath's "set that variable to an absolute path"), no directory (names the variables), undecodable file (card 4 replaces it), lost entries (re-run takes a former-name owner, `lym adopt` takes another name's), unpushed commit (re-run pushes, since `pushOwnerCommit` runs even with nothing written), the recovery lines, and the registry decode refusal (its own way forward, `internal/hubregistry/hubregistry.go:146-148`).

Tool call: one grep of `refuseUnreadable`, the Bolt skip detail functions and `beam start root`. My context lacked the registry refusal text, the skip-detail way forward, and whether `root` is a valid `beam start` slug.
