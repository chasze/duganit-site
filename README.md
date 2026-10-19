# duganit.com

Personal site for IT projects and study notes. Plain HTML and CSS, no build step.

## What's in here

| Path | What it is |
|---|---|
| `index.html` | Home page: hero with cert path, About, toolbox, projects, study log, contact |
| `notes/truenas-orphaned-snapshot.html` | Example write-up. Copy it to start a new note |
| `assets/css/site.css` | All styles. Colors are variables at the top, light and dark mode |
| `assets/img/favicon.svg` | Browser tab icon |
| `404.html` | Not-found page |
| `staticwebapp.config.json` | Azure Static Web Apps config: 404 page and security headers |

## Editing

- **Cert path:** in `index.html`, each step is an `<li>`. No `data-state` means earned, `data-state="current"` is the one you're studying, `data-state="planned"` is future. Keep `--steps` equal to the number of steps.
- **Projects:** copy a `<li class="project">` block. Status is `running`, `building` or `planned`.
- **Study log:** newest entry first. Copy an `<li>`.
- **New note:** copy the TrueNAS note into `notes/`, rename it, change the `<title>`, description, date and body, then link to it from a project or the study log.
- **Colors:** the aqua and yellow come from fiber jacket colors (OM3/OM4 aqua, single-mode yellow). Change them in the `:root` block in `site.css`.

## Preview locally

Links use root paths (`/assets/...`), so open the site through a local web server rather than double-clicking the file:

```powershell
cd dugan-site
python -m http.server 8080
# then browse to http://localhost:8080
```

To test the Azure config (headers, 404) locally, use the Static Web Apps CLI: `npm install -g @azure/static-web-apps-cli`, then `swa start .`

## Hosting: use Static Web Apps, not App Service F1

The free App Service plan (F1) **does not allow custom domains**. You'd be stuck on `yourapp.azurewebsites.net` until you scale up to Basic (B1) or higher, which costs money. Shared (D1) allows a custom domain but no TLS binding for it.

**Azure Static Web Apps Free** is the better fit for this site: free hosting, 2 custom domains, apex domain support, and free auto-renewing TLS certificates. It deploys from GitHub on every push.

### Deploy to Static Web Apps

1. Create a GitHub repo (for example `duganit.com`) and push the contents of this folder to the repo root.
2. In the Azure portal, create a **Static Web App**:
   - Plan type: **Free**
   - Source: **GitHub**, pick the repo and `main` branch
   - Build preset: **Custom**
   - App location: `/`
   - Api location: leave blank
   - Output location: leave blank
3. Azure commits a GitHub Actions workflow to `.github/workflows/`. Pull that commit locally so you don't overwrite it. Every push to `main` now redeploys the site.
4. Check the generated `*.azurestaticapps.net` URL to confirm it's live.

### Point duganit.com at it

In the Static Web App, open **Custom domains**.

**www.duganit.com**
1. Add `www.duganit.com`, choose CNAME validation.
2. At your DNS host, create `CNAME www -> <your-app>.azurestaticapps.net`.

**duganit.com (apex)**
1. Add `duganit.com`, choose **TXT** validation, and create the TXT record Azure gives you.
2. Point the apex at the app. Apex records can't be a plain CNAME, so either:
   - use your DNS host's **ALIAS / ANAME / CNAME flattening** record pointing to `<your-app>.azurestaticapps.net` (Cloudflare and some registrars support this), or
   - move the zone to **Azure DNS** and create an **alias record** targeting the Static Web App resource. Azure DNS has a small monthly charge per zone.
3. Once both domains show as ready, select the one you want as primary and choose **Set default**. Visitors on the other one get redirected.

The TLS certificate is issued and renewed automatically after validation.

**Heads up:** duganit.com already carries your Microsoft 365 DNS records (MX, autodiscover, SPF and verification TXT records). Only add the new records above. Leave the existing ones alone, and if you move the zone to Azure DNS, copy every existing record over before switching nameservers.

### If you'd still rather use App Service (good AZ-104 practice)

1. Create a Web App: Publish **Code**, Runtime **any** (static files are served as-is on a Windows plan), Plan **Free F1**.
2. Zip the folder contents and deploy:
   ```powershell
   Compress-Archive -Path .\* -DestinationPath ..\site.zip -Force
   az webapp deploy --resource-group <rg> --name <app> --src-path ..\site.zip --type zip
   ```
3. `staticwebapp.config.json` is ignored on App Service. The security headers would need a `web.config` instead.
4. Scale up to B1 when you're ready to attach duganit.com.
