# prospector-agent

**Role:** supplies the list of target companies and removes pages that are not real companies.

**Type:** deterministic rules, no Claude. (Automatic web search with Firecrawl was used to build the list; in the MVP the list that runs comes from a file.)

**Input:** `companii-demo.csv` — columns: `Nume`, `Website`, `Descriere`, `Industrie`, `DataAdaugare` (8 rows).

**Output:** the same rows, minus directory pages and listicles ("Top 10 firms...", rankings, reviews).

**n8n nodes:** `Read/Write Files from Disk` → `Extract from File` → `Filtru pagini-director`.

**Filter logic** (code node): rejects a row if the name or URL path matches patterns such as `top 10`, `cele mai bune`, `clasament`, `director`, `lista de`, `recenzii`, `reviews`, or if the website is missing. Every rejection is logged with the company name.

**Lesson learned:** CSV files exported from PowerShell carry an invisible BOM that breaks CSV parsing in n8n, so files are written without a BOM.
