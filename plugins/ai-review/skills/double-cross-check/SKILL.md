---
name: double-cross-check
description: "Ověř výsledek nezávislým modelem od jiného providera — Codex (OpenAI), Gemini (Google) nebo Claude Code (Anthropic). Použij na fráze: 'získej druhý názor', 'ověř z jiného modelu', 'zkontroluj přes Codex/Gemini/Claude Code', 'porovnej názory modelů', 'cross-check', 'dvojitá kontrola', 'fact-check', 'ověř fakta'. Vhodné pro ověřování faktografických dat a investigativní rešerše, kontrolu architektonických rozhodnutí, bezpečnostní audit kódu, porovnávání implementačních přístupů a multi-model konsenzus u rozhodnutí s vyššími stakes. Pro běžný dotaz, kde stačí jedna odpověď, skill nepoužívej — je drahý a pomalý. Používej cíleně."

allowed-tools:
  - AskUserQuestion
  - Read
  - Glob
  - Grep
  - WebSearch
  - WebFetch
  - Bash(command -v *)
  - Bash(codex exec:*)
  - Bash(claude -p:*)
  - Bash(gemini -p:*)
  - Bash(mktemp:*)
  - Bash(mkdir:*)
  - Bash(cat:*)
  - Bash(echo:*)
  - Bash(git diff:*)
  - Bash(git log:*)


argument-hint: "otázka / soubor / rozhodnutí k ověření"

---

# Double cross-check: kontrola jiným AI modelem

Získej nezávislý pohled od OpenAI (přes Codex CLI), Google (přes Gemini CLI) nebo Anthropic (přes Claude Code CLI) na výsledek, který sám/sama produkuješ. Smysl: chytit chyby, na které jsi slepý/á — ne potvrdit si vlastní odpověď.

---

## 1. Identifikuj sebe sama

Než začneš, urči, který model skill právě spouští, podle prostředí (system prompt, název CLI, autentizační kontext). Ověřovatele vybírej **z těch zbývajících**:

- Pokud jsi **Claude** → ověřuje **Codex** a/nebo **Gemini**.
- Pokud jsi **GPT/Codex** → ověřuje **Claude** a/nebo **Gemini**.
- Pokud jsi **Gemini** → ověřuje **Codex** a/nebo **Claude**.

Nikdy se neověřuj stejným modelem, který odpověď vyrobil — sdílí stejné slabiny.

---

## 2. Předpoklady prostředí

Skill předpokládá shellové prostředí s nainstalovanými relevantními CLI nástroji (typicky lokální stroj s Claude Code, Codex CLI nebo Gemini CLI). **V prostředí bez shellu** (čistý chat bez bash toolu) skill nelze použít — místo něj nabídni uživateli, ať verifikační prompt pošle do jiného modelu ručně, nebo použij web search pro grounding faktů.

Než začneš, ověř dostupnost:

```bash
codex --version  || echo "Codex CLI není nainstalované"
gemini --version || echo "Gemini CLI není nainstalované"
claude --version || echo "Claude Code CLI není nainstalované"
```

**Autentizace:**
- **Codex:** `codex login` nebo env var `OPENAI_API_KEY`
- **Gemini:** Google OAuth nebo env var `GEMINI_API_KEY`
- **Claude Code:** `claude login` nebo env var `ANTHROPIC_API_KEY`

**Workdir:** všechny příklady níže používají `WORK=$(mktemp -d)` na začátku a pak `$WORK/...` pro mezivýstupy. Vyhne se to kolizím při paralelním běhu a uklidí to po sobě (na rozdíl od pevného `/tmp/claude/`).

**Codex — povinný flag `--skip-git-repo-check`:** Při každém volání `codex exec ...` z tohoto skillu **vždy** přidej `--skip-git-repo-check`. Codex defaultně odmítne běžet mimo git repo (safety check proti nechtěným editům cizích souborů), ale v tomto skillu Codex pouze vrací odpověď do `--output-last-message` a nic nezapisuje, takže check je zbytečný a CWD při volání z Claude může být libovolné. Bez tohoto flagu volání selže s chybou *"not inside a Git repository"*.

**Fallback při selhání volání Codexu:** Pokud `codex exec` přesto selže (timeout, autentizační problém, runtime error), zkus delegovat ověření na `codex:codex-rescue` skill / subagent. Ten má vlastní robustní orchestraci přes `codex-companion.mjs` helper a obejde řadu okrajových případů. Použij jen na jeden retry; pokud selže i tak, přepni na jiného providera (Gemini nebo Claude) a poznač to v sekci *Nejistoty* finální odpovědi.

