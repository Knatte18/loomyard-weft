<!-- This is the note request `lyx orch` types into the orchestrator session in the handoff-requested phase, rendered by RenderHandoffInstruction (internal/orchengine/prompt.go).
     It must render to ONE line: shuttle's Send refuses multi-line text.
     Its markers are {{.handoff_path}}, the file the session writes its orch note to, and {{.note_template_path}}, the template the note follows.
lyx-stencil: sha256=f7a23d2af3ec82ada6ccc0ad8fb1c2f831766e6c7aecff56b65158c4ff62033f -->
Your context is about to be cycled: write the orch note to {{.handoff_path}} following the template {{.note_template_path}}, then end your turn.
