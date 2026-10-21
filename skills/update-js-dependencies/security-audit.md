# Security audit findings

Reached from [`SKILL.md`](SKILL.md) when the audit reports a vulnerability. Its guardrails still apply.

## 1. Describe

For each finding record: package, installed version, affected range, severity, advisory (GHSA / CVE), fixed version, and the chain from the direct dependency that pulls it in (`pnpm why <pkg> --prod`).

## 2. Fix

Take the first route that applies, always with an eligible fixed version:

1. **Direct dependency**: update it through the group or migration flow.
2. **Transitive, parent has a release with the fix**: update the parent the same way.
3. **Transitive, an official patch release fits its consumers' declared range**: add an override.
4. **Otherwise**: the vulnerability stays open (section 4).

## 3. Overrides

Use the package manager's mechanism (pnpm `overrides`, npm `overrides`, yarn `resolutions`), each entry with a comment naming the advisory.

- Scope it narrowly: the vulnerable package only, under its parent (`parent>pkg`) when other consumers already resolve a safe version.
- Pin a patch-level security fix. Incompatibilities and minor or major jumps go through a migration.
- After adding it: install, confirm the lockfile resolves the fixed version, run the full suite. If it breaks anything, remove it and treat the vulnerability as open.
- Delete existing overrides the tree no longer needs, and re-run the suite.

## 4. Open vulnerabilities

Keep the PR open and comment with: package and versions, severity and advisory, the direct dependency that pulls it in, the known fixed version, why the fix can't be applied yet (release-age gate, incompatibility, no fix released, pending migration), and the audit output. Note whether production code reaches the vulnerable path, but keep the severity as reported. Every open vulnerability also goes in the final report.
