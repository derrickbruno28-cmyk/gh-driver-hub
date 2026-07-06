# GH Logistics Driver Hub

A fleet and driver management web app for GH Logistics — driver roster, availability,
team pairings, recruitment pipeline, DQF/insurance tracking, safety violations, and
company pay/policy reference.

**Live site:** https://gh-driver-hub.netlify.app

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

```
.
├── public/
│   └── index.html      # The entire app (HTML/CSS/JS in one file)
├── netlify.toml        # Netlify build/redirect config
├── firebase.json       # Firebase Hosting config (alternate host)
├── .firebaserc         # Firebase project reference
├── package.json        # Convenience scripts
└── README.md
```

## Deployment

Hosted on **Netlify** with continuous deployment from this repo.

**To publish a change:** commit and push to `main` — Netlify builds and deploys
automatically.

```bash
git add .
git commit -m "Describe your change"
git push
```

The live site updates within a minute. Every change is versioned here, so any
deploy can be rolled back from the Netlify dashboard or with `git revert`.

- **Publish directory:** `public`
- **Build command:** none (static site)
- **Branch:** `main`

### Manual deploy (backup)

If you ever need to deploy outside of the git flow:

```bash
netlify deploy --prod
```

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
