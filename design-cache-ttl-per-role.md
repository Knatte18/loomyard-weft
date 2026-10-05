# Prompt-cache TTL per agent role

`cache-ttl-per-role` · Note · enhancement, shuttle

Claude Code gives a subscription's main session a 1-hour prompt-cache TTL (cache write 2× base input) and subagents/forks 5 minutes (1.25×); every lyx agent is a main session in tmux, so all of them write at the 1-hour price (see https://code.claude.com/docs/en/prompt-caching.md).

Measured on the `landing` run's transcripts: Discussion, Plan, Bouncer and Burler never paused more than 90 s between turns, so 5 minutes keeps them warm; ly-drive paused up to 508 s (three gaps over 5 minutes), so it needs 1 hour. Webster Master waits on forks via Monitor and is not yet measured.

Likely shape: shuttle sets `CLAUDE_CODE_PROMPT_CACHE_TTL=5m|1h` (Claude Code v2.1.242+) per role when it spawns a session, configurable per role. loomyard was developed with the 5-minute limit in mind, so 5 minutes matches its design assumption.