---

## 3. Workflow

1. **Získej ověření** od jednoho nebo více jiných modelů. Pro běžný cross-check stačí jeden; multi-model konsenzus si nech na rozhodnutí s vyššími stakes (publikovatelný výstup, bezpečnostní rozhodnutí, faktografická tvrzení v investigativním textu).
2. **Pročti zjištění bod po bodu.** Pro každou údajnou chybu: ověř ji nezávisle (zdroj, dokumentace, web search, vlastní úvaha) — kontrolor se může mýlit stejně jako autor. Zapracuj jen ty, které ti potvrzená data podpoří.
3. **Re-check.** Opravenou verzi pošli stejnému (nebo jinému) kontrolorovi znovu.
4. **Iterace max. 3×.** Skonči dřív, pokud kontrola nevrací žádné nové připomínky. Po 3 iteracích **shrň zbývající otevřené body uživateli k rozhodnutí** — neztrácej se v nekonečném ladění drobností, na kterých se modely nedohodnou.


---

## 4. Kdy AI cross-check nestačí

AI modely sdílejí trénovací data a s nimi i halucinace. Když ověřuješ **fakt** (existence osoby/účtu, datum, hodnota, citát, právní/historický nárok), nejsilnější kontrola není další model, ale **nezávislý zdroj**:

- Pro veřejné instituce a osoby: oficiální registry, Wikidata, parlamentní/justiční databáze.
- Pro novější fakta: web search s důrazem na primární zdroj.
- Pro kód a knihovny: oficiální dokumentace, repo, changelog.

Použij AI cross-check pro **uvažování, strukturu, logické skoky, code review, návrhová rozhodnutí**. Pro fakta ho použij jen jako první filtr a pak ověř proti zdroji.

---

## 5. Prompt engineering pro silnější verifikaci

Slabý prompt → slabá kontrola. Místo "Je tohle správně?" použij vstup uživatele a doporučený prompt.


### Doporučený prompt pro externí model

Použij tento tvar zadání a doplň jen nezbytný kontext:

(Pokud je `$ARGUMENTS` prázdné, ověř výstup z předchozího kola konverzace. Pokud je to cesta k souboru, načti ho. Pokud je to volný popis úlohy, použij ho jako kontext.)

```text
Posuď následující zadání kriticky. Nehledej potvrzení, hledej chyby.

Zaměř se na:
- chybné nebo slabé předpoklady,
- bezpečnostní a provozní rizika,
- opomenuté alternativy,
- nesoulad s běžnou praxí,
- tvrzení, která potřebují primární zdroj nebo test.

U každého nálezu uveď:
- závažnost,
- konkrétní důvod,
- jak nález ověřit,
- doporučenou opravu nebo další krok.

Zadání:
$ARGUMENTS

a přidej potřebný kontext, pokud je potřeba.
```


---

## 6. Rychlé vzory volání

### Codex / OpenAI

```bash
WORK=$(mktemp -d)

# Jednoduchá otázka
codex -a never exec --skip-git-repo-check -m gpt-5.5 -c 'model_reasoning_effort="high"' \
  --output-last-message "$WORK/answer.txt" \
  "Tvá otázka tady"
cat "$WORK/answer.txt"

# Strukturovaný výstup s validací JSON Schema
cat > "$WORK/schema.json" <<'EOF'
{
  "type": "object",
  "properties": {
    "assessment":     { "type": "string" },
    "strengths":      { "type": "array", "items": { "type": "string" } },
    "concerns":       { "type": "array", "items": { "type": "string" } },
    "recommendation": { "type": "string" }
  },
  "required": ["assessment", "strengths", "concerns", "recommendation"],
  "additionalProperties": false
}
EOF

codex -a never exec --skip-git-repo-check -m gpt-5.5 -c 'model_reasoning_effort="high"' \
  --output-schema "$WORK/schema.json" \
  --output-last-message "$WORK/result.json" \
  "Analyzuj [téma]. Vrať strukturované zhodnocení."
cat "$WORK/result.json"
```

**Pozn. ke schématu:** všechny objekty (i vnořené) musí mít `"additionalProperties": false` a `"required": [...]` se všemi vlastnostmi. Jinak Codex odmítne.

