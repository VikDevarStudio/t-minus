# t-minus

**The countdown before you ship.** A Claude skill that audits your app before launch and fixes what it finds.

Getting hacked is only one way a launch goes wrong. You can also get fined for a missing privacy policy, charge someone twice when they double-tap "Pay", or lose users because sign-in takes three tabs. t-minus checks all of it in one pass:

- **Will it get hacked?** 31 concrete attack scenarios, from changing an ID in the URL to faking a payment
- **Will it get you in legal trouble?** Privacy policy, cookie consent, age gate, auto-renewal rules, unsubscribe links
- **Will it break in real life?** Double taps, iPhone photos, time zones, lost signal after paying, 10,000 users at once
- **Will people actually like using it?** Sign-in friction, persistent login, realtime updates, native-app feel
- **Does it look AI-generated?** Removes the usual tells: gradient hero text, glassmorphism, untouched shadcn, Inter everywhere

Then it tests every fix and writes a report with a final **Go / No-Go** verdict.

> **Why "T-minus"?** In rocket launches, "T" is the moment of liftoff, and "T-minus 10" means ten seconds before launch. During the countdown, every system is checked one by one, and if anything is not safe, the launch is held. This skill does the same for your app.

## Install

**Claude Code (plugin)**
```
/plugin marketplace add VikDevarStudio/t-minus
/plugin install t-minus@t-minus
```
In the Claude desktop app, open the plugin browser in a local Code session and add the marketplace `VikDevarStudio/t-minus`.

**Any agent with the skills CLI** (Claude Code, Cursor, Codex and others)
```bash
npx skills add VikDevarStudio/t-minus --skill t-minus
```

**Manual**
```bash
git clone https://github.com/VikDevarStudio/t-minus
cp -r t-minus/skills/t-minus ~/.claude/skills/
```

## Usage

Open your project and say:

```
Run t-minus on this app before I launch.
```

Claude will:

1. Detect your stack (Next.js, Supabase, Stripe, Firebase, Express, whatever you use) and skip what does not apply
2. Audit everything and list findings before touching code
3. Fix security first, then UX, edge cases, legal, accessibility, design and SEO
4. Ask before anything destructive (rewriting git history, migrating data, rotating keys)
5. Test every fix
6. Write `LAUNCH_AUDIT_REPORT.md` with a status for every item, a Go / No-Go verdict, and a short list of things only you can do (2FA on your accounts, billing alerts, legal review)

## What it checks

| Area | Checklist |
|---|---|
| Security | [security.md](skills/t-minus/references/security.md) |
| UX and product | [ux.md](skills/t-minus/references/ux.md) |
| Reliability and edge cases | [reliability.md](skills/t-minus/references/reliability.md) |
| Legal and compliance | [legal.md](skills/t-minus/references/legal.md) |
| Accessibility and design | [accessibility-and-design.md](skills/t-minus/references/accessibility-and-design.md) |
| Public pages and SEO | [public-pages-seo.md](skills/t-minus/references/public-pages-seo.md) |

Items that only apply to some apps (payments, uploads, AI features, subscriptions, marketing pages) are marked conditional.

## Contributing

Found a launch problem t-minus missed? [Open an issue](https://github.com/VikDevarStudio/t-minus/issues) with what happened and how to check for it.

## Disclaimer

t-minus is a checklist and helper, not a guarantee. It does not replace a professional security audit or legal advice. Have a qualified person review your legal pages before launch.

## License

[MIT](LICENSE)
