<!-- This is the soft-trigger note request `lym orch` posts as creel mail to the orchestrator session at a natural break below the hard cap, rendered by RenderSoftHandoffInstruction (internal/orchengine/prompt.go).
     Its markers are {{.handoff_path}}, the file the session writes its orch note to, and {{.note_template_path}}, the template the note follows.
     The mail's id is assigned when it is posted, so the text names the id the `[creel]` line carrying it opens with, for the `defer` reply.
lym-stencil: sha256=f7cd65d01ffe2a4b81ef4e856ee8dc3b2142711d1b478ccd9a06791ea489dae2 build=ef912d4155479098331ad98253e9a38250450e09 time=2026-10-27T07:47:27Z -->
A context cycle is due but optional: if work a cycle would lose is in flight (a subagent or fork mid-task, an edit half done), answer this message with `lym creel post --reply <id> defer` and write no file, where `<id>` is the number after `#` in the `[creel]` line that carried this message.
Otherwise write the orch note to {{.handoff_path}} following the template {{.note_template_path}}, then end your turn, with no reply.
