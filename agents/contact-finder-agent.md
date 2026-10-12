# contact-finder-agent

**Role:** finds the decision maker (founder, administrator, director, manager, partner, head physician) of each qualified company.

**Model:** `claude-haiku-4-5`, `temperature: 0`, `max_tokens: 400`. Pages are fetched with **Firecrawl**; Claude extracts only from the text it receives (max. 12,000 characters).

**Page discovery (works on sites search engines don't know yet):**
1. a Firecrawl search restricted to the company's domain (`includeDomains`);
2. a Firecrawl site map of the domain;
3. the homepage read with all its links (`onlyMainContent: false`, so menu links are included);
4. the contact page and the team page are chosen from those links and read directly (`maxAge: 0`, to bypass the cache);
5. everything is merged without duplicates and handed to Claude.

**Input:** the qualified company + the text of its contact / team pages.

**Output:** JSON `{"gasit": bool, "nume", "rol", "email", "telefon", "sursa"}`, then `ContactStatus`: `gasit` / `gasit-fara-email` / `negasit`. Only contacts with a valid email continue as Contacts in HubSpot.

**n8n nodes:** `Search contact pages (Firecrawl)` → `Prepare site-map request` → `Site map (Firecrawl)` → `Prepare homepage read` → `Read homepage (Firecrawl)` → `Pick contact and team pages` → `Read contact page (Firecrawl)` → `Read team page (Firecrawl)` → `Merge contact pages` → `Build contact prompt` → `Extract contact (Claude)` → `Parse contact` → `Has email?` → `Merge branches` (the contact upsert and association nodes in between belong to hubspot-sync).

**Prompt (verbatim from the flow, in Romanian):**

```
Din textul paginilor de mai jos ale firmei "${Nume}", extrage persoana de decizie cea mai potrivita (fondator, administrator, director, manager, partener, medic-sef). Raspunde STRICT cu un JSON valid, fara alt text, de forma exacta: {"gasit": true sau false, "nume": "", "rol": "", "email": "", "telefon": "", "sursa": "url-ul paginii"}.
Reguli: NU inventa nimic. Daca pe pagini nu apare un nume de persoana, pune gasit=false si lasa campurile goale. Pune email doar daca apare explicit in text (poate fi si un email de contact al firmei).

TEXT PAGINI:
${siteText}
```

**Demo result:** 4 contacts with email, 1 found without email, 1 "negasit". Rule "never invent" matters more than the success rate.

**Known limitation:** LinkedIn blocks scraping, so it is not used; it remains a future step.

**Lessons learned:**
- `site:` did not work in Firecrawl search for some sites; the `includeDomains` parameter fixed it.
- A brand-new site is not in any search index, so search returns nothing; reading the site directly (map + homepage links) solved it.
- Firecrawl returns `429` when requests come too fast, so the reading nodes are paced (one request every 7 seconds) and retry.
- Firecrawl caches page reads for about two days; `maxAge: 0` forces a fresh read.
