# Plan Agenți — Automatizare Funnel de Vânzări B2B

## Rezumat

- Echipă de agenți AI care prospectează companii B2B potrivite, le califică, identifică persoana de contact, generează ofertă personalizată (email + PDF) și răspunde automat la lead-uri primite.
- Arhitectură hibridă: **n8n** este „sistemul nervos" (triggere, orchestrare, scriere/citire CRM), **agenții Claude** sunt „creierul" chemat doar la pașii care necesită judecată sau conținut.
- Scop dublu: sistemul e testat inițial pe nișa „servicii AI/automatizări pentru IMM-uri" (ca produs de portofoliu + potențial produs vandabil), dar rămâne reutilizabil pentru orice altă aplicație viitoare.
- Toate trimiterile efective (email de outreach, răspuns la lead) trec printr-o aprobare umană înainte de a pleca.
- Faza curentă: testare pe un set fix de ~8 companii reale (dry-run, nimic nu se trimite), generat o singură dată și refolosit prin toate etapele ulterioare de dezvoltare.

## Context

**Brief original:**
> Arhitectură de agenți AI pentru automatizarea unui funnel de vânzări B2B. Scop dublu: piesă de portofoliu ȘI produs/serviciu real vandabil. Echipa caută clienții potriviți, alimentează un CRM, pregătește pentru fiecare o ofertă personalizată gata de trimis, și răspunde automat la lead-uri.

**Surse de date:**
- Web search/scraping (Firecrawl) — identificare companii via website public
- LinkedIn/surse publice — doar pentru companiile deja calificate, ca să identificăm persoana de decizie (nume + rol + email direct)
- Email inbox (contul existent al Adelinei) și formular website — pentru lead-uri primite

**Output-uri / destinații:**
- HubSpot CRM — obiecte Company + Contact + Deal, actualizate pe măsură ce prospectul avansează
- Email personalizat de outreach (trimis din contul existent, după aprobare)
- Document PDF de ofertă (generat automat, din template, cu conținut personalizat de Claude)
- Răspuns automat la lead-uri (email), cu aprobare umană pentru lead-urile calificate

**Frecvență:** hibrid — prospectare pornită manual SAU pe cron (configurabil); răspuns la lead-uri este event-driven (declanșat de email/formular nou)
**Volum estimat:** batch de prospectare — volum exact TBD (decis ulterior, după ce se vede costul/timpul per prospect); pentru testare recomand un set fix de **8 companii reale**, generat o singură dată și refolosit prin toate etapele — suficient să validezi calitatea fiecărui pas fără cost/timp mare de iterare, dar destul cât să prindă variații reale (industrii diferite, site-uri de calitate diferită).

## Out of Scope

Sistemul **NU** va face, în această versiune (v1/MVP):
- Cold-calling sau SMS — doar canale scrise (email, formular)
- Negociere automată de preț în conversație — oferta pleacă cu preț/template standard
- Semnare de contracte sau procesare de plăți
- Construirea landing page-ului/formularului de website — decizie amânată; dacă nu există deja un site cu formular, se decide separat dacă intră în scope
- Dashboard multi-client — vezi secțiunea Roadmap

## Subagenți Claude Code existenți de folosit

Aceștia sunt utili **doar în faza de dezvoltare/testare** a sistemului, nu în arhitectura de producție (în producție, agenții rulează ca apeluri Claude API din noduri n8n, nu ca sesiuni Claude Code):

- **Explore** — research rapid în documentația HubSpot API / Firecrawl MCP înainte de a scrie integrările
- **general-purpose** — construirea și testarea inițială a promptului fiecărui agent, izolat, înainte de a-l cabla în fluxul n8n

## Agenți custom de creat

### Agent: prospector-agent

- **Rol:** caută pe web companii care se potrivesc unui profil general (industrie, arie geografică, dimensiune) și extrage date de bază (nume, website, descriere scurtă)
- **Tools:** MCP Firecrawl (search, scrape, map)
- **API/MCP:** Firecrawl — scope: search + scrape pagini publice; endpoint-uri cheie: `firecrawl_search`, `firecrawl_scrape`, `firecrawl_map`; auth: API key
- **Triggers:** pornit manual sau de cron n8n (configurabil), primește parametri de căutare (industrie, arie, nr. companii dorit)
- **Acceptance:**
  - Returnează listă de companii cu nume + website + descriere scurtă
  - Nu include duplicate față de companiile deja existente în HubSpot
  - La eșec sursă: retry automat 1x, apoi skip + log eroare
- **Complexitate:** M — logică de căutare + deduplicare + retry

