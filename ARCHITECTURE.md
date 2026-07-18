# Architecture Note

## What I prioritized

Given the timebox, I optimized for one coherent, correctly-working slice rather than partial
coverage of every possible feature:

1. **A genuinely usable editor**, not a `<textarea>`. TipTap gives real rich-text (bold, italic,
   underline, headings, lists) with a proper document model, which matters because...
2. **...file import and manual editing had to produce the same data shape.** Both a freshly
   created document and one imported from a `.txt`/`.md` file store the same TipTap-compatible
   HTML in `Document.content`. There's one content format throughout, not a separate "imported
   file" representation that has to be reconciled with hand-edited documents later.
3. **Permission logic lives in one place** (`lib/permissions.js`), as pure functions with no
   database or request dependency. Every API route that touches a document - read, rename, edit
   content, delete, share - calls into the same `canView`/`canEdit`/`isOwner` functions. This
   was the single highest-leverage piece of code to get right and keep simple, since access-control
   bugs are the worst kind to ship quietly. It's also the piece I unit-tested most thoroughly.
4. **Ship something that runs with zero external accounts.** SQLite + mocked auth means a
   reviewer can `npm install && npm run setup && npm run dev` and be looking at working sharing
   behavior in under a minute, with no signup flow, API keys, or cloud database to provision.

## Data model

```
User(id, name, email)
Document(id, title, content, ownerId -> User, createdAt, updatedAt)
DocumentShare(id, documentId -> Document, userId -> User, permission: "view" | "edit")
```

`DocumentShare` is a join table rather than an array column so that permission (view vs. edit)
is per-share, and so revoking access is a single row delete. A document's owner is a direct
foreign key rather than a share with an "owner" permission, because ownership (can delete, can
manage sharing) is a materially different capability from edit access and deserves its own check
(`isOwner`) rather than being one more string in a permission enum.

## Request flow for a document open

`app/doc/[id]/page.js` (server component) reads the current user from a cookie, loads the
document with its shares via Prisma, and runs `canView` before rendering anything. If access is
denied, the page renders a plain "you don't have access" state server-side - the client editor
component never receives content it isn't allowed to see. This was a deliberate choice over
fetching the document client-side and checking permissions after the fact, which would briefly
put the unauthorized content in a client-side fetch response even if the UI never painted it.

## Autosave over explicit "Save"

Both title and content save automatically ~700ms after the user stops typing (debounced), rather
than requiring a save button. This matches the mental model of "a Google Doc" the assignment is
explicitly inspired by, and avoids a whole class of "I edited and lost my changes" bugs. The
tradeoff: no "unsaved changes" warning on navigation, and concurrent edits from two people are
last-write-wins rather than merged. A real-time layer (see below) would need to replace this
entirely rather than build on top of it.

## What I deliberately did not build

- **True real-time collaboration** (cursors, live merge). This is the single most complex thing
  on the optional stretch list (CRDTs/OT, presence, websockets) and would have consumed the
  entire timebox on its own at the expense of the core flow working well. Autosave + last-write-
  wins was the right scope for a "lightweight" editor.
- **Real auth.** Sessions/passwords/OAuth are a well-understood, mostly-boilerplate problem that
  wouldn't have demonstrated anything about document editing, file handling, or sharing logic -
  the things this assignment is actually testing. Mocked auth (seeded users, cookie session) let
  me spend that time on the sharing model and editor instead, while keeping the *shape* of real
  auth (a `getCurrentUser()` call every route relies on) so swapping in real auth later is a
  localized change.
- **`.docx`/`.pdf` import.** Parsing real Word/PDF documents into a rich-text model correctly
  (tables, images, styles) is a project on its own. `.txt`/`.md` import demonstrates the same
  end-to-end flow - upload, parse, land as a new editable document - without that complexity.

## What I'd build next with another 2-4 hours

1. **Attachments** (upload a file *onto* an existing document, not just import-as-new-document),
   since the assignment explicitly lists this as an alternative valid interpretation.
2. **Version history** (append-only snapshots on each save) - straightforward given the data
   model already has `updatedAt`, and it's the stretch goal with the best effort-to-value ratio.
3. **Optimistic concurrency warning**: if a document's `updatedAt` changed since it was loaded,
   warn before overwriting on save, instead of silent last-write-wins.
4. **Export to Markdown/PDF**, reusing the same HTML content already stored per document.
