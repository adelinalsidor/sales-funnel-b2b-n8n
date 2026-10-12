# TODO — Funnel de vânzări (MVP), Ziua 1 (2026-10-05)

> Plan complet al agenților: `output\agenti-automatizare-funnel-vanzari-b2b\plan-agenti.md`. Aici e doar lista de lucru a zilei.
> La finalul sesiunii: bifează, notează în `README.ro.md` ce a mers și ce a fost greu.

**Starea de azi:** `prospector → icp-qualifier (Haiku, scor 1-10) → hubspot-sync` rulează pe cele 8 firme din `companii-test.csv`.
**Scop MVP:** de la o listă de firme la „ofertă scrisă, aprobată de Adelina". Fără trimitere reală de email.

- [x] 1. Deduplicare în HubSpot: caută firma înainte să o creeze (acum re-rularea dublează înregistrările)
- [x] 2. Filtru în prospector: ignoră pagini-director și liste „Top X"
- [x] 3. `contact-finder-agent`: găsește persoana de contact potrivită per firmă (Firecrawl) și o creează ca Contact în HubSpot
- [x] 4. `offer-writer-agent`: scrie oferta personalizată pe baza scorului ICP și a datelor firmei
- [x] 5. Aprobare manuală prin fișier: câte un fișier per firmă în `oferte-de-aprobat\`, cu câmpul `status: în așteptare / aprobat / respins` în capul fișierului
- [x] 6. Rulare completă pe cele 8 firme, fără dubluri în HubSpot
- [x] 7. `git init`, `.gitignore` (verifică că `.env` e exclus înainte de orice commit), README cu schema fluxului și rezultate
- [x] 8. Jurnal în `README.ro.md`: ce a mers, ce a fost greu, ce rămâne

**Bonus făcut tot în Ziua 1:** Deal în HubSpot, pipeline „Funnel B2B" (Nou → Calificat → Draft email → Verificare și trimitere), 8 Deal-uri fără dubluri.

---

## De făcut de mine (manual)

- [ ] Citesc cele 6 oferte din `oferte-de-aprobat\` și setez `status` (aprobat / respins)
- [ ] Completez prețul în loc de `[PREȚ DE COMPLETAT]`
- [ ] Mut ofertele aprobate în alt folder (o rulare nouă a flow-ului rescrie fișierele și resetează `status`)

## Următorii pași (nu blochează portofoliul)

**Pasul următor recomandat — trimitere reală de email, cu aprobare:**
- [ ] 9. Test pe adresa mea: flow-ul citește fișierele cu `status: aprobat` și trimite emailul (Gmail) doar către mine
- [ ] 10. Reguli clare înainte de trimiteri reale: cine aprobă, ce se trimite, limită de emailuri pe zi
- [ ] 11. După trimitere, Deal-ul se mută în etapa „Verificare și trimitere"; fișierul își schimbă `status` în „trimis" (ca să nu plece de două ori)
- [ ] 12. Protecție la rulări repetate: nu rescrie fișierele deja aprobate

**Mai târziu:**
- [ ] 13. `lead-responder-agent`: clasifică lead-urile primite (spam / în afara targetului / calificat) și pregătește răspunsul, cu aprobare
- [ ] 14. Contact-finder prin LinkedIn (acum 4 din 6 contacte ies „negăsit"; LinkedIn are protecție anti-scraping)
- [ ] 15. Generare PDF de ofertă din template (prevăzut în plan, încă nefăcut)

**Atenție:** `.env` conține cheile HubSpot și Firecrawl. Nu intră în git și nu se afișează în chat.
