# hubspot-sync-agent

**Role:** keeps the HubSpot CRM up to date: Company, Contact and Deal, **with no duplicates** on repeated runs.

**Type:** HubSpot API calls orchestrated by n8n, no Claude.

**What it creates / updates:**
- **Company** — looked up by `domain`; if it exists a `PATCH` is sent, otherwise a `POST` (one node with a conditional method and URL).
- **Deal** — in the "Funnel B2B" pipeline, looked up by `dealname` = `<domain> — Funnel B2B`; on update its stage is left untouched.
- **Contact** — upsert by email, then associated with the Company.

**Deal stages:** New → Qualified (if the company is qualified) → Draft email (once the offer file is written) → Review and send (not wired yet; it comes with real email sending).

**n8n nodes:** the Company and Deal nodes, `Este calificat`, `Muta Deal in Calificat`, the contact upsert and association nodes, and `Prepara mutare Draft` / `Muta Deal in Draft email`.

**Verified:** repeated runs leave each domain as a single company and a single deal.

**Lessons learned:**
- The HubSpot token needs the `crm.schemas.deals.write` scope to create the pipeline; scopes are edited under Development → Legacy apps and must also be saved with the button in the top right.
- HubSpot rejects emails ending in `.example` (`INVALID_EMAIL`), and it auto-creates a company from the email domain of a new contact. Demo emails therefore use the same domain as the company's site.
