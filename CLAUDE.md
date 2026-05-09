# Linux Identity — repo conventions

This file is loaded by Claude Code in every session. Read it before opening a PR.

## Mission
System of record for every actor — human, service account, AI agent — on a Linux fleet.

## Stack
- Agent: Go 1.22+, runs on Linux (Ubuntu 22.04, Debian 12, Amazon Linux 2023, RHEL/Rocky 9 — in that priority order)
- Control plane: Go API + Postgres 15+ (with row-level security for tenant isolation)
- Web: Next.js (App Router) + Tailwind + shadcn/ui
- Infra: Terraform on AWS (us-east-1 primary, us-west-2 secondary)
- CI: GitHub Actions
- Observability: OpenTelemetry → Grafana Cloud (until ARR justifies self-host)

## Repo layout
```
/agent           Go binary that runs on customer Linux hosts
/control-plane   Go API + workers
/web             Next.js marketing site + dashboard (one app, two surfaces)
/infra           Terraform modules + envs (dev, staging, prod)
/docs            User-facing docs (Mintlify)
/backlog         Issue lists (source of truth before we import to GH Projects)
/policies        SOC 2 / security policy drafts
/.claude/agents  Subagent system prompts
```

## Branch & PR rules
- Trunk-based: branches off `main`, merge via PR, squash merge.
- **No agent merges directly to `main`.** Every PR needs at least one human review.
- PRs touching `agent/`, `control-plane/auth/`, `control-plane/crypto/`, `control-plane/tenancy/`, `infra/secrets/` require `architect-sec` review (enforced via CODEOWNERS once we add it).
- PR titles: Conventional Commits (`feat:`, `fix:`, `chore:`, `docs:`, `refactor:`, `test:`).
- Every feature PR needs tests + a brief test-plan checklist in the PR description.

## Security guardrails (hard rules)
1. SSH CA private key lives in AWS KMS. **Never** in Postgres, env vars, or filesystem.
2. Audit log writes are append-only with hash chaining. No DELETE allowed (Postgres role lacks the privilege).
3. Tenant isolation: every Postgres query goes through RLS-protected views. No raw `WHERE tenant_id = ...` in app code.
4. Agent self-update: signed releases only, signature verified before exec. We will get this wrong if not careful — `architect-sec` reviews every change.
5. The agent must NOT be in the SSH critical path. If our agent dies, customers must still be able to SSH in via standard OpenSSH cert validation.

## Coding standards
- Go: `gofmt`, `golangci-lint` (config in repo), `gosec`, `govulncheck`. Errors are values, not exceptions. No panics outside `main`.
- TypeScript: strict mode, no `any`. Components are server-rendered by default; mark `"use client"` deliberately.
- SQL: migrations via `goose`. No destructive migrations without an ADR.
- Tests: unit (table-driven for Go), integration (testcontainers), e2e (Playwright for web, custom harness for agent). Coverage threshold: 70% for new code.

## Subagents
See `.claude/agents/`. Each agent has a single focused role; don't ask one agent to do another's job.

Critical agents:
- **architect-sec** (Opus): system design, threat modeling, mandatory security review.
- **backend-go**, **frontend-react**, **devops-iac** (Sonnet): bulk engineering work.
- **product-strat** (Opus): rare, high-stakes strategy questions.
- **compliance** (Opus): controls mapping and policy drafting.

Rules:
- Agents produce drafts. Humans approve and merge.
- Marketing/sales/legal agents output drafts only. Humans send/publish.
- Any claim about competitors, prices, customers, or compliance must be verified before publishing.

## Cadence
- Daily: agents pick up issues from the active milestone.
- Weekly (Mon): PM review — backlog grooming, milestone adjustment, design-partner check-in.
- Bi-weekly: ship to design partners.
- Monthly: revisit roadmap and pricing.
- Quarterly: strategic review (ICP, pricing, positioning).

## Definition of done (per issue)
- Code merged to `main` via PR
- Tests written and passing in CI
- Docs updated if user-facing
- Telemetry/metric added if production behavior changed
- `architect-sec` signed off if security-sensitive
