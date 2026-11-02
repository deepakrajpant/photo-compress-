# Dream Japan JFT Mock Test 3.0

This is a complete replacement for your uploaded HTML, CSS and JavaScript, with the Firebase backend needed for paid access, browser binding and server-calculated results. It keeps your existing Firebase project, question document and six set names.

**Uploading only index.html, style.css and script.js is not sufficient for the new premium system. Deploy the included functions and rules first.** The code has not been deployed to your production Firebase project or pushed to GitHub.

## 1. Back up before installation

Download your existing website files. In the existing admin page, export the question bank. Back up Firestore before replacing production rules. Your existing question bank remains at:

`jftoctnovset/QORwII8D7qLZ7wuCDW36`

This version supports the existing 11-column row format:

```
[setNumber, sectionKey, question, passage, image, audio, answer, A, B, C, D]
```

For example:

```json
[1,"sv","これは何ですか？","NA","photos/s1q1.jpg","NA","A","本","机","椅子","NA"]
```

Object questions are supported too:

```json
{"id":"unique-question-id","set":1,"section":"lc","question":"正しい答えを選んでください。","passage":"","image":"","audio":"audio/s1q2.mp3","answer":"B","A":"月曜日","B":"火曜日","C":"水曜日","D":"木曜日"}
```

Sections: `sv`, `ce`, `lc`, `rc`. Set numbers: 1 through 6. Optional `NA` values become empty fields. Do not import the different 12-column spreadsheet format into this version; convert it to the schema above first. Existing questions receive stable IDs when edited or reordered. Existing bookmarks without IDs remain usable by their previous set/section/index until the question order changes; re-save those bookmarks after reordering.

## 2. Deploy Firebase backend

Use Node.js 22 and the official Firebase CLI. Cloud Functions deployment requires a Firebase project with billing enabled and the necessary deployment permissions. No service-account key belongs in this repository.

From this project folder:

```bash
cd functions
npm install
npm test
cd ..
firebase login
firebase use dream-japan-9cad8
firebase deploy --only functions,firestore:rules
```

The callable functions use `us-central1`. If you change the region, update both `functions/index.js` and `getFunctions(app, "us-central1")` in `script.js`.

The Firebase web configuration in script.js is the configuration from your uploaded code. Keep your current Firebase Authentication providers enabled. In Firebase Authentication > Settings > Authorized domains, include `deepakrajpant.github.io`. For Google login, allow popups; redirect fallback remains available. If email verification was completed in a separate tab, sign out and sign in again before entering admin.

Only this email with `email_verified == true` is an administrator:

`deepakrajpant978@gmail.com`

Do not keep the duplicated rules you pasted. The included `firestore.rules` contains one service block. The browser no longer writes results or payment requests directly; callable functions handle them. Students cannot read the question collection directly.

## 3. Upload website files to GitHub

Put these files at the root of your `deepakrajpant/testapi` repository:

```
index.html
style.css
script.js
manifest.webmanifest
sw.js
icons/icon.svg
icons/icon-192.png
icons/icon-512.png
photos/esewa.jpg
photos/your-question-images.jpg
audio/your-question-audio.mp3
```

Keep `photos/` and `audio/` from your existing repository. The package does not contain your private question media or an actual eSewa QR. Add your own `photos/esewa.jpg` before publishing.

You may also keep the Firebase deployment files and `functions/` in the repository. Never commit service-account JSON, Firebase CLI tokens, passwords or `node_modules/`.

Media base URL is `https://deepakrajpant.github.io/testapi/`, which matches your Pages site. GitHub filenames are case sensitive. If your Pages site is hosted elsewhere, update `REPO_BASE` in script.js.

## 4. Configure admin settings

Sign in using the verified admin email, then open Profile > Admin Dashboard > Settings.

1. Enter the actual eSewa number. It is deliberately blank because no payment number was supplied.
2. Keep QR path `photos/esewa.jpg`, or change it to another image you upload.
3. Confirm the WhatsApp number. The supplied `+97766157755` is preserved as `97766157755`; verify the intended full number before using it.
4. Add or edit premium plans, prices, days and benefits. Hidden plans cannot be requested. If there are no existing plans, create at least one; the app does not invent a price.
5. Choose whether to show the install suggestion.

Active membership is visible together with every currently available plan. Payment requests include the selected plan, price, duration, transaction reference and browser binding. Students copy the eSewa number and their own email for the payment remark. Clicking payment completed creates a pending request; it does not claim that payment has been automatically verified.

The admin sees a live in-app request count and a toast when a new pending request arrives while signed in. This is not a background push, SMS or email notification. The WhatsApp action opens a prefilled draft for the student to review and send; the website does not automatically send messages.

## 5. Membership and browser binding

For payment requests, the browser installation used to request premium is bound when the admin approves payment. A second browser cannot retrieve paid questions or submit results. Admin approval updates membership and request status in one transaction, preventing a request from being approved twice.

Admin > Students supports:

