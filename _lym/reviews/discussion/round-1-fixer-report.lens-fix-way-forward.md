```yaml
lens: fix-way-forward
outcome: reported
```

Task: check that every way forward the fix added can be carried out from the state it names and makes progress.

Ways forward checked:
1. D5: hand-write `{"name": "<name>"}` after the collision check refuses a lost name over an undecodable file.
2. D5: restore a lost name in `names` by hand so its landings count again.
3. D5: refuse a write when no config directory resolves, naming D2's variables.
4. D5: a hub whose pull failed is reported as checked locally only.
5. D5 shared-name bound: give one machine a distinct name, then `lym adopt`.
6. D6: the dobby's owner-pause lift and its full pass.
7. D6: `lym tender resume <slug>` for a pause the probe does not lift.
8. D6: lift pauses only after the hub's sweep reports done.
9. Slack constraint: rename the archived channel in Slack.
10. D8: the test seams, and a bad developer file failing tests that only read it.

Items 1–4 and 6–10 can be carried out from the state they name and make progress. Item 8's wait ends when a re-run of the verb sweeps the hub. Item 7's lift gets a full pass (`internal/dobbyengine/doc.go`: "The covering pause's lift still gives it a full pass").

**[LOW]** D5, the shared-name bound's way forward: "giving one of the two machines a distinct name with `lym machine name` on it, then taking back with `lym adopt` each entry the other's sweep moved".
- It does not say which machine to rename, and the choice decides the damage.
- Take the case where the shared name is machine A's former name and B's current name. If A renames, A's sweep puts every former name in its replace set (D6), the shared name included. So A's own sweep also moves B's open entries to A's new name.
- "each entry the other's sweep moved" then points at the wrong machine's sweep: the moves come from the renamed machine's own sweep.
- Renaming B, the machine whose current name is shared, moves nothing of A's, since A already swept its own owners off that name.
- Evidence: D6, "the replace set, this machine's former names", and the write selects "every open beam project and the prime whose recorded owner is in the set".

I made no further tool calls; everything I needed was already in context.
