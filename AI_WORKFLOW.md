# AI Workflow Note

*Draft written by Claude (Anthropic) during the build session. Personalize the "verification"
section below before submitting - see the callout at the bottom.*

## Which AI tools I used

Claude (Sonnet), used agentically inside a sandboxed dev environment with file read/write and
shell access, for the entire initial build: project scaffolding, Prisma schema, all API routes,
React components, CSS, tests, and this documentation.

## Where AI materially sped up the work

- **Boilerplate elimination.** CRUD API routes (documents, sharing), Next.js App Router
  conventions, and Prisma schema syntax are exactly the kind of well-documented, pattern-heavy
  code AI generates quickly and reliably, freeing time for the parts that needed actual product
  judgment (what the permission model should be, what to cut).
- **Design-token-first CSS.** Generating a full custom design system (palette, type scale,
  component classes) by hand is normally a big time sink for a full-stack-focused build; having
  it produced in one pass and then reviewed/adjusted was much faster than iterating from scratch.
- **Parallel surface area.** Writing the API layer and the matching frontend components together,
  keeping request/response shapes consistent across ~10 files, is exactly where an agentic tool
  helps most - a human would normally context-switch between files far more.

## What AI-generated output I changed or rejected

- **Rejected: a markdown-parsing npm dependency** (e.g. `marked`) for file import, in favor of a
  small hand-written `mdToHtml` function. This is a real tradeoff, not a free win: it only covers
  the markdown subset the editor's own toolbar supports (headings, bold/italic, lists), not full
  CommonMark. I chose it because it's zero-dependency, trivially unit-testable without a
  markdown-spec test suite, and "supports what the UI supports" is arguably more correct for
  this app than a generic parser would be.
- **Rejected: fetching document permissions client-side after render.** The first draft had the
  editor page fetch the document via `useEffect` and check access in the browser. I moved the
  permission check server-side (in the page component, before any client code runs) so
  unauthorized content is never sent to a browser that shouldn't see it - see `ARCHITECTURE.md`.
- **Changed: TipTap version pin.** Initially pinned an older TipTap 2.4 line; bumped to 2.5.x and
  added `immediatelyRender: false` explicitly, which avoids a known SSR/hydration warning under
  Next.js App Router that 2.4 doesn't handle by default.
- **Simplified: dropped an `Attachment` model** from an early schema draft (for "attach a file to
  an existing document" in addition to "import a file as a new document"). Both are valid readings
  of the assignment's file-upload requirement; implementing only one, cleanly, beats a half-built
  version of both. This is called out explicitly as a scope cut in `ARCHITECTURE.md`.

## How correctness was verified in this session

- The two pieces of pure, dependency-free business logic - `lib/permissions.js` (access control)
  and `lib/fileImport.js` (txt/md → editor HTML conversion) - were executed directly with `node`
  against hand-picked cases (owner/editor/viewer/stranger access; headings, bold/italic, and
  mixed lists in markdown; unsupported file-type rejection) and matched expected output before
  the corresponding Vitest test files were written to formalize those same cases.
- **The full Next.js application (npm install, dev server, build, actually clicking through the
  UI) was *not* run in this session** - the build sandbox has no network access, so `npm install`
  could not fetch packages. The API routes, React components, and Prisma schema are written
  carefully against known Next.js 14 / Prisma / TipTap APIs, but they have not been executed
  end-to-end.

## ⚠️ Before you submit this

Because of the constraint above, you (the candidate) need to do the verification a real AI-native
workflow requires - don't submit on trust:

1. Run `npm run setup && npm run dev` locally and click through: create a doc, format text,
   rename it, refresh and confirm it persisted, upload a `.md` file, share a doc between two
   seeded users, confirm a third user can't open it.
2. Run `npm test` and confirm both test files pass.
3. Fix anything that doesn't work as described - it's realistic that a version pin or Next.js API
   detail needs a small correction once it's actually run.
4. Rewrite this note in your own words once you've done that verification, and be specific about
   anything you had to fix - that's exactly the kind of detail this section is meant to surface.
