<!-- This is the soft-trigger note request `lyx orch` posts as creel mail to the orchestrator session at a natural break below the hard cap, rendered by RenderSoftHandoffInstruction (internal/orchengine/prompt.go).
     Its markers are {{.handoff_path}}, the file the session writes its orch note to, and {{.note_template_path}}, the template the note follows.
     The mail's id is assigned when it is posted, so the text names the id `lyx creel read` prints beside it.
lyx-stencil: sha256=26d80e14916d14840210e648d5de83673f6d80210c93e07d78957eecdaffe5ee build=c353cc1ecefd257ec9d5d5ae1e7e86ad98a9f446 time=2026-10-18T13:36:30Z -->
A context cycle is due but optional: if work a cycle would lose is in flight (a subagent or fork mid-task, an edit half done), answer this message with `lyx creel post --reply <id> defer` and write no file; otherwise write the orch note to {{.handoff_path}} following the template {{.note_template_path}}.
Once the note is written, answer this message with `lyx creel post --reply <id> "written {{.handoff_path}}"`, then end your turn.
Here `<id>` is the id `lyx creel read` prints beside this message.
