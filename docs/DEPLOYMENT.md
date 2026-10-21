# Deployment: GitHub → Hostinger

> **Context:** The Trans Global platform is **already built and working**: public website, tracking engine, Express/SQLite API,
> admin console, documents and quotes. Claude is acting as a **senior professional developer** who maintains and improves this
> existing codebase. Nothing here is built from scratch; every task modifies the working system
> in place and must leave it working. See `CLAUDE.md §0`.

**Note:** the app has already been deployed to Hostinger once (see the comment in `vite.config.ts` about the `Public/` folder). This guide applies that proven setup to the Trans Global repo and domain.

The app is **one Node.js process**: Express serves the API (`/api/*`) and the built React site (`dist/`).
It stores data in a **SQLite file**. That shapes everything below.

---

## 1. Hosting requirements

| Need | Why |
|---|---|
| A Hostinger plan that runs **Node.js web apps** (Business / Cloud web hosting, or a VPS) | Plain static hosting cannot run the Express server, tracking API or admin |
| **Node.js ≥ 22.5** (choose 22.x LTS or newer) | `server/db.ts` uses the built-in `node:sqlite` |
| **Persistent storage** outside the deploy folder | The database must survive every redeploy |
| HTTPS (free SSL in hPanel) | The admin session cookie is `secure` in production |

Hostinger's hPanel screens change from time to time. If a label below doesn't match, look for the equivalent setting.

## 2. Repository setup

1. Create a **new private GitHub repo** (e.g. `transglobal-freight-logistics`). Push the code with fresh history:
   ```bash
   rm -rf .git && git init && git add . && git commit -m "Trans Global Freight Logistics: initial import"
   git branch -M main && git remote add origin https://github.com/<you>/transglobal-freight-logistics.git && git push -u origin main
   ```
2. Confirm `.gitignore` still excludes `node_modules/`, `dist/`, `dist-server/`, `/data/`, `.env`.
3. Add `"engines": { "node": ">=22.5" }` to `package.json`.

## 3. Hostinger app configuration

