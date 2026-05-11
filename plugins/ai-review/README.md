# ai-review

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-1.2.3-blue)](.claude-plugin/plugin.json)

Nezávislé AI review workflows — **druhý názor** a **cross-check** napříč Claude (Anthropic), Codex (OpenAI) a Gemini (Google).

Když si Claude (nebo Codex, nebo Gemini) sám/sama validuje vlastní výstup, sdílí stejné slabiny. ai-review abstrahuje volání tří hlavních AI CLI nástrojů, takže ověřovatelem je vždy model **od jiného providera**.

> Součást marketplace [`michalblaha-ai-plugins`](../../README.md). Funguje samostatně i jako companion k pluginu [`wiki-tools`](../wiki-tools/) (fact-check během ingestu, deep validace u dotazů).

---

## Instalace

### Claude Code

```bash
claude plugin marketplace add michalblaha/michalblaha-ai-plugins
claude plugin install ai-review@michalblaha-ai-plugins
```

### Codex CLI (0.130+)

```bash
codex plugin marketplace add michalblaha/michalblaha-ai-plugins
```

Aktivace v `~/.codex/config.toml`:

```toml
[plugins."ai-review@michalblaha-ai-plugins"]
enabled = true
```

### Předpoklady

Skill předpokládá nainstalované relevantní CLI nástroje na lokálním stroji:

- [Claude Code CLI](https://code.claude.com/) — `claude`
- [Codex CLI](https://github.com/openai/codex) — `codex`
- [Gemini CLI](https://github.com/google-gemini/gemini-cli) — `gemini`

Stačí mít alespoň jeden cizí model dostupný (vůči tomu, který skill spouští).

---

## Skills

| Skill | Kdy použít |
|-------|-----------|
| `second-opinion` | Validace architektonického rozhodnutí, bezpečnostní audit kódu, porovnání implementačních přístupů, multi-model konsenzus pro rozhodnutí s vyššími stakes |
| `double-cross-check` | Ověření faktografických tvrzení, investigativní rešerše, fact-check — typicky jako poslední krok před publikací nebo akcí |

### Triggery

- „získej druhý názor", „ověř z jiného modelu"
- „zkontroluj přes Codex / Gemini / Claude Code"
- „cross-check", „dvojitá kontrola", „fact-check", „ověř fakta"
- „porovnej názory modelů"

---

## Princip identifikace ověřovatele

Skill nejprve určí, který model ho právě spouští, a vybírá ověřovatele **z těch zbývajících**:

- Pokud jsi **Claude** → ověřuje **Codex** a/nebo **Gemini**
- Pokud jsi **Codex/GPT** → ověřuje **Claude** a/nebo **Gemini**
- Pokud jsi **Gemini** → ověřuje **Codex** a/nebo **Claude**

Nikdy se neověřuje stejným modelem, který odpověď vyrobil.

---

## Kdy NE

Skill **nepoužívej** pro běžné dotazy, kde stačí jedna odpověď — je drahý a pomalý. Cílené použití u rozhodnutí s vyššími stakes nebo u faktografie před publikací.

---

## Integrace s wiki-tools

Pokud je nainstalován společně s `wiki-tools`, wiki skills mohou cíleně využít `ai-review`:

- **`wiki-ingest`** — pro ověření faktografických tvrzení s vyšším dopadem (`double-cross-check`); cíleně, ne pro každý odstavec (skill je drahý a pomalý)
- **`wiki-query`** — v deep módu pro nezávislou validaci syntézy od jiného modelu (`second-opinion`)

Integrace je volitelná — `wiki-tools` funguje plně i bez `ai-review` a wiki skills v takovém případě pokračují bez externí validace.

---

## Další dokumentace

- [Marketplace README](../../README.md) — kompletní dokumentace všech pluginů
- [`skills/second-opinion/SKILL.md`](skills/second-opinion/SKILL.md) — detailní bash patterny pro Codex / Gemini / Claude Code CLI
- [`skills/double-cross-check/SKILL.md`](skills/double-cross-check/SKILL.md) — workflow pro fact-check a investigativní rešerše

---

*MIT License.*