**Pozn. k `--skip-git-repo-check`:** flag je povinný pro každé volání Codexu z tohoto skillu (viz sekce 2). Pokud volání i s ním selže, fallback je delegace na `codex:codex-rescue`.

### Gemini / Google

```bash
WORK=$(mktemp -d)

# Jednoduchá otázka — auto-routing vybere model
gemini -p "Tvá otázka tady" --output-format text > "$WORK/answer.txt"
cat "$WORK/answer.txt"

# Explicitní výběr modelu
gemini -m gemini-3-pro -p "Tvá otázka tady" \
  --output-format text > "$WORK/answer.txt"

# JSON výstup (bez schema validace)
gemini -p "Analyzuj [téma]. Odpověz JSONem s klíči assessment (string), strengths (array), concerns (array), recommendation (string)." \
  --output-format json > "$WORK/result.json"
cat "$WORK/result.json"
```

**Pozn.:** Gemini CLI nepodporuje validaci JSON Schema. Pro přísně strukturovaný výstup s validací sáhni po Codexu s `--output-schema`.

### Claude Code / Anthropic

```bash
WORK=$(mktemp -d)

# Jednoduchá otázka (alias 'opus' = nejlepší dostupný Opus dle providera)
claude -p "Tvá otázka tady" --model opus --output-format text > "$WORK/answer.txt"
cat "$WORK/answer.txt"

# Konkrétní pin verze (reprodukovatelnost)
claude -p "Tvá otázka tady" --model claude-opus-4-7 \
  --effort xhigh --output-format text > "$WORK/answer.txt"

# Levnější / rychlejší pro jednoduché úlohy
claude -p "Tvá otázka tady" --model haiku --output-format text > "$WORK/answer.txt"

# JSON výstup (zabalí odpověď do strukturovaného message objektu)
claude -p "Analyzuj [téma]. Odpověz JSONem s klíči assessment, strengths, concerns, recommendation." \
  --model opus --output-format json > "$WORK/result.json"
cat "$WORK/result.json"
```

**Pozn.:** Spuštěná instance běží nezávisle na aktuální session — kontrolor nemá tvůj kontext, dostává opravdu čerstvý pohled. Aliasy (`opus`, `sonnet`, `haiku`) ukazují na aktuálně doporučenou verzi pro daný provider; pro reprodukovatelnost pinuj `claude-opus-4-7`, `claude-sonnet-4-6`, `claude-haiku-4-5-20251001`.

---

## 7. Příklady použití

### Faktografická verifikace (typický investigativní use-case)

```bash
WORK=$(mktemp -d)

claude -p "Ověř následující tvrzení. Pro každé řekni: pravdivé / nepravdivé / nelze ověřit + důkaz nebo důvod nejistoty. Buď skeptický, nepředpokládej.

Tvrzení:
$(cat tvrzeni.txt)

Pokud potřebuješ pro ověření primární zdroj a nemáš ho, řekni 'potřebuji zdroj X' místo odhadu." \
  --model opus --effort high --output-format text > "$WORK/factcheck.txt"

cat "$WORK/factcheck.txt"
```

**Důležité:** model nemá přístup k živým zdrojům, takže "pravdivé" znamená "konzistentní s trénovacími daty modelu" — ne "ověřeno proti realitě". Pro fakta novější než cutoff modelu, nebo pro tvrzení o konkrétních osobách/účtech/datech, zkombinuj s web search nebo strukturovaným zdrojem (Wikidata, psp.cz, ARES, justiční databáze atd.).

### Cross-reference strukturovaného datasetu

```bash
WORK=$(mktemp -d)

codex -a never exec --skip-git-repo-check -m gpt-5.5 -c 'model_reasoning_effort="high"' \
  --output-last-message "$WORK/dataset_review.txt" \
  "Mám dataset přiřazení X účtů osobám (CSV níže). Najdi:
   - podezřelé záznamy (false-positive matching, různé osoby stejné jméno)
   - chybějící atribuce (osoba bez účtu — opravdu žádný, nebo nedohledán?)
   - nesrovnalosti mezi sloupci
   - vzorky vyžadující ruční ověření

   Pro každý nález: ID řádku + důvod podezření + konkrétní návrh, jak ověřit.

   Data:
   $(cat dataset.csv)"

cat "$WORK/dataset_review.txt"
```

### Kontrola architektury

