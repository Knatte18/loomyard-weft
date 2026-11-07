# Subagents spawned by a writer load no language skills, so their comments miss the scribe rules

`subagents-skip-language-skills` · Note · bug, shuttle, stencil

## Seen

The operator found that many workers in this repository's history did not load the scribe skills, and that many docstrings are poor for it (2026-11-07).
In `comment-cleanup`, all 502 subagents so far made no Skill call: neither the worker brief nor the lead brief named a skill.
`skills` in `internal/languageconfig/template.yaml` loads language skills for the code-writing and code-reviewing roles lym starts, such as the darn writer and the Webster master.
A subagent such a role spawns through the Agent tool is outside that list, so it gets a skill only if its brief says so.
Which earlier workers wrote without the skills, and through which route, is not traced.

## Idea

Have every agent that writes code or comments load the language skills, a spawned subagent included, through the stencils that tell a role to spawn one or through a hook at subagent start.

## Wanted

Trace which roles and subagents wrote without the skills, then close that route.
