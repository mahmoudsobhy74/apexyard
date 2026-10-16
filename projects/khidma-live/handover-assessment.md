# khidma-live — Handover Assessment

**Date**: 2026-10-16
**Assessor**: Mahmoud Sobhy
**Status**: handover

Outcome: khidma-live is a small, healthy demo module. All 27 tests pass. Harnessability is LOW, because the repo has no types, no lint baseline, no coverage threshold, and no CI. Open issues #10, #11, and #12 already track those gaps. The next action is to get write access from the repo owner, because this account can only read the repo.

## Origin

- **Where it came from**: A demo starter for a fictional government e-services portal. The README calls it "a tiny slice of Khidma".
- **Original owner**: `atlas-apex` (repo owner). Commits also come from Momen Yasser.
- **Repo location**: https://github.com/atlas-apex/khidma-live (public)
- **First commit date**: 2026-10-11
- **Last commit date**: 2026-10-15 (`e12561b docs: add AGENTS.md operating manual (#14)`)
- **Prior handover**: The repo owner ran `/handover` on 2026-10-15. That run added `AGENTS.md`, `CLAUDE.md`, and issues #9 to #13. This assessment is a second, independent read from this portfolio.

## Current State

### Tech stack

- Language: JavaScript (ES modules)
- Runtime: Node.js >= 20 (`package.json` `engines`)
- Framework: none. One plain module, `src/requests.js`, with no HTTP server.
- Database: none. Requests live in an in-process `Map` and reset on restart.
- Test framework: Node built-in test runner (`node --test`)
- CI: none. The repo has no `.github/workflows/` directory.

This stack differs from the portfolio's main product stack. See Open Questions.

### Build status

- Install: not needed. The repo has zero dependencies.
- Build: no build script is defined.
- `npm test`: ok. 27 tests, 27 pass, 0 fail, on Node v22.17.1 on 2026-10-16.
- Lint: no lint script is defined.

### Test coverage

- Estimated: unknown. No coverage tool or threshold is configured.

### Repo activity

