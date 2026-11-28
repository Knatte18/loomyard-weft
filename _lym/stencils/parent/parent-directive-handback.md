<!-- This is the hand-back variant of the parent directive, for a role whose verdict file is its only channel.
     internal/parentdirective.HandBack renders it when it is told a non-empty parent name;
     an empty one renders parent-directive-none instead.
     Its one marker is the parent name: the orch that parents the run.
lym-stencil: sha256=c0758dd6a281468a3f95bc4d16838d4b3e429a0ae6767895937bb9806f9db749 build=b439f157e432a2f6559d9fbf5e68daa2aae9995f time=2026-11-28T09:27:33Z -->

## Your parent

Your run's parent is `{{.parent_name}}`.
When you are blocked, write your report and stop; the run hands the stop back.
