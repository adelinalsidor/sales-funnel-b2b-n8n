# icp-qualifier-agent

**Role:** gives each company a fit score (1–10) against the ideal customer profile (ICP) and a label `calificat` (qualified) / `necalificat` (unqualified).

**Model:** `claude-haiku-4-5`, `temperature: 0`, `max_tokens: 300`.

**Input:** `Nume`, `Website`, `Descriere`, `Industrie`.

**Output:** JSON `{"scor": 1-10, "eticheta": "calificat|necalificat", "motiv": "..."}`. Threshold: score ≥ 6 = qualified. The result is stored in the `description` field of the company in HubSpot.

**n8n nodes:** `Build scoring prompt` → `Score ICP (Claude)` (HTTP call to the Anthropic API) → `Parse score, prepare HubSpot data`.

**Prompt (verbatim from the flow, in Romanian):**

```
Esti un expert in calificarea prospectilor B2B pentru o firma mica de servicii AI/automatizari pentru IMM-uri. Analizeaza compania de mai jos si raspunde STRICT cu un JSON valid, fara alt text in afara lui, de forma exacta: {"scor": <numar 1-10>, "eticheta": "calificat sau necalificat", "motiv": "un motiv scurt, o propozitie"}.

Companie: ${Nume}
Website: ${Website}
Descriere: ${Descriere}
Industrie: ${Industrie}

Reguli: pragul de calificare este scor mai mare sau egal cu 6. Companiile mici (IMM), fara semne vizibile de automatizare/chatbot pe site, sunt candidati mai buni.
```

**Robustness:** the code looks for the `text` block in the response and extracts the JSON with a regex; if parsing fails, the label becomes `eroare` (error) instead of an invented score.

**Lesson learned:** without `temperature: 0` the score varied between runs (7 vs 6 qualified). With `0` it is stable.
