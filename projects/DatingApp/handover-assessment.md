# DatingApp — Handover Assessment

**Date**: 2026-10-16
**Assessor**: Mahmoud Sobhy
**Status**: handover

Outcome: DatingApp is a working Angular 20 and ASP.NET Core 9 reference app. The API builds clean and has no known vulnerable NuGet packages. Harnessability is LOW because the repo has no tests and no lint baseline. The repo has no license, so its code cannot go into a commercial product without the author's permission. The next action is to decide how this portfolio will use the repo: as a reference, or as a base for code.

## Origin

- **Where it came from**: Open-source teaching repo for the Udemy course "Build an app with ASP.NET Core and Angular from scratch".
- **Original owner**: Neil Cummings (`TryCatchLearn` organisation). He wrote all 26 commits.
- **Repo location**: https://github.com/mahmoudsobhy74/DatingApp-1 (public fork, created 2026-10-16). Upstream: https://github.com/TryCatchLearn/DatingApp (public, 384 stars, 206 forks).
- **Local clone remotes**: `origin` is the fork. `upstream` is `TryCatchLearn/DatingApp`.
- **First commit date**: 2025-05-23
- **Last commit date**: 2025-07-02 (`73972be UpdatedWf`)
- **License**: none. The repo has no `LICENSE` file, and GitHub reports no license.

## Current State

### Tech stack

- Language: C# (API), TypeScript (client)
- Runtime: .NET 9 (`API/API.csproj` `net9.0`), Node.js 22 in CI
- Framework: ASP.NET Core Web API with SignalR. Angular 20 with Tailwind CSS 4 and DaisyUI 5.
- Database: SQL Server through Entity Framework Core 9. `docker-compose.yml` runs SQL Server 2022 for local development. The README says production uses Azure SQL.
- Auth: ASP.NET Core Identity, JWT bearer tokens, and a refresh token in a cookie
- Photo storage: Cloudinary (`CloudinaryDotNet` 1.27.5)
- Test framework: Karma and Jasmine are installed for the client. The repo has no test files.
- CI: one workflow, `.github/workflows/main_dating-app-course.yml`. It builds the client and API on push to `main` and deploys to an Azure Web App. It runs no tests.

### Build status

- `dotnet build API/API.csproj -c Release`: ok. 0 warnings, 0 errors, .NET SDK 9.0 on 2026-10-16.
- Client `npm install` and `ng build`: not attempted, at the operator's choice.
- Tests: none exist to run.
- Lint: no lint configuration exists.

### Test coverage

- Estimated: 0%. The repo has no .NET test project and no Angular `*.spec.ts` files.

### Repo activity

