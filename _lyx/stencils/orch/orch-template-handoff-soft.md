<!-- This is the soft-trigger note request `lyx orch` posts as creel mail to the orchestrator session at a natural break below the hard cap, rendered by RenderSoftHandoffInstruction (internal/orchengine/prompt.go).
     Its markers are {{.handoff_path}}, the file the session writes its orch note to, and {{.note_template_path}}, the template the note follows.
     The mail's id is assigned when it is posted, so the text names the id the `[creel]` line carrying it opens with.
lyx-stencil: sha256=680322e133dadd1a3744d12adb13d2447b95cd01056756be676abe4e4d0414f4 build=ff6b3e8c1315ed6dfccfa2a5549938c46da8cf6c time=2026-10-18T14:37:07Z -->
A context cycle is due but optional: if work a cycle would lose is in flight (a subagent or fork mid-task, an edit half done), answer this message with `lyx creel post --reply <id> defer` and write no file; otherwise write the orch note to {{.handoff_path}} following the template {{.note_template_path}}.
Once the note is written, answer this message with `lyx creel post --reply <id> "written {{.handoff_path}}"`, then end your turn.
Here `<id>` is the number after `#` in the `[creel]` line that carried this message.
