---
name: update-dependencies
description: Weekly dependency maintenance for Laria. Use when the user wants to analyze, review, or update project dependencies, handle open dependency PRs, or audit production dependencies for vulnerabilities.
---

# Update Dependencies

Keep Laria's direct dependencies current through PRs that contain only **published, eligible** versions: a version is _eligible_ when it is older than `minimumReleaseAge` or matched by `minimumReleaseAgeExclude` (both in `pnpm-workspace.yaml`). A dependency's code is always used exactly as published.

Every run ends in the [final report](final-report.md), including runs that open no PR.

## Guardrails

These hold for the whole run and override any step that seems to conflict with them:

- **Eligible versions only.** Leave `minimumReleaseAge` and `minimumReleaseAgeExclude` untouched. When `latest` is too young, take the newest eligible version below it and list the skipped one in the report.
- **Published code only.** A dependency that needs its own code corrected to work is a _blocker_, not a task: no `patchedDependencies`, `patch-package`, `patches/`, edits to `node_modules`, modified vendored copies, or monkey patches. Keep the published version, leave its PR open, document the failure.
- **Plain installs.** No `--force`, no `pnpm audit --fix`, no ignored advisories, no forced incompatible peers.
- **Failing PRs stay open.** A dependency PR is never closed because install, lint, types, tests, build, or audit fail; it is fixed or documented.
- **Isolated worktrees.** All work happens in worktrees created from the freshly fetched remote default branch (or from the PR's branch when fixing an existing PR). The main checkout and its uncommitted changes are never touched. Remove each worktree once its branch is pushed.

## The check suite

Run in this order inside the worktree, after the last `pnpm install`:

1. `pnpm lint`
2. `pnpm test:ci`
3. `pnpm build` — already runs `check-types` first, so a type error surfaces here; report it as a `check-types` failure. Run `pnpm check-types` on its own only to iterate on type errors without paying for the build.
4. `pnpm audit --prod` — mandatory security check. Any finding sends you to [security-audit.md](security-audit.md).

A branch is _verified_ only when the full suite ran on its final state: after the last install and the last change to versions or overrides. Re-run the suite after any such change. Run nothing beyond the suite unless a migration or security investigation requires it.

## Steps

### 1. Prepare

Read `AGENTS.md` and `docs/agents/git-workflow.md`. `git fetch origin` and resolve the remote default branch. Done when you hold the fetched default-branch SHA that every new worktree will start from.

### 2. Open dependency PRs first

List every open PR that touches dependencies: previous runs of this skill (`chore/deps-*`, `fix/*` security bumps), Dependabot, Renovate, or manual bumps (`gh pr list --state open`, then filter by branch, title, author, and files changed).

For each one, in a worktree on the PR's own branch:

- Check mergeability, conflicts, failing checks, and whether each proposed version is still eligible and still the newest eligible one.
- Fix what is fixable within the guardrails: rebase or merge the default branch, regenerate the lockfile with `pnpm install` (never hand-resolve `pnpm-lock.yaml` conflicts), apply project-side migration changes. Run the check suite, push to the same branch, update the PR description.
- When the fix would need patched dependency code or a not-yet-eligible version, leave the PR open and comment with the blocker.

Done when every open dependency PR is either verified and updated, or carries a comment documenting its blocker. The updates those PRs cover are _taken_: exclude them from the following steps.

### 3. Detect eligible updates

Inspect `package.json`, `pnpm-workspace.yaml`, and `pnpm-lock.yaml`. List outdated **direct** dependencies with `pnpm outdated --format json`. For every candidate, confirm eligibility from the registry publish time (`pnpm view <pkg> time --json`) against `minimumReleaseAge` (minutes) rather than trusting `latest`.

Done when every outdated direct dependency has a row: current version, target version (newest eligible), skipped younger versions, and whether it is taken by an open PR.

### 4. Classify

Put each target in one bucket:

- **Group**: no source changes expected — patches, minors, and majors whose release notes show no breaking change affecting this codebase.
- **Migration**: needs source or config changes. Official package families with coordinated versions (e.g. `@mui/*`, `@sentry/*`, `react` + `react-dom` + their types) move together as one migration.

Read changelogs, release notes, peer dependency ranges, and migration guides for every major and for any minor of a `0.x` package; grep the codebase for the APIs they break. Done when every target has a bucket and every major has a one-line justification citing its release notes.

### 5. Group PR

Skip when the group bucket is empty.

One worktree, one branch `chore/deps-weekly-YYYY-MM-DD` (reuse the branch if today's already exists). Apply all group updates to `package.json`, run `pnpm install`, run the check suite.

When a check fails, find the culprit by bisecting the group (revert half, reinstall, re-run the failing check). Move the culprit to the migration bucket and continue with the rest; one bad update never blocks the group.

Before committing, confirm the diff contains only: dependency manifests, `pnpm-lock.yaml`, security overrides allowed by [security-audit.md](security-audit.md), and strictly mechanical config changes.

Commit as `chore(deps): update weekly dependencies` (`fix(deps): …` when the PR's purpose is resolving an advisory), push, and open a **non-draft** PR whose body lists: each dependency with old → new version, security overrides added, vulnerabilities fixed, versions skipped by `minimumReleaseAge`, and the result of each check. When a failure attributable to the remaining group persists, publish anyway and state the failing check and its error in the PR.

Done when the PR exists and its branch is verified, or its failure is documented in the PR.

### 6. Migration PRs

One migration = one dependency (or one inseparable family) = one worktree = one branch `chore/deps-<name>-<version>` = one PR. Migrations are independent: dispatch each to its own subagent or thread when the harness allows, passing it this skill's guardrails and check suite; otherwise run them sequentially.

In each: update the dependency, `pnpm install`, make only the project-side changes the migration requires, run the check suite, review the diff, commit as `chore(deps): update <name> to <version>` (add `!` and a `BREAKING CHANGE` footer only when Laria's own behaviour changes), push, and open a non-draft PR **whether or not the checks pass**. A failing PR states the failed command, the relevant error output, and any audit findings.

Done when every migration has an open PR that is verified or documents its failure.

### 7. Baseline audit

When steps 5 and 6 opened nothing, run `pnpm audit --prod` in a worktree on the default branch. Findings go through [security-audit.md](security-audit.md): a fixable one gets its own branch and PR, an unfixable one goes in the report. Open no empty branches or PRs.

### 8. Report

Remove the worktrees you created, then deliver the [final report](final-report.md) in the user's language.
