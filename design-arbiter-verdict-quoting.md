# Arbiter verdict with an unquoted verbs item containing ': ' is unparseable

`arbiter-verdict-quoting` · Task · bug, shed, loom · done

Imported from [issue #754](https://github.com/Knatte18/loomyard/issues/754).

Seen 2026-10-19 in run ubuntu-2404-support, Plan-Bouncer round 3 (cause hard-cap). Arbiter attempt 2 wrote verdict.yaml with verdict: resume, and a verbs list item like - lyx loom decision add ... --title "D6 refined: card 9's ..." unquoted. The ': ' inside the plain scalar makes YAML read the item as a map, so verbs failed to unmarshal (cannot unmarshal !!map into string) and the driver turned the verdict into a hand-on with cause unparseable. The arbiter's verbs had already run (decision added, circling accept recorded), so only the resume was missing. Workaround: the orch read the attempt, confirmed the recorded accept, and ran lyx loom resume. Expected: the verdict format cannot carry a verb line that breaks parsing, e.g. the arbiter writes each verb as a quoted or block scalar, the stencil says so, and the parser tolerates or the arbiter's own check rejects an unparseable verdict before exit.