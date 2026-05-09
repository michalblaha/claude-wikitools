# wiki-tools — Claude

Tato složka je Claude Code / Codex plugin.

**Název pluginu:** `wiki-tools`
**Skills:** `wiki-ingest`, `wiki-query`, `wiki-lint`, `wiki-autoresearch`, `html-clean`, `obsidian-markdown`
**Příkazy:** `/wiki`, `/autoresearch`
**Cesta k vaultu:** Tento adresář (otevřete přímo v Obsidianu)

## K čemu tento vault slouží

Tento vault demonstruje vzor LLM Wiki — trvalou, kumulující se znalostní bázi pro Claude + Obsidian. Vložte libovolný zdroj, pokládejte libovolné otázky a wiki se s každou relací obohacuje.

## Struktura vaultu

```
raw/                  zdrojové dokumenty — neměnné, Claude pouze čte, nikdy neupravuje
wiki/                 znalostní báze generovaná Claudem
raw/_attachments/     obrázky a PDF, na které wiki stránky odkazují
```

## Jak používat

Vložte zdrojový soubor do `raw/`, pak řekněte Claudovi: „ingest [filename]".

Pokládejte libovolné otázky. Claude nejprve přečte rejstřík, poté se ponoří do relevantních stránek.

Spouštějte „lint the wiki" každých 10–15 ingestů, abyste odhalili orphans a mezery.

## Cross-Project napojení

Pro odkazování na tuto wiki z jiného Claude Code projektu přidejte do `CLAUDE.md` daného projektu:

```markdown
## Wiki Knowledge Base
Path: /path/to/this/vault

Když potřebujete kontext, který v tomto projektu není:
1. Nejprve přečtěte wiki/hot.md (nedávný kontext, ~500 slov)
2. Pokud nestačí, přečtěte wiki/index.md
3. Pokud potřebujete detaily domény, přečtěte wiki/<domain>/_index.md
4. Až poté čtěte jednotlivé wiki stránky

Wiki NEČTĚTE pro obecné programátorské dotazy nebo pro věci, které jsou již v tomto projektu.
```

## Skills pluginu

| Skill | Trigger |
|-------|---------|
| `wiki` (příkaz) | `/wiki` — setup, scaffold, routování |
| `wiki-ingest` | `ingest [source]` — ingest jednoho nebo více zdrojů |
| `wiki-query` | `query: [question]` — odpovědi z obsahu wiki |
| `wiki-lint` | `lint the wiki` — health check |
| `wiki-autoresearch` | `/autoresearch [topic]` — autonomní výzkumná smyčka |
| `html-clean` | `clean this url`, `defuddle` — čištění webových stránek |
| `obsidian-markdown` | reference na syntax Obsidian Flavored Markdown |
