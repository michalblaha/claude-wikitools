---
name: wiki-autoresearch
description: >
  Autonomní iterativní výzkumná smyčka. Vezme téma, spustí webové vyhledávání, stáhne
  zdroje, syntetizuje nálezy a založí vše do wiki jako strukturované stránky.
  Postaveno na vzoru Karpathyho autoresearch: program.md konfiguruje cíle a omezení,
  smyčka běží do dosažení požadované hloubky, výstup jde přímo do znalostní báze.
  Triggers on: "/autoresearch", "autoresearch", "research [topic]", "deep dive into [topic]",
  "investigate [topic]", "find everything about [topic]", "research and file",
  "go research", "build a wiki on".
allowed-tools: Read Write Edit Glob Grep WebFetch WebSearch
---

# wiki-autoresearch: Autonomní výzkumná smyčka

Jsi výzkumný agent. Vezmeš téma, spustíš iterativní webové vyhledávání, syntetizuješ nálezy a založíš vše do wiki. Uživatel dostane wiki stránky, ne chatovou odpověď.

Postaveno na Karpathyho autoresearch vzoru: konfigurovatelný program definuje tvé cíle. Smyčku běžíš do dosažení hloubky. Výstup jde do znalostní báze.

---


## Routing podle profilu

Identifikuj profil z prvního slova za `/autoresearch`:

- žádné slovo → načti `references/default.md`
- `gov-project` → načti `references/gov-project.md`

k načtení cílů a omezení výzkumu. Tento soubor je uživatelsky konfigurovatelný. Definuje, kterým zdrojům dávat přednost, jak hodnotit míru spolehlivosti (confidence) a jakákoli omezení specifická pro danou doménu.

---

## Výzkumná smyčka

```
Vstup: téma (z uživatelského příkazu)

Kolo 1. Široké vyhledávání
1. Rozlož téma na 3–6 odlišných výzkumných úhlů
2. Pro každý úhel: spusť 5–6 WebSearch dotazů
3. Pro top 3–5 výsledků na úhel: WebFetch stránky
4. Z každé extrahuj: klíčová tvrzení, entity, koncepty, otevřené otázky

Kolo 2. Doplnění mezer
5. Identifikuj, co chybí nebo si protiřečí z Kola 1
6. Spusť cílené vyhledávání pro každou mezeru (max 5 dotazů)
7. Stáhni top výsledky pro každou mezeru

Kolo 3. Kontrola syntézy (volitelně, pokud zůstávají mezery)
8. Pokud stále existují velké rozpory nebo chybějící části: jeden další cílený průchod
9. Jinak: pokračuj k zakládání

Max kol: 4 (jak je nastaveno v profilu). Skonči, když je dosažena hloubka nebo max kol.
```


---

## Struktura syntézní stránky

```markdown
---
type: synthesis
title: "Research: [Topic]"
created: YYYY-MM-DD
updated: YYYY-MM-DD
tags:
  - research
  - [topic-tag]
status: developing
related:
  - "[[Every page created in this session]]"
sources:
  - "[[wiki/sources/Source 1]]"
  - "[[wiki/sources/Source 2]]"
---

# Research: [Topic]

## Overview
[2–3 věty shrnující, co bylo nalezeno]

## Key Findings
- Finding 1 (Source: [[Source Page]])
- Finding 2 (Source: [[Source Page]])
- ...

## Key Entities
- [[Entity Name]]: role/význam

## Key Concepts
- [[Concept Name]]: jednořádková definice

## Contradictions
- [[Source A]] tvrdí X. [[Source B]] tvrdí Y. [Krátká poznámka, který je důvěryhodnější a proč]

## Open Questions
- [Otázka, kterou výzkum plně nezodpověděl]
- [Mezera, která vyžaduje více zdrojů]

## Sources
- [[Source 1]]: autor, datum
- [[Source 2]]: autor, datum
```

---

## Po založení

1. Aktualizuj wiki. Přidej všechny nové stránky.
2. Připoj na `wiki/log.md` (NA ZAČÁTEK):
   ```
   ## [YYYY-MM-DD] autoresearch | [Topic]
   - Rounds: N
   - Sources found: N
   - Pages created: [[Page 1]], [[Page 2]], ...
   - Synthesis: [[Research: Topic]]
   - Key finding: [jedna věta]
   ```
3. Aktualizuj `wiki/hot.md` shrnutím výzkumu
4. Spusť `wiki-lint`

---

## Report uživateli

Po založení všeho:

```
Research complete: [Topic]

Rounds: N | Searches: N | Pages created: N

Created:
  wiki/questions/Research: [Topic].md (synthesis)
  wiki/sources/[Source 1].md
  wiki/concepts/[Concept 1].md
  wiki/entities/[Entity 1].md

Key findings:
- [Finding 1]
- [Finding 2]
- [Finding 3]

Open questions filed: N
```

---

## Omezení

Respektuj omezení z načteného profilu z `references/*`:
- Max kol
- Max stránek za relaci
- Pravidla pro hodnocení míry jistoty (confidence)
- Pravidla preference zdrojů

Pokud omezení koliduje s úplností, respektuj omezení a co bylo vynecháno, poznamenej do sekce Open Questions.
