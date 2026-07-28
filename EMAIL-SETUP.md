# Turning on plan-of-action emails

The Repair CT Dashboard already does its half of the job: **on every plan-of-action
decision, the board writes an email into a Firestore collection called `mail`.**
You can see them piling up in the Firebase console under Firestore → `mail`.

Three events send mail, all to the same group:

| Event | Subject line | Includes |
|---|---|---|
| Plan **submitted** | `Plan of action submitted — …` | the plan, unit details, who submitted |
| Plan **approved** | `Plan of action APPROVED — …` | the plan, who approved, any note |
| **Changes requested** | `Plan of action — CHANGES REQUESTED — …` | what needs to change, plus the plan |

Nothing actually *sends* those yet. A web page can't send email on its own —
something server-side has to pick the message up and hand it to a mail server.
Below is the one-time setup that does it.

**Recipients** are hard-coded in `repair-site/index.html`, in the `PLAN_NOTIFY`
list near the top. Currently:

- Alfred@ghlogisticsllc.com
- Caleb@ghlogisticsllc.com
- Derrick@ghlogisticsllc.com
- Anna@ghlogisticsllc.com
- Andrea.b@ghlogisticsllc.com

---

## Setup — Firebase "Trigger Email from Firestore" extension

**Time:** about 15 minutes. **Cost:** effectively $0 for this volume, but it
requires a billing account on file (see the note at the bottom).

### 1. Get SMTP credentials (the mail server that does the sending)

Pick one:

**Option A — Google Workspace / Gmail** (you already have `@ghlogisticsllc.com`
mail, so this sends *from* a real company address):
- The sending account needs 2-Step Verification turned on.
- Go to https://myaccount.google.com/apppasswords and create an **App password**.
- Your connection string will be:
  `smtps://noreply@ghlogisticsllc.com:APPPASSWORD@smtp.gmail.com:465`
- Note: Gmail caps sending at ~2,000/day — far above what this needs.

**Option B — SendGrid / Brevo / Mailgun free tier** (cleaner separation, no
company-mailbox password involved):
- Create an account, generate an API key.
- SendGrid's string looks like:
  `smtps://apikey:YOUR_API_KEY@smtp.sendgrid.net:465`

### 2. Install the extension

1. Go to https://console.firebase.google.com → project **ghlogistics-driver-hub**
2. Left sidebar → **Extensions** → **Explore extensions**
3. Find **"Trigger Email from Firestore"** → **Install**
4. Fill in the configuration:
   - **SMTP connection URI:** the string from step 1
   - **Email documents collection:** `mail`  ← must match exactly
   - **Default FROM address:** e.g. `noreply@ghlogisticsllc.com`
     (with Gmail this must be the same account as the app password)
   - **Default REPLY-TO address:** optional, e.g. your own address
   - Leave the rest at defaults
5. Click **Install extension** and wait for it to finish (a few minutes)

### 3. Test it

1. Open the board, pick any repair, write a short plan, hit **Submit plan**
   (then try **Approve plan** and **Request changes** too)
2. You should see "Plan submitted — notification queued for 5 people."
3. Within a minute the five addresses get the email
4. If it doesn't arrive: Firebase console → Firestore → `mail` → open the newest
   document. The extension writes a `delivery` field onto it with the result —
   `delivery.state` will say `SUCCESS` or `ERROR`, and `delivery.error` will
   tell you exactly what went wrong (almost always a bad SMTP string).

---

## Until then: the manual fallback

Every time a plan is submitted, the confirmation banner has a **"Send it
myself"** button. That opens your normal mail app with all five recipients,
the subject, and the full plan details already filled in — you just hit Send.
It works today with zero setup, and it keeps working as a backup afterward.

---

## About the billing requirement

Firebase Extensions run on Cloud Functions, which requires the project to be on
the **Blaze (pay-as-you-go)** plan instead of the free Spark plan. Blaze keeps a
generous free allowance — a few emails a day costs nothing, and you can set a
budget alert (Firebase console → ⚙ → Usage and billing → Details & settings) so
there are no surprises. Upgrading requires a credit card on the Google account,
so that step has to be done by whoever owns the billing.

If you'd rather not enable billing at all, the "Send it myself" button above
covers the same ground with one extra click per plan.

---

## Changing who gets the emails

Edit the `PLAN_NOTIFY` list in `repair-site/index.html`, commit, and push —
the live site updates in about a minute. Or just ask Claude to change it.
