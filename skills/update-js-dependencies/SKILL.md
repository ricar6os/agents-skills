---
name: update-js-dependencies
description: Update JavaScript/TypeScript dependencies (npm, pnpm, yarn, bun) through eligible-only PRs. Use when the user wants to bump packages, handle Dependabot/Renovate or other open dependency PRs, or audit production dependencies for vulnerabilities.
---

# Update JS Dependencies

Keep a JS/TS project's direct dependencies current through PRs that contain only **eligible** versions, used exactly as published. Commands below use pnpm; translate them to the package manager the lockfile shows.

A version is _eligible_ when it is older than the release-age gate (pnpm `minimumReleaseAge` / `minimumReleaseAgeExclude`, or the equivalent setting) or the project has no gate.

## Guardrails

These override any step that seems to conflict:

- **Eligible versions only.** Leave the release-age gate untouched. When `latest` is too young, take the newest eligible version and report the skipped one.
- **Published code only.** A dependency that needs its own code changed to work is a _blocker_: keep the published version, leave the PR open, document the failure. No `patch-package`, `patchedDependencies`, `node_modules` edits, or monkey patches.
- **Plain installs.** No `--force`, `audit --fix`, ignored advisories, or forced peer ranges.
- **Failing PRs stay open**, either fixed or with the failure documented.
- **Isolated worktrees** from the freshly fetched default branch (or the PR's branch when fixing one). Leave the main checkout alone; remove each worktree after pushing.

## Check suite

Use the project's own `package.json` scripts, in this order, after the last install:

1. lint
2. tests (CI variant if one exists)
3. type check and build (skip a separate type check when the build already runs it)
4. `pnpm audit --prod`. Any finding: follow [security-audit.md](security-audit.md).

A branch is _verified_ when the full suite passed on its final state. Re-run it after any change to versions, overrides, or the lockfile.

## Steps

### 1. Prepare

Read the repo's agent instructions and its branch, commit, and PR conventions. `git fetch origin`. Done when you hold the default-branch SHA every new worktree starts from.

### 2. Existing dependency PRs

List open PRs that touch dependencies (Dependabot, Renovate, earlier runs of this skill, manual bumps). For each, in a worktree on its branch: update from the default branch, regenerate the lockfile with an install (never hand-merge it), confirm each version is still the newest eligible, apply project-side fixes, run the suite, push, update the description. A blocker gets a PR comment.

Done when every such PR is verified or carries its blocker comment. Their updates are _taken_: exclude them below.

### 3. Detect

`pnpm outdated --format json` for direct dependencies. Check each candidate's publish time (`pnpm view <pkg> time --json`) against the gate. Done when every outdated dependency has: current version, newest eligible version, skipped younger versions, taken or not.

### 4. Classify

- **Group**: no source changes expected. Patches, minors, and majors whose release notes break nothing this codebase uses.
- **Migration**: needs source or config changes. Coordinated families (`@mui/*`, `@sentry/*`, `react` + `react-dom` + `@types/react*`) move together as one migration.

For every major, and every minor of a `0.x` package, read the release notes and grep the codebase for the APIs they break. Done when every target has a bucket and every major a one-line justification citing its notes.

### 5. Group PR

One branch `chore/deps-weekly-YYYY-MM-DD`: apply all group updates, install, run the suite. When a check fails, bisect the group to find the culprit, move it to migrations, continue with the rest.

The diff holds only manifests, the lockfile, overrides allowed by [security-audit.md](security-audit.md), and mechanical config changes. Commit `chore(deps): update dependencies` (`fix(deps): …` when it resolves an advisory) and open a non-draft PR listing each `package: old → new`, overrides added, vulnerabilities fixed, versions skipped by the gate, and each check's result.

### 6. Migration PRs

One dependency or family = one worktree = branch `chore/deps-<name>-<version>` = one PR. Run them in parallel subagents when the harness allows (pass them the guardrails and suite), otherwise sequentially.

Update, install, make only the changes the migration requires, run the suite, commit `chore(deps): update <name> to <version>`, and open a non-draft PR whether or not checks pass. A failing PR states the command and its error.

### 7. Baseline audit

When steps 5 and 6 opened nothing, run `pnpm audit --prod` on the default branch and follow [security-audit.md](security-audit.md) for any finding. Open no empty PRs.

### 8. Report

Remove your worktrees, then deliver the [final report](final-report.md) in the user's language. Every run ends here, including runs that changed nothing.
