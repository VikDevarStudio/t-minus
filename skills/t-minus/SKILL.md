---
name: t-minus
description: Use before launching or shipping any web/mobile app (especially AI-built ones) to audit and fix security, legal compliance, UX, edge cases, accessibility, design slop and SEO, then report.
---

# T-Minus: Pre-Launch App Audit

A full pre-launch review and hardening pass for any app. Covers seven areas: UX/product, security, reliability edge cases, legal/compliance, accessibility, design anti-patterns, and public pages/SEO. Ends with a written report.

## How to work

1. **Detect the stack first.** Read the repo: framework, database, auth provider, storage, hosting, payment provider, email/SMS provider, analytics, AI features. Items below that name a tool (Supabase, Stripe, Next.js) apply only if that tool is used; otherwise apply the equivalent for the actual stack. Items marked *(if X)* apply only when the app has X.
2. **Never silently skip an item.** Every item ends up in the report as Fixed / Already OK / Not applicable (with reason) / Needs human.
3. **Work in phases, in this order:** (A) audit and write findings, (B) critical security fixes, (C) auth and UX fixes, (D) legal pages and consent flows, (E) accessibility and design cleanup, (F) public pages/SEO, (G) test everything, (H) write the report.
4. **Ask before anything destructive or irreversible:** rewriting git history, dropping/migrating tables with data, rotating live keys, changing auth provider.
5. **Do not break existing features.** Run the app and existing tests after each phase.
6. **Use general, trusted sources** for legal and security guidance: official provider docs, OWASP, WCAG, and the actual texts of GDPR, CCPA/CPRA, COPPA, CAN-SPAM, DMCA. Do not invent legal facts. Recommend human legal review before launch.
7. **Verify, do not assume.** Each fix is tested (automated test or a manual reproduction showing the issue is blocked).
8. **Extra tools:** run the Cloudflare security audit skill (https://github.com/cloudflare/security-audit-skill/tree/main/skills/security-audit) if available, and run `/security-review` before finishing (Claude Code).
9. If the user's brief conflicts with a checklist item, resolve it explicitly and note the resolution (see Known tensions).

## Known tensions and how to resolve them

- **"Never log me out" vs "sessions must expire":** short-lived access tokens + long-lived, rotating, revocable refresh tokens in secure httpOnly cookies. Logout and password change revoke all sessions.
- **"Add Google Analytics" vs "no trackers before consent":** load analytics only after the user accepts cookies.
- **"Use nice fonts" vs GDPR font issue:** self-host fonts instead of loading from Google Fonts.

---

## 1. UX and product

- **Branding everywhere:** app icon, favicon, logo and name visible throughout the app, not only on the login page.
- **Low-friction auth:** avoid magic-link-only sign-in that bounces users to their email app in a new tab. Prefer email + password + OTP verification, plus OAuth (e.g. Google sign-in). If an OAuth user is new, route them to a "complete your profile" step before they can use the app.
- **Persistent login** like big consumer apps (users practically never get logged out), resolved securely as in Known tensions.
- **Realtime status:** anything one party changes that another party is watching (order status, booking status, messages) updates live via websocket/realtime subscription, with reconnect and fallback. No manual refresh needed.
- **Native-app feel** for consumer apps: installable PWA (manifest, icons, splash, standalone display, offline shell), bottom navigation, native-style transitions, large touch targets; or a native wrapper (e.g. Capacitor) if better. Explain the choice.
- **Follow conventions of large established apps** in the same category for navigation, cart, checkout, profile and tracking.
- **Primary actions always reachable:** e.g. a sticky/floating cart or checkout bar once items are added (with count and total), not a button buried at the bottom of the page.
- **Profile page and logout** exist and are easy to find.
- **Real accounts where identity matters:** if staff/admins need to know who did something (who placed an order), require registration and show a username in admin views. Show only what is needed (username, not email).
- **Check for more:** walk every user flow end to end for each role and fix other UX problems.

---

## 2. Security

### 2.1 Secrets, keys and git
- All API keys and secrets live on the server, never in browser code. Confirm nothing sensitive is in the client bundle.
- Frontend uses only the **public, limited** database key (e.g. Supabase anon/publishable). Admin/master key (e.g. Supabase `service_role`) is server-only. An exposed admin key makes row-level security useless.
- `.env` files not committed; `.gitignore` correct; production values separate from development.
- Scan full git history for leaked keys (gitleaks/trufflehog). For each leak: **rotate it** (deleting is not enough), then purge from history (ask first).
- `.git` folder cannot be downloaded from the deployed site.
- No source maps exposed in production.
- Debug mode off; admin/debug endpoints removed or locked; production settings checked.

### 2.2 Authentication and sessions
- Real login system; **hash every password** (bcrypt/argon2 or provider hashing). Never plain text.
- Auth enforced on every protected route **and** inside every server action/API handler, not just middleware or UI.
- Login cannot be skipped with a special header (e.g. Next.js `x-middleware-subrequest` bypass). Patch the framework as soon as security fixes drop.
- JWT: strong server-side secret, verify signature and expiry on every request so logins cannot be forged.
- Tokens in **secure, httpOnly, SameSite cookies**, not localStorage.
- Sessions expire; stolen tokens do not work forever.
- **Rate limit** login, OTP verification, signup and password reset. OTP/login codes not guessable (length, short expiry, attempt limits, lockout).
- Defend against credential stuffing (rate limits, bot protection, optional breached-password check).
- **Password reset links** expire, work only once, and are built from a fixed configured domain (never the request Host header) so they cannot point to an attacker's server.
- Email verification required so nobody can pre-register someone else's email (account pre-hijacking). Link OAuth and password accounts safely.
- No user enumeration: same response whether or not an email has an account (login, signup, reset).
- **CSRF protection** on all state-changing requests.
- **Bot protection** (Turnstile/CAPTCHA) on signup and all public forms.

### 2.3 Authorization and access control
- Admin/staff routes locked server-side by role; typing `/admin` gets nobody in.
- **No IDOR:** changing an ID in the URL/request never returns another user's data. Check the logged-in user owns the record before returning or modifying it.
- Users cannot make themselves admin at signup or edit their own role, credits, balance or other privileged fields (server-side field whitelist).
- API responses trimmed to needed fields; never return other people's information.

### 2.4 Database
- **Row-level security on every table**, with policies so each user only accesses their own data.
- No table fully public by default; review all database rules. Run Supabase security advisor *(if Supabase)*.
- **Parameterized queries / ORM everywhere**: user input is data, not code. No SQL injection.
- Anonymous users cannot wipe or mass-edit data.
- Encrypt sensitive data at rest.
- Automated backups enabled.

### 2.5 Input, output and XSS
- Validate and sanitize **all input on the server**, not just in the form.
- Escape user content on render (a `<script>` in a username renders as text).
- Error messages never leak stack traces or internals.

### 2.6 Payments and business logic *(if payments, pricing, discounts or credits)*
- **Recheck every price and total on the server**; never trust client values.
- Reject invalid quantities (negative, zero, non-integer, absurdly large).
- Payments confirmed only via **verified webhooks** (verify every Stripe/other provider signature), never a client "success" call.
- Discount/promo codes cannot be redeemed repeatedly (per-user and global limits enforced atomically).
- Idempotency keys: double-tapping pay never creates duplicate orders or charges.

### 2.7 File uploads and storage *(if uploads)*
- Check real file type (magic bytes, not extension) and size; only allowed types.
- Storage buckets private unless intentionally public; nobody can list/read all files.
- Files promised to be deleted are actually deleted from storage.
- No path traversal (`../`, `//`) letting anyone read server files.

### 2.8 Network, headers and infrastructure
- **Force HTTPS** (redirect + HSTS).
- Security headers: CSP, `frame-ancestors`/X-Frame-Options (no clickjacking / invisible frames), X-Content-Type-Options, Referrer-Policy, Permissions-Policy.
- **CORS** allows only your own origins.
- No open redirects that bounce users to phishing pages.
- No SSRF: server never fetches arbitrary URLs (e.g. cloud metadata endpoints with keys).
- Third-party CDN scripts avoided or pinned with Subresource Integrity.
- No dangling DNS / forgotten subdomains open to takeover.

### 2.9 Dependencies and supply chain
- Scan dependencies (`npm audit` or equivalent); update risky packages; remove unused ones.
- Every installed package is real and correctly named (no hallucinated/typosquatted packages).
- Every package license is compatible.

### 2.10 Abuse and cost control
- Rate limit all routes, especially anything that costs money per request (SMS, email, AI, third-party APIs). Protect SMS/OTP from bill-pumping.
- Spend caps on paid APIs; billing alerts recommended.

### 2.11 AI features *(if any AI/chatbot)*
- Resist prompt injection; never leak the system prompt.
- Rate limit AI calls and cap spend.
- Safe self-harm response path in any chatbot.
- No training on customer data without consent; disclose AI use in the privacy policy.

### 2.12 Logging and monitoring
- Logging in place so attacks are noticed.
- Logs never contain passwords, tokens or card numbers.

### 2.13 Attack scenarios to test (each must be shown blocked)
1. Type `/admin` and get in.
2. Change an ID in the URL to see another user's data.
3. Find an API key in the browser.
4. Edit the price before/at checkout.
5. Order `-1` items.
6. Fake a payment.
7. Redeem a discount 100 times.
8. Become admin at signup.
9. Guess the login/OTP code.
10. Tell the AI to ignore its instructions / leak its prompt.
11. Put a script in a username.
12. Upload something that is not an allowed file.
13. Find `.env`/secrets in git.
14. Type a SQL query into a form.
15. Skip login with one header.
16. Download the `.git` folder.
17. Read the storage bucket.
18. Log in with leaked passwords.
19. Wipe an open database.
20. Inject malware via a CDN script.
21. Run up the SMS bill.
22. Forge a login with the JWT secret.
23. Sign up with someone's email before they do.
24. Take over a forgotten subdomain.
25. Make the server fetch its own cloud keys.
26. Read server files via path traversal.
27. Redirect a password reset link to an attacker's server.
28. Read source code from source maps.
29. Load the site in an invisible frame.
30. Bounce users to a phishing page.
31. Discover which emails have accounts.

---

## 3. Reliability and real-world edge cases

Test and handle:
- Double-tapping the pay/submit button.
- Email with capital letters (trim + lowercase for lookup and uniqueness).
- Apostrophes and non-ASCII characters in names.
- Photos straight from an iPhone (HEIC, large size, EXIF orientation; strip EXIF location).
- Full password reset flow.
- Different time zones (store UTC, display local; opening hours, bookings, timestamps).
- Connection lost right after paying (state reconciled via webhook; correct status when back online).
- Account deletion (works; data actually deleted or anonymized).
- Sign in with Apple using Hide My Email (relay addresses); missing/private emails in general.
- Sudden traffic spike (e.g. 10,000 users at once after going viral): check bottlenecks, connection limits, N+1 queries, caching, and that rate limits do not block legitimate users.

---

## 4. Legal and compliance

### 4.1 Pages and documents
- **Privacy Policy** matching exactly what is collected: states what user data is collected, whether AI is used, and names every third party/sub-processor data is sent to.
- **Published sub-processor list.**
- **Terms of Service** with a liability cap.
- **Refund Policy.**
- **Cookie Policy.**
- **DPA** offered to business customers *(if B2B)*.
- **Business details** (legal entity, contact).

### 4.2 Consent and data rights
- Unticked checkbox required to accept Terms; all form consents reviewed.
- **Cookie consent banner** with a **Reject** button as easy as Accept.
- No trackers/analytics loaded before consent.
- Data minimization: do not collect unnecessary data.
- Users can **export** their data and **request deletion**.
- **Age gate:** ask age at signup; no children's data without parental consent (COPPA, under 13).

### 4.3 Subscriptions *(if any recurring billing or free trial)*
- Auto-renewal terms disclosed before checkout, right next to the pay button.
- Reminder before a free trial auto-charges.
- Cancel online, with no more steps than signing up.

### 4.4 Remove or fix
- Audit third-party SDKs and what data they send.
- Remove dark patterns and hidden fees (all fees shown before checkout).
- Remove fake reviews/testimonials; show only real reviews.
- Remove unsupported claims: no SOC 2 or other certifications you lack, no uptime promises you cannot meet.
- No customer logos without permission.
- Remove unlicensed images and fonts; license any stock images.
- **Self-host fonts** (Google Fonts sends visitor IPs to Google; German court precedent).
- **Session replay** off, or fully masked (keystroke recording can count as wiretapping in California).
- **Unsubscribe link in every marketing email**, including the launch email (CAN-SPAM).
- *(If user uploads)* flag registering a DMCA copyright agent as a human task.

---

## 5. Accessibility

- Alt text on all meaningful images.
- Color contrast meets WCAG AA, including dark mode.
- Full keyboard navigation with visible focus.
- Passes automated checks (axe/Lighthouse); fix what they report.

---

## 6. Design: the app must NOT contain

- Gradient hero text
- Emojis in headings
- Inter font everywhere
- Colored left-border cards
- Glassmorphism cards
- Low-contrast dark mode
- Three icon boxes in a row
- A badge above the headline
- Lucide icons everywhere
- Untouched shadcn components (customize to the brand)
- Sections fading in on scroll
- Cursor-following line/glow
- Buttons that fade on hover
- Inconsistent spacing (use one spacing scale)
- Em dashes everywhere in UI copy
- Generic buzzwords
- Serif italics for accent words
- Space Grotesk + Instrument Serif pairing
- Grain texture

---

## 7. Public pages and SEO *(if the app has a marketing site or public pages)*

- Custom 404 page
- Clear call to action above the fold
- Internal links
- Thank-you page after inquiry/form submission
- Breadcrumbs
- Case study section
- 5 FAQs
- Response time promise
- Sticky mobile call to action
- `robots.txt` and sitemap
- Unique page titles
- Meta descriptions
- Social share images (Open Graph)
- Map plus directions *(if physical location)*
- Real customer reviews only
- Alt text on images
- Local business schema *(if local business)*
- Privacy policy page linked in footer
- Google Analytics, loaded only after consent

---

## 8. Human-only tasks (list in report)

- Two-factor auth on every account: hosting, database, domain registrar, git provider, payment provider.
- Rotate leaked keys in provider dashboards.
- Billing alerts and spend caps in each paid service.
- Confirm automated backups in provider dashboards.
- Register a DMCA copyright agent *(if user uploads)*.
- Legal review of policies by a qualified person.

---

## 9. Report

Write `LAUNCH_AUDIT_REPORT.md` with:
1. Detected stack and which conditional items applied.
2. Every item in sections 1 to 7: status (Fixed / Already OK / Not applicable + reason / Needs human), what changed, which files.
3. Every attack scenario (2.13) and edge case (3): how tested, result.
4. Dependency scan before/after; packages updated and removed.
5. Git history scan results; keys needing rotation.
6. `/security-review` and other audit tool findings and actions.
7. Remaining risks and recommendations.
8. Human-only task checklist (section 8).

If the audit is a follow-up to an earlier pass, verify earlier fixes instead of redoing them, mark them "Already OK", and append to the existing report.
