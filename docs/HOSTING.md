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

Replace `example.com` below with the client's actual domain.

**a. Tell GitHub the domain.** Create a file `site/CNAME` containing one line
and nothing else:

```
example.com
```

**b. Add the DNS records** at the domain's registrar (where the client bought
it — the place that controls the nameservers).

For the apex domain (`example.com`), four `A` records, all with host `@`:

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

If the registrar offers `ALIAS` or `ANAME` records, a single one pointing at
`qashanilikamva-cmyk.github.io` is preferable to the four `A` records — it keeps
working if GitHub changes its IPs.

**c. Wait for DNS to propagate.** Usually minutes, up to 24 hours. Check with:

```sh
dig +short example.com
dig +short www.example.com
```

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
