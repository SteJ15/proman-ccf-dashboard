# CCF Labour Provider KPI Dashboard

Internal prototype dashboard for the PROMAN ↔ Cranswick Continental Foods labour-provider partnership.

This is the demonstrator version, hosted on GitHub Pages for review and stakeholder feedback. Production hosting will be migrated to PROMAN's own infrastructure once the architecture is finalised.

## What's in this repo

| File | Purpose |
|---|---|
| `index.html` | The dashboard itself. Single self-contained page. Fetches `./data.json` on load |
| `data.json` | Sample data — 19 weeks of fabricated figures with W16 anchored to the real April 2026 submission |
| `DATA_SCHEMA.md` | Schema documentation for whoever builds the Power Automate flow |

## Viewing the dashboard

Once deployed to GitHub Pages, the URL will be something like `https://<your-account>.github.io/<repo-name>/`. The dashboard loads in any modern browser and is responsive down to phone screens.

Use the **Weekly / Monthly / YTD** tabs at the top to switch view, and the dropdown to pick a specific week or month.

## How the data flows in production

```
SharePoint workbook (CCF_KPI_Input.xlsx)
        │
        │  edit
        ▼
Power Automate flow
        │
        │  read → transform → write
        ▼
data.json (hosted alongside index.html)
        │
        │  fetch on page load
        ▼
Dashboard renders
```

The dashboard is **read-only** and **stateless** — it does not modify the source data, and nothing is stored client-side. Each page load fetches a fresh `data.json`.

## Branding

Both logos load from external URLs (PROMAN from `proman-uk.com`, CCF from the Azure preview site). If either URL changes, update the `logoUrl` values in `data.json`.

Brand colours:
- Primary: `#003680` (PROMAN blue)
- Accent: `#F58220` (PROMAN orange)

## Updating the data

For the demonstrator: edit `data.json` directly and push to GitHub. The dashboard will reflect the change next time it's loaded.

For production: see `DATA_SCHEMA.md`. Power Automate will write to `data.json` automatically when the source workbook changes.

## Local preview

The dashboard fetches `./data.json` so it needs to be served over HTTP, not opened directly from the file system. Quick options:

```bash
# Python 3
cd <this folder>
python3 -m http.server 8000
# then open http://localhost:8000

# Node
npx serve .
```

## Notes

- **Tested in**: latest Chrome, Edge, Firefox, Safari (desktop and mobile)
- **Not supported**: Internet Explorer (uses modern JS features)
- **Print-friendly**: there's a print stylesheet that strips the period selector and keeps each panel intact for paper copies
- **Issues raised log**: free-text fields ("N/A" entries are filtered out automatically in monthly/YTD views so the log only shows real content)
