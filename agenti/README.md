# Agenții din Funnel-ul de vânzări B2B

Agenții **nu sunt fișiere separate** (de tip `.claude/agents/`) — rulează în producție ca **noduri în flow-ul n8n** `workflow/sales-funnel.json`. n8n e orchestratorul (ordine, ramificații, HubSpot, fișiere), iar Claude e chemat prin API doar la pașii care cer judecată sau text. Documentele din acest folder descriu fiecare agent: rol, intrări, ieșiri, model și promptul real extras din flow.

```
companii-test.csv
   │
   ▼
[1] prospector-agent ──► [2] icp-qualifier-agent (Claude Haiku)
                                   │
                                   ▼
                        [3] hubspot-sync-agent (Company + Deal)
                                   │  (doar firmele calificate)
                                   ▼
                        [4] contact-finder-agent (Firecrawl + Claude Haiku)
                                   │
                                   ▼
                        [5] offer-writer-agent (Claude Sonnet)
                                   │
                                   ▼
                  oferte-de-aprobat/*.md  ──► aprobare manuală (Adelina)
```

| # | Agent | Folosește Claude? | Model | Noduri n8n |
|---|-------|-------------------|-------|------------|
| 1 | [prospector-agent](prospector-agent.md) | Nu (CSV + reguli) | — | 0–3 |
| 2 | [icp-qualifier-agent](icp-qualifier-agent.md) | Da | Haiku 4.5 | 4–6 |
| 3 | [hubspot-sync-agent](hubspot-sync-agent.md) | Nu (API HubSpot) | — | 7–17, 23–26, 32–33 |
| 4 | [contact-finder-agent](contact-finder-agent.md) | Da | Haiku 4.5 | 18–22 |
| 5 | [offer-writer-agent](offer-writer-agent.md) | Da | Sonnet 5.5 | 28–31 |

**De ce modele diferite:** Haiku (ieftin, rapid) pentru clasificare și extragere, unde răspunsul e un JSON mic; Sonnet doar pentru scrierea ofertei, unde contează calitatea textului. `temperature: 0` la Haiku, ca scorul să fie stabil între rulări.

**Principii comune:** nu se inventează date (contact negăsit = „negăsit"), fără trimitere de email fără aprobare umană, deduplicare în HubSpot la fiecare rulare.

Planul inițial al agenților: `../output/agenti-automatizare-funnel-vanzari-b2b/plan-agenti.md`.
