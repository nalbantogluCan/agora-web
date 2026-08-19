# Agora support site

The public site behind the two URLs App Store Connect requires: the **Support
URL** and the **Privacy Policy URL**. Plain HTML and CSS, no build step, no
dependencies. Editing a file and pushing it is the whole deployment process.

```
docs/
├── index.html           Support page: contact details + FAQ        (EN)
├── privacy.html         Privacy policy (GDPR)                      (EN)
├── delete-account.html  Account deletion instructions              (EN)
├── tr/
│   ├── index.html       Destek: iletişim + SSS                     (TR)
│   ├── privacy.html     Gizlilik Politikası (GDPR)                 (TR)
│   └── delete-account.html  Hesap silme yönergeleri                (TR)
├── style.css            The only stylesheet, shared by all six pages
├── .nojekyll            Serve the files as they are, without Jekyll
└── README.md            This file — not published
```

English is the default and lives at the root; Turkish lives under `/tr/`. Every
page carries a language switch in its header, and the two languages use
**identical anchor ids**, so any deep link maps across by adding or removing
`tr/`: `…/#safe-meetups` ⇄ `…/tr/#safe-meetups`.

---

## Deploying to GitHub Pages

The site is written to be served from the `docs/` folder of the
`Agora-Exchange/agora-mobile` repository.

1. Commit and push these files to the default branch.

   ```sh
   git add docs
   git commit -m "Add support and privacy site"
   git push
   ```

2. On GitHub, open the repository → **Settings** → **Pages**.

