# Enabling GitHub Pages

The `docs/` folder serves three pages Apple requires for App Store submission:

| File | Purpose | App Store Connect field |
|---|---|---|
| `index.html` | Marketing landing page | **Marketing URL** (optional) |
| `privacy.html` | Privacy policy | **Privacy URL** (required) |
| `support.html` | Support / contact / FAQ | **Support URL** (required) |

## Steps

1. Push this repository to GitHub (any visibility — public, private with Pages plan, or private with GitHub Pro/Team).
2. In your GitHub repo: **Settings → Pages**.
3. Under **Build and deployment**:
   - **Source**: `Deploy from a branch`
   - **Branch**: `main`, folder: `/docs`
4. Click **Save**. First deploy takes ~30–60 seconds. A green checkmark + URL appears at the top of the page.
5. Your URLs will be:
   ```
   https://<your-github-username>.github.io/<repo-name>/             # marketing
   https://<your-github-username>.github.io/<repo-name>/privacy.html # privacy
   https://<your-github-username>.github.io/<repo-name>/support.html # support
   ```
6. (Optional) **Custom domain**: add a `CNAME` file at `docs/CNAME` containing your domain (e.g. `restock.app`). Then configure your DNS to point `CNAME` to `<username>.github.io`. Wait for HTTPS to provision.

## Before submitting

Two placeholders remain in `support.html` — replace them before going live:

- `__SUPPORT_EMAIL__` — your contact email (Apple verifies the page resolves; a `mailto:` won't be tested but should work)
- `__GITHUB_ISSUES_URL__` — `https://github.com/<your-username>/<repo>/issues`

Use a simple find/replace:

```bash
sed -i '' 's|__SUPPORT_EMAIL__|support@yourdomain.com|g' docs/support.html
sed -i '' 's|__GITHUB_ISSUES_URL__|https://github.com/yourname/restock/issues|g' docs/support.html
```

## After Pages is live

Fill the three URLs into `Secrets/secrets.yaml`:

```yaml
urls:
  marketing: "https://yourname.github.io/restock/"
  support:   "https://yourname.github.io/restock/support.html"
  privacy:   "https://yourname.github.io/restock/privacy.html"
```

Then re-run the populator:

```bash
python3 Secrets/populate_fastlane.py
```

It writes those URLs into `fastlane/metadata/en-US/marketing_url.txt`, `support_url.txt`, `privacy_url.txt`, which `bundle exec fastlane metadata` pushes to App Store Connect.

## Validation checklist

After Pages goes live, verify in a private browser window:

- [ ] `/` (index) renders, links to /privacy.html and /support.html work
- [ ] `/privacy.html` renders, all sections visible, links back to / work
- [ ] `/support.html` renders, FAQ accordions expand, placeholders replaced
- [ ] All three return HTTP 200 (not redirects); Apple's review pipeline does not follow redirects reliably
- [ ] Page loads under 2 seconds on a cold network (lighthouse score 95+ is easy with static HTML)

## Why not use a different host?

GitHub Pages is:
- Free for public repos (and bundled with private repos on most paid plans)
- HTTPS by default, even for `github.io` subdomains
- Auto-deploys on every push to `main`
- No vendor lock-in: the docs/ folder is just static HTML; move it to Cloudflare Pages, Vercel, Netlify, or your own server in ~3 minutes if you outgrow it
