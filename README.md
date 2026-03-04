# Legal Reviews Pipeline (CZ)

Cíl: průběžně sbírat a analyzovat recenze advokátních kanceláří (CZ) v jednotném JSON formátu, automaticky je:
- sloučit
- deduplikovat
- obohatit (sentiment + faktory) pomocí OpenAI API
- publikovat jednoduchý frontend (static) pro vyhodnocení

## Co tady JE a co NENÍ
- ✅ Je to **pipeline** pro práci s JSON soubory (data dodáš ty / nebo je získáš legální cestou).
- ✅ Umí **automaticky přidávat nové JSONy** z `/data/inbox` do datasetu.
- ✅ Umí **deduplikovat** a **enrichovat** (sentiment + faktory).
- ✅ Má jednoduchý **dashboard** v `/public`.
- ❗️Neobsahuje automatický scraping Google Maps. Google a další platformy často zakazují scraping v ToS.
  Doporučení: export/ruční sběr, nebo oficiální API, nebo datový provider.

---

## 1) Jednotný vstupní formát (RAW JSON)

Každý soubor, který přidáš do `/data/inbox`, musí být validní JSON ve tvaru:

```json
{
  "source": "google_maps_reviews",
  "location": "Praha",
  "collected_at": "2026-03-04",
  "reviews": [
    {
      "firm": "HAVEL & PARTNERS",
      "city": "Praha",
      "review_text": "…",
      "review_date": "2025-11-16",
      "rating": 1,
      "sentiment": "negative"
    }
  ]
}
```

Pravidla:
- `rating` vynech, pokud není dostupný
- `review_date` vynech, pokud není dostupný
- `sentiment` můžeš dodat, nebo ho nech prázdné/vynech → pipeline dopočítá

---

## 2) Jak přidávat data (nejjednodušší workflow)

1. Vytvoř nový soubor například:
   - `data/inbox/reviews_google_maps_praha_01.json`
2. Commit + push do GitHub.
3. GitHub Action:
   - sloučí vše do `public/data/dataset.json`
   - z deduplikovaných review vytvoří `public/data/enriched.json` (sentiment + faktory)

---

## 3) Nastavení OpenAI API klíče (GitHub Secrets)

V repo nastav secret:

- `OPENAI_API_KEY` = tvůj API klíč

Volitelné:
- `OPENAI_MODEL` (default: `gpt-4.1-mini` pokud není nastaveno)

---

## 4) Lokální spuštění

```bash
npm install
npm run build:data
npm run dev
```

Otevři:
- http://localhost:5173

---

## 5) Deployment

Tento projekt je čistě static (Vite). Můžeš ho:
- deploynout na Vercel (doporučeno)
- nebo GitHub Pages

### Vercel (rychlé)
- Import repo do Vercel
- Build command: `npm run build`
- Output: `dist`

---

## 6) Co pipeline dělá

- `scripts/merge.mjs`  → sloučení RAW souborů
- `scripts/dedupe.mjs` → deduplikace (hash podle firm+city+text+date+rating+source)
- `scripts/enrich.mjs` → sentiment + faktory přes OpenAI (incremental cache)
- `scripts/build-data.mjs` → orchestrace (merge → dedupe → enrich → export do public/data)

---

## 7) Seznam firem (centrálně)

`config/config.json`

---

## 8) Bezpečnost / ToS

Používej jen zdroje a metody sběru, které jsou v souladu s podmínkami platforem a právem.
