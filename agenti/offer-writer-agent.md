# offer-writer-agent

**Rol:** scrie emailul de prezentare și oferta personalizată pentru fiecare firmă calificată. Rezultatul **nu se trimite**: se salvează pentru aprobare manuală.

**Model:** `claude-sonnet-5-5`, `max_tokens: 6000`. E singurul pas unde se folosește modelul mai scump, pentru că aici contează calitatea textului.

**Intrare:** datele firmei, scorul ICP și motivul, persoana de contact (sau „NEGĂSIT" → salut generic) și textul de pe site.

**Ieșire:** un fișier `.md` per firmă în `oferte-de-aprobat/`, cu `status: în așteptare` în cap. Adelina îl schimbă în `aprobat` sau `respins`. Deal-ul trece în „Draft email".

**Noduri n8n:** `Pregateste prompt oferta` → `Scrie oferta Claude` → `Pregateste fisier aprobare` → `Salveaza oferta pentru aprobare`.

**Prompt (exact din flow):**

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

**Garanții anti-halucinare:** prețul rămâne placeholder (decide omul), pain-point-urile trebuie ancorate în textul de pe site, iar la lipsă de date apare „DETALII INSUFICIENTE pe site".

**Lecții învățate:** Sonnet răspunde cu un bloc `thinking` înaintea textului, deci codul caută blocul `type: text`, nu `content[0]`. La `max_tokens: 2500` ofertele ieșeau tăiate; la 6000 sunt complete.