- Commits in last 90 days: 0. The last commit is 15 months old.
- Open issues: 2 (#11 "Blinking Carret", #12 "Request")
- Open PRs: 0
- Top contributors: Neil Cummings (26 commits)
- Other branches: `PhotoManagementChallenge`, `dokploy`

## Harnessability assessment

**Overall verdict**: `low`

> ⚠ Harnessability: LOW
>
> Rex's architecture handbooks will fire advisory-only on this codebase. The blocking gate (`ENFORCEMENT: blocking`) will generate false positives. Recommended: adopt as advisory-only, plan a follow-up to add the missing scaffolding (typescript strict, lint baseline, etc.)

| Dimension | Score | Evidence |
|-----------|-------|----------|
| Type safety | `strong` | `client/tsconfig.json` line 6: `"strict": true`, plus `strictTemplates`. `API/API.csproj` sets `<Nullable>enable</Nullable>`. |
| Module boundaries | `partial` | One API project, organised by technical folder (`Controllers/`, `Data/`, `Services/`, `Interfaces/`). It has no separate domain, application, or infrastructure projects. |
| Framework opinionation | `strong` | ASP.NET Core with EF Core, Identity, and built-in DI. Angular on the client. |
| Test coverage signal | `absent` | `karma-coverage` is installed, but no tests exist and no threshold is set. CI runs no tests. |
| Lint baseline | `absent` | No ESLint config in `client/`, no `.editorconfig`, and no .NET analyzer settings. |

Two of five dimensions are in the top bucket, so the verdict is `low`. See AgDR-0042 for the scoring rationale and v1 thresholds.

## Quality Risks

### Security

- `API/appsettings.Development.json` is committed. It holds the local SQL Server `SA` password and a JWT signing key ("super secret key …"). `docker-compose.yml` repeats the SA password. These are development values. Never reuse them in any deployed environment.
- The login endpoint returns "Invalid email address" and "Invalid password" as two different errors (`AccountController.cs` lines 60 and 64). An attacker can use the difference to find registered email addresses.
- The JWT setup sets `ValidateIssuer = false` and `ValidateAudience = false` (`Program.cs`).
- The API stores refresh tokens in the user table as plain values (`AccountController.cs` line 78 compares the raw cookie value).
- `BuggyController` and `WeatherForecastController` are teaching or template endpoints. They are still present in the API.
- No rate limiting on the login endpoint was observed. This was a static read, so other protections may exist that it did not find.

### Dependencies

- `dotnet list package --vulnerable` reports no vulnerable NuGet packages on 2026-10-16.
- The client depends on Angular `^20.0.0-next.0` and `@angular/cli ^20.0.0-rc.3`. These are pre-release version ranges. Client packages were not audited because `npm install` was not run.
- The repo has no license. This is a legal risk, not a technical one. See Open Questions.

### Technical debt

- The repo has no automated tests on either side.
- The repo has no lint or formatting baseline.
- The API is one project with no domain layer. It applies migrations and seeds users at startup (`Program.cs`, `MigrateAsync` and `Seed.SeedUsers`).
- The CORS policy allows only `localhost:4200` origins. That works because the API serves the built client from `wwwroot`.

### Operational

- CI deploys to the author's Azure Web App on every push to `main`. A copy needs its own deployment target and credentials.
- CI builds and deploys but runs no tests.
- No error tracking or monitoring was found. `ExceptionMiddleware` only logs.
- `main` on the upstream repo has no server-side branch protection, or the check could not confirm it. The fork has none yet.
- This portfolio works in the fork `mahmoudsobhy74/DatingApp-1`, where this account has push access and Issues are enabled. It has read access only on the upstream repo.
- The fork carries the upstream deploy workflow. It triggers on every push to `main` and targets the author's Azure Web App. It will fail without that app's credentials. Disable or replace it before the first push to the fork's `main`.
- The project has been dormant for 15 months.

## Integration Plan

### Roles that apply

- `tech-lead` (always)
- `backend-engineer` (ASP.NET Core API, EF Core schema)
- `frontend-engineer` (Angular client)
- `platform-engineer` (GitHub Actions workflow)
- `sre` (Azure Web App and Azure SQL deployment)
- `security-auditor` (Identity, JWT, and refresh-token handling)

### Workflows that kick in

- [ ] PR workflow (`.claude/rules/pr-workflow.md`): every change goes through a PR
- [ ] AgDR for technical decisions, for example how to split the API into layers
- [ ] Code Reviewer agent on every PR
- [ ] Security Reviewer agent on the first pass and on any auth change
- [ ] Migration gate for any change under `API/Data/Migrations/`
- [ ] `/audit-deps` on adoption and monthly after that

### Hooks to enable

- [ ] `block-git-add-all`
- [ ] `block-main-push`
- [ ] `validate-branch-name` (`ticket_prefix: GH`)
- [ ] `validate-pr-create`
- [ ] `pre-push-gate` (`dotnet build`, `ng build`, and tests once they exist)
- [ ] `check-secrets`

### CI templates to copy in

- [ ] `golden-paths/pipelines/ci.yml` (adapt to `dotnet test` and `ng test`)
- [ ] `golden-paths/pipelines/security.yml`
- [ ] `golden-paths/pipelines/pr-title-check.yml`

### Registry entry

```yaml
- name: DatingApp
  repo: mahmoudsobhy74/DatingApp-1
  public: true
  workspace: workspace/DatingApp
  docs: projects/DatingApp
  status: handover
  roles:
    - tech-lead
    - backend-engineer
    - frontend-engineer
    - platform-engineer
    - sre
    - security-auditor
```

## Next Steps

1. The fork exists, so work can start there. The license question stays open: ask Neil Cummings for a license before any fork code ships in a commercial product.
2. Disable or replace the inherited Azure deploy workflow in the fork.
3. Turn on branch protection for `main` in the fork, with a required pull request.
4. Set up test coverage reporting with `dotnet test` and `ng test --code-coverage` before the first feature.
5. Run `/audit-deps DatingApp` on the client packages after `npm install`.
6. Run `/threat-model DatingApp` on the auth flow (login responses, JWT validation flags, refresh-token storage).
7. `/decide` on observability (Application Insights or Sentry are the common choices for this stack).

File these items as issues in the fork, `mahmoudsobhy74/DatingApp-1`, never in the upstream repo.

## Post-Handover Checklist

- [ ] Decide reference or base, and get a license if you choose base. Close this before any code reuse.
- [ ] Add a test project and a coverage baseline in the first two weeks, in your own copy
- [ ] Replace the committed development secrets with user secrets or environment variables in your own copy
- [ ] Add `DatingApp` to the weekly `/stakeholder-update` rollup only if you adopt it as a base
- [ ] Run `/audit-deps DatingApp` monthly for the next 3 months if you adopt it as a base

## Open Questions

- What is the purpose of this repo in the portfolio? It is a dating app owned by a third party.
- Will the author grant a license? With no license, the code is all rights reserved by default.
- If you adopt it as a base, will you fork it, or start a fresh repo and port selected patterns?
- Do the `PhotoManagementChallenge` and `dokploy` branches hold work that matters? This read covered `main` only.
