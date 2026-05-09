# Module 1 — SSH & Sudo Governance

Goal: replace static SSH keys with SSO-tied short-lived certificates; record every sudo invocation and interactive session; expose access graph + audit log via control plane.

GA target: Month 6. Alpha (5 design partners): Month 3.

---

## Phase 1 — Foundation (Weeks 1–2)

### M1-001 [arch] Threat model SSH CA + agent
- **Owner agent:** `architect-sec`
- **Type:** research
- **Labels:** module-1, phase-1-foundation, security-sensitive
- **Acceptance:** ADR committed at `docs/adr/0001-threat-model-ssh-ca.md` covering: attacker model, trust boundaries, key custody, agent-control-plane auth, compromise recovery, customer lockout failure modes.

### M1-002 [arch] ADR: stack selection
- **Owner agent:** `architect-sec` (with `product-strat` input)
- **Type:** docs
- **Labels:** module-1, phase-1-foundation
- **Acceptance:** `docs/adr/0002-stack-selection.md` formalizing Go + Postgres + Next.js + Terraform on AWS.

### M1-003 [infra] Bootstrap monorepo structure
- **Owner agent:** `devops-iac`
- **Type:** chore
- **Labels:** module-1, phase-1-foundation
- **Acceptance:** `/agent`, `/control-plane`, `/web`, `/infra`, `/docs`, `/policies` directories created. `make build && make test` returns 0 even if empty stubs.

### M1-004 [infra] CI pipeline
- **Owner agent:** `devops-iac`
- **Type:** chore
- **Labels:** module-1, phase-1-foundation
- **Acceptance:** GitHub Actions runs build + test + lint (golangci-lint, ESLint) + gosec + govulncheck on every PR. Branch protection requires green CI.

### M1-005 [infra] Secrets management plan (KMS)
- **Owner agent:** `architect-sec` + `devops-iac`
- **Type:** research
- **Labels:** module-1, phase-1-foundation, security-sensitive
- **Acceptance:** `docs/adr/0003-secrets-and-key-custody.md`. AWS KMS for SSH CA private key. IaC stub in `/infra/modules/kms`.

### M1-006 [control-plane] Multi-tenant data model + RLS
- **Owner agent:** `backend-go` + `architect-sec` review
- **Type:** feat
- **Labels:** module-1, phase-1-foundation, security-sensitive
- **Acceptance:** Postgres schema with `tenant_id` on every table; RLS policies enforced; integration test proves cross-tenant SELECT returns empty.

### M1-007 [control-plane] CODEOWNERS + branch protection
- **Owner agent:** `devops-iac`
- **Type:** chore
- **Labels:** module-1, phase-1-foundation
- **Acceptance:** `agent/`, `control-plane/auth/`, `control-plane/crypto/`, `control-plane/tenancy/`, `infra/secrets/` require `architect-sec` review.

### M1-008 [docs] Repo onboarding doc
- **Owner agent:** `docs-eng`
- **Type:** docs
- **Labels:** module-1, phase-1-foundation
- **Acceptance:** `CONTRIBUTING.md` covers local dev setup, test commands, PR conventions.

---

## Phase 2 — SSH CA + agent core (Weeks 3–6)

### M1-009 [control-plane] Wrap step-ca as managed CA
- **Owner agent:** `backend-go`
- **Type:** feat
- **Labels:** module-1, phase-2-ssh, security-sensitive
- **Acceptance:** API `POST /v1/ca/issue` accepts SSO token + principal, returns short-lived SSH cert. Cert TTL configurable per tenant (default 8h, max 24h).

### M1-010 [control-plane] OIDC SSO integration
- **Owner agent:** `backend-go`
- **Type:** feat
- **Labels:** module-1, phase-2-ssh
- **Acceptance:** OIDC flow with Okta, Google Workspace, Microsoft Entra. Tested end-to-end against each. Use Dex internally.

### M1-011 [agent] Linux agent — host registration
- **Owner agent:** `backend-go`
- **Type:** feat
- **Labels:** module-1, phase-2-ssh
- **Acceptance:** `linuxid-agent register` enrolls host, fetches tenant CA pubkey, installs into `/etc/ssh/trusted_user_ca_keys`, configures sshd `TrustedUserCAKeys`.

### M1-012 [cli] CLI tool — SSH login
- **Owner agent:** `backend-go`
- **Type:** feat
- **Labels:** module-1, phase-2-ssh
- **Acceptance:** `linuxid ssh user@host` triggers SSO login if needed, requests cert from CA, opens SSH connection. Works from macOS, Linux, Windows (WSL).

### M1-013 [control-plane] Cert revocation + KRL distribution
- **Owner agent:** `backend-go` + `architect-sec` review
- **Type:** feat
- **Labels:** module-1, phase-2-ssh, security-sensitive
- **Acceptance:** Revoked certs hit hosts within 60s via KRL push. Test: revoke cert in CP → SSH attempt to host fails within 60s.

### M1-014 [control-plane] Audit log — append-only with hash chain
- **Owner agent:** `backend-go` + `architect-sec` review
- **Type:** feat
- **Labels:** module-1, phase-2-ssh, security-sensitive
- **Acceptance:** Every cert issuance + every SSH login event is logged. Postgres role used by app cannot DELETE/UPDATE rows. Hash chain verifiable by external script.

