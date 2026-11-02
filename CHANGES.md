# Review of the uploaded version

The original implementation had these concrete problems:

* The admin question metric used the first set's question count. Default totals could also be misleading before loading finished.
* Scores and maximum marks came from the first set, producing incorrect results for sets with different question counts.
* The question collection allowed public reads, so hiding a Start button did not protect premium content.
* Premium requests used new random documents without an atomic pending-request guard. Approval changed request status before membership, allowing partial failures.
* The leaderboard read private student profiles, which the supplied Firestore rules correctly denied.
* Login startup waited for unrelated data, while profile and plan caches could remain stale.
* A profile role field was trusted in the UI; the new UI uses the same verified admin identity as the rules and functions.
* Question preview showed media filenames rather than playable/rendered media. Question editing and deletion matched text, which could affect duplicate questions.
* Native image sizes and an unfixed exam footer caused layout movement. The old audio player provided no useful duration/progress feedback.
* The supplied rules were duplicated and did not include server-protected payment, attempt and leaderboard collections.

# Delivered features

| Request | Implementation |
| --- | --- |
| Faster startup | Immediate shell/cached profile; plans, history, config and question bank load concurrently |
| Correct admin count | Actual complete bank count, including zero |
| Better admin UI | Revised visual theme, action cards, responsive controls and readable forms |
| Clickable overview | Cards open the related question/request/student view and student filter |
| Membership control | Custom days, extension, remove premium, note editing, filters, block and device reset |
| Duplicate requests | Deterministic request plus transactional pending check, including legacy requests |
| Question management | Set/section/search filters, rendered preview, full filtered preview, checkbox bulk deletion, move up/down and stable question IDs |
| Device restriction | Server checks active membership and the bound browser credential |
| Watermark | Visible Dream Japan watermark on question cards |
| Bookmarks | Save/remove from exam or practice; persistent user bookmarks with question IDs |
| Leaderboard | Minimal dedicated collection written by backend grading |
| Plans and payment | Current status plus all available plans; QR, copy number/email, reference, request and WhatsApp draft |
| Stable media | White 16:9 image area, fixed audio player and fixed bottom navigation |
| Installable app | Manifest, icons, service worker and admin-controlled install suggestion |
| Center confirmation | Dialog centered in the viewport with scroll handling |
| Modern audio | Play/pause, duration, progress, play counts and playback error messages |
| Preload | Whole selected set downloads before the exam timer starts |
| Marks | No per-question marks in test/practice; independent weighted 250-point calculation for each set |

# Limits to review before production

Backend deployment and real Firebase tests are required. A browser binding is not hardware identity. Public GitHub assets and browser-visible content cannot be made copy-proof. The normal browser URL bar remains visible; installed standalone PWA mode removes the normal bar. Admin receives in-app notices while the app is open. eSewa payments are manually verified. The existing one-document question bank remains subject to Firestore size limits. See README.md for deployment and staging checks.
