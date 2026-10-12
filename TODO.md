# TODO — B2B Sales Funnel

> Agent descriptions: `agents/`. Visual diagram: `docs/schema-flux.html`. Original agent plan (Romanian): `output/agenti-automatizare-funnel-vanzari-b2b/plan-agenti.md`.

**MVP goal:** from a list of companies to an "offer written and approved by a human", with the CRM updated in HubSpot. No real email sending.

## Done

- [x] Deduplication in HubSpot (company by domain, Deal by name, contact by email)
- [x] Filter for directory pages and "Top X" lists
- [x] `contact-finder-agent` (Firecrawl + Claude Haiku) and Contact in HubSpot, with discovery that works on sites search engines don't know (site map, homepage links, direct page reads, paced requests, cache bypass)
- [x] `offer-writer-agent` (Claude Sonnet) and manual approval through files in `oferte-de-aprobat/`
- [x] Deals in the "Funnel B2B" pipeline (New, Qualified, Draft email, Review and send)
- [x] Agents documented in `agents/` (role, inputs, outputs, real prompt from the flow)
- [x] Demo with fictional companies: `companii-demo.csv` and 6 mock sites on Vercel (`demo-sites/`)
- [x] Repository cleaned of real company data
- [x] Visual diagram `docs/schema-flux.html` (42 nodes, 7 steps)
- [x] Clean export of the flow in `workflow/sales-funnel.json` (42 nodes, no keys)

## Left to publish

- [ ] Screenshots on demo data only (n8n flow, HubSpot pipeline, a generated offer) in `docs/img/`
- [ ] Public GitHub repository (MIT license already added)

## Roadmap

**Real email sending, with approval:**
- [ ] Test on my own address: the flow reads files with `status: aprobat` and sends the email (Gmail) only to me
- [ ] Clear rules before real sending: who approves, what is sent, a daily email limit
- [ ] After sending, the Deal moves to "Review and send"; the file's `status` becomes "sent" (so nothing goes out twice)
- [ ] Protection on repeated runs: do not overwrite files that are already approved

**Later:**
- [ ] `lead-responder-agent`: classify incoming leads (spam / off-target / qualified) and prepare the reply, with approval
- [ ] Contact finder through LinkedIn (LinkedIn has anti-scraping protection)
- [ ] PDF offer generated from a template
- [ ] Surface HubSpot errors: today, when a request is rejected, the flow continues without a signal (that is how contacts went unsaved without any visible error)
- [ ] Stricter scoring prompt for companies with a very poor description (a workshop described only as "Atelier." came out qualified with 7)
