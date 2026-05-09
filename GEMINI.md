# wiki-tools: Instrukce pro Gemini CLI

Tento repozitář je společník znalostní báze, který buduje trvalé, kumulující se Obsidian wiki vaulty podle vzoru LLM Wiki od Andreje Karpathyho. Skills jsou napsány v cross-platform Agent Skills formátu a fungují v Gemini CLI / Antigravity vedle Claude Code.

## Discovery skills

Skills jsou v `skills/<name>/SKILL.md`. Pro zpřístupnění v Gemini CLI:

```bash
ln -s "$(pwd)/skills" ~/.gemini/skills/wiki-tools
```

## Skills

| Skill | Co dělá |
|---|---|
| `wiki-ingest` | Čte zdroje (soubory, URL, obrázky) a vytváří 8–15 wiki stránek za každý |
| `wiki-query` | Odpovídá na otázky z wiki ve třech hloubkových módech |
| `wiki-lint` | Health check: orphans, dead links, zastaralá tvrzení, mezery |
| `wiki-autoresearch` | Autonomní výzkumná smyčka: search → fetch → synthesize → file |
| `html-clean` | Čistí webové stránky před ingestem (úspora 40–60 % tokenů) |
| `obsidian-markdown` | Reference na Obsidian Flavored Markdown |

Bootstrap příkaz `/wiki` (`commands/wiki.md`) provede setup vaultu.

## Trigger fráze (příklady)

- „set up wiki" → příkaz `/wiki`
- „ingest this article" → `wiki-ingest`
- „ingest https://example.com/article" → `wiki-ingest` (URL mód)
- „what do you know about X" → `wiki-query`
- „lint the wiki" → `wiki-lint`
- „research [topic]" → `wiki-autoresearch`

## Konvence vaultu

- `raw/`: zdrojové dokumenty, neměnné (nikdy neupravujte)
- `wiki/`: znalostní báze generovaná agentem (vlastněno vámi)
- `wiki/hot.md`: cache nedávného kontextu (~500 tokenů), čte se první na začátku relace
- `wiki/index.md`: hlavní katalog
- `.manifest.json`: delta tracking pro ingest

## Bootstrap

Při první relaci:
1. Přečtěte tento soubor + projektový `CLAUDE.md`
2. Pokud existuje `wiki/hot.md`, tiše jej přečtěte pro obnovu nedávného kontextu
3. Počkejte, až uživatel napíše `/wiki`, `ingest` nebo `query`

## Odkazy na projekt

- Plugin: https://github.com/michalblaha/claude-wikitools
- Vzor: https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f
