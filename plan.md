# SEO & Search Visibility Plan — charlie.onl

Site purpose: personal portfolio for Charlie Hill, a UK-based full-stack developer. Pages: home (`index.html`), `about.html`, `projects.html`, `books.html`, `tools/`, plus external `blog.charlie.onl`. Static HTML/CSS/JS, Tailwind + Alpine.js, hosted on Vercel with `cleanUrls: true`.

Goal: improve how well this site is found and represented in search, without changing the visual design, tone, or wording of existing content in a way that would feel scripted or "AI-generated." Every change below is technical/structural — metadata, markup, and crawlability — not a content rewrite.

Do this work on the `seo-improvements` branch (already created). Commit in small, reviewable steps per section below so each can be checked before moving on.

---

## 1. Fix crawlability of core content (highest impact)

`projects.html` and `books.html` render their entire content client-side via Alpine.js fetching `data/*.yaml`. A crawler that doesn't execute JS (or executes it with a budget) sees an empty shell — the actual project/book listings are invisible. This is the single biggest gap since "projects" is presumably the most important page on a dev portfolio.

- [ ] Add static fallback content in the HTML itself: render the project cards (title, description, links) directly in `projects.html` as plain HTML, and have Alpine.js progressively enhance/hydrate over it (or simply leave the static markup and drop the fetch-based re-render if it's not adding real value). Same for `books.html`.
- [ ] Keep the YAML files as the single source of truth if desired, but generate the static HTML from them at build/deploy time rather than at request time in the browser (a tiny script during your existing deploy step, or even just hand-maintain both since the list is short).
- [ ] Verify with "view source" (not devtools) that project titles/descriptions/links are present without JS.

## 2. Add missing meta descriptions

Only `index.html` has a `meta description`. `about.html`, `projects.html`, and `books.html` have none, so Google will auto-generate a snippet from page text, which is unpredictable.

- [ ] Write a one-sentence, natural meta description for each page reflecting what's actually there (e.g. about.html: who you are and what you do; projects.html: the kind of projects listed; books.html: what the reading list is). Keep them factual, ~120–155 characters, in your own voice — no keyword stuffing.

## 3. Canonical tags

No page has a `<link rel="canonical">`. Since `vercel.json` has `cleanUrls: true`, both `/about` and `/about.html` likely resolve — without a canonical tag this risks duplicate-content dilution.

- [ ] Add `<link rel="canonical" href="https://charlie.onl/about">` (extensionless, matching the clean URL) to each page, pointing at itself.

## 4. Open Graph & Twitter Card tags

No page has OG or Twitter meta tags, so links shared on LinkedIn, Twitter/X, Slack, iMessage etc. render with no preview image/title/description.

- [ ] Add `og:title`, `og:description`, `og:type`, `og:url`, `og:image` and `twitter:card`, `twitter:title`, `twitter:description`, `twitter:image` to each page.
- [ ] Use a proper local image (not the hotlinked GitHub avatar) for `og:image` — e.g. a simple 1200×630 banner with your name, or reuse the existing portrait at proper social-card size. This can live in `assets/`.

## 5. Structured data (JSON-LD)

No page has schema.org markup, so Google has no explicit signal for "this is a Person" / "this is a portfolio."

- [ ] Add a `Person` JSON-LD block to `index.html` (and optionally `about.html`) with `name`, `url`, `jobTitle`, `sameAs` (GitHub, LinkedIn, Instagram, YouTube — the same links already on the page).
- [ ] Optionally add `CreativeWork`/`SoftwareSourceCode` entries for a couple of the more notable projects if it's not overkill for a personal site — low priority, skip if it feels forced.

## 6. robots.txt

There is no `robots.txt` at the site root.

- [ ] Add a minimal `robots.txt` allowing all crawlers and pointing to the sitemap:
  ```
  User-agent: *
  Allow: /

  Sitemap: https://charlie.onl/sitemap.xml
  ```

## 7. Sitemap fixes

`sitemap.xml` is missing `books.html`/`tools/`'s siblings (tools is present, books is not), and every `lastmod` is hardcoded to `2025-01-01` regardless of actual changes.

- [ ] Add the missing `books.html` (`/books`) entry.
- [ ] Update `lastmod` values to reflect real last-edit dates going forward (or drop `lastmod` entirely if you don't want to maintain it by hand — a missing tag is better than a permanently stale one).
- [ ] Re-check the sitemap after each future content change is a habit worth keeping, not a one-off fix.

## 8. Fix the orphaned books page

`books.html` isn't linked from `index.html` (the link is commented out) or from `projects.html`'s nav, so it's only reachable by a direct URL — bad for internal link equity and discovery.

- [ ] Either re-enable the "Reading List" link in the nav (index.html and the shared page nav) if the page is meant to be public, or intentionally leave it unlisted and drop it from the sitemap too — pick one, don't leave it in the current half-state.

## 9. Heading & internal-linking consistency

- [ ] `about.html`/`projects.html`/`books.html`/tools' `h1`s are styled as small nav-bar text (`text-lg`) rather than a visually distinct page heading — this is a design choice, not strictly an SEO bug, but consider giving each page a slightly more prominent, descriptive `h1` (e.g. "Charlie Hill — Projects" instead of just "Projects") to reinforce topical relevance without changing the overall look much.
- [ ] Fix the `site.webmanifest` path inconsistency: `index.html` links `./assets/site.webmanifest`, other pages link `./site.webmanifest` — confirm which file actually exists at deploy (root `site.webmanifest`) and make all pages point at the same, correct path.

## 10. Performance clean-up (indirect ranking signal)

Page experience/Core Web Vitals are a minor but real ranking factor, and this site currently loads unminified Tailwind, Meyer reset, and animate.css from three different CDNs render-blocking in `<head>`, plus a hotlinked avatar image from GitHub.

- [ ] Host the (already-tiny) portrait image locally in `assets/` instead of hotlinking `avatars.githubusercontent.com` — removes a third-party dependency and DNS lookup.
- [ ] Consider swapping the full unpkg Tailwind build for the actual small subset of utility classes used (or a purged build) — not urgent, but worth a look given this is a lightweight static site.
- [ ] Add `loading="lazy"` to below-the-fold images (project thumbnails).

## 11. Verify

- [ ] After changes, validate: Google Rich Results Test (structured data), a plain `curl`/"view source" pass on `projects.html`/`books.html` to confirm content is present without JS, and Search Console (if not already set up, register the site and submit the updated sitemap).

---

### Explicitly out of scope
No rewording of bio copy, project descriptions, or page tone. No new content, no keyword stuffing in visible text, no restructuring the visual design. Everything above is markup/metadata/build-process level so the site still reads and feels exactly like it does today.