```bash
WORK=$(mktemp -d)
gemini -p "Posuď toto architektonické rozhodnutí: [popis].
  Buď skeptický. Hodnoť: škálovatelnost, udržovatelnost, bezpečnostní rizika, alternativy.
  Pro každou výhradu: konkrétní scénář, kde rozhodnutí selže." \
  --output-format text > "$WORK/arch.txt"
cat "$WORK/arch.txt"
```

### Bezpečnostní audit

```bash
WORK=$(mktemp -d)
codex -a never exec --skip-git-repo-check -m gpt-5.5 -c 'model_reasoning_effort="xhigh"' \
  --output-last-message "$WORK/security.txt" \
  "Bezpečnostní review následujícího kódu:

   $(cat src/auth.py)

   Hledej: vstupní validaci, autentizaci/autorizaci, expozici dat, race conditions, injection.
   Pro každou nalezenou zranitelnost: lokace (řádek), exploit scénář, oprava."
cat "$WORK/security.txt"
```

### Code review

```bash
WORK=$(mktemp -d)
claude -p "Review následujícího diffu na: bugy, výkonnostní problémy, udržovatelnost, test coverage gaps.
  Pro každou připomínku: lokace + závažnost (blocker / major / minor / nit) + návrh opravy.
  Buď skeptický k autorově optimismu.

  Diff:
  $(git diff main..HEAD -- src/)" \
  --model opus --effort high --output-format text > "$WORK/review.txt"
cat "$WORK/review.txt"
```

---

## 8. Co dělat při neshodě modelů

Když si Codex a Gemini protiřečí, neber žádný z nich za autoritu:

1. **Konkrétnější tvrzení s odkazem na zdroj > obecné tvrzení.** Pokud Codex říká "X je špatně, viz CVE-2024-1234" a Gemini říká "X je v pořádku", začni s tím konkrétním a ověř CVE.
2. **Pro fakta zvol nezávislý zdroj** (web search, dokumentace, registry) místo třetího modelu — modely sdílejí halucinace, takže "tie-breaker" třetím modelem je iluze nezávislosti.
3. **Pro názory a preference** (např. "Redis vs PostgreSQL pro session storage") neshoda často odráží reálný trade-off, ne chybu. Surface obě perspektivy uživateli s tím, na čem rozhodnutí závisí.
4. **Pokud spor zůstává nerozhodnutý**, přiznej to v odpovědi explicitně — neforsírovej falešný konsenzus.

---

## 9. Náklady a latence

Multi-model cross-check není zadarmo. Reasoning modely na high/xhigh effort běží desítky sekund a stojí znatelně víc než single call. Doporučení:

- **Single cross-check** (jeden další model): default pro většinu kontrol.
- **Multi-model konsenzus** (všechny tři): nech si na rozhodnutí, kde stojí čas a peníze za to — publikovatelný výstup, faktografická tvrzení v investigativním materiálu, bezpečnostně-citlivý kód, architektonické rozhodnutí s dlouhodobým dopadem.
- **Levnější tier pro rychlé checky:** `gpt-5.4-mini`, `gemini-3-flash`, `claude-haiku-4-5-20251001`.

V Claude Code se vyplatí omezit `--max-turns` (default je bez limitu) a případně `--max-budget-usd`, aby ti neutekly náklady při delší smyčce.

---

## 10. Klíčové volby CLI

### Codex CLI

| Volba | Účel |
|---|---|
| `-m gpt-5.5` | Aktuální frontier model (doporučeno pro hloubkovou kontrolu) |
| `-m gpt-5.5-codex` | Codex-optimalizovaná verze 5.5 pro agentické kódování |
| `-m gpt-5.4-mini` | Rychlý a levný pro jednoduché úlohy |
| `-c 'model_reasoning_effort="high"'` | Reasoning effort. Hodnoty: `low`, `medium`, `high`, `xhigh` |
| `--output-schema file.json` | Strukturovaný JSON s validací schématu (Codex umí) |
| `--output-last-message file.txt` | Uložení odpovědi do souboru |
| `-i image.png` | Přidání obrázku k analýze |

### Gemini CLI

| Volba | Účel |
|---|---|
| Auto-routing (default) | CLI sám volí mezi Pro/Flash dle složitosti |
| `-m gemini-3-pro` | Explicitně Gemini 3 Pro |
| `-m gemini-3-flash` | Explicitně Gemini 3 Flash (rychlejší, levnější) |
| `-m gemini-3.1-pro-preview` | Nejnovější preview (vyžaduje Pro/Ultra účet) |
| `-p "prompt"` | Neinteraktivní režim (povinné pro skripty) |
| `--output-format text\|json` | Formát výstupu (JSON bez schema validace) |

