# t-minus

The countdown before you ship. A Claude skill that audits and fixes your app before launch.

> **Why "T-minus"?** In rocket launches, "T" is the moment of liftoff, and "T-minus 10" means ten seconds before launch. During the countdown, every system is checked one by one, and if anything is not safe, the launch is held. This skill does the same for your app: a final go/no-go check of every system before it goes live.

Built for apps made fast with AI ("vibe coding"), but it works on any web or mobile app. Claude reads your codebase, works through a checklist of security holes, legal gaps, UX problems, real-world edge cases, accessibility issues and generic AI design patterns, fixes what it can, tests each fix, and writes a report.

## What it checks

| Area | Examples |
|---|---|
| **Security** | Keys in the browser, missing row-level security, IDOR (changing an ID in the URL), SQL injection, XSS, price tampering, fake payments, unverified webhooks, rate limiting, CSRF, session cookies, security headers, leaked secrets in git history, 31 concrete attack scenarios |
| **Reliability** | Double-tapped pay buttons, capitalized emails, apostrophes in names, iPhone HEIC photos, time zones, lost connection after paying, account deletion, traffic spikes |
| **Legal** | Privacy policy that matches what you collect, terms, refund and cookie policies, cookie banner with a real reject button, no trackers before consent, data export and deletion, age gate, auto-renewal disclosure, unsubscribe links, self-hosted fonts, session replay |
| **UX** | Branding, low-friction sign-in, persistent login, realtime updates, native-app feel, always-reachable primary actions, profile and logout |
| **Accessibility** | Alt text, color contrast, keyboard navigation, automated checks |
| **Design** | Removes common AI-generated design tells (gradient hero text, glassmorphism, untouched shadcn, Inter everywhere, and more) |
| **Public pages / SEO** | 404 page, meta tags, Open Graph images, robots.txt, schema, analytics after consent |

Items that only apply to some apps (payments, uploads, AI features, subscriptions, marketing pages) are marked conditional. Every item ends up in the report as **Fixed**, **Already OK**, **Not applicable** (with a reason) or **Needs human**.

## Install

### Claude Code (as a plugin)

```
/plugin marketplace add VikDevarStudio/t-minus
/plugin install t-minus@t-minus
```

### Any agent with the skills CLI

```bash
npx skills add VikDevarStudio/t-minus --skill t-minus
```

### Claude Code (manual)

Copy the skill folder into your skills directory:

```bash
# for all projects
mkdir -p ~/.claude/skills
cp -r skills/t-minus ~/.claude/skills/

# or for one project only
mkdir -p .claude/skills
cp -r skills/t-minus .claude/skills/
```

### Claude apps (claude.ai / desktop)

Zip the `skills/t-minus` folder and upload it in the Skills section of your Claude settings.

## Usage

Open your project and ask:

```
Run t-minus on this app before I launch.
```

Or just say you are about to ship; the skill triggers on launch and shipping requests. Claude will:

1. Detect your stack
2. Audit and list findings
3. Fix security issues, then UX, legal, accessibility, design and SEO
4. Ask before anything destructive (rewriting git history, migrating data, rotating keys)
5. Test every fix
6. Write `LAUNCH_AUDIT_REPORT.md`, including a list of tasks only you can do (2FA on your accounts, billing alerts, legal review)

## Disclaimer

This skill is a checklist and a helper, not a guarantee. It does not replace a professional security audit or legal advice. Have a qualified person review your legal pages before launch.

## License

[MIT](LICENSE)
