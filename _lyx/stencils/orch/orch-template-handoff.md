<!-- This is the handoff instruction `lyx orch` types into the orchestrator session in the handoff-requested phase, rendered by RenderHandoffInstruction (internal/orchengine/prompt.go).
     It must render to ONE line: shuttle's Send refuses multi-line text.
     Its one marker is {{.handoff_path}}, the file the session writes its handoff to.
lyx-stencil: sha256=ae387dfa136819c8d447ee2c99eef61ba42c22dfb4b44314491ad809fa5c4120 -->
/scribe:handoff {{.handoff_path}} Your context is about to be cycled, so make the handoff complete.
