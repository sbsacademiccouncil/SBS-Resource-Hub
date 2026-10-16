# Class Hub: how the code works and how to change it

## What's in the folder
| File | Purpose |
|---|---|
| `index.html` | The entire app: styles, markup, and JavaScript in one file. |
| `firestore.rules` | Who can read/write each Firestore collection. **Merged with the Opportunities site's rules.** One ruleset covers the whole database. |
| `storage.rules` | Who can upload/delete PDFs. |
| `README.md` | First-time Firebase and GitHub Pages setup. |

There is no build step. Edit `index.html`, push it to GitHub, and Pages serves it.

## How `index.html` is organized
Top to bottom:
1. **`<style>`** starts with the shared theme tokens in `:root` (colors, spacing), then component styles.
2. **`<header>`, `.layout` (sidebar + `<main>`), `<footer>`, `<dialog>`**: the fixed page shell. Only `<main>` and the sidebar scroll, so the page itself never does.
3. **`<script type="module">`**, in this order:
   - Firebase config and imports (loaded from Google's CDN).
   - **State variables:** `user`, `admin` (effective, false in student view), `realAdmin`, `asStudent`, `classes`, `allRes`, `cur` (resources currently on screen), `filt`, `catSel`.
   - **Auth:** `onAuthStateChanged` checks the `@sbs.org` domain and looks the user up in `admins`. `renderAuth()` draws the sign-in area and the "View as student" button.
   - **Sidebar:** `loadClasses()`, `visibleClasses()` (applies search + filters), `renderSide()`.
   - **Routing:** `route()` reads `location.hash` (`#/class/ID`, `#/search/TEXT`, `#/stats`, `#/admin`) and calls the matching view. No server config needed.
   - **Views:** `viewClass`, `viewSearch`, `viewStats`, `viewAdmin`, each builds HTML and puts it in `<main>` via `main()`.
   - **Resources:** `loadRes()`, `card()` (one resource's HTML), `feed()` (category chips + cards).
   - **Actions:** one delegated click handler reads `data-act` attributes (`like`, `comments`, `report`, `edit`, `del`, `approve`, `dismiss`, ...).
   - **Forms (modals):** `addForm`, `editForm`, `classForm`, `suggestForm`, all via `modal()`.
   - **Helpers:** `helpHtml()`, `catOpts()`, `renderComments()`, `toast()`, `esc()` (always use it when putting user text into HTML).

## Data model (Firestore)
- `classes/{id}`: `classCode`, `className`, `teacher`, `discipline`, `semesters[]`, `blocks[]`
- `resources/{id}`: `classId`, `title`, `type` (`pdf`|`link`), `url`, `storagePath`, `category`, `uploadedBy` (email), `createdAt`, `isApproved`, `reportsCount`, `likedBy[]` (user IDs)
  - `resources/{id}/comments/{id}`: `text`, `author`, `createdAt`
- `reports/{resourceId_userId}`: one per student per resource
- `suggestions/{id}`: `text`, `from`, `createdAt`
- `settings/global`: `moderation` (boolean)
- `admins/{email}`: empty doc; existence = admin (shared with the Opportunities site)

## Who can do what
Enforced by `firestore.rules`, not by the page. Hiding a button is cosmetic; the rules are the real lock.
- **Anyone:** read classes, approved resources, comments, stats.
- **Signed-in `@sbs.org`:** add resources (starts approved unless moderation is on), edit their own title/category/link, like, comment, report once, send suggestions.
- **Admins:** everything else: classes, approvals, deleting resources/comments, reading reports and suggestions, the moderation switch.

## Look and feel
All colors live in `:root` at the top of `<style>`. The same token block sits in the Opportunities site; **change both together** so the sites stay matched. `--accent` is the main navy blue, `--deep` the darkest navy (header), `--sky` and `--tint` the soft blue highlights, `--red` for destructive actions. In the Opportunities site the same colors sit under older names (`--green` is the main blue, `--green-dark` the navy, `--ochre*` the light blues). Fonts are Source Serif 4 (headings) and IBM Plex Sans (everything else), loaded by the `<link>` in `<head>`.

## Common changes
**Change colors:** edit the `:root` values (both sites).

**Change the allowed email domain:** replace `sbs.org` in four places: `isSbs()` regex and `setCustomParameters({hd:...})` in `index.html`, and the `.matches('.*@sbs[.]org')` lines in `firestore.rules` and `storage.rules`. Republish both rules.

**Change default category suggestions:** edit the array inside `catOpts()`. Students can still type any new category.

**Change the PDF size limit:** change `MAX_MB` in `index.html` **and** `15 * 1024 * 1024` in `storage.rules`. Change them together or uploads fail.

**Turn off PDF uploads (links only):** remove the "Upload a PDF instead" radio and `#pdfbox` from `addForm`, and the `pdf` branch in its submit handler. You can then delete `storage.rules` and Storage.

**Edit the instructions text:** edit `helpHtml()`. It feeds both the home page panel and the Help button.

**Add a field to resources (example: `unit`):**
1. Add an input in `addForm` and `editForm`, and include it in the `addDoc` / `updateDoc` call.
2. Show it in `card()`.
3. In `firestore.rules`, add the key to the create `hasOnly([...])` list and to the owner-edit `hasOnly` lists. Without this, saves are rejected with "permission-denied".

**Add a class field:** add an input in `classForm`, the `data` object in its submit handler, and show it in `viewClass`.

**Add another page:** add a `v==='yourpage'` line in `route()`, write a `viewYourPage()` that calls `main('<h1>...</h1>')`, and link to `#/yourpage`.

## Troubleshooting
- **"Missing or insufficient permissions":** usually a rules mismatch (see "Add a field"), a missing `settings/global` document, or the rules not being republished.
- **Sign-in popup fails:** add your site's domain under Authentication → Settings → Authorized domains.
- **Admin buttons missing:** the `admins` document ID must exactly match your email, and you must not be in student view.
- **Upload fails but links work:** Storage rules not published, or the file isn't a real PDF / is over the limit.

## Known limits
- Comment counts and like counts are read when a page loads; other people's new activity appears after a refresh.
- Statistics and search load all approved resources at once, which is fine for a school, but would need paging at thousands of items.
- Unapproved PDFs are hidden from the feed but their direct URLs are technically public.