### Claude Code CLI

| Volba | Účel |
|---|---|
| `--model opus` | Alias na nejlepší dostupný Opus (4.7 na API, 4.6 na Bedrock/Vertex) |
| `--model claude-opus-4-7` | Pin konkrétní verze (reprodukovatelnost) |
| `--model sonnet` / `claude-sonnet-4-6` | Vyvážená rychlost a kvalita |
| `--model haiku` / `claude-haiku-4-5-20251001` | Rychlý a levný |
| `--effort xhigh` | Hluboké uvažování (Opus 4.7 default). Hodnoty: `low`, `medium`, `high`, `xhigh`, `max` |
| `-p "prompt"` | Neinteraktivní print mode (povinné pro skripty) |
| `--json-schema file.json` | Strukturovaný JSON s validací schématu (Claude umí) |
| `--max-turns N` | Strop agentních kol (default: bez limitu) |
| `--max-budget-usd N` | Strop útraty na session |
| `--permission-mode bypassPermissions` | Pro CI / non-interactive bez schvalování nástrojů, použij pouze se souhlasem uživatele |

---

## 11. Doporučení modelu podle úlohy

| Úloha | Codex | Gemini | Claude Code |
|---|---|---|---|
| Komplexní rešerše / architektura | `gpt-5.5` + xhigh | `gemini-3.1-pro-preview` nebo auto | `claude-opus-4-7` + xhigh |
| Rychlé code review | `gpt-5.4-mini` | `gemini-3-flash` | `claude-haiku-4-5-20251001` |
| Bezpečnostní audit (hluboký) | `gpt-5.5` + xhigh | auto-routing | `claude-opus-4-7` + xhigh |
| Strukturovaný výstup s validací | `gpt-5.5` + `--output-schema` | — (bez schema validace) | — (bez schema validace) |
| Faktografická verifikace | `gpt-5.5` (kombinuj se zdroji) | `gemini-3-pro` | `claude-opus-4-7` |
| Multi-model konsenzus | spusť všechny tři a porovnej | spusť všechny tři a porovnej | spusť všechny tři a porovnej |

---
## 12. Vyhodnocení a prezentace výsledků

Výstup externího modelu ber jako **hypotézy, ne jako pravdu**. Druhý model může mít stejné slepé místo jako ty — "ověřeno cross-checkem" není totéž co "ověřeno proti realitě".

### Postup vyhodnocení

1. Přečti nálezy externího modelu bod po bodu.
2. Každý konkrétní nález ověř proti dostupným souborům, testům, dokumentaci nebo primárnímu zdroji.
3. Přijmi jen nálezy, které jsou konkrétní, relevantní a ověřené.
4. Odmítni nálezy, které jsou spekulativní, neplatné nebo mimo rozsah.


Nikdy nezapracovávej doporučení jen proto, že ho uvedl jiný model. Zapracuj ho **až po vlastním ověření**.

### Struktura finální odpovědi uživateli

Jasně odděl:

- **Druhý názor** — který provider/model byl použit (např. *"OpenAI / Codex – gpt-5.5"* nebo *"Konsenzus: Codex + Gemini"*) a stručné shrnutí, co řekl.
- **Přijaté nálezy** — co jsi po vlastním ověření přijal/a jako pravdivé. Pro každý: lokace + důkaz nebo zdůvodnění.
- **Odmítnuté nálezy** — co jsi nepotvrdil/a a proč.
- **Změny nebo doporučení** — co bylo provedeno nebo co má následovat.
- **Nejistoty** — co vyžaduje další primární zdroj, test nebo rozhodnutí uživatele.

### Doplňková pravidla

- **Porovnej se svou původní analýzou.** Zdůrazni, kde ses lišil/a od externího modelu.
- **Při více kontrolorech** vyjmenuj oblasti shody i neshody mezi nimi předtím, než přejdeš k vlastnímu vyhodnocení.
- **Pokud spor mezi modely zůstává nerozhodnutý** i po vlastním ověření, nech ho v sekci *Nejistoty* otevřený a popiš, na čem rozhodnutí závisí — neforsírovej falešný konsenzus.

