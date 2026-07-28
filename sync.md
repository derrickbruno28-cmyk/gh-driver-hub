# Sync Guide — Master Dispatch / GH Driver Hub

This folder is shared between two machines (**home desktop** and **work desktop**) through
GitHub. GitHub is the shared "locker" in the middle. Each machine has its own copy of this
folder; `pull` and `push` are how each copy stays in sync with GitHub.

- **Repo:** https://github.com/derrickbruno28-cmyk/gh-driver-hub.git
- **Work folder:** `C:\Users\Bruno\Driver Management`
- **Home folder:** (your home copy of the same repo)

---

## The one rule that prevents all problems

> **Pull when you sit down. Push before you stand up.**

- **Sit down at a machine →** get the latest before editing anything.
- **Done working →** save your changes up to GitHub before you leave.

Do this on **both** machines and home and work never clash.

---

## What to say to Claude

You don't run any commands yourself — just tell Claude:

| You say... | Claude does... |
|------------|----------------|
| **"sync"** | `git pull` — grabs the latest from GitHub, then confirms it's done |
| **"save"** or **"push my changes"** | `git add -A` + `git commit` + `git push` — sends your work up, then confirms |
| **"status"** | Shows whether you have unsaved edits or are behind GitHub |

Claude reports back in the chat when each step finishes.

---

## The daily loop

**Arriving (either machine):**
1. Say **"sync"** → Claude pulls, confirms "up to date."
2. Make your edits.

**Leaving (either machine):**
3. Say **"save"** → Claude commits + pushes, confirms "pushed, safe to leave."

That's it. The two machines take turns, and each one catches up before it does anything.

---

## If you're ever unsure

- **Forgot whether you pushed from the other machine?** Just say **"sync"** first anyway.
  Pulling is always safe — it never destroys your work. Only *editing before pulling* causes trouble.
- **Merge conflict?** This only happens if both machines were edited without pulling in between.
  It is not dangerous — just tell Claude "resolve the conflict" and it'll walk you through it.

---

## The manual commands (reference only — Claude handles these for you)

```bash
git pull                          # sync down
git add -A                        # stage everything
git commit -m "describe changes"  # save a checkpoint
git push                          # send up to GitHub
git status                        # see where things stand
```
