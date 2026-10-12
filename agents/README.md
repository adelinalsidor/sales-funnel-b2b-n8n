# Agents in the B2B Sales Funnel

The agents are **not separate files** (such as `.claude/agents/`). In production they run as **nodes in the n8n flow** `workflow/sales-funnel.json`. n8n is the orchestrator (order, branching, HubSpot, files); Claude is called through the API only at the steps that need judgment or text. Each document in this folder describes one agent: role, inputs, outputs, model, and the real prompt extracted from the flow.

> The prompts are quoted **verbatim, in Romanian**, because the flow targets Romanian SMEs. The node names are also the ones used in n8n.

```
companii-demo.csv
   │
   ▼
[1] prospector-agent ──► [2] icp-qualifier-agent (Claude Haiku)
                                   │
                                   ▼
                        [3] hubspot-sync-agent (Company + Deal)
                                   │  (qualified companies only)
                                   ▼
                        [4] contact-finder-agent (Firecrawl + Claude Haiku)
                                   │
                                   ▼
                        [5] offer-writer-agent (Claude Sonnet)
                                   │
                                   ▼
                  oferte-de-aprobat/*.md  ──► manual approval
```

| # | Agent | Uses Claude? | Model | Nodes (numbers from the diagram in `docs/schema-flux.html`) |
|---|-------|--------------|-------|---------------------------------------------------------------|
| 1 | [prospector-agent](prospector-agent.md) | No (CSV + rules) | — | 1–4 |
| 2 | [icp-qualifier-agent](icp-qualifier-agent.md) | Yes | Haiku 4.5 | 5–7 |
| 3 | [hubspot-sync-agent](hubspot-sync-agent.md) | No (HubSpot API) | — | 8–18, 32–35, 41–42 |
| 4 | [contact-finder-agent](contact-finder-agent.md) | Yes | Haiku 4.5 | 19–30 |
| 5 | [offer-writer-agent](offer-writer-agent.md) | Yes | Sonnet 5.5 | 37–40 |

**Why different models:** Haiku (cheap, fast) for classification and extraction, where the answer is a small JSON; Sonnet only for writing the offer, where text quality matters. `temperature: 0` on Haiku keeps the score stable between runs.

**Common principles:** never invent data (contact not found = "negasit"), never send an email without human approval, deduplicate in HubSpot on every run.