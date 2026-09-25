# Global agent instructions

## Subagents

Work **solo**: the main agent does the work itself, including reading across many files and multi-step investigation. Use parallel tool calls for independent mechanical operations.

### Gate

Spawn a subagent only when all three hold:

1. The task is bounded, read-only exploration, or an independent review of a consequential change, with a deliverable you can state in one sentence.
2. You have useful work of your own to do while it runs. If you would wait for its result, do the task solo.
3. You can name the concrete saving in time or main-context tokens, and it clearly exceeds the subagent's full cost: its own tokens, your briefing, and your review of its report.

In doubt, stay solo. A benefit you cannot name is a failed gate.

### Spawning

Before the call, state in one line: the task, the saving, and the model and reasoning effort you chose.

Set the model explicitly on every spawn; a subagent left to inherit the main agent's model runs at the most expensive tier. Pick the smallest tier, with low or medium reasoning:

- Claude: `sonnet`, with the `Explore` agent type for searches.
- GPT / Codex: `luna`, the smallest of luna < terra < sol, reasoning effort `low`. Use `medium` when the task needs inference across files.
- Any other provider: its smallest general-purpose model at the lowest reasoning setting that completes the task.

Escalate one tier only after a small-model attempt returned an unusable report, and say that you are escalating and why.

Brief the subagent with only the context it needs and a word cap for its report. Run one subagent at a time. Subagents work solo too: they finish their task without spawning further agents.

### After

The main agent checks the report against the code before relying on it. An explicit user instruction about delegation or budget overrides this section.
