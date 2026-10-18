<!-- This is the note request `lyx orch` posts as creel mail to the orchestrator session in the handoff-requested phase, rendered by RenderHandoffInstruction (internal/orchengine/prompt.go).
     Its markers are {{.handoff_path}}, the file the session writes its orch note to, and {{.note_template_path}}, the template the note follows.
     The mail's id is assigned when it is posted, so the text names the id the `[creel]` line carrying it opens with.
lyx-stencil: sha256=74a6cf498f8c05a78195b9c55eacc490a49268d2cca756cd4708b014e835f5e2 build=ff6b3e8c1315ed6dfccfa2a5549938c46da8cf6c time=2026-10-18T14:37:07Z -->
Your context is about to be cycled: write the orch note to {{.handoff_path}} following the template {{.note_template_path}}.
Once the note is written, answer this message with `lyx creel post --reply <id> "written {{.handoff_path}}"`, where `<id>` is the number after `#` in the `[creel]` line that carried this message, then end your turn.
