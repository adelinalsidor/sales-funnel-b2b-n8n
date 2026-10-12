# offer-writer-agent

**Role:** writes the introduction email and the personalized offer for each qualified company. The result is **not sent**: it is saved for manual approval.

**Model:** `claude-sonnet-5-5`, `max_tokens: 6000`. It is the only step that uses the more expensive model, because text quality matters here.

**Input:** company data, ICP score and reason, the contact person (or "NEGĂSIT" → generic greeting) and the text from the site.

**Output:** one `.md` file per company in `oferte-de-aprobat/`, with `status: în așteptare` (pending) in its header. The owner changes it to `aprobat` or `respins`. The Deal moves to "Draft email".

**n8n nodes:** `Pregateste prompt oferta` → `Scrie oferta Claude` → `Pregateste fisier aprobare` → `Salveaza oferta pentru aprobare`.

**Prompt (verbatim from the flow, in Romanian):**

```
Ești Adelina Sidor, consultant care oferă servicii de automatizare și AI pentru IMM-uri (de exemplu: chatbot pe site, automatizarea răspunsurilor la clienți, integrare CRM, rapoarte automate). Scrie un email de prezentare și conținutul unei oferte personalizate pentru firma de mai jos, în limba română, cu diacritice, ton profesionist și cald.

REGULI STRICTE:
- Nu inventa clienți, cifre, statistici, prețuri sau servicii pe care textul de pe site nu le menționează.
- Fiecare pain-point trebuie legat de un detaliu concret din textul de pe site. Dacă textul nu are destule detalii, scrie exact „DETALII INSUFICIENTE pe site" în loc să inventezi.
- Emailul are maxim 150 de cuvinte, fără formulări generice de tip „am observat compania dumneavoastră".
- La prețul din ofertă scrie exact: [PREȚ DE COMPLETAT de Adelina].

DATE FIRMĂ:
Nume: ${Nume}
Website: ${Website}
Industrie: ${Industrie}
Scor ICP: ${ScorICP}/10 — ${MotivICP}
Persoană de contact: ${contactInfo}

TEXT DE PE SITE:
${siteText}

FORMAT EXACT AL RĂSPUNSULUI (fără alt text în afara lui):
### SUBIECT
(o singură linie)
### EMAIL
(începe cu: ${salut} — semnează: Adelina Sidor)
### PAIN POINTS
(2-3 puncte cu liniuță)
### OFERTA
(secțiuni cu titluri: Situația actuală; Ce propunem; Pași de implementare; Rezultate așteptate; Investiție; Pasul următor)
```

**Guardrails against hallucination:** the price stays a placeholder (a human decides), pain points must be anchored in the site text, and when data is missing the offer says "DETALII INSUFICIENTE pe site" (insufficient details).

**Lessons learned:** Sonnet answers with a `thinking` block before the text, so the code looks for the block with `type: text` instead of reading `content[0]`. At `max_tokens: 2500` offers came out truncated; at 6000 they are complete.
