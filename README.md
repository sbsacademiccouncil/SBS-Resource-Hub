# SBS Class Hub: setup

## 1. Firebase project
1. Create a project at console.firebase.google.com.
2. **Authentication → Sign-in method:** enable **Google**.
3. **Authentication → Settings → Authorized domains:** add `YOUR-USERNAME.github.io`.
4. **Firestore Database:** create it (production mode). Then **Rules** tab → paste `firestore.rules` → Publish.
5. **Storage:** create it (needs the Blaze plan; free usage tier still applies). **Rules** tab → paste `storage.rules` → Publish.
6. **Project settings → Your apps → Web (</>):** register an app and copy the `firebaseConfig` into `index.html`.

## 2. Seed the data (Firestore console)
- Collection `settings`, document ID `global`: field `moderation` (boolean) = `false`. **Required**, since the rules read it on every upload.
- Collection `admins`: one document per admin, **document ID = their exact email** (e.g. `teacher@sbs.org`). No fields needed.
- Sign in once, then use **+ New class** in the sidebar to create classes (admins only).

## 3. Deploy to GitHub Pages
Push `index.html` to a repo, then **Settings → Pages → Deploy from branch → main / root**.
Routing uses `#/…` URLs, so no 404 workarounds are needed.

## Notes
- The `@sbs.org` limit is enforced in the security rules (not just in the UI), and requires a verified email.
- Anyone can read the site and open PDFs without signing in. PDFs awaiting approval are hidden from the feed but their direct URL is technically public.
- Reports: one per student per resource. "Dismiss reports" clears them so people can report again.
- The Firebase `apiKey` in `index.html` is not a secret; security comes from the rules.