### Agent: icp-qualifier-agent

- **Rol:** analizează fiecare companie găsită și calculează un scor de potrivire (ICP) pe baza unei combinații de factori — mărime, industrie, semnale de pe website (ex. lipsă chatbot/automatizări vizibile)
- **Tools:** Firecrawl (scrape semnale de pe website), Claude reasoning
- **API/MCP:** Firecrawl — scope: citire pagini publice
- **Triggers:** apelat automat de n8n imediat după ce prospector-agent returnează o listă
- **Acceptance:**
  - Fiecare companie primește scor + etichetă (calificat/necalificat) + motivare scurtă
  - Pragul de calificare este configurabil (nu hardcodat)
- **Complexitate:** M — necesită un prompt de scoring calibrat și testat pe eșantion real

### Agent: contact-finder-agent

- **Rol:** pentru companiile deja calificate, identifică persoana de decizie — nume, rol, email direct — via LinkedIn/surse publice
- **Tools:** Firecrawl (search + scrape), eventual MCP LinkedIn dedicat dacă devine disponibil
- **API/MCP:** Firecrawl — scope: căutare/citire pagini publice; auth: API key
- **Triggers:** apelat **doar** pentru companiile marcate „calificat" de icp-qualifier-agent (nu pentru toate — economisește cost)
- **Acceptance:**
  - Returnează nume + rol + email direct, dacă e găsibil public
  - Dacă nu găsește date, raportează explicit „contact negăsit" (nu inventează)
- **Complexitate:** M/L — extragerea de date LinkedIn e greu de făcut consistent; posibile limitări de acces

### Agent: offer-writer-agent

