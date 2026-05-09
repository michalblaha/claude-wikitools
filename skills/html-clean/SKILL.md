---
name: html-clean
description: "Odstraňuje rušivé prvky z webových stránek před ingestem do wiki. Maže reklamy, navigaci, hlavičky, patičky a boilerplate: nechává čistý čitelný markdown, který šetří 40–60 % tokenů. Triggers on: defuddle, clean this page, strip this url, fetch and clean, clean web content before ingesting, strip ads, remove clutter, clean URL content, readable markdown from URL."
allowed-tools: Read Bash
---

# html-clean: Čistič webových stránek

Tento skill používá CLI nástroj `defuddle` (npm balíček `defuddle-cli`), který extrahuje smysluplný obsah z webové stránky a odstraní vše ostatní: reklamy, cookie bannery, navigační lišty, související články, patičky a tlačítka pro sdílení. Co zůstane, je tělo článku jako čistý markdown.

Použij to před libovolným URL ingestem. Je volitelné, ale silně doporučené. Snižuje spotřebu tokenů u typických webových článků o 40–60 % a produkuje čistší wiki stránky.

---

## Instalace

```bash
npm install -g defuddle-cli
```

Ověření: `defuddle --version`

---

## Použití

### Přímé čištění URL
```bash
defuddle https://example.com/article
```
Čistý markdown se vypíše na stdout.

### Uložení do raw/
```bash
defuddle https://example.com/article > raw/articles/article-slug-$(date +%Y-%m-%d).md
```

### Přidání frontmatter hlavičky po uložení
Po spuštění defuddle přidej zdrojové URL a datum stažení:
```bash
SLUG="article-slug-$(date +%Y-%m-%d)"
{ echo "---"; echo "source_url: https://example.com/article"; echo "fetched: $(date +%Y-%m-%d)"; echo "---"; echo ""; defuddle https://example.com/article; } > raw/articles/$SLUG.md
```

### Čištění lokálního HTML souboru
```bash
defuddle page.html
```

---

## Kdy použít

**Použij defuddle, když:**
- Ingestuješ článek, blog post nebo dokumentační stránku z URL
- Stránka má hodně obklopujícího obsahu (většina webových stránek má)
- Chceš se vejít do tokenového rozpočtu u dlouhého článku

**Defuddle přeskoč, když:**
- Zdroj je již čistý markdown nebo PDF soubor
- Stránka je dashboard, app nebo strukturovaná data (defuddle očekává článkový obsah)
- Defuddle není nainstalován a článek je dostatečně krátký na zpracování raw

---

## Fallback

Pokud není defuddle nainstalován, zkontroluj:

```bash
which defuddle 2>/dev/null || echo "not installed"
```

Pokud není nainstalován: použij přímo WebFetch. Obsah bude méně čistý, ale použitelný.

---

## Integrace s wiki-ingest

Skill `wiki-ingest` automaticky kontroluje defuddle, když je předáno URL. Před ingestem URL nemusíš defuddle spouštět ručně. Skill ho zavolá, je-li dostupný.

Pro ruční čištění stránky a uložení před ingestem:
1. Spusť výše uvedený save příkaz
2. Pak: `ingest raw/articles/[slug].md`
