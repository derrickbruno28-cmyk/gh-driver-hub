# GH Logistics Driver Hub

A fleet and driver management web app for GH Logistics — driver roster, availability,
team pairings, recruitment pipeline, DQF/insurance tracking, safety violations, and
company pay/policy reference.

**Live site:** https://ghlogistics-driver-hub.firebaseapp.com

> Always share this `.firebaseapp.com` link with the team. Google sign-in is
> pinned to the same Firebase origin the app is served from, so opening the app
> from another address triggers sign-in errors:
> - the old `gh-driver-hub.netlify.app` link → "missing initial state" (storage
>   partitioning), and
> - the `.web.app` link → "Error 400: redirect_uri_mismatch", because Google's
>   OAuth client only trusts the `.firebaseapp.com` sign-in handler by default.
>
> To also allow the `.web.app` URL, add
> `https://ghlogistics-driver-hub.web.app/__/auth/handler` to the OAuth 2.0
> client's **Authorized redirect URIs** in the Google Cloud console.

## Overview

The app is a single self-contained `public/index.html` file (HTML + CSS + JavaScript,
no build step). It uses **Firebase** as its backend and falls back to browser
`localStorage` so it keeps working offline. Sections include:

- **Overview** — Dashboard, Master Driver List
- **Status** — Availability, Time Off, Call-Out Log, Constraints, Return Dates
- **Roster** — Team Pairings, Solo Drivers, Programs
- **Recruitment** — Leads / Call Log, Road Tests / New Hires, Terminated / Removed
- **Admin** — RSI Insurance List, YOE & Check Calls, DQF Files
- **Safety** — Points Memo, Driver Violations, Safety Training
- **Reference** — Policies & Pay (SOP)

## Project structure

Two sites deploy from this repo (both on the same Firebase project):

| Site | Folder | Live URL |
|---|---|---|
| Driver Hub | `public/` | https://ghlogistics-driver-hub.firebaseapp.com |
| Repair CT Dashboard | `repair-site/` | https://fleet-repair-ctdashboard.firebaseapp.com |

```
.
├── public/
│   ├── index.html      # Driver Hub (HTML/CSS/JS in one file)
│   └── repair/
│       └── index.html  # redirect stub to the Repair CT Dashboard's URL
├── repair-site/
│   └── index.html      # Repair CT Dashboard (repair board, self-contained)
├── netlify.toml        # Legacy Netlify config (retired)
├── firebase.json       # Firebase Hosting config (both sites, via targets)
├── .firebaserc         # Firebase project + hosting-target mapping
├── package.json        # Convenience scripts
└── README.md
```

## Deployment

Hosted on **Firebase Hosting** (project `ghlogistics-driver-hub`). Serving the
app and Google sign-in from the same Firebase origin is what keeps the sign-in
flow working across Safari, Chrome, and Firefox.

**To publish a change:** commit and push to `main` — a GitHub Action
(`.github/workflows/firebase-hosting-merge.yml`) deploys to Firebase Hosting
automatically, just like the old Netlify flow.

```bash
git add .
git commit -m "Describe your change"
git push
```

The action requires a one-time secret named `FIREBASE_SERVICE_ACCOUNT_GHLOGISTICS`
(setup instructions are in the workflow file). To deploy by hand instead:

```bash
firebase deploy --only hosting   # or: npm run deploy:firebase
```

Every change is versioned here, so any deploy can be rolled back from the
Firebase console (Hosting → Release history) or with `git revert`.

- **Publish directory:** `public`
- **Build command:** none (static site)
- **Canonical URL:** https://ghlogistics-driver-hub.firebaseapp.com

### Legacy Netlify config

`netlify.toml` is kept only for reference/backup. The team should use the
`.firebaseapp.com` URL; the Netlify address is being retired.

## Local preview

Because it is a static file, you can open `public/index.html` directly in a browser,
or serve the folder:

```bash
npx serve public
```

## Notes

- **Data & auth:** the app talks to Firebase from the browser; no server code lives
  in this repo.
- **Single-file design:** all markup, styles, and logic are in `public/index.html`
  for portability — edit that one file to change the app.
