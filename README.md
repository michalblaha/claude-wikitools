# michalblaha-ai-plugins

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude_Code-marketplace-8B5CF6)](https://code.claude.com/docs/en/discover-plugins)
[![Codex CLI](https://img.shields.io/badge/Codex_CLI-marketplace-10B981)](https://github.com/openai/codex)

Marketplace AI pluginů od Michala Bláhy. Funguje jako katalog pro **Claude Code** (`.claude-plugin/marketplace.json`) i pro **Codex CLI** (`.agents/plugins/marketplace.json`). Aktuálně obsahuje dva pluginy, které lze používat samostatně i společně.

| Plugin | Verze | Co dělá |
|---|---|---|
| **[wiki-tools](plugins/wiki-tools)** | 1.3.0 | Trvalá kumulující se znalostní báze nad Obsidianem — ingest zdrojů, dotazy s citacemi, autonomní výzkum, health check vaultu |
| **[ai-review](plugins/ai-review)** | 1.2.3 | Druhý názor a cross-check napříč Claude, Codex (OpenAI) a Gemini (Google) — multi-model validace |

---

## Quick Start

### Claude Code

```bash
# Krok 1: přidat marketplace
claude plugin marketplace add michalblaha/michalblaha-ai-plugins

# Krok 2: nainstalovat plugin (jeden nebo oba)
claude plugin install wiki-tools@michalblaha-ai-plugins
claude plugin install ai-review@michalblaha-ai-plugins
```

Ověření:
```bash
claude plugin list
```

### Codex CLI

Marketplace je deklarován v `.agents/plugins/marketplace.json` (Codex CLI 0.130+). Přidání marketplace:

```bash
# Z GitHubu
codex plugin marketplace add michalblaha/michalblaha-ai-plugins

# Nebo z lokálního klonu
git clone https://github.com/michalblaha/michalblaha-ai-plugins
codex plugin marketplace add ./michalblaha-ai-plugins
```

Aktivace jednotlivých pluginů — edituj `~/.codex/config.toml` a doplň:

```toml
[plugins."wiki-tools@michalblaha-ai-plugins"]
enabled = true

[plugins."ai-review@michalblaha-ai-plugins"]
enabled = true
```

Aktualizace marketplace na nejnovější verzi:

```bash
codex plugin marketplace upgrade michalblaha-ai-plugins
```

Odebrání:

```bash
codex plugin marketplace remove michalblaha-ai-plugins
```

> Codex CLI **nemá** příkaz `codex plugin install`. Po `marketplace add` se pluginy aktivují přes `~/.codex/config.toml` (nebo přes interaktivní UI Codexu).

### Naklonovat jako Obsidian vault (jen pro wiki-tools)

```bash
git clone https://github.com/michalblaha/michalblaha-ai-plugins
cd michalblaha-ai-plugins
```

Otevřete složku `plugins/wiki-tools/` v Obsidianu: **Manage Vaults → Open folder as vault**. Otevřete Claude Code ve stejné složce a popište záměr (např. „nastav mi wiki pro výzkum státních zakázek").

---

## Plugin: wiki-tools

Společník pro znalostní bázi nad Claude. Průběžný „zapisovatel", který buduje a udržuje trvalou, kumulující se wiki. Každý zdroj, který přidáte, se integruje. Každá otázka, kterou položíte, čerpá ze všeho, co bylo přečteno. Znalosti se kumulují jako úroky.

Postaveno na vzoru [LLM Wiki od Andreje Karpathyho](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f).

### Co dělá

- **Ingest zdrojů** — Claude přečte vstup, extrahuje entity a koncepty, aktualizuje křížové reference a uloží do strukturované Obsidian wiki.
- **Dotazy s citacemi** — Claude přečte hot cache, prohledá rejstřík, ponoří se do relevantních stránek a syntetizuje odpověď. Cituje konkrétní wiki stránky, ne tréninková data.
- **Autonomní výzkum** — `/wiki-autoresearch` spustí iterativní webovou výzkumnou smyčku se syntézou nálezů přímo do wiki.
- **Health check** — lint v 8 kategoriích (orphans, dead links, mezery ve frontmatter, prázdné sekce, zastaralá tvrzení, ...).
- **Hot cache mezi relacemi** — na konci každé relace Claude aktualizuje `hot.md`. Další relace začíná s plným nedávným kontextem bez nutnosti rekapitulace.

### Skills a příkazy

| Co napíšete | Co Claude udělá | Skill |
|---------|------------|------|
| „set up wiki for [topic]" | Bootstrap: kontrola nastavení, scaffold, nebo pokračování tam, kde jste skončili | — |
| `ingest [file]` | Načte zdroj, vytvoří 8–15 wiki stránek, aktualizuje rejstřík a log | `wiki-ingest` |
| `ingest all of these` | Dávkové zpracování více zdrojů a následné křížové propojení | `wiki-ingest` (paralelní agent) |
| `what do you know about X?` | Přečte rejstřík → relevantní stránky → syntetizuje odpověď | `wiki-query` |
| `/wiki-autoresearch [topic]` | Spustí autonomní výzkumnou smyčku: vyhledá, stáhne, syntetizuje, založí | `wiki-autoresearch` |
| `lint the wiki` | Health check: orphans, dead links, mezery, návrhy oprav | `wiki-lint` |
| `clean this url` | Strip ads, navigace a boilerplate před ingestem (defuddle) | `html-clean` |
| `update hot cache` | Obnoví `hot.md` aktuálním shrnutím kontextu | — |

Doplňkové skills: `obsidian-markdown` (referenční syntax pro psaní Obsidian Flavored Markdown).

### Šest módů wiki

| Mód | Použití |
|------|---------|
| A: Website | Sitemap, audit obsahu, SEO wiki |
| B: GitHub | Mapa codebase, architektonická wiki |
| C: Business | Projektová wiki, konkurenční zpravodajství |
| D: Personal | Druhý mozek, cíle, syntéza deníku |
| E: Research | Články, koncepty, teze |
| F: Book/Course | Tracker kapitol, poznámky ke kurzu |

Módy lze kombinovat.

### Co se vytváří

Typický scaffold vytvoří:
- Strukturu složek pro zvolený mód
- `wiki/index.md`: hlavní katalog
- `wiki/log.md`: append-only log operací
- `wiki/hot.md`: cache nedávného kontextu
- `wiki/overview.md`: výkonné shrnutí
- `_templates/`: Templater šablony pro každý typ poznámky (concept, entity, source, question, comparison)
- Vault `CLAUDE.md`: automaticky načítané instrukce projektu

### AutoResearch: konfigurovatelný program

Příkaz `/wiki-autoresearch` je konfigurovatelný přes `plugins/wiki-tools/skills/wiki-autoresearch/references/`:

- `default.md` — obecný výzkum
- `gov-project.md` — profil pro státní/veřejnou sféru

Ovládá: které zdroje preferovat, pravidla pro hodnocení míry jistoty, maximální počet kol a stránek na relaci, omezení specifická pro doménu. Default funguje pro obecný výzkum. Pro svou doménu jej přepište — lékařský výzkumník by přidal „prefer PubMed", byznysový analytik „focus on market data and filings".

---

## Plugin: ai-review

Druhý názor a cross-check napříč modely. Když si Claude (nebo Codex, nebo Gemini) sám/sama validuje vlastní výstup, sdílí stejné slabiny — proto má smysl ptát se modelu od jiného providera. Plugin abstrahuje volání tří hlavních AI CLI nástrojů:

- **Codex CLI** (OpenAI, GPT-5.x)
- **Gemini CLI** (Google, Gemini 2.x)
- **Claude Code CLI** (Anthropic, Claude 4.x)

### Skills

| Skill | Kdy použít |
|-------|-----------|
| `second-opinion` | Validace architektonického rozhodnutí, bezpečnostní audit kódu, porovnávání implementačních přístupů, multi-model konsenzus pro rozhodnutí s vyššími stakes |
| `double-cross-check` | Ověření faktografických tvrzení, investigativní rešerše, fact-check — typicky jako poslední krok před publikací nebo akcí |

### Triggery

- „získej druhý názor", „ověř z jiného modelu"
- „zkontroluj přes Codex / Gemini / Claude Code"
- „cross-check", „dvojitá kontrola", „fact-check", „ověř fakta"

Skill **nepoužívej** pro běžné dotazy, kde stačí jedna odpověď — je drahý a pomalý. Cílené použití.

### Integrace s wiki-tools

Pokud je `ai-review` nainstalován společně s `wiki-tools`, wiki skills ho mohou cíleně využít:

- **`wiki-ingest`** — pro ověření faktografických tvrzení s vyšším dopadem (`double-cross-check`); cíleně, ne pro každý odstavec (skill je drahý)
- **`wiki-query`** — v deep módu pro nezávislou validaci syntézy od jiného modelu (`second-opinion`)

`wiki-tools` funguje plně samostatně i bez něj — pokud `ai-review` není nainstalován, wiki skills pokračují bez externí validace.

---

## Cross-Project napojení (wiki-tools)

Můžete na vault ukázat z jiného Claude Code projektu. Do `CLAUDE.md` daného projektu přidejte:

```markdown
## Wiki Knowledge Base
Path: ~/path/to/vault

Když potřebujete kontext, který v tomto projektu není:
1. Nejprve přečtěte wiki/hot.md (cache nedávného kontextu)
2. Pokud nestačí, přečtěte wiki/index.md
3. Pokud potřebujete detaily domény, přečtěte relevantní sub-index domény
4. Až poté otevírejte konkrétní wiki stránky

Wiki NEČTĚTE pro obecné programátorské dotazy nebo úkoly nesouvisející s [doménou].
```

---

## MCP Setup pro Obsidian (volitelné)

MCP umožňuje Claudovi číst a zapisovat poznámky vault přímo bez kopírování.

**Možnost A — přes REST API:**
1. Nainstalujte plugin **Local REST API** v Obsidianu
2. Zkopírujte API klíč
3. Spusťte:
```bash
claude mcp add-json obsidian-vault '{
  "type": "stdio",
  "command": "uvx",
  "args": ["mcp-obsidian"],
  "env": {
    "OBSIDIAN_API_KEY": "your-key",
    "OBSIDIAN_HOST": "127.0.0.1",
    "OBSIDIAN_PORT": "27124"
  }
}' --scope user
```

**Možnost B — přes filesystem, bez Obsidian pluginu:**
```bash
claude mcp add-json obsidian-vault '{
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "@bitbonsai/mcpvault@latest", "/path/to/your/vault"]
}' --scope user
```

---

## Doporučené Obsidian pluginy

### Core (vestavěné)

| Plugin | Účel |
|--------|---------|
| **Bases** | Nativní databázové pohledy (od Obsidian v1.9.10, srpen 2025) |
| **Properties** | Vizuální editor frontmatter |
| **Backlinks**, **Outline**, **Graph view** | Standardní navigace |

### Community (Settings → Community Plugins)

| Plugin | Účel |
|--------|---------|
| **Templater** | Automaticky vyplňuje frontmatter ze složky `_templates/` |
| **Obsidian Git** | Auto-commit vault každých 15 minut |
| **Dataview** | Pro Dataview dotazy (volitelné) |
| **Calendar** | Kalendář v pravém panelu |
| **Banners** | Notion-style hlavičkový obrázek přes `banner:` ve frontmatter |

Doporučujeme také rozšíření prohlížeče **[Obsidian Web Clipper](https://obsidian.md/clipper)** — odesílá webové stránky do `raw/` jedním kliknutím.

---

## Struktura repozitáře

```
michalblaha-ai-plugins/
├── .claude-plugin/
│   └── marketplace.json         # Claude Code marketplace katalog
├── .agents/
│   └── plugins/
│       └── marketplace.json     # Codex CLI marketplace katalog
├── plugins/
│   ├── wiki-tools/              # plugin: znalostní báze
│   │   ├── .claude-plugin/      # plugin metadata pro Claude Code
│   │   ├── .codex-plugin/       # plugin metadata pro Codex CLI
│   │   ├── skills/
│   │   │   ├── wiki-ingest/
│   │   │   ├── wiki-query/
│   │   │   ├── wiki-lint/
│   │   │   ├── wiki-autoresearch/
│   │   │   │   └── references/
│   │   │   │       ├── default.md
│   │   │   │       └── gov-project.md
│   │   │   ├── html-clean/
│   │   │   └── obsidian-markdown/
│   │   ├── agents/
│   │   │   ├── wiki-ingest.md   # paralelní ingest agent
│   │   │   └── wiki-lint.md     # health check agent
│   │   ├── commands/
│   │   │   └── wiki-autoresearch.md
│   │   ├── hooks/
│   │   │   └── hooks.json       # SessionStart + PostCompact + PostToolUse + Stop hooks (hot cache + auto-commit)
│   │   ├── _templates/          # Templater šablony
│   │   ├── docs/
│   │   │   └── install-guide.md
│   │   ├── AGENTS.md            # instrukce pro Codex / OpenCode
│   │   ├── GEMINI.md            # instrukce pro Gemini CLI
│   │   ├── CLAUDE.md            # instrukce pro Claude Code
│   │   ├── ATTRIBUTION.md
│   │   └── LICENSE
│   └── ai-review/               # plugin: cross-model review
│       ├── .claude-plugin/
│       ├── .codex-plugin/
│       └── skills/
│           ├── second-opinion/
│           └── double-cross-check/
└── README.md                    # tento soubor
```

---

## Proč tento marketplace?

Většina Obsidian AI pluginů jsou chatovací rozhraní — odpovídají na otázky o vašich existujících poznámkách. **wiki-tools** je znalostní engine — autonomně poznámky vytváří, organizuje, udržuje a rozvíjí. **ai-review** přidává disciplínu cross-model validace, kterou jediný model sám se sebou nedokáže poskytnout.

| Schopnost | wiki-tools | ai-review |
|---|---|---|
| Automatická organizace poznámek | ano | — |
| Označování rozporů (callouty `[!contradiction]`) | ano | — |
| Paměť relace (hot cache) | ano | — |
| Údržba vaultu (lint v 8 kategoriích) | ano | — |
| Autonomní výzkum (vícekolový web research) | ano | — |
| Dotazy s citacemi konkrétních stránek | ano | — |
| Multi-model druhý názor | přes ai-review | ano |
| Cross-provider fact-check | přes ai-review | ano |
| Multi-model podpora | Claude Code, Codex CLI, Gemini CLI a další Agent Skills-kompatibilní nástroje | Claude Code, Codex CLI, Gemini CLI |
| Open source (MIT) | ano | ano |

---

## Komunita

- [LLM Wiki vzor](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) — Andrej Karpathy
- [Issues](https://github.com/michalblaha/michalblaha-ai-plugins/issues)

---

*wiki-tools postaveno na vzoru [LLM Wiki od Andreje Karpathyho](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f). Inspirováno projektem [claude-obsidian](https://github.com/AgriciDaniel/claude-obsidian).*
