# wiki-tools: Instrukce pro agenty

Tento repozitář je Claude Code plugin **a zároveň** Obsidian vault, který buduje trvalé, kumulující se znalostní báze podle vzoru LLM Wiki od Andreje Karpathyho. Funguje s **libovolným AI coding agentem**, který podporuje standard Agent Skills, včetně Codex CLI, OpenCode a podobných.

Původně postaveno pro Claude Code, skills nyní následují cross-platform Agent Skills specifikaci. Frontmatter používá pouze `name` a `description` (žádná Claude-specifická rozšíření).

## Discovery skills

Všechny skills jsou v `skills/<name>/SKILL.md`. Codex / OpenCode / další agenti kompatibilní s Agent Skills je auto-discoverují, když si symlinkujete adresář:

```bash
# Codex CLI
ln -s "$(pwd)/skills" ~/.codex/skills/wiki-tools

# OpenCode
ln -s "$(pwd)/skills" ~/.opencode/skills/wiki-tools
```


## Dostupné skills

| Skill | Trigger fráze |
|---|---|
| `wiki-ingest` | ingest, ingest this url, ingest this image, batch ingest |
| `wiki-query` | query, what do you know about, query quick:, query deep: |
| `wiki-lint` | lint the wiki, health check, find orphans |
| `wiki-autoresearch` | autoresearch, autonomous research loop |
| `html-clean` | clean this url, defuddle |
| `obsidian-markdown` | reference Obsidian Markdown syntaxe (wikilinks, callouts, frontmatter) |

Bootstrap příkaz `/wiki` (`commands/wiki.md`) provede setup vaultu.

## Klíčové konvence

- **Kořen vaultu**: adresář obsahující `wiki/` a `raw/`
- **Hot cache**: `wiki/hot.md` (čte se na začátku relace, aktualizuje se na konci)
- **Zdrojové dokumenty**: `raw/` (neměnné: agenti je nikdy neupravují)
- **Generovaná znalostní báze**: `wiki/` (vlastněno agentem, odkazuje na zdroje přes wikilinks)
- **Manifest**: `.manifest.json` sleduje ingestované zdroje (delta tracking)

## Bootstrap

Když uživatel projekt poprvé otevře:

1. Přečtěte tento soubor (`AGENTS.md`) a projektový `CLAUDE.md` pro plný kontext
2. Pokud existuje `wiki/hot.md`, tiše jej přečtěte pro obnovu nedávného kontextu
3. Pokud uživatel napíše `/wiki` (nebo „set up wiki"), spusťte scaffold workflow z `commands/wiki.md`
