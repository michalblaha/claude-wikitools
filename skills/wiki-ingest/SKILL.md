---
name: wiki-ingest
description: "Ingest zdrojů do Obsidian wiki vaultu. Načte zdroj, extrahuje entity a koncepty, vytvoří nebo aktualizuje wiki stránky, křížově odkazuje a zaloguje operaci. Podporuje soubory, URL a batch mód. Triggers on: ingest, process this source, add this to the wiki, read and file this, batch ingest, ingest all of these, ingest this url."
allowed-tools: Read Write Edit Glob Grep Bash WebFetch
---

# wiki-ingest: Ingest zdrojů

Přečti zdroj. Napiš wiki. Vše křížově propoj. Jeden zdroj se typicky dotkne 8–15 wiki stránek.

**Standard syntaxe**: Všechny Obsidian Markdown stránky piš ve správném Obsidian Flavored Markdown. Wikilinky jako `[[Note Name]]`, callouty jako `> [!type] Title`, embeds jako `![[file]]`, properties jako YAML frontmatter. Pokud je nainstalován plugin kepano/obsidian-skills, preferuj jeho kanonický skill obsidian-markdown jako referenci. Jinak postupuj podle skillu `obsidian-markdown` v tomto projektu.

---

## Delta tracking

Před ingestem libovolného souboru zkontroluj `.manifest.json`, abys znovu nezpracoval nezměněné zdroje.

```bash
# Zkontrolovat existenci manifestu
[ -f .manifest.json ] && echo "exists" || echo "no manifest yet"
```

**Formát manifestu** (pokud chybí, vytvoř):
```json
{
  "sources": {
    "raw/articles/article-slug-2026-04-08.md": {
      "hash": "abc123",
      "ingested_at": "2026-04-08",
      "pages_created": ["wiki/sources/article-slug.md", "wiki/entities/Person.md"],
      "pages_updated": ["wiki/index.md"]
    }
  }
}
```

**Před ingestem souboru:**
1. Spočítej hash: `md5sum [file] | cut -d' ' -f1` (nebo `sha256sum` na Linuxu).
2. Zkontroluj, zda cesta v `.manifest.json` existuje se stejným hashem.
3. Pokud hash sedí, přeskoč. Nahlas: „Already ingested (unchanged). Use `force` to re-ingest."
4. Pokud chybí nebo se hash liší, pokračuj s ingestem.

**Po ingestu souboru:**
1. Zaznamenej `{hash, ingested_at, pages_created, pages_updated}` do `.manifest.json`.
2. Zapiš aktualizovaný manifest zpět.

Delta kontrolu přeskoč, pokud uživatel řekne „force ingest" nebo „re-ingest".

---

## URL ingest

Trigger: uživatel předá URL začínající `https://`.

Kroky:

1. **Fetch** stránky pomocí WebFetch.
2. **Clean** (volitelně): pokud je dostupný `defuddle` (`which defuddle 2>/dev/null`), spusť `defuddle [url]` pro odstranění reklam, navigace a balastu. Typicky šetří 40–60 % tokenů. Pokud není nainstalován, použij raw výstup z WebFetch.
3. **Odvoď slug** z URL cesty (poslední segment, malá písmena, mezery → pomlčky, odstraň query stringy).
4. **Save** do `raw/articles/[slug]-[YYYY-MM-DD].md` s frontmatter hlavičkou:
   ```markdown
   ---
   source_url: [url]
   fetched: [YYYY-MM-DD]
   ---
   ```
5. Pokračuj **Single Source Ingest** od kroku 2 (soubor je nyní v `raw/`).

---

## Image / Vision ingest

Trigger: uživatel předá cestu k obrázku (`.png`, `.jpg`, `.jpeg`, `.gif`, `.webp`, `.svg`, `.avif`).

Kroky:

1. **Read** obrázku pomocí Read nástroje. Claude umí obrázky zpracovávat nativně.
2. **Describe** obsahu obrázku: extrahuj všechen text (OCR), identifikuj klíčové koncepty, entity, diagramy a data viditelná v obrázku.
3. **Save** popisu do `raw/images/[slug]-[YYYY-MM-DD].md`:
   ```markdown
   ---
   source_type: image
   original_file: [original path]
   fetched: YYYY-MM-DD
   ---
   # Image: [slug]

   [Plný popis obsahu obrázku, přepsaný text, viditelné entity atd.]
   ```
4. Pokud obrázek ještě není ve vaultu, zkopíruj ho do `_attachments/images/[slug].[ext]`.
5. Pokračuj **Single Source Ingest** nad uloženým popisným souborem.

Použití: fotky tabulí, screenshoty, diagramy, infografiky, scany dokumentů.

---

## Single Source Ingest

Trigger: uživatel vloží soubor do `raw/` nebo vloží obsah.