* Name/email/note search and active, blocked, premium or expired filters.
* Grant any whole number of days from 1 through 3650.
* Remove premium, block/unblock, edit notes and reset the bound browser.
* Extend an active membership by adding purchased days to its existing expiry.

For a manual premium grant with no prior payment binding, the first browser to open paid content becomes the bound browser. Existing premium users are bound on their next paid access. Reset Device allows the next browser to bind again. Clearing browser data, using incognito or switching browsers requires an admin reset.

This is a browser-installation credential, not a physical hardware identity. Someone who deliberately copies their browser credential can share it. A public HTML/JavaScript application cannot provide hardware DRM or prevent screenshots. Watermarks are a visible ownership reminder and can be removed by a determined user. GitHub media URLs remain public. For confidential media, use authenticated storage or expiring server URLs instead of a public repository.

## 6. Exams, marks and media

Section weights remain 3, 4, 6 and 7. Each set independently calculates:

```
originalFullMarks = sum(sectionQuestionCount * sectionOriginalMarks)
newQuestionMarks = (originalQuestionMarks * 250) / originalFullMarks
score = (earnedOriginalMarks * 250) / originalFullMarks
```

Scores display to two decimals, maximum 250, with the existing 200/250 mock pass target. This is your mock-test scoring model, not a claim to reproduce the official JFT scoring algorithm.

Question marks are hidden during exams and practice. The backend grades submitted answer indices using the question snapshot taken for that attempt; the browser cannot submit an arbitrary numeric score to the leaderboard. Answers still reach the student browser for practice and review, so these are practice rankings, not a secure proctored-examination system. Leaderboard entries contain only a display name, score and streak. Existing user best scores are not automatically imported; new verified submissions populate the new leaderboard collection.

All media for the selected exam set is fetched before the server timer starts, with five concurrent downloads and a timeout per file. A failed file offers cancel, retry or explicit continue. Media cannot be guaranteed instantly available on an unreliable connection. Images reserve a white 16:9 slot and contain the whole picture. Audio has a fixed-height player, elapsed/total time, progress and remaining plays. Previous/Next stay fixed at the bottom of the viewport. Questions without media do not gain a blank media image.

Listening allows two starts per question in exam mode and unlimited starts in practice. Pausing and resuming the same playback does not consume another play. Speech-synthesis examples are supported when the browser provides Japanese speech.

Refresh restores the saved attempt for the same signed-in user and keeps the original deadline. The timer automatically submits at zero. A 15-minute server submission grace window permits a temporary connection failure; it is not a proctoring guarantee. Completed-section confirmation is centered, and previous sections cannot be reopened through the normal UI.

## 7. PWA and address bar

Open the HTTPS site in a supported browser and choose Install or Add to Home Screen. Launching the installed PWA uses standalone display without the normal browser address bar. A normal browser tab retains its address bar; website JavaScript cannot permanently hide it.

Admin settings control the app's install suggestion, not the browser's own installation menu. Browser install prompts are subject to browser eligibility rules and may not appear immediately. On iPhone/iPad, use Safari Share > Add to Home Screen. The service worker caches only the app shell; Firebase SDK loading, authentication, premium checks and cloud saving require a connection. Change the cache version in `sw.js` for later releases.

## 8. Verify after deployment

Use a test student account before accepting real payments:

1. Sign in/out with email and Google. Check password reset and verification.
2. As an unpaid student, open Tests, Practice and Bookmarks: plans must appear.
3. Choose a plan, verify QR/payment details and submit a test transaction reference. A repeated request must be denied.
4. In admin, verify email/reference/price, approve, and confirm paid access on the requesting browser.
5. Sign in from another browser: question retrieval must be denied. Reset Device in admin and verify rebinding.
6. Grant 2, 15 and 30 days, edit notes, remove premium and block the test account.
7. Filter questions by set/section, preview media, reorder, edit and bulk delete disposable test questions. Export a backup first.
8. Complete a test with known answers, confirm maximum 250 and compare the leaderboard. Check a set with a different number of questions.
9. Check image/audio errors, offline submission recovery, bookmarks, section confirmation and mobile browser Back.
10. Install the PWA and confirm installed display and the admin install-suggestion toggle.

The included 11 Node tests cover grading and mocked callable authorization/membership behavior. They do not replace Firestore Rules Emulator tests or a live Firebase staging test. JavaScript syntax and local browser flows were checked; production login/payment/device/PWA behavior has not been verified on your Firebase deployment.

The existing single-document bank schema is preserved for compatibility. Very large banks still need a per-question collection and pagination; do not exceed Firestore's document size limit. Server attempt snapshots also need periodic cleanup; configure a retention process for old `serverAttempts` documents before high-volume use.

## Sources used for implementation

* Firebase transactions: https://firebase.google.com/docs/firestore/manage-data/transactions
* Firebase callable functions: https://firebase.google.com/docs/functions/callable
* Firebase security rules: https://firebase.google.com/docs/firestore/security/get-started