### M1-015 [agent] Heartbeat + health metrics
- **Owner agent:** `backend-go`
- **Type:** feat
- **Labels:** module-1, phase-2-ssh
- **Acceptance:** Agent reports alive/version/OS/uptime to control plane every 60s. Stale agents flagged in dashboard.

### M1-016 [test] E2E: SSO → cert → SSH login
- **Owner agent:** `qa-test`
- **Type:** test
- **Labels:** module-1, phase-2-ssh
- **Acceptance:** GitHub Actions e2e job spins up containerized OIDC, control plane, agent on a Linux container; performs full login flow.

---

## Phase 3 — Sudo + session recording (Weeks 7–10)

### M1-017 [agent] PAM module / sudo plugin
- **Owner agent:** `backend-go` + `architect-sec` co-write
- **Type:** feat
- **Labels:** module-1, phase-3-sudo, security-sensitive
- **Acceptance:** Sudo invocations logged with user, command, exit code, timestamp on Ubuntu 22.04, Debian 12, Amazon Linux 2023. Failure mode: if our plugin crashes, sudo still works (graceful degradation).

### M1-018 [agent] Session recording — PTY capture
- **Owner agent:** `backend-go`
- **Type:** feat
- **Labels:** module-1, phase-3-sudo
- **Acceptance:** Interactive shell sessions recorded (input + output) in asciinema-compatible format; ≤5% CPU overhead; recording survives session end.

### M1-019 [control-plane] Session storage — S3 with Object Lock
- **Owner agent:** `backend-go` + `architect-sec` review
- **Type:** feat
- **Labels:** module-1, phase-3-sudo, security-sensitive
- **Acceptance:** Recordings encrypted (KMS), uploaded to S3 with Object Lock (compliance mode, retention configurable). Tampering = audit failure.

### M1-020 [web] Session replay UI
- **Owner agent:** `frontend-react`
- **Type:** feat
- **Labels:** module-1, phase-3-sudo
- **Acceptance:** Engineer can play back any session, scrub timeline, search for keywords (`grep`-style) in recorded output.

### M1-021 [web] Sudo command search
- **Owner agent:** `frontend-react`
- **Type:** feat
- **Labels:** module-1, phase-3-sudo
- **Acceptance:** "Who ran `rm -rf` in the last 7 days" returns ≤2s.

### M1-022 [agent] Chaos test — agent failure does not block SSH
- **Owner agent:** `qa-test` + `architect-sec`
- **Type:** test
- **Labels:** module-1, phase-3-sudo, security-sensitive
- **Acceptance:** Kill `linuxid-agent` mid-session → existing SSH session continues. New SSH login still works (sshd validates cert against CA pubkey directly). Documented in runbook.

---

## Phase 4 — Control plane + UI (Weeks 8–11, parallel with Phase 3)

### M1-023 [web] Host inventory dashboard
- **Owner agent:** `frontend-react`
- **Type:** feat
- **Labels:** module-1, phase-4-cp
- **Acceptance:** Lists all enrolled hosts with last-seen, OS, agent version, environment tags. Filterable, sortable.

### M1-024 [web] User access matrix
- **Owner agent:** `frontend-react`
- **Type:** feat
- **Labels:** module-1, phase-4-cp
- **Acceptance:** Grid of users × hosts (or roles × host-groups). Click reveals last 10 sessions. Export as CSV.

### M1-025 [web] Audit log search
- **Owner agent:** `frontend-react`
- **Type:** feat
- **Labels:** module-1, phase-4-cp
- **Acceptance:** Free-text + filters (user, host, event type, time range). Results paginated, exportable.

### M1-026 [control-plane] Org / team / billing scaffolding
- **Owner agent:** `backend-go`
- **Type:** feat
- **Labels:** module-1, phase-4-cp
- **Acceptance:** Stripe integration: org → trial → paid → usage metering. Free tier enforced (≤10 hosts).

### M1-027 [web] Onboarding flow — under 5 minutes
- **Owner agent:** `frontend-react` + `docs-eng`
- **Type:** feat
- **Labels:** module-1, phase-4-cp
- **Acceptance:** Sign up → connect SSO → copy `curl | bash` → first SSH login under 5 minutes for a tester who's never seen the product. Timed test recorded.

### M1-028 [web] User invite + RBAC
- **Owner agent:** `frontend-react` + `backend-go`
- **Type:** feat
- **Labels:** module-1, phase-4-cp
- **Acceptance:** Roles: owner, admin, auditor, member. Invites via email. RBAC enforced server-side (not just UI).

### M1-029 [docs] Install guide
- **Owner agent:** `docs-eng`
- **Type:** docs
- **Labels:** module-1, phase-4-cp
- **Acceptance:** Mintlify docs site live at docs.linuxidentity.com. Install guide for each supported distro. Troubleshooting section.

---

## Phase 5 — Hardening + ship (Weeks 12–14)

