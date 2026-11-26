# Reliability and edge-case checklist

Part of the t-minus skill. Work through every item; record each in the report as Fixed / Already OK / Not applicable (with reason) / Needs human.

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