Kroky:

1. **Read** zdroje kompletně. Nečti rychle.
2. **Discuss** klíčové poznatky s uživatelem. Zeptej se: „Co mám zdůraznit? Jak granulárně?" Přeskoč, pokud uživatel řekne „just ingest it."
3. **Create** shrnutí zdroje v `wiki/sources/`. Použij source frontmatter schéma z `_templates/source.md`.
4. **Create or update** stránky entit pro každou zmíněnou osobu, organizaci, produkt a repozitář. Jedna stránka na entitu.
5. **Create or update** stránky konceptů pro významné myšlenky a frameworky.
6. **Update** relevantní doménové stránky a jejich `_index.md` sub-rejstříky.
7. **Update** `wiki/overview.md`, pokud se změnil celkový obraz.
8. **Update** `wiki/index.md`. Přidej záznamy pro všechny nové stránky.
9. **Update** `wiki/hot.md` kontextem tohoto ingestu.
10. **Append** do `wiki/log.md` (nové záznamy NA ZAČÁTEK):
    ```markdown
    ## [YYYY-MM-DD] ingest | Source Title
    - Source: `raw/articles/filename.md`
    - Summary: [[Source Title]]
    - Pages created: [[Page 1]], [[Page 2]]
    - Pages updated: [[Page 3]], [[Page 4]]
    - Key insight: Jedna věta o tom, co je nového.
    ```
11. **Check for contradictions.** Pokud nové info koliduje s existujícími stránkami, přidej `> [!contradiction]` callouty na obě stránky.

---

## Batch Ingest

Trigger: uživatel vloží více souborů nebo řekne „ingest all of these."

Kroky:

1. Vypiš všechny soubory ke zpracování. Před začátkem potvrď s uživatelem.
2. Zpracuj každý zdroj single ingest postupem. Křížové odkazování mezi zdroji odlož na krok 3.
3. Po všech zdrojích: udělej cross-reference průchod. Hledej spojení mezi nově ingestovanými zdroji.
4. Aktualizuj rejstřík, hot cache a log na konci (ne při každém zdroji).
5. Nahlas: „Processed N sources. Created X pages, updated Y pages. Here are the key connections I found."

Batch ingest je méně interaktivní. Pro 30+ zdrojů očekávej významnou dobu zpracování. Po každých 10 zdrojích se uživateli ohlas.

---

## Disciplína kontextového okna

Tokenový rozpočet je důležitý. Při ingestu dodržuj:

- Nejprve čti `wiki/hot.md`. Pokud obsahuje relevantní kontext, neotvírej znovu plné stránky.
- Čti `wiki/index.md`, abys našel existující stránky před vytvořením nových.
- Čti maximálně 3–5 existujících stránek na ingest. Pokud potřebuješ 10+, čteš příliš široko.
- Pro chirurgické úpravy používej PATCH. Nikdy nečti celý soubor jen kvůli aktualizaci jednoho pole.
- Drž wiki stránky krátké. Max 100–300 řádků. Pokud stránka přeroste 300 řádků, rozděl ji.
- Použij vyhledávání (`/search/simple/`) pro nalezení konkrétního obsahu bez čtení plných stránek.

---

## Rozpory

> [!note] Závislost na vlastním calloutu
> Typ calloutu `[!contradiction]` použitý níže je **vlastní callout** definovaný v `.obsidian/snippets/vault-colors.css` (auto-instalovaný `/wiki` scaffoldem). Renderuje se s červenohnědým stylem a alert-triangle ikonou, je-li snippet zapnutý. Pokud snippet chybí, Obsidian použije výchozí styl calloutu, takže stránka funguje i bez vizuálního zvýraznění.

Když nové info odporuje existující wiki stránce:

Na existující stránku přidej:
```markdown
> [!contradiction] Conflict with [[New Source]]
> [[Existing Page]] tvrdí X. [[New Source]] tvrdí Y.
> Vyžaduje vyřešení. Zkontroluj data, kontext a primární zdroje.
```

Na shrnutí nového zdroje to referencuj:
```markdown
> [!contradiction] Contradicts [[Existing Page]]
> Tento zdroj tvrdí Y, ale existující wiki tvrdí X. Detaily viz [[Existing Page]].
```

Stará tvrzení tiše nepřepisuj. Označ a nech rozhodnout uživatele.

---

## Co NEDĚLAT

- Nic v `raw/` neupravuj. Jsou to neměnné zdrojové dokumenty.
- Nevytvářej duplicitní stránky. Před vytvořením vždy zkontroluj rejstřík a vyhledávání.
- Nevynechávej log záznam. Každý ingest musí být zaznamenán.
- Nevynechávej aktualizaci hot cache. Ta drží budoucí relace rychlé.
