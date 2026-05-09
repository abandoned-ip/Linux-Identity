# Backlog

Source of truth for work items before they are imported as GitHub Issues. Once `gh` is installed, run `scripts/import-backlog.sh` to push these into GitHub Issues + Projects.

## Files
- `module-1-ssh-sudo.md` — Module 1 (SSH/sudo governance) — fully detailed
- `module-2-jit.md` — TBD (after M1 alpha)
- `module-3-compliance.md` — TBD
- `module-4-nhi.md` — TBD
- `module-5-agent-identity.md` — TBD
- `module-6-behavioral.md` — TBD
- `site-marketing.md` — Marketing site + content track (will be added in next iteration)

## Issue conventions

### Title format
`[<area>] <imperative verb phrase>`
e.g. `[agent] Implement SSH cert request flow`, `[web] Build host inventory dashboard`

### Labels
- **Area** (one): `agent`, `control-plane`, `web`, `infra`, `docs`, `marketing`, `compliance`, `legal`
- **Module** (one): `module-1`, `module-2`, ..., `module-6`, `site`, `cross-cutting`
- **Phase** (one): `phase-1-foundation`, `phase-2-ssh`, `phase-3-sudo`, `phase-4-cp`, `phase-5-harden`, `phase-launch`
- **Type** (one): `feat`, `fix`, `chore`, `docs`, `research`, `review`, `compliance-evidence`
- **Risk** (optional): `security-sensitive` — triggers `architect-sec` review
- **Owner agent** (one): `agent:backend-go`, `agent:frontend-react`, `agent:architect-sec`, etc.

### Milestones
- `M1-Phase1-Foundation` (Weeks 1–2)
- `M1-Phase2-SSH-Core` (Weeks 3–6)
- `M1-Phase3-Sudo-Sessions` (Weeks 7–10)
- `M1-Phase4-CP-UI` (Weeks 8–11, parallel)
- `M1-Phase5-Harden` (Weeks 12–14)
- `M1-Launch` (Week 14+)

### Issue body template
```markdown
## Summary
<1-2 sentence description>

## Acceptance criteria
- [ ] ...
- [ ] ...

## Dependencies
- Depends on: #<issue>
- Blocks: #<issue>

## Owner agent
`agent:<name>`

## Notes
<any context, links, ADRs>
```
