<!-- This is the soft-trigger note request `lyx orch` posts as creel mail to the orchestrator session at a natural break below the hard cap, rendered by RenderSoftHandoffInstruction (internal/orchengine/prompt.go).
     Its markers are {{.handoff_path}}, the file the session writes its orch note to, and {{.note_template_path}}, the template the note follows.
     The mail's id is assigned when it is posted, so the text names the id the `[creel]` line carrying it opens with, for the `defer` reply.
lyx-stencil: sha256=069e7d8f5d091d51b56a81e6e6bbe76efc8be107aeeec5a7e7cb83c539d63e85 build=e634efb9060e4d49a9b7c66339d46e91cbeef10a time=2026-10-22T17:21:04Z -->
A context cycle is due but optional: if work a cycle would lose is in flight (a subagent or fork mid-task, an edit half done), answer this message with `lyx creel post --reply <id> defer` and write no file, where `<id>` is the number after `#` in the `[creel]` line that carried this message.
Otherwise write the orch note to {{.handoff_path}} following the template {{.note_template_path}}, then end your turn, with no reply.
