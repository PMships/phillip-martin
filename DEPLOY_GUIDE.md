# phillip-martin.com — Deploy & DNS Guide

**Prepared by:** Damien Croft
**Site bundle:** `/outputs/phillip-martin-site/` (folder) and `/outputs/phillip-martin-site.zip` (one-file)
**Target host:** Netlify free tier
**Domain registrar:** GoDaddy

---

## What's in the bundle

| File | Purpose |
|---|---|
| `index.html` | The site. Single-file, no build step. |
| `404.html` | Branded "not found" page Netlify auto-serves. |
| `phillip-martin-cv.pdf` | Your CV. Linked from the hero. Also reachable at `/cv`. |
| `netlify.toml` | Security headers, caching, and the `/cv` and `/linkedin` redirects. |
| `robots.txt` | Allows all crawlers. |
| `sitemap.xml` | One-URL sitemap. |

**No build step. No framework. No dependencies.** Drag the folder into Netlify, it's live in 30 seconds.

---

## Step 1 — Deploy to Netlify (3 minutes)

### Option A — Drag-and-drop (recommended — zero config)

1. Sign in at **app.netlify.com** (free; sign in with GitHub or email).
2. From the dashboard, look for the drop zone labelled **"Want to deploy a new site without connecting to Git? Drag and drop your site output folder here."**
3. Drag the **`phillip-martin-site`** folder (or the `phillip-martin-site.zip`) onto the drop zone.
4. Netlify gives you a temporary URL like `random-name-123abc.netlify.app`. Visit it. Confirm the site renders correctly.
5. In **Site configuration → Site information → Change site name**, rename it to `phillip-martin` so the staging URL becomes `phillip-martin.netlify.app`. Cleaner for sharing pre-domain.

### Option B — GitHub-connected deploy (better long-term)

If you want every git push to auto-deploy:

1. Create a new private repo on GitHub: `phillip-martin-site`.
2. Push the contents of `phillip-martin-site/` to that repo.
3. In Netlify: **Add new site → Import an existing project → GitHub → pick the repo**.
4. Build settings: leave **Build command** empty. **Publish directory:** `.` (just a dot). Click Deploy.

Use Option A today. Move to Option B later if you want CI on edits.

---

## Step 2 — Add your custom domain in Netlify (2 minutes)

1. In your Netlify site dashboard → **Domain management** → **Add a domain**.
2. Enter `phillip-martin.com` and click **Verify**.
3. Netlify will say *"This domain is not registered with Netlify DNS — would you like to..."*. You have two choices:

   - **Choice A — Use Netlify DNS (recommended).** Netlify hosts your DNS. You point your GoDaddy nameservers at Netlify once, and never touch DNS again. Simpler, faster, supports their automatic SSL and apex-domain redirects cleanly.
   - **Choice B — Keep DNS at GoDaddy.** You add specific A/CNAME records in GoDaddy pointing at Netlify. More fiddly but works fine.

4. Netlify will show you the records or nameservers you need. Keep that tab open — you'll need it for Step 3.
5. Also add `www.phillip-martin.com` as a domain alias. In Netlify domain settings, set the **primary domain** to `phillip-martin.com` (the apex, without `www`). Netlify will auto-301 `www.` to apex.

---

## Step 3 — Point GoDaddy at Netlify (5 minutes)

Pick one of the two paths below. **Path A is what I'd do.**

### Path A — Switch nameservers to Netlify DNS (recommended)

1. In Netlify, when you chose "Use Netlify DNS" above, it gave you **4 nameservers** like:
   ```
   dns1.p01.nsone.net
   dns2.p01.nsone.net
   dns3.p01.nsone.net
   dns4.p01.nsone.net
   ```
   (The actual values will differ — copy them from your Netlify screen.)

2. Open GoDaddy → **My Products → Domains → phillip-martin.com → DNS** (or **Manage DNS**).
3. Scroll to **Nameservers** → click **Change**.
4. Choose **"I'll use my own nameservers"** (or "Custom").
5. Paste the four Netlify nameservers, one per row. Save.
6. **Propagation:** typically 15 minutes to 4 hours. Sometimes up to 24-48h in the worst case. You can check with `dig NS phillip-martin.com` from Terminal, or use **dnschecker.org**.
7. Once live, Netlify will auto-provision a free **Let's Encrypt SSL certificate**. It happens within minutes of the nameservers resolving. The padlock will appear in browsers.

### Path B — Keep GoDaddy DNS, add records pointing at Netlify

