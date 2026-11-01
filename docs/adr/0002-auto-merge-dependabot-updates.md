# 0002. Auto-merge Dependabot updates

- Status: Accepted
- Date: 2026-11-01

## Context

Dependabot opens weekly PRs for GitHub Actions and npm tooling. Most are minor
or patch bumps that only need CI to pass, so merging them by hand is busywork
and lets updates pile up. Major updates are more likely to break things or
need code changes, so they still deserve a human look.

## Decision

We will enable GitHub auto-merge (squash) on Dependabot PRs whose highest
update is minor or patch, using `.github/workflows/dependabot-auto-merge.yml`.
GitHub then merges the PR once the required status checks pass. Version
update groups in `.github/dependabot.yml` exclude major updates, so majors
arrive as separate PRs and are never auto-merged. Grouped security updates
are auto-merged unless they include a major update.

The workflow only acts when both the PR author and the triggering actor are
`dependabot[bot]`, so a human push to a Dependabot branch does not enable
auto-merge.

## Consequences

- Routine updates land without manual effort; majors still need review.
- Auto-merge relies on repository settings that live outside the code:
  "Allow auto-merge" must be enabled, and `main` needs a branch protection
  rule or ruleset with required status checks (Format, Commit messages,
  Secret scan). Without required checks, auto-merge merges immediately.
- Merges made with `GITHUB_TOKEN` do not trigger other workflows, so the CI
  run on `push` to `main` is skipped for these merges. The same checks have
  already passed on the PR.
- A broken minor or patch release can reach `main` if CI does not catch it.
  The three-day cooldown in `dependabot.yml` reduces this risk.
