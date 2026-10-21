# Final report

Delivered at the end of every [`update-dependencies`](SKILL.md) run, in the user's language. Keep every section; write "none" where a section is empty. Link every PR.

1. **Pre-existing dependency PRs** — each PR open at the start, actions taken, final state.
2. **Updates detected** — eligible updates as `package: old → new`, split into grouped and migration.
3. **Skipped by `minimumReleaseAge`** — `package: version`, publish date, date it becomes eligible.
4. **PRs opened or updated** — link, branch, verified or failing.
5. **Check results per PR** — `pnpm lint`, `pnpm check-types`, `pnpm test:ci`, `pnpm build`, `pnpm audit --prod`: pass or fail.
6. **Production vulnerabilities**
   - detected;
   - fixed by direct update;
   - fixed by override — package, previous version, pinned patch version;
   - still open — package, affected version, severity, advisory / CVE, originating direct dependency, known fixed version, reason the fix is not applied yet.
7. **Blockers and failed checks** — for each: dependency and version, failed command, one-paragraph synthesis of the error.

When there was nothing eligible and nothing open, say so and give the baseline `pnpm audit --prod` result.
