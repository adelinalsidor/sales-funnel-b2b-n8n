# prospector-agent

**Rol:** furnizează lista de firme-țintă și elimină din ea paginile care nu sunt firme reale.

**Tip:** reguli deterministe, fără Claude. (Căutarea automată pe web cu Firecrawl a fost folosită la construirea listei; în MVP lista rulată vine din fișier.)

**Intrare:** `companii-test.csv` — coloane: `Nume`, `Website`, `Descriere`, `Industrie` (8 firme).

**Ieșire:** aceleași rânduri, fără paginile-director și listicle-urile („Top 10 firme...", clasamente, recenzii).

**Noduri n8n:** `Read/Write Files from Disk` → `Extract from File` → `Filtru pagini-director`.

**Logica filtrului** (nod de cod): respinge rândul dacă numele sau calea URL-ului se potrivește cu tipare precum `top 10`, `cele mai bune`, `clasament`, `director`, `lista de`, `recenzii`, `reviews`, sau dacă lipsește website-ul. Fiecare respingere se loghează cu numele firmei.

**Lecție învățată:** CSV-urile exportate din PowerShell au un BOM invizibil care strică parsarea în n8n — fișierele se scriu fără BOM.
