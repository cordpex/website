# cordpex.com

The Cordpex website — a static Astro site hosted on Cloudflare Pages.

Live at **https://cordpex.com** (and `www.cordpex.com`).

## Stack

| Layer | What |
|---|---|
| Framework | [Astro](https://astro.build) — static output, no runtime JS shipped |
| Source | GitHub, `cordpex/website` |
| Hosting | Cloudflare Pages, project `cordpex` (`cordpex.pages.dev`) |
| DNS / TLS / email | Cloudflare |

The page has no external fonts, scripts, or images — everything is inline or in
`public/`. That keeps it fast and avoids depending on third-party hosts.

## Brand

Design tokens live in `src/styles/brand.css`. Colours are taken from the logo
artwork in the brand kit:

| Token | Hex | Use |
|---|---|---|
| `--navy` | `#12263f` | Text, logo, primary |
| `--orange` | `#fd6a18` | Accent only |
| `--cream` | `#f5f2eb` | Page background |
| `--periwinkle` / `--blue-muted` / `--tan` | `#c3c9e0` / `#7181b5` / `#e2c293` | Secondary, unused so far |

Typefaces are **Utendo** (display/headings) and **Satoshi** (body, variable),
self-hosted from `public/fonts/` — no external font CDN, so nothing leaks to a
third party and there is no render-blocking request to another origin.

The brand kit ships TrueType. The `.woff2` files in `public/fonts/` were
converted from it, which cut them from ~200KB to ~74KB total:

```bash
npx --yes -p wawoff2 node -e "
  const {compress}=require('wawoff2'), fs=require('fs');
  compress(fs.readFileSync('Utendo-Bold.ttf')).then(b=>fs.writeFileSync('Utendo-Bold.woff2',b));
"
```

Source assets (full logo set, patterns, original TTFs) are kept outside this
repo in `../brand assets/` — they are large and not needed to build the site.

## Local development

Requires **Node 22+** (Astro 7 will not run on older versions).

```bash
npm install
npm run dev      # dev server at http://localhost:4321
npm run build    # static output into dist/
npm run preview  # serve the built dist/ locally
```

## Structure

```
src/pages/index.astro   the entire landing page — markup + scoped styles
public/favicon.svg      inline SVG favicon
astro.config.mjs        sets `site` so canonical/OG URLs resolve absolutely
```

Astro treats every file in `src/pages/` as a route, so adding `about.astro`
there would publish `/about` with no routing config.

## Deployment

Fully automatic. **Any push to `main` triggers a Cloudflare Pages build and
deploy** — there is no manual deploy step and no CI config in this repo.

Pages build settings (configured in the Cloudflare dashboard, not in-repo):

| Setting | Value |
|---|---|
| Framework preset | Astro |
| Build command | `npm run build` |
| Build output directory | `dist` |
| Production branch | `main` |
| `NODE_VERSION` | `22` |

`NODE_VERSION` is pinned deliberately. Without it the build inherits whatever
Node version Cloudflare currently defaults to, which can change and would break
the build without any commit landing here.

Pushes to other branches produce preview deployments at their own URLs; only
`main` updates production.

## Domains and TLS

Both `cordpex.com` and `www.cordpex.com` are Pages custom domains, each a CNAME
to `cordpex.pages.dev` (Cloudflare flattens the apex CNAME automatically).

Certificates are issued and renewed by Cloudflare Universal SSL — nothing to
manage. Zone settings: **Full (strict)** encryption, **Always Use HTTPS**,
minimum **TLS 1.2**, with TLS 1.3 and Automatic HTTPS Rewrites enabled.

> Note: a Cloudflare Pages custom domain must live in the *same* Cloudflare
> account as the Pages project. A proxied CNAME pointing across accounts is
> rejected with [Error 1014](https://developers.cloudflare.com/support/troubleshooting/http-status-codes/cloudflare-1xxx-errors/error-1014/).

## Email

`info@cordpex.com` is handled by **Cloudflare Email Routing**, which forwards to
an external inbox. A catch-all rule covers every other address at the domain.
MX, SPF and DKIM records are managed by Email Routing and are locked in the DNS
dashboard — don't edit them by hand.

Email Routing is **receive-only**. Mail can arrive at `info@cordpex.com`, but
nothing can be *sent* from it: Cloudflare provides no SMTP. Sending as this
address would require an external relay (Resend, Brevo, Mailgun) wired up as a
"Send mail as" identity, or moving the mailbox to a real mail host.
