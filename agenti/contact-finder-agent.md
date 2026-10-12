# contact-finder-agent

**Rol:** găsește persoana de decizie (fondator, administrator, director, manager, partener, medic-sef) a fiecărei firme calificate.

**Model:** `claude-haiku-4-5`, `temperature: 0`, `max_tokens: 400`. Paginile sunt aduse cu **Firecrawl** (căutare restrânsă la domeniul firmei, `includeDomains`), iar Claude extrage doar din textul primit (max. 12.000 de caractere).

**Intrare:** firma calificată + textul paginilor de echipă/contact.

**Ieșire:** JSON `{"gasit": bool, "nume", "rol", "email", "telefon", "sursa"}`, apoi `ContactStatus`: `gasit` / `gasit-fara-email` / `negasit`. Doar contactele cu email valid merg mai departe ca Contact în HubSpot.

**Noduri n8n:** `Cauta pagini contact Firecrawl` → `Pregateste prompt contact` → `Extrage contact Claude` → `Interpreteaza contact` → `Are email`.

**Prompt (exact din flow):**

```
Din textul paginilor de mai jos ale firmei "${Nume}", extrage persoana de decizie cea mai potrivita (fondator, administrator, director, manager, partener, medic-sef). Raspunde STRICT cu un JSON valid, fara alt text, de forma exacta: {"gasit": true sau false, "nume": "", "rol": "", "email": "", "telefon": "", "sursa": "url-ul paginii"}.
Reguli: NU inventa nimic. Daca pe pagini nu apare un nume de persoana, pune gasit=false si lasa campurile goale. Pune email doar daca apare explicit in text (poate fi si un email de contact al firmei).

TEXT PAGINI:
${siteText}
```

**Rezultat actual:** 2 contacte găsite din 6 firme calificate (1 cu email). Restul sunt marcate „negăsit" — regula „nu inventa" contează mai mult decât rata de succes.

**Limitare cunoscută:** LinkedIn blochează scraping-ul, deci nu e folosit; rămâne un pas viitor.

**Lecție învățată:** `site:` nu funcționa în căutarea Firecrawl la 2 cabinete de avocatură; parametrul `includeDomains` a rezolvat imediat.
