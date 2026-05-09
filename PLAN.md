# Linux Identity — Operating Plan

Owner: PM (Claude) · Founder: Saheed · Last updated: 2026-05-09

## North star
Become the system of record for every actor on a Linux fleet. Wedge with SSH/sudo governance; expand into compliance, NHI, AI agent identity, and behavioral detection.

## Strategic decisions
- **Path:** Bootstrap to ~$2M ARR, then re-decide on venture. Default optimization: capital efficiency.
- **First wedge:** Module 1 (SSH & sudo governance) for Series A/B infra-heavy startups.
- **First ICP:** 50–300 engineers, Linux-dominant, Head of Platform or first Security hire as buyer.
- **Pricing target (Module 1):** $25/host/mo or $15/user/mo, whichever is higher; free under 10 hosts/5 users.
- **Brand:** "Linux Identity" — narrow, declarative, defensible domain.

## Modules — sequence and target dates

| # | Module | GA target | Effort (calendar) | Cumulative ARR target (bootstrap) |
|---|---|---|---|---|
| 1 | SSH & sudo governance | Month 6 | 14 weeks | $400–600K by M12 |
| 2 | JIT privileged access | Month 9 | 6 weeks | $1M by M15 |
| 3 | Compliance evidence | Month 18 | 8 weeks | $3–5M by M24 |
| 4 | Non-human identity | Month 30 | 10 weeks | $8–14M by M36 |
| 5 | AI agent identity | Month 42 | 12 weeks | $20–32M by M48 |
| 6 | Behavioral analytics | Month 54 | 16 weeks | $40–65M by M60 |

These dates are aggressive but possible solo + AI. Slippage of 4–8 weeks per module is normal; protect the wedge (M1) date hardest.

## Active milestone
**M1-Phase1: Foundation** — Weeks 1–2 of Module 1. See `backlog/module-1-ssh-sudo.md`.

## Agent roster (see `.claude/agents/`)
| Agent | Model | Primary use |
|---|---|---|
| architect-sec | Opus | Threat models, ADRs, mandatory security review |
| backend-go | Sonnet | Go agent + control plane API |
| frontend-react | Sonnet | Next.js dashboard + marketing site |
| devops-iac | Sonnet | Terraform, CI/CD, packaging |
| qa-test | Haiku | Tests, fixtures, fuzz harnesses |
| docs-eng | Haiku | Developer docs, install guides |
| product-strat | Opus | Pricing, positioning, competitor monitoring |
| product-spec | Sonnet | PRDs, user stories, acceptance criteria |
| seo-content | Sonnet | Keyword-targeted blog content |
| aiseo-content | Sonnet | AI-overview / LLM citation content |
| marketing-copy | Sonnet | Landing pages, emails, ads |
| sales-outreach | Sonnet | Cold outreach drafts (human sends) |
| customer-research | Sonnet | Prospect company research |
| compliance | Opus | SOC 2 / ISO controls + policy drafts |
| legal-privacy | Sonnet | TOS / privacy / DPA drafts |

## Cadence
- **Mon 9am**: PM weekly review. Backlog grooming, milestone adjustment, design-partner check-in.
- **Wed**: Mid-week unblock — clear blockers on active milestone.
- **Fri**: Demo any user-facing changes; ship to design partners if bi-weekly cycle.
- **Last Friday of month**: Roadmap review.

## Risk register (top 5)
1. **Agent locks customers out of own servers.** Mitigation: out-of-band design, OpenSSH CA validation works without our agent.
2. **SSH CA key compromise.** Mitigation: AWS KMS custody, rotation runbook, no plaintext exposure.
3. **Teleport launches SMB tier.** Mitigation: lock customers via compliance + NHI attach in years 2–3.
4. **AI agent category doesn't materialize.** Mitigation: Module 5 is optionality, not core.
5. **Founder bandwidth.** Mitigation: agents handle 80% of typing; founder focuses on customers + security review + decisions.

## KPIs (current quarter)
- Design partners signed: target 5 by Month 3
- Module 1 alpha shippable: Month 3
- Module 1 GA: Month 6
- First paid customer: Month 5
- ARR end of Q4: $400K
- SOC 2 Type 1 in progress: by Month 6
- NPS from design partners: ≥40

## Decision log
- 2026-05-08: Repo created on GitHub under `abandoned-ip` account, private.
- 2026-05-09: Bootstrap path chosen as default until $2M ARR. Stack locked: Go + Postgres + Next.js + Terraform on AWS. GitHub Projects + Issues as tracker. PR-based workflow non-negotiable.
