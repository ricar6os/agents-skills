# Final report

Delivered at the end of every [`update-js-dependencies`](SKILL.md) run, in the user's language. Keep every section, writing "none" where empty. Link every PR.

1. **Existing dependency PRs**: actions taken and final state.
2. **Updates detected**: `package: old → new`, split into group and migration.
3. **Skipped by the release-age gate**: `package: version`, publish date, date it becomes eligible.
4. **PRs opened or updated**: link, branch, verified or failing, and each check's result.
5. **Vulnerabilities**: detected; fixed by update; fixed by override (package, pinned version); still open (package, version, severity, advisory, direct dependency, fixed version, reason).
6. **Blockers**: dependency and version, failed command, one-paragraph summary of the error.

When nothing was eligible and nothing was open, say so and give the baseline audit result.