3. Under **Build and deployment**:
   - **Source:** Deploy from a branch
   - **Branch:** `master` (or `main`, whichever the repo's default is) and
     folder **`/docs`**
   - Click **Save**.

4. Wait a minute or two. The first build is the slow one; later pushes are
   usually live within about 30 seconds. Progress shows under the repo's
   **Actions** tab.

5. The site is then at:

   ```
   https://agora-exchange.github.io/agora-mobile/
   https://agora-exchange.github.io/agora-mobile/privacy.html
   https://agora-exchange.github.io/agora-mobile/delete-account.html
   https://agora-exchange.github.io/agora-mobile/tr/
   https://agora-exchange.github.io/agora-mobile/tr/privacy.html
   https://agora-exchange.github.io/agora-mobile/tr/delete-account.html
   ```

   Open all six on a phone before you rely on them, and check that the
   language switch in the header moves between them.

### Preview locally first

```sh
cd docs
python3 -m http.server 8000
# then open http://localhost:8000
```

Opening the files directly with `file://` works too, but the local server is a
closer match to how Pages serves them.

### The repository must be public

GitHub Pages on a private repository requires a paid plan, and Apple's reviewer
has to be able to open the URLs without signing in. If `agora-mobile` needs to
stay private, put these four files in a small public repo of their own
(`agora-support`, served from its root) and set Pages up there instead — the
links between the pages are all relative, so nothing needs changing.

### A custom domain, later

If you ever point a domain at this, add it under Settings → Pages → Custom
domain, tick **Enforce HTTPS**, and then update the URLs in App Store Connect.
Apple checks that the URLs work at review time, so change the DNS first and the
listing second.

---

## Where these URLs go in App Store Connect

| Field | Where | Value |
| --- | --- | --- |
| Support URL | App Store → your app → the version → **App Information** | `https://agora-exchange.github.io/agora-mobile/` |
| Privacy Policy URL | App Store → **App Privacy** | `https://agora-exchange.github.io/agora-mobile/privacy.html` |
| Account deletion URL | App Store → **App Privacy** → Account deletion | `https://agora-exchange.github.io/agora-mobile/delete-account.html` |

Give App Store Connect the **English** URLs. Apple takes one URL per field, and
a Turkish reader reaches their version through the language switch in the page
header. (App Store Connect does allow per-localisation URLs if you add a Turkish
localisation for the listing — in that case point the Turkish one at `…/tr/`.)

Guideline 5.1.1(v) is satisfied by the in-app deletion flow; the deletion page
is there to answer the reviewer's question quickly and to serve people who have
already uninstalled.

---

## Placeholders and things still to check

Nothing on the site is left as a literal `[placeholder]` — every value is
filled in with what the repository actually says. What follows is the list of
values that were **assumed, guessed, or read out of the code** and that you
should confirm. Each one is also marked in the HTML with a
`<!-- REVIEW THIS -->` comment, so:

```sh
grep -rn "REVIEW THIS" docs/
```

will always give you the current list.

### Must confirm before submitting

- **Legal name.** The site says `Can Nalbantoğlu` in the copyright line, in the
  GDPR controller section. It must match the name on
  the Apple Developer account. If that account is a company, replace the name
  everywhere and adjust "an individual developer" in `privacy.html`.
- **Support email.** `agora.exchange14@gmail.com`, taken from
  `src/lib/support.ts`. Make sure someone reads it — Apple sometimes writes to
  the support address, and the site promises a reply within 2 business days.
- **Hosting regions.** `privacy.html` describes Supabase and Railway generally
  because the regions were not determinable from the repo. Check both
  dashboards; if either is in the EU, say so explicitly.
- **Is the Railway API still live?** `privacy.html` names
  `agora-backend-production-7d3a.up.railway.app` as a processor, because
  `src/lib/orval-data.ts` routes profile, book, and request traffic through it.
  If that service is gone, delete the "Railway — the Agora API service"
  section.
- **Retention windows.** 90 days for infrastructure logs and 12 months for
  moderation records appear in both `privacy.html` and `delete-account.html`.
  They are conventional figures, not measurements. Change them to what you
  actually intend, and change them in both files.

### Facts stated in more than one place

These are the sentences that exist in both the app and this site. They agreed
as of the last edit; a change to any one of them has to be made everywhere it
appears, or a reviewer reading both will find a contradiction.

- **Minimum age — 16+**, the rating Apple assigned after the App Store Connect
  questionnaire. Four places: `privacy.html` and `tr/privacy.html` (Children),
  the in-app Terms (`src/app/settings/terms.tsx`, Eligibility) and the in-app
  Privacy Policy (`src/app/settings/privacy-policy.tsx`, Children's Privacy).
- **Location precision.** Both policies now say a place label *and* its
  coordinates are stored, which is what `sql/location-geo.sql` and
  `src/lib/distance.ts` actually do. Do not let either drift back to claiming
  coordinates are not kept.
- **Contact address — `agora.exchange14@gmail.com`**, and only that one. It
  lives in `src/lib/support.ts` for the app and is hard-coded on all six pages
  here. An older `support@agora.app` appears nowhere any more except in two
  comments describing the history.
- **Report triage within 24 hours**, promised in the in-app Terms and on the
  support page. That one is a commitment about your behaviour rather than the
  code's, and it is the easiest of these to quietly stop being true.

### Housekeeping

- **Dates.** Three files carry `Last updated: 19 August 2026`, and
  `privacy.html` also carries `Effective 19 August 2026`. Update them whenever
  you change the text — a policy whose effective date predates a change is
  worse than one with no date at all.
- **Policy version.** `LEGAL_VERSION` in `src/lib/legal.ts` is currently
  `2026-02-01` and stamps what each user agreed to. If you publish a materially
  different policy, bump it so acceptances can be told apart.
- **Keep the two languages in step.** Every change to an English page needs the
  same change in its `tr/` counterpart, and vice versa. Two things make drift
  visible: the anchor ids are identical across languages, and the `REVIEW THIS`
  comments in `tr/` are written in English and say `(mirrors …)` so one grep
  still finds the whole list. The figures that appear in more than one place —
  the 90-day and 12-month retention windows, the effective date, the 2-business-day
  reply — are the ones worth re-checking after an edit.
- **There is no KVKK section any more.** It was removed on 2026-08-19 at the
  owner's request. KVKK still applies to processing carried out from Türkiye —
  removing the text changed the disclosure, not the obligation — so if you ever
  reinstate it, `git log -- privacy.html` has the original wording in both
  languages.

---

## Editing notes

- **One stylesheet, no external requests.** No CDN, no web fonts, no
  frameworks. The type is a system serif stack, so the page loads instantly and
  works offline. Keep it that way — an external font or script is the sort of
  thing that fails behind a corporate proxy during review.
- **The page works without JavaScript.** The only script in the site reopens a
  collapsed FAQ answer when someone follows a direct link to it. Every answer
  is open by default, so with JavaScript off the page is complete.
- **Both colour schemes.** Light and dark are defined as custom properties at
  the top of `style.css` and switch on `prefers-color-scheme`. Change a colour
  in one place, and check the other scheme before pushing.
- **FAQ anchors are permanent links.** Each answer has an id, so you can send
  someone straight to it — for example
  `…/agora-mobile/#safe-meetups` or `…/agora-mobile/#book-not-found`. Don't
  rename an id once you have sent it to anyone; add a new answer instead.
- **The language switch is plain links**, one per page, pointing at that page's
  counterpart — no JavaScript, no cookie, no automatic redirect based on browser
  language. A reviewer landing on the English page stays on it.
- **The content width is capped around 65 characters** and the line height is
  generous, because these are pages people read rather than scan. Resist
  widening it.
