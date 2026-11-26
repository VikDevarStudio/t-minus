# UX and product checklist

Part of the t-minus skill. Work through every item; record each in the report as Fixed / Already OK / Not applicable (with reason) / Needs human.

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
