---
name: t-minus
description: Use before launching or shipping any web/mobile app (especially AI-built ones) to audit and fix security, legal compliance, UX, edge cases, accessibility, design slop and SEO, then report.
---

# T-Minus: Pre-Launch App Audit

The countdown before launch. Check every system, fix what is unsafe, hold the launch on anything that is not ready, and write it all down.

The detailed checklists live in `references/`. Read each one when you reach its phase, not all at once.

| Phase | Checklist |
|---|---|
| Security | [references/security.md](references/security.md) (includes 31 attack scenarios to test) |
| UX and product | [references/ux.md](references/ux.md) |
| Reliability and edge cases | [references/reliability.md](references/reliability.md) |
| Legal and compliance | [references/legal.md](references/legal.md) |
| Accessibility and design | [references/accessibility-and-design.md](references/accessibility-and-design.md) |
| Public pages and SEO | [references/public-pages-seo.md](references/public-pages-seo.md) |

## How to work

1. **Detect the stack first.** Read the repo: framework, database, auth provider, storage, hosting, payment provider, email/SMS provider, analytics, AI features. Items that name a tool (Supabase, Stripe, Next.js) apply only if that tool is used; otherwise apply the equivalent for the actual stack. Items marked *(if X)* apply only when the app has X.
2. **Never silently skip an item.** Every item ends up in the report as Fixed / Already OK / Not applicable (with reason) / Needs human.
3. **Work in phases, in this order**, reading the matching reference file at each phase:
   - (A) Audit everything and write findings before changing code.
   - (B) Critical security fixes (`security.md`).
   - (C) Auth and UX fixes (`ux.md`), then edge cases (`reliability.md`).
   - (D) Legal pages and consent flows (`legal.md`).
   - (E) Accessibility and design cleanup (`accessibility-and-design.md`).
   - (F) Public pages and SEO (`public-pages-seo.md`), only if the app has public or marketing pages.
   - (G) Test every fix.
   - (H) Write the report.
4. **Ask before anything destructive or irreversible:** rewriting git history, dropping or migrating tables with data, rotating live keys, changing the auth provider.
5. **Do not break existing features.** Run the app and existing tests after each phase.
6. **Use general, trusted sources** for legal and security guidance: official provider docs, OWASP, WCAG, and the actual texts of GDPR, CCPA/CPRA, COPPA, CAN-SPAM, DMCA. Do not invent legal facts. Recommend human legal review before launch.
7. **Verify, do not assume.** Each fix is tested with an automated test or a manual reproduction showing the issue is blocked.
8. **Extra tools:** run the Cloudflare security audit skill (https://github.com/cloudflare/security-audit-skill/tree/main/skills/security-audit) if available, and run `/security-review` before finishing (Claude Code).
9. If the user's brief conflicts with a checklist item, resolve it explicitly and note the resolution (see Known tensions).

## Known tensions and how to resolve them

- **"Never log me out" vs "sessions must expire":** short-lived access tokens + long-lived, rotating, revocable refresh tokens in secure httpOnly cookies. Logout and password change revoke all sessions.
- **"Add Google Analytics" vs "no trackers before consent":** load analytics only after the user accepts cookies.
- **"Use nice fonts" vs GDPR font issue:** self-host fonts instead of loading from Google Fonts.

## Human-only tasks (list in the report)

- Two-factor auth on every account: hosting, database, domain registrar, git provider, payment provider.
- Rotate leaked keys in provider dashboards.
- Billing alerts and spend caps in each paid service.
- Confirm automated backups in provider dashboards.
- Register a DMCA copyright agent *(if user uploads)*.
- Legal review of policies by a qualified person.

## Report

Write `LAUNCH_AUDIT_REPORT.md` with:
1. Detected stack and which conditional items applied.
2. Every checklist item: status (Fixed / Already OK / Not applicable + reason / Needs human), what changed, which files.
3. Every attack scenario and edge case: how it was tested, result.
4. Dependency scan before/after; packages updated and removed.
5. Git history scan results; keys needing rotation.
6. `/security-review` and other audit tool findings and actions.
7. Remaining risks and recommendations.
8. A final **Go / No-Go** verdict: No-Go if any critical security item is still open.
9. The human-only task checklist.

If the audit is a follow-up to an earlier pass, verify earlier fixes instead of redoing them, mark them "Already OK", and append to the existing report.
