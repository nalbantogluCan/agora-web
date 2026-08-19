# Agora support site

The public site behind the two URLs App Store Connect requires: the **Support
URL** and the **Privacy Policy URL**. Plain HTML and CSS, no build step, no
dependencies. Editing a file and pushing it is the whole deployment process.

```
docs/
├── index.html           Support page: contact details + FAQ        (EN)
├── privacy.html         Privacy policy (GDPR + KVKK)               (EN)
├── delete-account.html  Account deletion instructions              (EN)
├── tr/
│   ├── index.html       Destek: iletişim + SSS                     (TR)
│   ├── privacy.html     Gizlilik Politikası (GDPR + KVKK)          (TR)
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
  GDPR controller section, and in the KVKK section. It must match the name on
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

### Inconsistencies with the app, worth fixing in the app

- **Minimum age.** The Terms (`src/app/settings/terms.tsx`) require users to be
  17 or older and the App Store rating is 17+, but the in-app privacy policy
  (`src/app/settings/privacy-policy.tsx`) says the app is "not directed to
  children under 13". This site says 17+. Update the in-app text so the two
  agree — a reviewer comparing them is a plausible scenario.
- **Location precision.** The in-app policy says precise GPS coordinates are
  not stored. `sql/location-geo.sql` adds `location_lat` and `location_lng` to
  the profile and `src/lib/distance.ts` computes distances from them, so
  coordinates *are* stored. This site describes it accurately; the in-app text
  needs the same correction.
- **Contact address.** The in-app Terms and Privacy Policy have historically
  disagreed (`agora.exchange14@gmail.com` vs `support@agora.app`). This site
  uses only the first. Worth a pass through the app to make sure nothing still
  points at the other one.

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
- **The Turkish KVKK text is the one that counts.** `tr/privacy.html#kvkk` is
  what a Turkish reader, or the Kurum, would actually rely on. If you get any
  legal text reviewed locally, make it that section.

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
