# Submission

*Fill in the bracketed items before sending. This file exists so a reviewer can see at a glance
what's included and what's still outstanding.*

## What's included in this folder

- [x] Source code (this repository)
- [x] `README.md` - local setup and run instructions
- [x] `ARCHITECTURE.md` - what was prioritized and why
- [x] `AI_WORKFLOW.md` - AI tool usage, what was changed/rejected, verification (needs your
      personalization once you've run and tested the app - see the callout in that file)
- [x] `SUBMISSION.md` - this file
- [ ] Live product URL: `[ADD DEPLOYED URL HERE]`
- [ ] Walkthrough video URL (3-5 min): `[ADD LOOM/YOUTUBE LINK HERE]`
- [ ] Screenshots / demo GIF: `[ADD IF SETUP NEEDS EXTRA STEPS]`

## Test accounts (seeded, no password)

| Name | Email |
|---|---|
| Alice Chen | alice@ajaia.dev |
| Bob Farrow | bob@ajaia.dev |
| Carol Nunez | carol@ajaia.dev |

Alice owns a sample document already shared with Bob (edit access) - useful for demoing the
sharing flow immediately without setting it up manually. Log in as either from the app's login
screen (click their name/email card, no password needed).

## Status

**Working end to end:**
- Document create, rename, rich-text edit (bold/italic/underline/H1-H3/bulleted+numbered lists),
  autosave, persistence across refresh
- `.txt`/`.md` file upload → new editable document
- Sharing: owner grants view/edit access by email; dashboard separates owned vs. shared;
  non-shared users are denied access
- Automated tests for permission logic and file-import logic (`npm test`)

**Incomplete / not attempted:**
- Real-time collaboration (see `ARCHITECTURE.md` for why this was cut)
- `.docx`/`.pdf` import
- Version history

**What I'd build next with another 2-4 hours:** see the closing section of `ARCHITECTURE.md`.

## Local verification

```bash
npm run setup   # install, generate prisma client, create db, seed demo users
npm run dev     # http://localhost:3000
npm test        # runs the automated test suite
```