| Setting | Value |
|---|---|
| Source | GitHub → `transglobal-freight-logistics`, branch `main` |
| Node version | 22.x (or newer LTS) |
| Install command | `npm install` (the `postinstall` script already runs `npm run build`) |
| Build command | `npm run build` (only if the panel requires a separate build step; don't build twice if postinstall already did) |
| Start command | `npm start` → `node dist-server/server/index.js` |
| Port | Use the port the platform provides (`PORT` env var). The server reads `process.env.PORT` |

## 4. Environment variables (set in hPanel, never commit)

```ini
NODE_ENV=production
PORT=<provided by Hostinger or 5000>
ADMIN_PASSWORD_HASH=<bcrypt hash — see below>
SESSION_SECRET=<64 hex chars — see below>
DB_PATH=/home/<hostinger-user>/transglobal-data/transglobal.db   # a folder OUTSIDE the app/deploy directory
UPLOADS_DIR=/home/<hostinger-user>/transglobal-data/uploads # same persistent folder: package photos + payment proofs
```

Generate these locally:
```bash
node -e "console.log(require('bcryptjs').hashSync('YOUR-STRONG-PASSWORD', 12))"
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

**Database file:** the default is `data/transglobal.db` (relative to the working directory) when `DB_PATH` is unset. The
site starts with a fresh, empty database; the schema and default settings are created on first start.

**Uploads folder:** `UPLOADS_DIR` holds package photos (`images/`, shown on the track page) and payment-proof
screenshots (`proofs/`, private, admin-only). Both subfolders are created on start, and the startup log prints the
path in use. When unset it defaults to an `uploads` folder next to the database (`data/uploads` locally). Like
`DB_PATH`, it **must** point outside the app/deploy directory on Hostinger, or every redeploy deletes the files.

**Check persistence before launch:** the server has a temporary endpoint, `GET /api/diag/storage`. It is admin-only,
so log in to the admin console first, then open `https://transglobalfreightlogistics.ltd/api/diag/storage` in the same
browser. Redeploy and open it again. If `markerFirstSeen` stays the same, storage persists. **After confirming, remove
this endpoint.**

## 5. Domains & DNS

| Host | Points to | Purpose |
|---|---|---|
| `transglobalfreightlogistics.ltd` | Node app | Public site + admin at `/private-site/` |
| `www.transglobalfreightlogistics.ltd` | **Redirect → bare domain** (required, see below) | |

- **Admin console: `https://transglobalfreightlogistics.ltd/private-site/`**, a path on the same Node app, domain
  and API as the public site (no subdomain or DNS record). `isAdminPath()` in `src/config/brand.ts` (`ADMIN_PATH`)
  matches `/private-site` and `/private-site/` exactly and case-sensitively; `src/App.tsx` checks it against
  `location.pathname`, and on that path the app shows only the admin login or console.
- The server (`server/index.ts`) redirects `GET /private-site` 301 → `/private-site/` and serves `dist/index.html`
  there with `Cache-Control: no-cache` and `X-Robots-Tag: noindex, nofollow`; the page also adds
  `<meta name="robots" content="noindex, nofollow">`. Don't list `/private-site` in `robots.txt` or `sitemap.xml`
  (that would advertise it).
- Anywhere else, `#/admin` falls back to Home (it opens the admin on localhost only, for development).
- The session cookie is host-only (no `Domain`), so `www` **must** redirect to the bare domain; otherwise a login on
  one host is not seen on the other.
- Logging out returns to the login at `/private-site/`. Leaving a signed-in console for the public site ends the
  session first.
- If the old `private.` subdomain alias still exists in hPanel, remove it or redirect it to `/private-site/`.
- Turn on SSL for both hostnames, and force HTTPS.

## 6. Email (info@transglobalfreightlogistics.ltd)

- Create the mailbox (Hostinger Email or Google Workspace).
- DNS: **MX** records from the provider · **SPF** `v=spf1 include:<provider> ~all` · **DKIM** (from the provider) ·
  **DMARC** `v=DMARC1; p=quarantine; rua=mailto:info@transglobalfreightlogistics.ltd`.
- Send a test to Gmail and check "Show original" → SPF/DKIM/DMARC = PASS.

## 7. Launch checklist (smoke test on the live domain)

- [ ] **Demo data removed (launch blocker, tracker 6.7).** No demo shipments, quotes or documents on the public site or in the
  production database; `grep -rn "DEMO DATA" src server` returns nothing. Tracking any made-up ID (e.g. TGF7K2M9) on
  the live site must say "not found".
- [ ] Home loads over HTTPS, and the globe loads (or falls back) with no console errors.
- [ ] `/api/health` returns `ok` with the Trans Global service name.
- [ ] `/private-site` and `/private-site/` → login works with the new password; refresh keeps the session; logout
  returns to the login. `/` and `/#/admin` do **not** open admin. `curl -I .../private-site/` shows `X-Robots-Tag: noindex, nofollow`.
- [ ] Create a shipment in admin → it gets a `TGF` + 5-character ID → the public Track page finds it.
- [ ] Download the waybill / POD PDFs; both show Trans Global branding only.
- [ ] Quote request → appears in admin → quote link opens publicly.
- [ ] Contact and callback forms submit and show a `TGF-TKT-` reference.
- [ ] Redeploy once, then confirm the shipment above still exists (persistence).
- [ ] View source + `robots.txt` + `sitemap.xml` + OG preview (e.g. paste the link in WhatsApp): all Trans Global.
- [ ] **OG image is live:** `https://transglobalfreightlogistics.ltd/brand/og-image.jpg` returns 200 (`image/jpeg`). `index.html` points `og:image`,
  `twitter:image`, `og:url` and `canonical` at that exact domain, so if the site is served on `www.` instead, update all four
  first. Then run the home URL through the Facebook Sharing Debugger and LinkedIn Post Inspector ("Scrape again") so they
  drop any cached preview from before launch.
- [ ] View source of the live HTML/JS bundle: only Trans Global names, domain and email appear.
- [ ] Lighthouse mobile on Home meets the MOTION_3D_SPEC §2 targets.

## 8. After launch

- **Backups:** copy `transglobal.db` **and the `UPLOADS_DIR` folder** daily to off-server storage (a Hostinger cron job +
  download, or a VPS cron to cloud storage). The database stores only file names; the photos and proofs live in
  the folder. Test a restore once.
- **Updates:** push to `main` → Hostinger redeploys. Use a `staging` branch/subdomain for bigger changes.
- **Monitoring:** a free uptime monitor on `/api/health` (5-minute interval) with email alerts to info@.
- **Security:** rotate `ADMIN_PASSWORD_HASH` and `SESSION_SECRET` if anyone leaves; keep dependencies updated
  (`npm audit` monthly).