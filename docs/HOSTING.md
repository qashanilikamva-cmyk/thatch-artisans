# Hosting Thatch Artisans

The site is hosted on **GitHub Pages**, deployed automatically by
`.github/workflows/deploy-pages.yml` on every push to `main`.

Cost: free, including HTTPS on the custom domain.

Everything published to the public lives in `site/`. Nothing outside that folder
is served.

---

## One-time setup

### 1. Turn on GitHub Pages

This is a repository setting and can only be changed in the browser:

1. Go to **Settings → Pages** in the repository.
2. Under **Build and deployment → Source**, choose **GitHub Actions**.

That's it — the workflow does the rest. The first deploy publishes to
`https://qashanilikamva-cmyk.github.io/thatch-artisans/`.

### 2. Add the site files

Put the real site into `site/`, replacing the placeholder `site/index.html`.
The home page must be named `index.html`.

Push to `main` and the workflow deploys it.

### 3. Point the custom domain at it

The domain is **thatchartisans.co.za**, registered with GoDaddy.

**a. Tell GitHub the domain.** Already done — `site/CNAME` contains one line:

```
thatchartisans.co.za
```

That names the apex as the canonical address. Once both sets of DNS records
below are in place, GitHub redirects `www.thatchartisans.co.za` to it
automatically.

**b. Add the DNS records** at the domain's registrar. This domain is registered
with **GoDaddy** — see the GoDaddy walkthrough below for the exact clicks and
the two gotchas specific to them.

For the apex domain (`thatchartisans.co.za`), four `A` records, all with host
`@`:

| Type | Host | Value           |
| ---- | ---- | --------------- |
| A    | @    | 185.199.108.153 |
| A    | @    | 185.199.109.153 |
| A    | @    | 185.199.110.153 |
| A    | @    | 185.199.111.153 |

And, for IPv6, four `AAAA` records, also with host `@`:

| Type | Host | Value               |
| ---- | ---- | ------------------- |
| AAAA | @    | 2606:50c0:8000::153 |
| AAAA | @    | 2606:50c0:8001::153 |
| AAAA | @    | 2606:50c0:8002::153 |
| AAAA | @    | 2606:50c0:8003::153 |

For `www`, one `CNAME` record:

| Type  | Host | Value                          |
| ----- | ---- | ------------------------------ |
| CNAME | www  | qashanilikamva-cmyk.github.io. |

> Confirm these IPs against GitHub's current documentation before relying on
> them: <https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site>

Some registrars offer `ALIAS` or `ANAME` records, where a single record pointing
at `qashanilikamva-cmyk.github.io` replaces the four `A` records. GoDaddy does
not support these, so the four `A` records are the only option here.

#### GoDaddy walkthrough

Getting to the DNS editor:

1. Sign in at godaddy.com.
2. **My Products** (or the account menu → **My Products**).
3. Find the domain under **Domains**, click the **⋮** next to it, then
   **Edit DNS**.

You land on a list of existing records. Now the two gotchas:

**Gotcha 1 — edit the parked `A` record, don't just add to it.** Every GoDaddy
domain ships with a record `A / @ / <some parking IP>` pointing at GoDaddy's
"future home of..." page. If you leave it there and add the GitHub IPs
alongside, DNS round-robins between them and the site loads GoDaddy's parking
page roughly a fifth of the time — an intermittent fault that looks like
caching but isn't. **Edit** that existing record to `185.199.108.153`, then
**Add** the other three `A` records.

**Gotcha 2 — the `www` record already exists too.** GoDaddy pre-creates
`CNAME / www / @`. Edit it to `qashanilikamva-cmyk.github.io.` rather than
adding a second `www` record — GoDaddy will reject a duplicate host anyway.

Also worth checking while you're in there:

- **Domain forwarding.** If the client ever set up forwarding (Domain Settings
  → Forwarding), remove it. It silently overrides the DNS records and the site
  will never appear.
- **Nameservers.** At the bottom of the DNS page, these must be GoDaddy's own
  (`ns__.domaincontrol.com`). If they point somewhere else, that other provider
  controls DNS and the records must be added there instead.
- **TTL.** GoDaddy defaults to 1 hour. Setting it to 600 seconds on each record
  before you start makes mistakes faster to correct. You can put it back later.

GoDaddy's editor saves each record individually — there's no single "save all"
button, so confirm each row committed before moving on.

**c. Wait for DNS to propagate.** Usually minutes, up to 24 hours. Check with:

```sh
dig +short thatchartisans.co.za
dig +short www.thatchartisans.co.za
```

On Windows, `nslookup thatchartisans.co.za` instead.

As of the last check, both the apex and `www` resolved to `160.153.0.94` — a
GoDaddy parking address. That is the "before" state: once the records above are
saved and propagated, the apex should return the four `185.199.10x.153`
addresses instead. Seeing `160.153.0.94` still means the parked `A` record was
added to rather than edited (Gotcha 1), or propagation hasn't finished.

**d. Turn on HTTPS.** Once **Settings → Pages** shows the domain as verified,
tick **Enforce HTTPS**. The certificate is issued automatically and is free.
The tickbox stays greyed out until DNS resolves, so if it's unavailable, DNS
hasn't propagated yet.

---

## Day-to-day: publishing a change

1. Edit files in `site/`.
2. Commit and push to `main`.
3. Watch the run under the repository's **Actions** tab.

The change is live roughly a minute after the run goes green. Browsers cache
aggressively, so use a hard refresh when checking.

---

## Troubleshooting

**The workflow fails with "Pages is not enabled".** Step 1 above hasn't been
done.

**The domain shows a 404.** `site/index.html` is missing, or `site/CNAME`
doesn't match the domain in **Settings → Pages**.

**"Enforce HTTPS" is greyed out.** DNS hasn't propagated yet, or one of the
records above is wrong. Re-check with `dig`.

**The domain works but `www` doesn't (or vice versa).** The missing half of the
DNS records — apex needs `A`/`AAAA`, `www` needs `CNAME`. GitHub redirects one
to the other automatically once both resolve.
