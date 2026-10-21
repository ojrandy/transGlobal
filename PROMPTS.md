# Trans Global Freight Logistics — Prompt Pack for Claude in VS Code

**The job:** rebrand this working logistics site from *Navora Global Freight* to **Trans Global Freight Logistics**
on **transglobalfreightlogistics.ltd**. This is a **words-and-logo rebrand only**:

- **Changes:** company name, domain, email, logo/icons, tracking-ID and reference prefixes, internal names, and the
  admin console moves from a subdomain to **`transglobalfreightlogistics.ltd/private-site/`**.
- **Stays exactly the same:** every piece of copy (only the name/domain/email inside it changes), the tagline, design,
  colour palette, fonts, layout, photos and every feature. **No photos are replaced**: none show the old brand.

**Target time: under 3 hours** (estimates per prompt add up to ~2 h 35 min, with margin for checks).

**How to use**
- Send the prompts in order, one per chat turn. After each one, check the site (`npm run dev` →
  http://localhost:3000) before sending the next.
- If something breaks, send **Prompt F (Fix)**. New chat? Send **Prompt R (Resume)** first.

> ⛔ **Don't delete or change the old Navora project yet.** Navora keeps running from its own, separate repo. The
> new site goes to the **webspectron** repo. Leave the existing git remotes, the `navora` branch and the old
> domain/Hostinger app alone; they are cleaned up only when we deploy, and only when the owner says so.

---

## Project values

| Key | Value |
|---|---|
| Name | **Trans Global Freight Logistics** |
| Short name (UI) | **Trans Global** ⚠️ *confirm in Prompt 1* |
| Legal name | **Trans Global Freight Logistics** ⚠️ *confirm: add "Ltd" only if the company is registered as such* |
| Tagline | **Connecting Markets, Delivering Trust** (unchanged) |
| Domain | **transglobalfreightlogistics.ltd** (`www.` redirects to it) |
| Email | **info@transglobalfreightlogistics.ltd** ⚠️ *confirm the mailbox exists* |
| Admin console | **https://transglobalfreightlogistics.ltd/private-site/** (same app, same domain) · `http://localhost:3000/#/admin` and `http://localhost:3000/private-site/` locally |
| Tracking ID | **`TGF` + 5 characters = 8 total**, e.g. `TGF7K2M9` ⚠️ *confirm the prefix* |
| Reference prefixes | `TGF-TKT-######`, `TGF-INV-######`, `TGF-SL-######` |
| Internal code prefix | `tgf` (CSS `--tgf-*`, classes `tgf-*`, storage keys `tgf_*`, cookie `tgf.sid`, DB file `transglobal.db`) |
| Example cashtag (payment form placeholder) | `$TransGlobalPay` |
| Logo source | `Public/images/Global Freight Ship and Globe Logo.png` (1774×887, RGBA) → moved to `images/brand-source/transglobal-logo.png` |
| Palette, fonts, photos | **Unchanged.** If the new logo's colours clash with the red `#D3070B`/ink palette, STOP and ask; don't recolour the site on your own |
| Phone / WhatsApp / address / socials | **Not supplied yet. Leave the brand.ts values empty and hidden.** Never invent them. Add them later with Prompt O |
| Old brand terms to remove | `Navora`, `navora`, `NAVORA`, `navoraglobalfreight`, `NavoraPay`, `NGF`/`ngf` (as prefix), `private.navoraglobalfreight.com` |
| Target repo | **webspectron** remote (`https://github.com/webspectron/novaF.git`) |

---

### Prompt 1: Safety net & inventory (no code changes, ~10 min)
```
We're rebranding this working site from "Navora Global Freight" to "Trans Global Freight Logistics" on transglobalfreightlogistics.ltd. Read PROMPTS.md (the values table is the source of truth), CLAUDE.md, src/config/brand.ts, index.html, server/seo.ts, server/index.ts, src/App.tsx (isAdminHost and the hash/path parsing), scripts/optimize-images.mjs and docs/DEPLOYMENT.md.

Rules for the whole job: words and logo only. No changes to copy (except the name/domain/email inside it), tagline, design, colours, fonts, layout, photos or features. Don't delete or change git remotes, the `navora` branch, or anything about the old domain/Hostinger app.

1. `git status` must be clean. Create and switch to a new branch `transglobal` from the current `navora` HEAD.
2. Run `npm run build` and every test in scripts/ (*.test.ts); record the results as the baseline.
3. Count every occurrence of the old brand terms (values table) in src, server, scripts, index.html, Public, docs, CLAUDE.md, package.json and .env.example, grouped as: visible text / metadata & SEO / PDFs & documents / tracking IDs & references / admin-host logic / internal names (CSS vars, classes, storage keys, cookie, DB file) / comments / docs.
4. List every place that decides "is this the admin console?" (client, index.html inline script, server, docs), since Prompt 4 moves the admin from a subdomain to /private-site/.
5. Open the new logo file and describe it: real transparency or a solid background, colours, aspect ratio, whether there is a symbol that can be cropped into a square icon, and whether it works on the dark ink background.

STOP and report: baseline build/tests, counts per group, the admin-logic list, the logo notes, and ask me to confirm the four ⚠️ values in the table (short name, legal name, email, TGF prefix).
```

### Prompt 2: New logo & icons (~20 min)
```
Replace the logo with the Trans Global Freight Logistics logo. Same placement and sizes, no layout change.

1. Move "Public/images/Global Freight Ship and Globe Logo.png" to images/brand-source/transglobal-logo.png (source files are not served; Public/ is). Use git mv.
2. In scripts/optimize-images.mjs (so it can be re-run), from that source produce in Public/brand/, overwriting the existing files with the same names:
   - logo.png: full colour on transparent, trimmed margins, for light backgrounds. If the source has any background left, remove it cleanly (no halo).
   - logo-white.png: version for dark backgrounds (header over the hero, footer, admin sidebar): dark parts turned white, accent colours kept.
   - logo-web.webp and logo-white-web.webp: web-sized copies; update LOGO_WEB_SIZE in brand.ts to the real output size.
   - mark.png: square icon cropped from the ship-and-globe symbol.
   - favicon.png (512), favicon-32.png, favicon-16.png, icon-192.png, icon-maskable-512.png (safe padding), apple-touch-icon.png (180) from mark.png.
   - og-image.jpg (1200×630): logo centred on the current brand background.
   Don't upscale anything.
3. File names stay the same, so brand.ts paths don't change. Grep for any other direct reference to the logo/mark files (barcode labels, JSON-LD, documents/PDFs, admin) and confirm they all resolve.
4. Bump the ?v=N cache-bust in index.html and Public/site.webmanifest.
5. The logo's shape differs from the old one (about 2:1). Check the header, mobile menu, footer, admin sidebar and login, a PDF, and the browser tab. If it reads too small or too large anywhere, adjust ONLY that logo's height/width rule.

`npm run build` must pass. Commit "brand: Trans Global Freight Logistics logo and icons". STOP and show me how the logo looks on light and dark backgrounds.
```

### Prompt 3: Name, domain, email, tracking IDs, references (~30 min)
```
Apply the values table everywhere users can see it. Keep every piece of copy word-for-word apart from the name, domain and email. The tagline does not change. No new marketing text.

1. src/config/brand.ts: COMPANY, COMPANY_SHORT, LEGAL_NAME, EMAIL, DOMAIN "transglobalfreightlogistics.ltd", TRACKING_PREFIX "TGF", REFERENCE_PREFIX "TGF" (as confirmed in Prompt 1). TAGLINE stays. PHONE, WHATSAPP, HQ_ADDRESS and SOCIAL stay empty and hidden. Leave ADMIN_SUBDOMAIN/ADMIN_HOST for now; Prompt 4 replaces them.
2. Hard-coded strings: index.html (title, description, canonical, OG/Twitter, JSON-LD Organization name/url/email/logo), Public/site.webmanifest (name, short_name), App.tsx page titles/descriptions, page copy ("Why ___", "your ___ tracking ID", "___ coordinator", "___ Operations Centre", "___ Intake Desk"), image alt text, src/data (legalDocs.ts, help articles), DocumentBrand.tsx and every document/PDF label, server/index.ts (health service name, start log), server/db.ts settings defaults (new installs only; don't rewrite stored data). In .ts/.tsx, import from brand.ts instead of hard-coding the new name.
   Watch grammar where the name changes length or article: "a Navora …" → "a Trans Global …", possessives, and places where the short name now reads awkwardly; list any sentence you had to adjust beyond the name swap.
3. Payment form example: "$NavoraPay" → "$TransGlobalPay" in src/shared/paymentMethods.ts and paymentValidation.ts, plus the tests.
4. Tracking IDs: trackingId.ts builds its regex from TRACKING_PREFIX. Update every example/placeholder (NGF7K2M9 → TGF7K2M9, "NGF·····" → "TGF·····"), help text and scripts/trackingId.test.ts.
5. References: generate and display TGF-TKT / TGF-INV / TGF-SL, and update scripts/references.test.ts.
6. Docs copy: docs/CONTENT.md and docs/BRAND_GUIDE.md get the same name/domain/email/prefix swap (copy text otherwise unchanged), so the docs stay the source of truth.

`npm run build` and all tests must pass. Check Home (hero, "Why" section), Track, Contact, Legal and one PDF at 375px, 768px and 1440px; the longer name must not overflow the header, footer or documents (adjust only the affected rule if it does). Create a shipment in admin and confirm it gets a TGF ID that tracks publicly. Commit "brand: Trans Global name, domain, email and IDs". STOP and report.
```

### Prompt 4: Admin console at /private-site/ (~35 min)
```
Move the admin console from the subdomain (private.<domain>) to a path on the main domain: https://transglobalfreightlogistics.ltd/private-site/. Same app, same API, same auth. Don't change server/middleware/auth.ts security logic, rate limits or session settings (other than what is listed below). Explain your plan before editing.

1. brand.ts: replace ADMIN_SUBDOMAIN/ADMIN_HOST with ADMIN_PATH = '/private-site/' (plus a helper, e.g. isAdminPath(pathname), that matches "/private-site" and "/private-site/" exactly, case-sensitively, nothing else). ADMIN_CONSOLE_NAME becomes "Trans Global Operations Console" via COMPANY_SHORT.
2. src/App.tsx: isAdminHost() → isAdminPath() based on window.location.pathname. Keep every existing behaviour: on the admin path the app only ever shows the admin (login or console), robots noindex meta is set, the live-chat script is not loaded, and #/admin on the public site still does NOT open the admin in production (localhost still allows #/admin for dev). Logout/leaving the console goes to /private-site/ (login), never to a public page with a session.
3. index.html inline hero-preload script: skip when the path is /private-site (instead of the hostname check). Any other hostname check in index.html → path check.
4. server: express.static only serves "/" as index.html. Add an explicit route so GET /private-site redirects 301 to /private-site/ and GET /private-site/ serves dist/index.html with Cache-Control no-cache and X-Robots-Tag "noindex, nofollow". Assets must keep loading from absolute /assets/ and /brand/ paths (check the built HTML). Don't add other catch-all routes.
5. server/seo.ts: isAdminRequest() → path-based (req.path starts with /private-site). Do NOT list /private-site in robots.txt or the sitemap (that would advertise it); the noindex header and meta are enough.
6. ADMIN_PROXY_TARGET (the separate admin-subdomain deployment mode in server/index.ts) is no longer needed. Remove that mode and its env var from .env.example only if it's self-contained; if removing it touches anything else, leave it and tell me.
7. Session cookie: stays host-only on transglobalfreightlogistics.ltd; this works because admin and API are now on the same origin. Note that www must redirect to the bare domain (Prompt 8) so the cookie is always on one host.
8. Update docs/DEPLOYMENT.md and CLAUDE.md §2–§3 (admin URL, how admin access is decided).

Verify on `npm run build` + `npm start` (production mode, not just dev): /private-site and /private-site/ open the login; login → dashboard → create shipment → logout works; refresh on the console keeps the session; "/" and "/#/admin" show the public site only; the response headers on /private-site/ include noindex; robots.txt doesn't mention it. Commit "feat(admin): serve the console at /private-site/". STOP and report, with any security trade-off you see.
```

### Prompt 5: Internal clean-up: remove every old-brand trace (~30 min)
```
Remove the remaining internal old-brand names. This is a mechanical rename; behaviour and appearance must not change. The new site starts with a fresh, empty database, so nothing needs migrating.

1. CSS: --ngf-* → --tgf-*, class prefix ngf- → tgf-, in one scoped pass across src/ and index.html (word-boundary-aware; don't touch unrelated words). Then grep that no `--ngf-` or `ngf-` class remains and no CSS var is left undefined.
2. Storage keys ngf_* → tgf_*, session cookie ngf.sid → tgf.sid (only the SESSION_COOKIE name in server/middleware/auth.ts; no logic changes), and update the cookie/storage names listed in legalDocs.ts.
3. DB file navora.db → transglobal.db (server/db.ts, server/index.ts, .env.example, docs). Don't rename tables or columns. Don't touch my local data/ folder; tell me what to do with the old file instead.
4. Comments, file names, test names and package.json "name" ("transglobal-freight-logistics"; regenerate package-lock.json with `npm install --package-lock-only`) that mention the old brand → Trans Global or brand-neutral.
5. Docs: rewrite CLAUDE.md title, §1 and §2 for Trans Global Freight Logistics (no history of earlier brands). Update company facts in docs/BRAND_GUIDE.md, CONTENT.md, DEPLOYMENT.md, MOTION_3D_SPEC.md and PROJECT_TRACKER.md (add a dated "Trans Global rebrand" Change log entry and list the ⚠️ owner confirmations under Blocked / Needs owner). docs/PROMPT-UPDATE.md and docs/update.md are old prompt packs: ask me whether to update or delete them.

Final grep (case-insensitive) over src, server, scripts, index.html, Public, docs, CLAUDE.md, package.json, .env.example for: navora, navoraglobalfreight, ngf (as prefix/word), private.navora. Goal: zero hits outside PROMPTS.md; list any you can't remove and why. Then `npm run build`, all tests, `npm run dev` and confirm: pages look identical, admin login at /private-site/, create → track, a PDF, a quote, a contact message. Commit "chore: remove remaining old-brand names". STOP and report.
```

### Prompt 6: New admin password & session secret (~5 min)
```
This is a separate business, so it gets its own admin password and session secret. Configuration only: don't edit auth logic.

1. Give me the two commands to run MYSELF in the project folder. Don't run them, and never ask for the password:
   - node -e "console.log(require('bcryptjs').hashSync('MY-NEW-PASSWORD', 12))"
   - node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
2. Tell me exactly what to put in my local .env (ADMIN_PASSWORD_HASH, SESSION_SECRET, DB_PATH if used), and whether the `$` characters in the hash need quoting. Remind me the same values go into Hostinger in Prompt 8.
3. After I say "done", restart `npm run dev` and have me confirm at http://localhost:3000/private-site/: the old password is rejected, the new one works, and logout/login works.
STOP.
```

### Prompt 7: Final check (~15 min)
```
1. Fresh `npm run build`, then sweep src, server, Public, index.html AND dist/ + dist-server/ for the old brand terms. Zero hits.
2. Full run with `npm start` (production build): admin login at /private-site/ → create shipment (TGF ID) → track publicly → change status → generate each document type → quote → contact message. Confirm the new name and logo show everywhere (header, footer, tab icon, PDFs, share preview tags) and nothing else changed visually. Check 375px, 768px, 1440px.
3. Confirm the admin opens only at /private-site/ in production (not via #/admin, not via any other path), and #/admin still works on localhost in dev.
4. Confirm there's no demo data in code or seed and SEED_DEMO_DATA is unset/false.
5. Tick the tasks in docs/PROJECT_TRACKER.md and make sure Blocked / Needs owner lists everything still open (contact details, any ⚠️ value not yet confirmed).
Commit any fixes as "chore: pre-deploy checks". STOP with a short ready / not-ready report.
```

### Prompt 8: Deploy to transglobalfreightlogistics.ltd on Hostinger (~20 min, mostly me in hPanel)
```
Prepare the deploy per docs/DEPLOYMENT.md. Don't deploy anything yourself and don't push without my go-ahead; give me instructions. Don't touch the old Navora repo, app, domain or database.

1. GitHub: the commands to push the `transglobal` branch to the webspectron remote (https://github.com/webspectron/novaF.git) as its main branch. Show me first what is currently on that remote (`git ls-remote webspectron`) and warn me before anything that would overwrite existing history. Check .gitignore excludes node_modules, dist, dist-server, data, .env.
2. hPanel: create a Node.js app for transglobalfreightlogistics.ltd, connect the repo, Node ≥ 22.5, install/build/start commands, entry dist-server/server/index.js.
3. Env vars: NODE_ENV=production, PORT if needed, ADMIN_PASSWORD_HASH and SESSION_SECRET (from Prompt 6), DB_PATH in persistent storage (e.g. …/persistent-storage/transglobal.db). The database starts empty.
4. Domain: transglobalfreightlogistics.ltd + www on the app, SSL on both, force HTTPS, www → bare domain 301 redirect, and the DNS records if the domain is registered elsewhere. No admin subdomain is needed any more.
5. Email: remind me that info@transglobalfreightlogistics.ltd must exist as a mailbox before launch, and that any outgoing-mail settings (if used) need the new domain's records.
6. Smoke test: new logo, name and tab icon; share preview uses transglobalfreightlogistics.ltd; https://transglobalfreightlogistics.ltd/#/admin does NOT open admin; https://transglobalfreightlogistics.ltd/private-site/ shows the login and the new password works; create → track a TGF shipment; data survives a redeploy.
STOP and wait for my results. Removing the old Navora site/repo is a separate, later step, only on my instruction.
```

---

## Utility prompts

### Prompt R: Resume in a new chat
```
Resume the Trans Global Freight Logistics rebrand. Read CLAUDE.md, PROMPTS.md and docs/PROJECT_TRACKER.md, run `git status`, `git branch --show-current` and `git log --oneline -10`, then tell me which prompt we're on and what's next. Don't change anything until I confirm.
```

### Prompt O: Owner contact details arrived
```
Here are Trans Global Freight Logistics' contact details: [phone / WhatsApp / HQ address / social links].
Add them to src/config/brand.ts (and the JSON-LD in index.html), confirm the UI that was hidden now shows them correctly on Header, Footer, Contact and documents, update the values table in PROMPTS.md and CLAUDE.md §2, build, and commit "content: add Trans Global contact details".
```

### Prompt F: Fix something that broke
```
Something broke: [what you see, which page, steps, any error message].
Find the root cause (check the latest commits first) and explain it before fixing. Apply the smallest fix that doesn't change design or unrelated behaviour, confirm `npm run build` passes and admin login at /private-site/ → create → track still works, and commit "fix: …".
```
