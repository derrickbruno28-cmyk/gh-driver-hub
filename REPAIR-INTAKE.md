# Repair CT Dashboard — collaboration & Telegram bot guide

This doc is for anyone working on the **Repair CT Dashboard**
(`public/repair/index.html`), especially the Telegram intake bots.

- **Live board:** https://ghlogistics-driver-hub.firebaseapp.com/repair/
- **Firebase project:** `ghlogistics-driver-hub` (Firestore + Storage + Auth)

---

## 1. Working on the site together

The whole app is one file: `public/repair/index.html` (React via inline JSX,
no build step). To collaborate:

1. **Get repo access.** The owner invites you on GitHub:
   repo **derrickbruno28-cmyk/gh-driver-hub** → Settings → Collaborators → Add people.
2. **Clone it:**
   ```bash
   git clone https://github.com/derrickbruno28-cmyk/gh-driver-hub.git
   ```
3. **Edit → commit → push.** Every push to `main` auto-deploys to Firebase
   Hosting via GitHub Actions (~1 minute). Nothing else to run.
4. **Pull before you edit** (`git pull`) so you don't step on each other.

To preview locally: `npx serve public` then open http://localhost:3000/repair/

For the bot you'll also want **Firebase console access**: the owner adds you at
console.firebase.google.com → project settings → Users and permissions.

---

## 2. How the "Incoming issues" queue works

The board polls the Firestore collection **`repairIntake`** every 30 seconds.
Every document in that collection shows up in the **Incoming issues** panel
(top bar button, with a count badge, filterable Trucks/Trailers). From there a
dispatcher either:

- **Accepts it** — the "Report a unit down" form opens pre-filled from the doc;
  when they save, the repair lands on the board and the intake doc is
  **deleted**; or
- **Dismisses it** — the intake doc is just **deleted**.

So the bot's only job: **parse a Telegram post → write one document to
`repairIntake`**. The board does the rest.

## 3. Intake document schema

All fields are strings unless noted. Everything is optional except `unitType`,
`unitNumber`, and `issue` — but the more you fill, the less the dispatcher has
to fix by hand. Matches the Telegram template.

```jsonc
{
  "unitType": "Trailer",              // "Truck" or "Trailer" (drives the filter)
  "unitNumber": "TR-9082",
  "issue": "Landing gear crank sheared, ABS light",
  "priority": 2,                      // number: 1 Critical / 2 High / 3 Routine
  "date": "7/28/2026",                // from the post's "Date:" line
  "requestedBy": "J. Alvarez",
  "trailerType": "Dry van",           // trailers only
  "attachedTruck": "T-4417",          // trailers only
  "location": "San Antonio, TX",      // city/state or yard; if it contains
                                      // San Antonio / Austin / Dallas / Memphis
                                      // the form picks that terminal, else OTR
  "safeToMove": "YES",                // YES/NO (or boolean)
  "loaded": "NO",                     // YES/NO (or boolean)
  "deliveryBy": "7/29 14:00",         // if loaded
  "neededBy": "7/30 08:00",           // "needed back in service by"
  "media": [                          // photos/videos from the post
    {
      "url": "https://firebasestorage.googleapis.com/...",
      "name": "IMG_4412.jpg",
      "mime": "image/jpeg",           // used to tell photo vs video
      "size": 2048576,                // bytes (optional)
      "path": "driverDocs/tg_...jpg"  // Storage path (optional, enables delete)
    }
  ],
  "createdAt": "2026-07-28T15:30:00.000Z",  // ISO — used to sort newest first
  "raw": "full original message text"        // optional, for debugging
}
```

**Media:** download the Telegram file, upload it to this project's
**Firebase Storage** under `driverDocs/` (any filename), then put the download
URL in `media`. Don't link straight to Telegram URLs — they expire. Extensions
`.mp4 .mov .m4v .webm .avi` (or a `video/*` mime) count as video; everything
else is treated as a photo.

## 4. Bot setup (server side)

Use the **firebase-admin** SDK with a service account (bypasses security rules,
no login flow):

1. Firebase console → Project settings → **Service accounts** →
   Generate new private key. **Keep this JSON secret** — never commit it.
2. Minimal example:

```js
const admin = require("firebase-admin");
admin.initializeApp({
  credential: admin.credential.cert(require("./serviceAccount.json")),
  storageBucket: "ghlogistics-driver-hub.firebasestorage.app",
});
const db = admin.firestore();

async function postIntake(item) {
  item.createdAt = new Date().toISOString();
  await db.collection("repairIntake").add(item);
}
```

3. Point your Telegram bot's message handler at `postIntake(parsedTemplate)`.

Notes:
- One doc per post. The board deletes docs on accept/dismiss — don't re-post
  the same issue on edits unless you want it to reappear.
- Uploading media: `admin.storage().bucket().upload(localPath, { destination: "driverDocs/tg_" + Date.now() + "_" + filename })`,
  then `file.getSignedUrl()` or make it public-read via `getDownloadURL`-style
  token — simplest is `await file.makePublic()` and use the public URL, or use
  the Firebase Storage download-token URL.
- Test without Telegram: add a doc by hand in Firebase console → Firestore →
  `repairIntake` → Add document, and watch it appear on the board within 30s.

## 5. Other data the board uses (don't clash with these)

| Where | What |
|---|---|
| Firestore `repairBoard/fleet-repair-board-v1` | the whole shared board (one JSON blob, versioned by `rev`) |
| Firestore `repairIntake/*` | incoming queue (bot writes, board deletes) |
| Storage `driverDocs/*` | uploaded invoices/photos/videos (shared with the Driver Hub's driver documents) |
| Firestore `drivers`, `recruits`, `leads`, `meta` | the Driver Hub app — leave alone |

Sign-in on the site is limited to `@ghlogisticsllc.com` / `@ajgtransport.com`
Google accounts.