- **Rol:** analizează website-ul companiei calificate, identifică 1-3 pain-points concrete, scrie emailul personalizat de outreach + conținutul pentru documentul PDF de ofertă
- **Tools:** Firecrawl (scrape website companie), Claude API (model de calitate mare pentru scriere)
- **API/MCP:** Firecrawl — scope: scrape website; Claude API — model de top pentru generare conținut
- **Triggers:** apelat după ce contact-finder-agent confirmă persoana de contact
- **Acceptance:**
  - Emailul conține minim un detaliu concret real despre compania țintă (nu generic de tip „am observat compania dvs.")
  - Conținutul ofertei respectă structura template-ului PDF
  - Rezultatul e marcat explicit „draft — neaprobat" până la revizuire umană
- **Complexitate:** L — calitatea personalizării e factorul critic pentru rata de succes a funnel-ului

### Agent: hubspot-sync-agent

- **Rol:** creează/actualizează Company + Contact + Deal în HubSpot pentru fiecare prospect, la fiecare pas relevant al fluxului
- **Tools:** MCP HubSpot (sau REST API HubSpot direct)
- **API/MCP:** HubSpot API — scope: `crm.objects.companies.write`, `crm.objects.contacts.write`, `crm.objects.deals.write`; auth: Private App token (OAuth)
- **Triggers:** apelat la creare inițială, după calificare, după trimitere ofertă, după răspuns lead
- **Acceptance:**
  - Niciun Company/Contact duplicat pentru aceeași companie (verificare după domeniu website)
  - Deal-ul reflectă corect stage-ul curent (Nou → Calificat → Ofertă trimisă → În discuție)
- **Complexitate:** S/M — API HubSpot e bine documentat; principala grijă e deduplicarea

### Agent: lead-responder-agent

- **Rol:** clasifică fiecare lead nou primit (spam / în afara targetului / calificat) și pregătește răspunsul potrivit
- **Tools:** Claude reasoning, citire date existente din HubSpot pentru context (dacă lead-ul are istoric)
- **API/MCP:** HubSpot API — scope: read contacts/deals
- **Triggers:** declanșat de n8n la eveniment (email nou primit sau formular completat)
- **Acceptance:**
  - **Spam:** zero răspuns trimis, doar log — criteriile de detectare spam sunt documentate explicit și ajustabile
  - **În afara targetului:** răspuns politicos generic trimis + marcaj „atenție umană" în HubSpot
  - **Calificat:** răspuns personalizat generat, trimis spre aprobare, cu SLA de răspuns în aceeași zi lucrătoare
- **Complexitate:** M — logica de clasificare trebuie testată pe cazuri variate ca să nu respingă lead-uri bune

## Orchestrare

**Pattern:** orchestrator-workers, cu n8n ca orchestrator (triggere + control flow) și fiecare agent Claude ca „worker" chemat prin API la pasul potrivit.

**Flow — Prospectare → Ofertă:**

```
[Trigger: manual sau cron n8n]
        │
        ▼
[prospector-agent] → listă companii (nume, website, descriere)
        │
        ▼
[icp-qualifier-agent] → scor + etichetă
        │
   ┌────┴────┐
   │         │
necalificat  calificat
   │         │
[log, skip]  ▼
        [contact-finder-agent] → nume + rol + email decision maker
             │
             ▼ (dacă negăsit → flag „contact negăsit", skip trimitere)
        [hubspot-sync-agent] → creează Company + Contact + Deal
             │
             ▼
        [offer-writer-agent] → email personalizat + conținut ofertă
             │
             ▼
        [n8n: generare PDF din template]
             │
             ▼
        [n8n: notificare aprobare → Adelina]
             │
        ┌────┴────┐
        │         │
     respins   aprobat
        │         │
   [revizuire]  [trimite email + PDF, prin contul Gmail]
                     │
                     ▼
              [hubspot-sync-agent] → update Deal: „Ofertă trimisă"
```

**Flow — Răspuns la lead-uri (event-driven):**

```
[Trigger: email nou SAU formular completat]
        │
        ▼
[lead-responder-agent: clasificare]
        │
   ┌────┼──────────────┐
   │    │              │
 spam  în afara       calificat
   │   targetului        │
   │    │              │
[log] [răspuns generic] [răspuns personalizat]
      [flag CRM]           │
                            ▼
                    [n8n: notificare aprobare]
                            │
                         aprobat
                            │
                            ▼
                    [trimite răspuns, SLA aceeași zi]
                            │
                            ▼
                    [hubspot-sync-agent: update status]
```

**Dependențe:**
- `prospector-agent` → `icp-qualifier-agent`: primește lista brută de companii
- `icp-qualifier-agent` → `contact-finder-agent`: trimite doar companiile calificate (economie de cost pe căutarea LinkedIn)
- `contact-finder-agent` → `hubspot-sync-agent` → `offer-writer-agent`: contactul confirmat e scris în CRM înainte de generarea ofertei
- `lead-responder-agent` → `hubspot-sync-agent`: orice clasificare de lead actualizează statusul în CRM, inclusiv flag-urile de atenție umană

## Decizii umane in-the-loop

- **Înainte de trimiterea oricărui email/ofertă către un prospect real:** sistemul se oprește, Adelina vede draftul complet (email + PDF) și aprobă/respinge/editează. Recomand notificare **Slack cu link de aprobare** (nod „wait for webhook" în n8n) ca variantă MVP — mai rapidă decât să deschidă HubSpot de fiecare dată; varianta „draft direct în HubSpot" rămâne o alternativă bună dacă preferă totul într-un singur loc mai târziu.
- **Lead în afara targetului:** primește răspuns generic automat, dar e marcat explicit pentru atenție umană în HubSpot — nu se pierde, dar nu blochează fluxul.
- **Lead marcat spam:** nu primește niciun răspuns automat; criteriile care îl clasifică drept spam trebuie documentate clar și pot fi ajustate.
- **Contact negăsit pentru o companie calificată:** flag pentru revizuire manuală, în loc să fie sărit silențios — ca să nu se piardă prospecți buni din cauza unei căutări eșuate.
- **Erori tehnice (Firecrawl fără rezultat, date lipsă):** retry automat o dată, apoi skip + log — nu blochează restul batch-ului.

## Skills Claude Code relevante

- **brainstorming** — util pentru rafinări viitoare de scope pe măsură ce sistemul evoluează
- **claude-api** — referință pentru alegerea modelului potrivit (cost vs. calitate) când se integrează Claude API în nodurile n8n — relevant direct pentru decizia „echilibrat": model mai ieftin pentru clasificare/scoring (icp-qualifier, lead-responder), model de top doar pentru scriere efectivă (offer-writer)

## Next Steps recomandate

1. Configurează HubSpot (Private App token) și testează manual crearea unui Company + Contact + Deal, ca să confirmi scope-ul de permisiuni
2. Configurează Firecrawl API key și testează un search + scrape pe 2-3 companii reale, ca reper de calitate a datelor
3. Generează setul de test: **8 companii reale**, salvate o singură dată (dry-run — nimic nu se trimite), refolosite prin toate etapele următoare de dezvoltare
4. Construiește primul flow n8n minimal: `prospector-agent` → `icp-qualifier-agent` → `hubspot-sync-agent` (fără ofertă, fără trimitere) și validează-l pe cele 8 companii
5. Adaugă `contact-finder-agent` + `offer-writer-agent` + nodul de aprobare (Slack) înainte de a conecta trimiterea efectivă de email

## Progres — unde ne-am oprit (actualizat 2026-09-29)

**Pașii 1, 2 și 3 din Next Steps sunt finalizați.**

- **Pasul 1 (HubSpot):** token-ul Private App e configurat (`HUBSPOT_TOKEN` în `.env`, local, nu pe git).
- **Pasul 2 (Firecrawl):** cheia API refolosită de la contul existent din curs, salvată în `.env`. Testată cu succes pe `scrape` (o pagină) și `search` (căutare pe internet) — vezi detalii și observația despre calitatea inconsecventă a datelor în `README.ro.md`.
- **Pasul 3 (set de test):** generat setul de **8 companii reale**, dry-run, salvat în `companii-test.csv` (rădăcina proiectului), acoperind 4 industrii diferite (avocatură, contabilitate, stomatologie, imobiliare). Notă: din cele 9 rezultate brute ale căutării inițiale (3 industrii), 4 erau pagini-director/listicle, nu companii individuale — au fost filtrate manual, apoi completate cu o a 4-a industrie de căutare.

**Pasul 4 e finalizat.** Primul flow n8n minimal funcționează end-to-end: `prospector-agent` (CSV) → `icp-qualifier-agent` (Claude Haiku, scoring 1-10) → `hubspot-sync-agent` (creare Company în HubSpot). Rulat cu succes pe toate cele 8 companii din `companii-test.csv` — flow-ul se numește **„SALES FUNNEL"** în n8n local. Rezultat: 6 calificate, 2 necalificate — scoruri și motivații corecte, vizibile în câmpul „description" al fiecărei companii din HubSpot.

**Lecție de reținut pentru `prospector-agent`:** logica de deduplicare/filtrare (acceptance criteria de mai sus) trebuie extinsă să excludă explicit paginile-director și listicle-urile ("Top X firme...", pagini de recenzii), nu doar duplicate față de HubSpot — altfel intră zgomot în pipeline direct de la sursă.

**Trei probleme reale întâlnite și rezolvate la construcția flow-ului (utile pentru orice flow n8n viitor):**
1. **BOM invizibil în CSV** — fișierele `.csv` exportate din PowerShell au un marcaj invizibil la început care strică parsarea CSV în n8n. Fix: rescriere fără BOM (`UTF8Encoding($false)`).
2. **IPv6 rupt pe rețeaua locală** — n8n prefera IPv6 la conectare și rămânea blocat (timeout), deși IPv4 funcționa normal. Fix: pornit n8n cu `NODE_OPTIONS=--dns-result-order=ipv4first`.
3. **Domeniu invalid pentru HubSpot** — proprietatea „domain" din HubSpot vrea doar domeniul (`exemplu-firma.ro`), nu un URL cu cale (`exemplu-firma.ro/servicii-contabilitate`). Fix: extragere cu `new URL(...).hostname` în loc de tăiere manuală de text.

**⚠️ De reținut pentru rulări viitoare ale acestui flow:** nodul HubSpot nu verifică duplicate — o rulare repetată creează companii duplicate. Înainte de re-testare, șterge manual companiile de test din HubSpot (sau adaugă un pas de căutare-înainte-de-creare, ca îmbunătățire viitoare — vezi acceptance criteria `hubspot-sync-agent`).

**La reluare, pornim de aici — Pasul 5 din Next Steps:**
Adaugă `contact-finder-agent` + `offer-writer-agent` + nodul de aprobare (Slack) înainte de a conecta trimiterea efectivă de email.

## Roadmap — idei notate pentru viitor (fără prioritizare acum)

- **Agent de negociere:** ar răspunde la obiecții/întrebări de preț în numele Adelinei, tot cu aprobare umană la început — necesită mai întâi ca MVP-ul de bază (prospectare → ofertă → răspuns lead) să funcționeze stabil
- **Dashboard pentru clienți:** interfață unde un client care cumpără serviciul vede rapoarte/log-uri despre ce s-a făcut în numele lui — relevant dacă funnel-ul devine produs vândut altor companii (multi-tenant); momentan log-urile sunt vizibile doar pentru Adelina
- **Landing page + formular:** dacă nu există deja un website cu formular de contact, intră în discuție separat
- **Migrare canal de trimitere:** dacă volumul crește semnificativ, de luat în calcul un email dedicat de business (în loc de contul personal) și/sau un tool de cold-email dedicat (ex. Instantly/Smartlead) pentru deliverability mai bună la volum mare
