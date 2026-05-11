# wiki-tools

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-1.3.0-blue)](.claude-plugin/plugin.json)

Trvalá kumulující se znalostní báze pro **Claude Code + Obsidian**. Ingest zdrojů, dotazy s citacemi, autonomní výzkum a health check vaultu.

Postaveno na vzoru [LLM Wiki od Andreje Karpathyho](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f).

> Součást marketplace [`michalblaha-ai-plugins`](../../README.md). Volitelně se integruje s pluginem [`ai-review`](../ai-review/) pro cross-model fact-check.

---

## Instalace

### Claude Code

```bash
claude plugin marketplace add michalblaha/michalblaha-ai-plugins
claude plugin install wiki-tools@michalblaha-ai-plugins
```

### Codex CLI (0.130+)

```bash
codex plugin marketplace add michalblaha/michalblaha-ai-plugins
```

Aktivace v `~/.codex/config.toml`:

```toml
[plugins."wiki-tools@michalblaha-ai-plugins"]
enabled = true
```

### Jako Obsidian vault

Otevřete tuto složku přímo v Obsidianu: **Manage Vaults → Open folder as vault**.

---

## Skills a příkazy

| Co napíšete | Skill | Co Claude udělá |
|---------|------|------------|
| `ingest [file/url]` | `wiki-ingest` | Načte zdroj, vytvoří 8–15 wiki stránek, aktualizuje rejstřík a log |
| `ingest all of these` | `wiki-ingest` (paralelní agent) | Dávkové zpracování více zdrojů + křížové propojení |
| `what do you know about X?` | `wiki-query` | Hot cache → rejstřík → relevantní stránky → odpověď s citacemi |
| `/wiki-autoresearch [topic]` | `wiki-autoresearch` | Autonomní výzkumná smyčka: vyhledá, stáhne, syntetizuje, založí |
| `lint the wiki` | `wiki-lint` | Health check: orphans, dead links, mezery, návrhy oprav |
| `clean this url` | `html-clean` | Defuddle: strip ads, navigace a boilerplate před ingestem |

Doplňkový skill `obsidian-markdown` slouží jako referenční syntax pro psaní Obsidian Flavored Markdown.

---

## Struktura vaultu

```
raw/                  zdrojové dokumenty (NEMĚNNÉ, Claude pouze čte)
wiki/                 znalostní báze generovaná Claudem
wiki/index.md         hlavní katalog
wiki/log.md           append-only záznam operací
wiki/hot.md           cache nedávného kontextu (~500 slov)
wiki/overview.md      výkonné shrnutí vaultu
_templates/           Templater šablony (concept, entity, source, question, comparison)
```

---

## Šest módů wiki

| Mód | Použití |
|------|---------|
| A: Website | Sitemap, audit obsahu, SEO wiki |
| B: GitHub | Mapa codebase, architektonická wiki |
| C: Business | Projektová wiki, konkurenční zpravodajství |
| D: Personal | Druhý mozek, cíle, syntéza deníku |
| E: Research | Články, koncepty, teze |
| F: Book/Course | Tracker kapitol, poznámky ke kurzu |

Módy lze kombinovat. Při bootstrapu (`set up wiki for [topic]`) Claude pomůže zvolit a nascaffolduje strukturu.

---

## Konfigurace AutoResearch

Příkaz `/wiki-autoresearch` se konfiguruje přes `skills/wiki-autoresearch/references/`:

- `default.md` — obecný výzkum
- `gov-project.md` — profil pro státní/veřejnou sféru

Definuje preferované zdroje, pravidla pro confidence, max počet kol a omezení specifická pro doménu.

---

## Cross-Project napojení

Z jiného Claude Code projektu na vault ukažte přidáním do `CLAUDE.md`:

```markdown
## Wiki Knowledge Base
Path: ~/path/to/vault

Když potřebujete kontext, který v projektu není:
1. Nejprve přečtěte wiki/hot.md
2. Pokud nestačí, přečtěte wiki/index.md
3. Pak relevantní sub-index domény
4. Až poté konkrétní stránky
```

---

## Další dokumentace

- [`CLAUDE.md`](CLAUDE.md) — instrukce pro Claude Code (workflow, formát stránky, pravidla citací)
- [`AGENTS.md`](AGENTS.md) — instrukce pro Codex / OpenCode
- [`GEMINI.md`](GEMINI.md) — instrukce pro Gemini CLI
- [`docs/install-guide.md`](docs/install-guide.md) — detailní instalační průvodce
- [`ATTRIBUTION.md`](ATTRIBUTION.md) — uvedení zdrojů a inspirací
- [Marketplace README](../../README.md) — kompletní dokumentace všech pluginů

---

*MIT License. Inspirováno [LLM Wiki vzorem od Andreje Karpathyho](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) a projektem [claude-obsidian](https://github.com/AgriciDaniel/claude-obsidian).*
