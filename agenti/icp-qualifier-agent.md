# icp-qualifier-agent

**Rol:** dă fiecărei firme un scor de potrivire (1–10) cu profilul clientului ideal (ICP) și o etichetă `calificat` / `necalificat`.

**Model:** `claude-haiku-4-5`, `temperature: 0`, `max_tokens: 300`.

**Intrare:** `Nume`, `Website`, `Descriere`, `Industrie`.

**Ieșire:** JSON `{"scor": 1-10, "eticheta": "calificat|necalificat", "motiv": "..."}`. Prag: scor ≥ 6 = calificat. Rezultatul ajunge în câmpul `description` al firmei în HubSpot.

**Noduri n8n:** `Pregateste prompt Claude` → `Scoreaza ICP Claude` (HTTP către API-ul Anthropic) → `Interpreteaza scor si pregateste HubSpot`.

**Prompt (exact din flow):**

```
Esti un expert in calificarea prospectilor B2B pentru o firma mica de servicii AI/automatizari pentru IMM-uri. Analizeaza compania de mai jos si raspunde STRICT cu un JSON valid, fara alt text in afara lui, de forma exacta: {"scor": <numar 1-10>, "eticheta": "calificat sau necalificat", "motiv": "un motiv scurt, o propozitie"}.

Companie: ${Nume}
Website: ${Website}
Descriere: ${Descriere}
Industrie: ${Industrie}

Reguli: pragul de calificare este scor mai mare sau egal cu 6. Companiile mici (IMM), fara semne vizibile de automatizare/chatbot pe site, sunt candidati mai buni.
```

**Robustețe:** codul caută blocul de tip `text` din răspuns și extrage JSON-ul cu regex; dacă parsarea eșuează, eticheta devine `eroare`, nu se inventează un scor.

**Lecție învățată:** fără `temperature: 0`, scorul varia între rulări (7 vs 6 calificate). Cu `0`: stabil, 6 calificate din 8.
