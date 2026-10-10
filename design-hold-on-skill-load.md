# shuttle raises a hold notice at the skill-load turn end, before the start prompt is sent

`hold-on-skill-load` · Note · bug, shuttle

Imported from [issue #485](https://github.com/Knatte18/loomyard/issues/485).

On 2026-10-10 at about 07:07 the orch got a hold notice for `ly:bolt-and-store:discussion`: "ended a turn without its output files and is held … Its last message: «ok scribe:prose, scribe:conversation»".
That last message is the reply to shuttle's own skill-load prompt.
The pane shows the start prompt ("Read …/prompt.md in full first …") typed right after it, and the agent working on its task within seconds.
So the hold check judged the skill-load turn end as a task turn end that lacked its outputs, and the orch was woken for nothing.

## Expected

- The turn end that answers the skill-load prompt (and any other turn shuttle itself injected before the start prompt) never counts as a task turn end for the hold check.
- A hold notice is raised only after the start prompt was sent and a later turn ends without the outputs.
