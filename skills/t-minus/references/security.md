# Security checklist

Part of the t-minus skill. Work through every item; record each in the report as Fixed / Already OK / Not applicable (with reason) / Needs human.

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
