# Kumo Index Test

A static test site for observing how Google discovers, crawls, and indexes pages.
Served at `https://pages.kumo.my.id`.

## Files

```
index.html      Home page
about.html      Second page (internal linking target)
404.html        Not-found page, carries noindex
styles.css      Single stylesheet, no JS, no external fonts
robots.txt      Allows crawl, points to sitemap
sitemap.xml     Both public URLs with absolute domain
```

No build step, no dependencies, no JavaScript. Deploy the directory as-is.

## Deploy to Cloudflare Pages

Direct upload (fastest for a test site):

```sh
npx wrangler pages deploy . --project-name kumo-index-test
```

Or connect the directory as a Git repo in the Cloudflare dashboard and use:

- **Build command:** *(leave empty)*
- **Build output directory:** `/`

Both routes produce the same static output. Cloudflare Pages serves `404.html`
automatically for unknown paths, so no routing config is required.

## Attach the custom domain

`pages.kumo.my.id` is a subdomain, so it needs a CNAME.

1. Dashboard → Workers & Pages → your project → **Custom domains** → **Set up a domain**.
2. Enter `pages.kumo.my.id`.
3. If `kumo.my.id` is already on Cloudflare DNS, the CNAME is created for you.
   Otherwise create it at your DNS provider:
   `pages.kumo.my.id  CNAME  <project-name>.pages.dev`
4. Wait for the certificate to show **Active** before testing indexing.

Do not test indexing while the site is only reachable at `<project>.pages.dev`.
`pages.dev` subdomains are shared infrastructure and index slowly and unreliably.

## Verify ownership in Search Console

Use a **Domain property** for `kumo.my.id`, not a URL-prefix property for the
subdomain. A DNS-verified Domain property covers every subdomain at once,
including `pages.kumo.my.id`, so you never have to re-verify if you add more.

1. Search Console → **Add property** → **Domain** → `kumo.my.id`.
2. Copy the `google-site-verification=...` TXT value.
3. Add it as a TXT record on `kumo.my.id` at your DNS provider.
4. Click **Verify**.

## Submit the sitemap

Search Console → **Sitemaps** → enter `sitemap.xml` → **Submit**.

Then confirm the crawler can actually fetch the supporting files by opening
these in a browser. All three must return 200 with the expected content:

- `https://pages.kumo.my.id/robots.txt`
- `https://pages.kumo.my.id/sitemap.xml`
- `https://pages.kumo.my.id/about.html`

## Check whether a page is indexed

Search Console → **URL Inspection** → paste the exact URL.

- **URL is on Google** — indexed.
- **URL is not on Google** — often normal for a new host. Check the sub-reason:
  - *Discovered — currently not indexed*: queued but not fetched yet.
  - *Crawled — currently not indexed*: fetched, judged not worth indexing yet.
- **Excluded by noindex tag** — a directive is blocking it; fix and re-crawl.

Use **Request Indexing** only after a real content or metadata change. It is
rate-limited and does not guarantee indexing.

## Expectations worth setting

Indexing a new subdomain is not immediate. Twenty-four hours is optimistic;
several days is typical, and a site this small may sit in "Crawled — currently
not indexed" for a while because it carries little signal. That is a property of
the content, not a bug in the markup. The files here remove the technical causes
of non-indexing (blocked crawl, broken sitemap, noindex, canonical pointing at
the wrong host, JS-only content); they cannot manufacture relevance.

Do not use "search for my own site" as the check. Rankings fluctuate and
results are personalised, so absence from a self-search proves nothing.

## Changing the domain

If the domain changes, update it in all of:

- `index.html` — canonical, `og:url`, both `@id`/`url` values in JSON-LD
- `about.html` — canonical, `og:url`, JSON-LD values
- `robots.txt` — `Sitemap:` line
- `sitemap.xml` — both `<loc>` entries

Then update `lastmod` in `sitemap.xml` and resubmit.
