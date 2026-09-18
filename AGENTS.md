# Global agent instructions

## Subagent use and cost

- Work in the main agent by default. Create a subagent only when a concrete benefit in completion time or result quality justifies the additional token cost, duplicated context, coordination, and review effort.
- Delegate bounded, independent work with a clear deliverable, such as a substantial investigation that can run alongside useful main-agent work or an independent review of a consequential change. Handle simple edits, routine lookups, and tightly dependent steps directly.
- Before spawning, briefly state the delegated task and why its expected benefit outweighs its cost. If the benefit is unclear, continue locally.
- Use the fewest subagents needed, pass only relevant context, and choose the least expensive available model capable of the task when model selection is supported and permitted. Prefer parallel tool calls for independent mechanical operations.
- Avoid duplicate investigations and recursive delegation. A subagent must apply this same cost-benefit test before creating another agent. Reuse an existing agent when suitable and stop redundant work.
- The main agent remains responsible for integrating and checking results. Respect any explicit user budget or restriction on delegation.