- Commits in last 90 days: 5
- Open issues: 5 (#9, #10, #11, #12, #13)
- Open PRs: 1 (#15, `chore(#12): add lint baseline and type checking`, by `atlas-apex`)
- Top contributors: Momen Yasser (4 commits under two name spellings), atlas-apex (1)

## Harnessability assessment

**Overall verdict**: `low`

> ⚠ Harnessability: LOW
>
> Rex's architecture handbooks will fire advisory-only on this codebase. The blocking gate (`ENFORCEMENT: blocking`) will generate false positives. Recommended: adopt as advisory-only, plan a follow-up to add the missing scaffolding (typescript strict, lint baseline, etc.)

| Dimension | Score | Evidence |
|-----------|-------|----------|
| Type safety | `none` | No `tsconfig.json` or `jsconfig.json` on `main`. The code is plain JavaScript. PR #15 proposes a `tsconfig.json`. |
| Module boundaries | `flat` | One module, `src/requests.js`, with no domain, application, or infrastructure split. |
| Framework opinionation | `weak` | `package.json` has no dependencies and no framework. |
| Test coverage signal | `absent` | The `test` script is `node --test` with no coverage flag or threshold. The repo has no CI. |
| Lint baseline | `absent` | No ESLint or Prettier config on `main`. PR #15 proposes `eslint.config.js`. |

The override rule applies: type safety is `none` and framework opinionation is `weak`. See AgDR-0042 for the scoring rationale and v1 thresholds.

## Quality Risks

### Security

- The module does not authenticate callers. Each function trusts the `citizenId` and `reviewerId` it receives. This is acceptable for a demo. A real service needs an auth layer before it calls this module.
- The module has no dependencies, so no dependency CVEs apply.
- No secrets or `.env` files are in the repo.

### Dependencies

- None. The repo has zero runtime and dev dependencies on `main`. PR #15 adds dev dependencies and a `package-lock.json`.

### Technical debt

- The repo has no type checking and no lint baseline. Issue #12 and PR #15 track this.
- The repo has no coverage threshold. Issue #11 tracks this.
- `transition()` calls `new Date()` directly, so history timestamps cannot be injected in tests. Source: the repo's `AGENTS.md`, Conventions section.
- Two features are not built yet: KH-107 (withdraw a request) and KH-109 (return a request for more information). They exist only as markdown in `tickets/`. Issue #9 tracks their move into GitHub Issues.
- KH-109 has an open product question: does a resubmitted request go back to the same reviewer, or to any reviewer?

### Operational

- The repo has no CI. `npm test` runs only on a developer machine. Issue #10 tracks this.
- `main` has no server-side branch protection, or the check could not confirm it. Anyone with push rights can push straight to `main`.
- This portfolio's GitHub account has read access only (`push: false`). It cannot push branches or open PRs from a branch in this repo.
- State is per process. The store and the reference-number counter are not shared across processes.

## Integration Plan

### Roles that apply

- `tech-lead` (always)
- `backend-engineer` (all code is domain logic in `src/requests.js`)

No UI, CI, or deployment exists yet, so the frontend, platform, and SRE roles do not apply. Add `platform-engineer` when issue #10 adds CI.

### Workflows that kick in

- [ ] PR workflow (`.claude/rules/pr-workflow.md`): every change goes through a PR
- [ ] AgDR for technical decisions (for example, the KH-109 reviewer-assignment question)
- [ ] Code Reviewer agent on every PR
- [ ] Security Reviewer agent before any auth or HTTP layer is added
- [ ] `/audit-deps` once PR #15 adds dependencies

### Hooks to enable

- [ ] `block-git-add-all`
- [ ] `block-main-push`
- [ ] `validate-branch-name` (`ticket_prefix: GH`)
- [ ] `validate-pr-create`
- [ ] `pre-push-gate`
- [ ] `check-secrets`

### CI templates to copy in

- [ ] `golden-paths/pipelines/ci.yml` (adapt to `npm test`. The repo has no TypeScript build yet.)
- [ ] `golden-paths/pipelines/pr-title-check.yml`

### Registry entry

```yaml
- name: khidma-live
  repo: atlas-apex/khidma-live
  public: true
  workspace: workspace/khidma-live
  docs: projects/khidma-live
  status: handover
  roles:
    - tech-lead
    - backend-engineer
```

## Next Steps

1. Ask `atlas-apex` for write access to the repo. Without it, this portfolio cannot open PRs or work on any ticket below.
2. Ask `atlas-apex` to turn on branch protection for `main` that requires a pull request before merge.
3. ~~Add a lint baseline and type checking~~ → Filed as [#12](https://github.com/atlas-apex/khidma-live/issues/12). PR #15 is open for it.
4. ~~Re-enable CI on this repo. Copy in `golden-paths/pipelines/ci.yml`~~ → Filed as [#10](https://github.com/atlas-apex/khidma-live/issues/10)
5. ~~Set up test coverage reporting before the first feature~~ → Filed as [#11](https://github.com/atlas-apex/khidma-live/issues/11)
6. ~~`/code-review` the most recent PR as Rex to calibrate review standards~~ → Filed as [#13](https://github.com/atlas-apex/khidma-live/issues/13). PR #15 is a good first candidate.
7. ~~Migrate open tickets KH-107 and KH-109 into GitHub Issues~~ → Filed as [#9](https://github.com/atlas-apex/khidma-live/issues/9)

The repo owner filed items 3 to 7 during the 2026-10-15 handover. Items 1 and 2 are requests to the owner, not repo work, so they are not tickets.

## Post-Handover Checklist

- [ ] Review this assessment with `atlas-apex`
- [ ] Get write access to the repo. Close this before the first feature PR.
- [ ] Get branch protection on `main` turned on in the first week
- [ ] Review and merge PR #15 (lint and types), then rescore harnessability
- [ ] Add `khidma-live` to the weekly `/stakeholder-update` rollup
- [ ] Set a test coverage baseline after issue #11 lands
- [ ] Run `/audit-deps khidma-live` after PR #15 adds dependencies

## Open Questions

- How does this repo relate to the portfolio's main product? This repo is a JavaScript demo of a government e-services portal. Is it a practice project, or a real portfolio project?
- Will `atlas-apex` give this account write access, or will this portfolio only observe the repo?
- Who owns the KH-109 decision about which reviewer gets a resubmitted request?
- Is the 2026-10-15 handover's `AGENTS.md` still accurate after PR #15 merges?