If you'd rather keep DNS at GoDaddy (e.g. because you also use GoDaddy email):

1. In Netlify domain settings, you'll see the required records. They are usually:

   | Type | Host / Name | Value |
   |---|---|---|
   | A | @ (apex) | `75.2.60.5` *(or whatever Netlify shows you — they sometimes rotate)* |
   | CNAME | www | `phillip-martin.netlify.app` |

   **⚠️ Always use the values Netlify shows you in the dashboard at deploy time — not these.** The A-record IP can change.

2. In GoDaddy → **DNS Management** for `phillip-martin.com`:
   - Delete or edit the default **Parking** A-record at `@`. Set it to the IP Netlify gave you.
   - Add a **CNAME** record at `www` pointing to `phillip-martin.netlify.app`.
   - Remove any conflicting CNAME or A-records GoDaddy auto-created for marketing pages.
3. Save. Propagation: same as Path A.
4. Back in Netlify, click **Verify DNS configuration** — it'll confirm the records and provision SSL.

### Both paths — forcing HTTPS

Once SSL is provisioned, in Netlify → **Domain management → HTTPS** → toggle **"Force HTTPS"** ON. All `http://` requests will 301 to `https://`.

---

## Step 4 — Verify (5 minutes)

Run through this checklist after DNS propagates:

- [ ] `https://phillip-martin.com` — loads, padlock visible, renders correctly on desktop
- [ ] `https://www.phillip-martin.com` — redirects to apex
- [ ] `http://phillip-martin.com` — redirects to `https://`
- [ ] `https://phillip-martin.com/cv` — opens the CV PDF
- [ ] `https://phillip-martin.com/linkedin` — redirects to your LinkedIn
- [ ] Mobile check: open on your phone. Tap-targets work, layout doesn't break.
- [ ] **Page speed:** run https://pagespeed.web.dev/analysis?url=https://phillip-martin.com — should score 95+ on both desktop and mobile.
- [ ] **Meta preview:** paste your URL into https://www.opengraph.xyz/ — confirms the LinkedIn/Twitter share card looks right.

---

## Step 5 — After it's live

Three updates that need to land within 48 hours of the site going live:

1. **LinkedIn profile** → add `phillip-martin.com` to your contact info, and to your headline if you want (Option C from the LinkedIn rewrite doc — "Writing about Agentic AI × Blockchain at phillip-martin.com").
2. **Email signature** → add the URL alongside your LinkedIn.
3. **CV update** → swap the LinkedIn link footer on your CV for `phillip-martin.com` (one stable URL is stronger than two).

---

## Future updates (the right way)

When you want to update the site:

- **Option A path (drag-drop):** edit `index.html` locally, then in Netlify → **Deploys** → drag the updated folder onto the deploy zone. Live in 30 seconds.
- **Option B path (GitHub):** edit, commit, push. Netlify auto-deploys.

When the **INA-Agent side project goes live**, update the "Building In Public" section to point at the live URL and the GitHub repo — that's the most important edit you'll make in the first 30 days.

---

## What I'd NOT do

- **Don't pay for Netlify Pro** for this site. The free tier covers everything you need — bandwidth, SSL, custom domain, 99.99% uptime SLA in practice.
- **Don't add Google Analytics on day one.** Privacy-respecting and faster without it. If you want lightweight analytics later, use **Plausible** (€9/mo) or Netlify's own basic analytics ($9/mo).
- **Don't add a contact form.** Static-form services (Netlify Forms) work, but spam is real and your email is already on the page. Direct email is better.
- **Don't add a blog section here.** Your writing lives on LinkedIn first — that's where the recruiter funnel is. The "Writing" section on the site links out to LinkedIn posts. Don't fork your content distribution.

---

## If something breaks

| Symptom | Likely cause | Fix |
|---|---|---|
| DNS still showing GoDaddy parking page after 6 hours | Nameservers haven't propagated yet | Wait. Check `dig NS phillip-martin.com`. If still showing GoDaddy, re-check Step 3. |
| SSL certificate not provisioning | DNS not fully propagated | Wait 1 hour, then in Netlify click **Renew certificate**. |
| Site loads but no styles | Browser cache | Hard refresh (Cmd+Shift+R). |
| 404 on `/cv` | Redirect rule not picked up | Re-deploy the folder; `netlify.toml` must be at the root. |

Email me (your recruiter, in role) if anything blocks for more than an hour. We don't lose a day to a DNS issue.

Move.
