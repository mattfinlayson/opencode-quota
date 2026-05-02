# Contributing to opencode-quota

Thanks for contributing. This repo has strict local-only behavior and regression guardrails, so please follow this workflow.

## Issue-First (Preferred)

- Prefer opening an issue before starting features, bug fixes, refactors, or behavioral changes.
- If you already have a fix ready, opening an issue and PR together is fine.
- When an issue exists, link it in the PR description using `Fixes #<issue>` or `Refs #<issue>`.
- If no issue exists, include a short rationale/scope summary in the PR description.

## Issue and PR Templates

- GitHub Issue Forms are enabled and blank issues are disabled.
- Use `.github/ISSUE_TEMPLATE/bug_report.yml` for bug reports.
- Use `.github/ISSUE_TEMPLATE/feature_request.yml` for feature requests.
- Use template title prefixes for consistent issue titles.
- Inactive issues may be marked stale after 23 days and closed 7 days later if there are still no updates.
- Bug title format: `[bug]: <short description>`
- Feature title format: `[feature]: <short description>`
- Pull requests use `.github/pull_request_template.md` and should include tested OpenCode version details.

## Development Setup

- Node.js must be `>=18.0.0` (matches `package.json` engines).
- Install dependencies with:

```sh
npm install
```

`npm install` runs `prepare`, which installs Husky hooks.

## Local Quality Gates

Pre-commit hooks currently run:

- `npx lint-staged` (formats staged files via Prettier)
- `npm run typecheck`
- `npm test`

Pre-push hooks currently run:

- `npm ci`

Run checks manually before opening a PR:

```sh
npm run typecheck
npm test
npm run build
```

Use `npm run test:watch` for local iteration.

## CI Checks (Automated)

PR and `main` pushes trigger `.github/workflows/ci.yml` (`CI` workflow):

- Job: `build`
- Matrix: Node `18.x`, `20.x`, `22.x`
- Steps: `npm ci`, `npm run typecheck`, `npm run build`, `npm test`

PR CI also runs `npm test` in the matrix job.

Release workflow `.github/workflows/publish-npm.yml` runs on release/manual dispatch and includes typecheck, test, and build before publish. It is not a normal PR required check.

## Branch Protection (Maintainers)

Recommended settings for `main`:

- Require a pull request before merging.
- Require branches to be up to date before merging.
- Require status checks from workflow `CI` for every Node matrix entry.
- Select checks exactly as GitHub displays them in repository settings.
- Typical names look like `build (18.x)`, `build (20.x)`, `build (22.x)` or `CI / build (18.x)` variants.
- Block direct pushes to `main` for non-admin users.

## Repo Guardrails

- Never invoke an LLM/model API to compute toast/report output. Everything must remain local and deterministic.
- Preserve slash command handled-sentinel behavior in `command.execute.before`.
- Do not catch `isCommandHandledError(...)` and return normally.
- Keep `tests/plugin.command-handled-boundary.test.ts` aligned with this invariant.

Additional boundary tests to keep healthy when touching plugin/provider logic:

- `tests/plugin.qwen-hook.test.ts`
- `tests/quota-provider-boundary.test.ts`

## Provider Changes

When adding a provider, keep the README setup wording tied to real behavior.

- For API-key/token providers that support `Existing OpenCode auth, global config, or env`, start from `contributing/provider-template/`.
- Copy the template files to the target paths listed in `contributing/provider-template/README.md`.
- Replace the example names, ids, env vars, and config keys before coding.
- Add tests for each supported auth source before using the shared README wording; do not leave copied template tests skipped, todo-only, or unresolved.
- In the PR checklist, state whether you started from the provider template; if not, explain why it does not apply.
- Do not use that wording for OAuth-only providers such as OpenAI.

## Quality Bar for Fixes

- Prefer the smallest safe fix that addresses the root cause.
- Align behavior with current OpenCode production behavior rather than adding extra hook/output mutation layers.
- Preserve existing invariants and update/add boundary tests when behavior contracts change.
- We appreciate PRs that verify the fix against the current production released OpenCode version and note the tested version in the PR.

## Pull Request Checklist

- Linked issue (`Fixes #...` or `Refs #...`) when available, or included a short no-issue rationale in the PR.
- `npm run typecheck` passes.
- `npm test` passes.
- `npm run build` passes.
- Verified behavior against the current production released OpenCode version, and included the tested version in the PR notes.
- Updated docs when user-facing commands/config/workflow changed (usually `README.md`; update this file when contributor workflow changes).
- For new API-key/token providers, started from `contributing/provider-template/` or explained why the template does not apply.
- For provider setup/auth wording changes, checked `contributing/provider-template/` and verified README wording against implementation/tests.
