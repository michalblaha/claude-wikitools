# wiki-tools

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Claude Code](https://img.shields.io/badge/Claude_Code-plugin-8B5CF6)](https://code.claude.com/docs/en/discover-plugins)

Společník pro znalostní bázi nad Claude. Průběžný „zapisovatel", který buduje a udržuje trvalou, kumulující se wiki. Každý zdroj, který přidáte, se integruje. Každá otázka, kterou položíte, čerpá ze všeho, co bylo přečteno. Znalosti se kumulují jako úroky.

Postaveno na vzoru [LLM Wiki od Andreje Karpathyho](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f).

---

## Co dělá

Vložíte zdroje. Claude je přečte, extrahuje entity a koncepty, aktualizuje vzájemné reference a vše uloží do strukturované Obsidian wiki. Wiki se s každým zpracovaným zdrojem obohacuje.

Pokládáte otázky. Claude přečte hot cache (poslední kontext), prohledá rejstřík, ponoří se do relevantních stránek a syntetizuje odpověď. Cituje konkrétní wiki stránky, ne tréninková data.

Spouštíte lint. Claude najde osamocené stránky, mrtvé odkazy, zastaralá tvrzení a chybějící křížové reference. Wiki zůstává zdravá bez ručního úklidu.

Na konci každé relace Claude aktualizuje hot cache. Další relace začíná s plným nedávným kontextem, bez nutnosti rekapitulace.

---

## Proč wiki-tools?

Většina Obsidian AI pluginů jsou chatovací rozhraní — odpovídají na otázky o vašich existujících poznámkách. wiki-tools je znalostní engine — autonomně poznámky vytváří, organizuje, udržuje a rozvíjí.

| Schopnost | wiki-tools |
|---|---|
| **Automatická organizace poznámek** | Vytváří entity, koncepty, křížové reference |
| **Označování rozporů** | Callouty `[!contradiction]` se zdroji |
| **Paměť relace** | Hot cache přetrvává mezi konverzacemi |
| **Údržba vault** | Lint v 8 kategoriích (orphans, dead links, gaps) |
| **Autonomní výzkum** | Vícekolový web research s vyplňováním mezer |
| **Multi-model podpora** | Claude, Gemini, Codex, Cursor, Windsurf |
| **Dotazy s citacemi** | Cituje konkrétní wiki stránky |
| **Open source** | MIT |

---

## Quick Start

### Možnost 1: Naklonovat jako vault

```bash
git clone https://github.com/michalblaha/claude-wiki-tools
cd claude-wiki-tools
```

Otevřete složku v Obsidianu: **Manage Vaults → Open folder as vault → vyberte `claude-wiki-tools/`**

Otevřete Claude Code ve stejné složce. Napište `/wiki`.

---

### Možnost 2: Instalace jako Claude Code plugin

Instalace pluginu v Claude Code je dvoukroková. Nejprve přidáte marketplace katalog, poté plugin nainstalujete.

```bash
# Krok 1: přidat marketplace
claude plugin marketplace add michalblaha/claude-wiki-tools

# Krok 2: nainstalovat plugin
claude plugin install wiki-tools@wiki-tools-marketplace
```

V libovolné Claude Code relaci napište `/wiki`. Claude vás provede nastavením vaultu.

Ověření:
```bash
claude plugin list
```

---

## Příkazy

| Co napíšete | Co Claude udělá |
|---------|------------|
| `/wiki` | Kontrola nastavení, scaffold, nebo pokračování tam, kde jste skončili |
| `ingest [file]` | Načte zdroj, vytvoří 8–15 wiki stránek, aktualizuje rejstřík a log |
| `ingest all of these` | Dávkové zpracování více zdrojů a následné křížové propojení |
| `what do you know about X?` | Přečte rejstřík → relevantní stránky → syntetizuje odpověď |
| `/autoresearch [topic]` | Spustí autonomní výzkumnou smyčku: vyhledá, stáhne, syntetizuje, založí |
| `lint the wiki` | Health check: orphans, dead links, mezery, návrhy |
| `update hot cache` | Obnoví `hot.md` aktuálním shrnutím kontextu |

---

## Cross-Project napojení

Můžete na tuto wiki ukázat z jiného Claude Code projektu. Do `CLAUDE.md` daného projektu přidejte:

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

## Šest módů wiki

| Mód | Použití |
|------|---------|
| A: Website | Sitemap, audit obsahu, SEO wiki |
| B: GitHub | Mapa codebase, architektonická wiki |
| C: Business | Projektová wiki, konkurenční zpravodajství |
| D: Personal | Druhý mozek, cíle, syntéza deníku |
| E: Research | Články, koncepty, teze |
| F: Book/Course | Tracker kapitol, poznámky ke kurzu |

Módy lze kombinovat.

---

## Co se vytváří

Typický scaffold vytvoří:
- Strukturu složek pro zvolený mód
- `wiki/index.md`: hlavní katalog
- `wiki/log.md`: append-only log operací
- `wiki/hot.md`: cache nedávného kontextu
- `wiki/overview.md`: výkonné shrnutí
- `_templates/`: Templater šablony pro každý typ poznámky
- Vault `CLAUDE.md`: automaticky načítané instrukce projektu

---

## MCP Setup (volitelné)

MCP umožňuje Claudovi číst a zapisovat poznámky vault přímo bez kopírování.

Možnost A (přes REST API):
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
    "OBSIDIAN_PORT": "27124",
    "NODE_TLS_REJECT_UNAUTHORIZED": "0"
  }
}' --scope user
```

Možnost B (přes filesystem, bez pluginu):
```bash
claude mcp add-json obsidian-vault '{
  "type": "stdio",
  "command": "npx",
  "args": ["-y", "@bitbonsai/mcpvault@latest", "/path/to/your/vault"]
}' --scope user
```

---

## Pluginy

### Core pluginy (vestavěné v Obsidianu)

| Plugin | Účel |
|--------|---------|
| **Bases** | Nativní databázové pohledy (od Obsidian v1.9.10, srpen 2025) |
| **Properties** | Vizuální editor frontmatter |
| **Backlinks**, **Outline**, **Graph view** | Standardní navigace |

### Doporučené Community pluginy

Nainstalujte ze **Settings → Community Plugins**:

| Plugin | Účel |
|--------|---------|
| **Templater** | Automaticky vyplňuje frontmatter ze složky `_templates/` |
| **Obsidian Git** | Auto-commit vault každých 15 minut |
| **Dataview** | Pro Dataview dotazy (volitelné) |
| **Calendar** | Kalendář v pravém panelu |
| **Banners** | Notion-style hlavičkový obrázek přes `banner:` ve frontmatter |

Doporučujeme také rozšíření prohlížeče **[Obsidian Web Clipper](https://obsidian.md/clipper)** — odesílá webové stránky do `raw/` jedním kliknutím.

---

## Struktura souborů

```
claude-wiki-tools/
├── skills/
│   ├── ingest/                  # zpracování zdrojů (interní jméno: wiki-ingest)
│   ├── wiki-query/              # dotazy nad wiki
│   ├── wiki-lint/               # health check
│   ├── autoresearch/            # autonomní výzkumná smyčka
│   │   └── references/
│   │       ├── default.md       # default profil výzkumu
│   │       └── gov-project.md   # profil pro státní/veřejnou sféru
│   ├── html-clean/              # čištění webových stránek (defuddle)
│   └── obsidian-markdown/       # reference na Obsidian Markdown syntax
├── agents/
│   ├── wiki-ingest.md           # paralelní ingest agent
│   └── wiki-lint.md             # health check agent
├── commands/
│   ├── wiki.md                  # /wiki bootstrap příkaz
│   └── autoresearch.md          # /autoresearch příkaz
├── hooks/
│   └── hooks.json               # SessionStart + Stop hot cache hooks
├── _templates/                  # Templater šablony
├── docs/
│   └── install-guide.md         # instalační průvodce
├── AGENTS.md                    # instrukce pro Codex / OpenCode
├── GEMINI.md                    # instrukce pro Gemini CLI
├── CLAUDE.md                    # instrukce pro Claude Code
└── README.md                    # tento soubor
```

---

## AutoResearch: program.md

Příkaz `/autoresearch` je konfigurovatelný. Upravte `skills/autoresearch/references/default.md` (nebo `gov-project.md`) a ovládněte:

- Které zdroje preferovat (akademické, oficiální dokumentace, zprávy)
- Pravidla pro hodnocení míry jistoty (confidence)
- Maximální počet kol a stránek na relaci
- Omezení specifická pro doménu

Default program funguje pro obecný výzkum. Pro svou doménu jej přepište. Lékařský výzkumník by přidal „prefer PubMed". Byznysový analytik by přidal „focus on market data and filings".

---

## Komunita

- [LLM Wiki vzor](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f) — Andrej Karpathy
- [Issues](https://github.com/michalblaha/claude-wiki-tools/issues)

---

*Postaveno na vzoru [LLM Wiki od Andreje Karpathyho](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f). Inspirováno projektem [claude-obsidian](https://github.com/AgriciDaniel/claude-obsidian).*
