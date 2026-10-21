# Security audit findings

Reached from [`SKILL.md`](SKILL.md) when `pnpm audit --prod` reports a vulnerability. The guardrails there still apply.

## 1. Describe each finding

Record: vulnerable package, installed version, affected range, severity, advisory ID (GHSA / CVE), fixed version if one exists, and the chain that installs it (`pnpm why <pkg> --prod`) down to the direct dependency that introduces it.

## 2. Pick the fix

Take the first route that applies; the fixed version must be eligible in every route.

1. **Direct dependency** — update it to an eligible fixed version through the normal group or migration flow.
2. **Transitive, parent has a release that pulls the fix** — update the parent the same way. Prefer this over an override: it leaves nothing to maintain.
3. **Transitive, fix is an official published patch release** compatible with the range its consumers declare — add an override.
4. **None of the above** — the vulnerability stays open; document it (section 4).

## 3. Override rules

Overrides live under `overrides:` in `pnpm-workspace.yaml`, each with a comment naming the advisory and why the override is safe (follow the existing entries).

- Scope it as narrowly as possible: the vulnerable package only, scoped to the parent (`parent>pkg`) when other consumers resolve a safe version on their own.
- An override selects a patch-level security fix. It is never a way around a known incompatibility, a jump to an arbitrary minor or major, or a substitute for a migration.
- After adding it: `pnpm install`, confirm in `pnpm-lock.yaml` that the fixed version is what actually resolves, then run the full check suite.
- When the override causes incompatibilities or functional failures, remove it and treat the vulnerability as open.
- While here, delete existing overrides that the current dependency tree no longer needs (the parent now resolves a safe version by itself) and verify with the suite.

## 4. Open vulnerabilities

When no safe fix applies, keep (or publish) the PR open and add a PR comment with:

- package, installed version, affected versions;
- severity and advisory / CVE;
- the direct dependency that introduces it, if transitive;
- known fixed version, if any;
- why the fix cannot be applied yet: `minimumReleaseAge`, incompatibility, no fixed release, or pending migration;
- the `pnpm audit --prod` output.

Whether the production app actually executes the vulnerable code is worth stating, but it never downgrades the severity or removes the finding. Every open vulnerability also goes in the final report.
