# Funnel de vânzări B2B automatizat (MVP)

De la o listă de firme la **ofertă personalizată, scrisă și gata de aprobat**, cu CRM actualizat pe parcurs. Construit cu **n8n** (orchestrare), **Claude** (judecată și scriere), **Firecrawl** (citirea site-urilor) și **HubSpot** (CRM).

> **Stadiu:** MVP funcțional. Demo-ul din acest repo rulează pe **firme și site-uri fictive** (`companii-demo.csv`, `demo-sites/`). **Nu trimite niciun email**: fiecare ofertă așteaptă aprobarea unui om.

Cei cinci agenți (prospector, icp-qualifier, hubspot-sync, contact-finder, offer-writer), cu rolul și promptul fiecăruia, sunt descriși în [`agenti/`](agenti/README.md). Ei rulează ca noduri în n8n, nu ca fișiere separate.

## Schema fluxului

```
CSV cu firme
   │
   ▼
Filtru pagini-director ──► aruncă listicle-uri ("Top 10 firme...")
   │
   ▼
Claude Haiku ──► scor ICP 1-10 + etichetă (calificat ≥ 6) + motiv
   │
   ▼
HubSpot: caută firma după domeniu ──► există? PATCH : POST   (fără dubluri)
   │
   ▼
HubSpot: Deal în pipeline-ul „Funnel B2B", etapa NOU (căutat după nume, fără dubluri)
   │
   ▼
Este calificat?
   ├─ nu ──► gata (Deal-ul rămâne în „Nou", cu scorul în firmă)
   └─ da ──► Deal → etapa CALIFICAT
       ▼
   Firecrawl: pagini de contact / echipă ale firmei
       ▼
   Claude Haiku: extrage persoana de decizie (nu inventează; altfel „negăsit")
       ▼
   Are email? ── da ──► HubSpot: upsert Contact (după email) + asociere cu Company
       │ nu ─────────────────────────────┐
       ▼                                 ▼
                 Unifică
                    ▼
   Claude Sonnet: email + pain-points + ofertă (fără prețuri/cifre inventate)
                    ▼
   oferte-de-aprobat/<firma>.md   (status: în așteptare / aprobat / respins)
                    ▼
   Deal → etapa DRAFT EMAIL
                    ▼
   (pasul următor) verificare + trimitere ──► Deal → etapa VERIFICARE ȘI TRIMITERE
```

**Pipeline HubSpot „Funnel B2B":** Nou → Calificat → Draft email → Verificare și trimitere. Ultima etapă e pregătită, dar încă nu se mută automat nimic în ea; se leagă de trimiterea reală de email.

Fluxul exportat e în [`workflow/sales-funnel.json`](workflow/sales-funnel.json) (fără chei; credențialele stau în n8n).

## Decizii de proiectare

- **Idempotent:** rulat de oricâte ori, nu creează dubluri. Firmele se caută după domeniu, contactele după email.
- **Nu inventează:** dacă pe site nu există un nume sau un detaliu, rezultatul spune explicit „negăsit" / „DETALII INSUFICIENTE".
- **Model potrivit pentru sarcină:** Haiku (ieftin) pentru scor și extragere, Sonnet pentru scrierea ofertei. `temperature: 0` la scor, ca rezultatul să fie repetabil.
- **Om în buclă:** oferta pleacă doar după ce `status` din fișier e schimbat manual în `aprobat`.
- **Rezistent la date murdare:** apelurile externe reîncearcă o dată, iar o eroare pe o firmă nu oprește restul lotului.

## Rezultate pe un set de test de 8 firme (rulare reală, numele omise)

| Pas | Rezultat |
|---|---|
| Calificare ICP | 6 calificate, 2 necalificate (stabil pe rulări repetate) |
| HubSpot | 8 firme și 8 Deal-uri, **fără dubluri** după mai multe rulări |
| Deal-uri | 6 în „Draft email", 2 în „Nou" (necalificate) |
| Contact găsit | 1 cu email, 1 fără email, 4 „negăsit" (fără invenții) |
| Oferte | 6 draft-uri, toate cu detalii concrete de pe site-ul firmei |

## Cum se rulează

1. Creează `.env` cu `HUBSPOT_TOKEN`, `FIRECRAWL_API_KEY`, `ANTHROPIC_API_KEY`, `N8N_API_KEY` (nu intră pe git).
2. Pornește n8n cu acces la folderul proiectului:
   ```powershell
   $env:NODE_OPTIONS = "--dns-result-order=ipv4first"
   $env:N8N_RESTRICT_FILE_ACCESS_TO = "C:\Users\<tu>\.n8n-files;<calea către acest folder>"
   n8n start
   ```
3. Importă `workflow/sales-funnel.json`, leagă credențialele (Anthropic, HubSpot, Firecrawl). Pune în `.n8n-files` lista de firme dorită (pentru demo: `companii-demo.csv`, cu site-urile fictive din `demo-sites/` publicate pe Vercel), setează-i numele în nodul `Read/Write Files from Disk` și apasă **Execute workflow**.
4. Deschide fișierele din `oferte-de-aprobat/`, citește-le și setează `status`.

## Limitări cunoscute

- O nouă rulare **rescrie** fișierele din `oferte-de-aprobat/` și resetează `status`. Mută ofertele aprobate în altă parte.
- Găsirea contactului depinde de ce e public pe site; LinkedIn nu e acoperit (protecție anti-scraping).
- Prețul din ofertă e un placeholder (`[PREȚ DE COMPLETAT]`), completat manual.

## Următorii pași

- Trimitere reală de email, cu aprobare (mută Deal-ul în „Verificare și trimitere")
- `lead-responder-agent` (răspuns la lead-uri primite)
- Contact-finder pentru LinkedIn
