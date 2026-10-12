# TODO — Funnel de vânzări B2B

> Plan complet al agenților: `output\agenti-automatizare-funnel-vanzari-b2b\plan-agenti.md`. Descrierea agenților: `agenti\`. Schema vizuală: `docs\schema-flux.html`.

**Scop MVP:** de la o listă de firme la „ofertă scrisă, aprobată de Adelina", cu CRM actualizat în HubSpot. Fără trimitere reală de email.

## Gata

**Ziua 1 (2026-10-05)**
- [x] Deduplicare în HubSpot (firmă după domeniu, Deal după nume, contact după email)
- [x] Filtru pagini-director și liste „Top X"
- [x] `contact-finder-agent` (Firecrawl + Claude Haiku) și Contact în HubSpot
- [x] `offer-writer-agent` (Claude Sonnet) și aprobare manuală prin fișiere în `oferte-de-aprobat\`
- [x] `git init`, `.gitignore`, README, jurnal
- [x] Deal-uri în pipeline-ul „Funnel B2B" (Nou, Calificat, Draft email, Verificare și trimitere)

**Ziua 2 (2026-10-12)**
- [x] Agenții documentați în `agenti\` (rol, intrări, ieșiri, prompt real din flow)
- [x] Demo cu firme fictive: `companii-demo.csv` și 6 site-uri mock pe Vercel (`demo-sites\`)
- [x] Repo curățat de firmele reale (un singur commit nou, fără date reale în istoric)
- [x] Contact-finder reparat pentru site-uri neindexate: harta site-ului, pagina principală cu linkuri, citire directă a paginii de contact și de echipă; ritm lent către Firecrawl (limită `429`)
- [x] Schema vizuală `docs\schema-flux.html` (42 de noduri, 7 pași)

## De terminat ca să declar proiectul „MVP finalizat"

- [ ] Rulare finală pe lista demo, cu contactele salvate în HubSpot (emailurile demo trec la `.example.com`, pentru că HubSpot respinge `.example`)
- [ ] Export curat al flow-ului în `workflow\sales-funnel.json` (42 de noduri, fără chei, fără nume reale)
- [ ] Capturi de ecran doar pe datele demo (flow n8n, pipeline HubSpot filtrat pe `vercel.app`, o ofertă)
- [ ] Commit cu tot ce s-a schimbat
- [ ] Șterg cheia API din n8n (Settings → n8n API), după export
- [ ] Decid: repo public pe GitHub pentru link în CV (acum repo-ul e doar local)
- [ ] Text scurt pentru CV / LinkedIn („MVP funcțional", cu tehnologiile folosite)

**Atenție:** `.env` conține cheile HubSpot, Firecrawl, Anthropic și n8n. Nu intră în git și nu se afișează în chat. Firmele reale din HubSpot nu apar în capturi.

## Ofertele pe firme reale (nu țin de portofoliu)

Ofertele pe firmele reale sunt în `oferte-rezerva-reale\` (ignorat de git).
- [ ] Citesc ofertele și setez `status` (aprobat / respins)
- [ ] Completez prețul în loc de `[PREȚ DE COMPLETAT]`

## Următorii pași (roadmap, nu blochează „MVP finalizat")

**Trimitere reală de email, cu aprobare:**
- [ ] 9. Test pe adresa mea: flow-ul citește fișierele cu `status: aprobat` și trimite emailul (Gmail) doar către mine
- [ ] 10. Reguli clare înainte de trimiteri reale: cine aprobă, ce se trimite, limită de emailuri pe zi
- [ ] 11. După trimitere, Deal-ul trece în „Verificare și trimitere"; fișierul își schimbă `status` în „trimis" (ca să nu plece de două ori)
- [ ] 12. Protecție la rulări repetate: nu rescrie fișierele deja aprobate

**Mai târziu:**
- [ ] 13. `lead-responder-agent`: clasifică lead-urile primite (spam / în afara targetului / calificat) și pregătește răspunsul, cu aprobare
- [ ] 14. Contact-finder prin LinkedIn (LinkedIn are protecție anti-scraping)
- [ ] 15. Generare PDF de ofertă din template
- [ ] 16. Erorile HubSpot nu trebuie să treacă neobservate: acum, la o cerere respinsă, flow-ul continuă fără semnal (așa au rămas contactele nesalvate fără nicio eroare vizibilă)
- [ ] 17. Prompt de scor mai strict pentru firme cu descriere foarte săracă (un atelier cu descrierea „Atelier." a ieșit calificat cu 7)