### M1-030 [security] External pen test
- **Owner agent:** human (vendor) + `architect-sec`
- **Type:** review
- **Labels:** module-1, phase-5-harden, security-sensitive
- **Acceptance:** Pen test report with no critical or high findings. All findings filed and resolved before GA.

### M1-031 [compliance] SOC 2 Type 1 readiness audit
- **Owner agent:** `compliance` (drafts) + human (auditor)
- **Type:** compliance-evidence
- **Labels:** module-1, phase-5-harden
- **Acceptance:** Drata or Vanta connected; gap assessment complete; readiness audit passed.

### M1-032 [infra] Status page + incident runbook
- **Owner agent:** `devops-iac` + `docs-eng`
- **Type:** chore
- **Labels:** module-1, phase-5-harden
- **Acceptance:** status.linuxidentity.com live. Runbook in `/docs/runbooks/incident-response.md`. PagerDuty rotation defined.

### M1-033 [agent] Self-update with signature verification
- **Owner agent:** `backend-go` + `architect-sec` review
- **Type:** feat
- **Labels:** module-1, phase-5-harden, security-sensitive
- **Acceptance:** Agent fetches signed release manifests; verifies signature; opt-in update windows; rollback path documented.

### M1-034 [infra] Disaster recovery runbook
- **Owner agent:** `devops-iac` + `architect-sec`
- **Type:** docs
- **Labels:** module-1, phase-5-harden
- **Acceptance:** RTO/RPO defined. Quarterly DR drill scheduled. KMS key rotation drill scripted.

### M1-035 [security] Beta → GA gate review
- **Owner agent:** `architect-sec` (chair) + product
- **Type:** review
- **Labels:** module-1, phase-5-harden
- **Acceptance:** Sign-off checklist complete: pen test clean, SOC 2 Type 1 in flight, runbooks current, on-call live, rollback tested.

---

## Launch (Week 14+)

### M1-036 [marketing] Landing page V1
- **Owner agent:** `marketing-copy` + `frontend-react`
- **Type:** feat
- **Labels:** module-1, site, phase-launch
- **Acceptance:** linuxidentity.com live with: hero, problem statement, 3 product screenshots, "Get a demo" CTA, pricing, security page, status page link.

### M1-037 [seo] 5 pillar SEO articles
- **Owner agent:** `seo-content` (draft) + human (edit)
- **Type:** docs
- **Labels:** module-1, site, phase-launch
- **Acceptance:** Articles live: "SSH key management for Series A startups," "JIT sudo without CyberArk," "OpenSSH CA in production: a complete guide," "What SOC 2 actually requires for Linux access," "Replacing static SSH keys: a 90-day plan." Indexed by Google. Internal links + schema.

### M1-038 [aiseo] 3 AISEO content pieces
- **Owner agent:** `aiseo-content` (draft) + human (edit)
- **Type:** docs
- **Labels:** module-1, site, phase-launch
- **Acceptance:** FAQ schema, definitive-answer formats, llms.txt published. Targets AI overviews and LLM citations. Tracking via Bing Webmaster + manual prompts.

### M1-039 [sales] Outreach to 50 design partners
- **Owner agent:** `sales-outreach` (drafts) + human (sends)
- **Type:** chore
- **Labels:** module-1, phase-launch
- **Acceptance:** 50 emails sent with personalized opener; 5–8 design partners signed.

### M1-040 [marketing] Show HN launch
- **Owner agent:** `marketing-copy` (draft) + human (post)
- **Type:** chore
- **Labels:** module-1, phase-launch
- **Acceptance:** Show HN post drafted, reviewed, posted Tue–Thu 8am Pacific. 24h response monitoring.

### M1-041 [marketing] Pricing page
- **Owner agent:** `marketing-copy`
- **Type:** docs
- **Labels:** module-1, site, phase-launch
- **Acceptance:** Free / Starter / Pro tiers; comparison vs Teleport, StrongDM published as separate page.

### M1-042 [legal] Terms of service + privacy + DPA
- **Owner agent:** `legal-privacy` (drafts) + human (lawyer review)
- **Type:** docs
- **Labels:** module-1, phase-launch
- **Acceptance:** TOS, Privacy Policy, DPA published. Reviewed by counsel. GDPR + CCPA compliant.

---

## Cross-cutting (start week 1, run through GA)

### M1-100 [research] Customer discovery — 25 interviews
- **Owner agent:** `customer-research` (prep) + human (interview)
- **Type:** research
- **Labels:** module-1, cross-cutting
- **Acceptance:** 25 interviews with target ICP completed. Notes synthesized into ICP doc + jobs-to-be-done framework.

### M1-101 [product] PRD — Module 1
- **Owner agent:** `product-spec`
- **Type:** docs
- **Labels:** module-1, cross-cutting
- **Acceptance:** Living PRD in `/docs/prd/module-1.md` with user stories, acceptance criteria, scope-cut list.

### M1-102 [product] Competitor monitoring
- **Owner agent:** `product-strat`
- **Type:** research
- **Labels:** module-1, cross-cutting
- **Acceptance:** Monthly competitor tracker covering Teleport, StrongDM, JumpCloud, smallstep, HashiCorp Boundary, Tailscale SSH.
