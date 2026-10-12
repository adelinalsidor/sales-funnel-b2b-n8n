# hubspot-sync-agent

**Rol:** ține HubSpot CRM la zi: Company, Contact și Deal, **fără dubluri** la rulări repetate.

**Tip:** apeluri API HubSpot orchestrate de n8n, fără Claude.

**Ce creează / actualizează:**
- **Company** — căutată după `domain`; dacă există se face `PATCH`, altfel `POST` (un singur nod cu metodă și URL condiționate).
- **Deal** — pipeline „Funnel B2B", căutat după `dealname` = `<domeniu> — Funnel B2B`; la actualizare nu i se schimbă etapa.
- **Contact** — upsert după email, apoi asociat cu Company.

**Etapele Deal-ului:** Nou → Calificat (dacă firma e calificată) → Draft email (după ce fișierul ofertei e scris) → Verificare și trimitere (încă nelegată; vine odată cu trimiterea reală de email).

**Noduri n8n:** 7–17 (Company și Deal), 23–26 (Contact și asociere), 32–33 (mutare în Draft email).

**Verificat:** 3 rulări la Company și 2 la Deal, fiecare domeniu apare o singură dată.

**Lecție învățată:** token-ul HubSpot are nevoie de scope-ul `crm.schemas.deals.write` ca să creeze pipeline-ul; scope-urile se editează din Development → Legacy apps și trebuie salvate și cu butonul din dreapta sus.
