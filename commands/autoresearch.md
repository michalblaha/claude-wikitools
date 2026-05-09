---
description: Spustí autonomní výzkumnou smyčku nad zadaným tématem. Prohledá web, syntetizuje nálezy a založí vše do wiki jako strukturované stránky.
---

Načti skill `wiki-autoresearch`. Poté spusť výzkumnou smyčku.

Použití:
- `/autoresearch [topic]` — výzkum konkrétního tématu
- `/autoresearch` — zeptej se „K jakému tématu mám provést výzkum?"

Před začátkem si načti `skills/wiki-autoresearch/references/default.md` (nebo `gov-project.md`, pokud je vyžádán) pro nahrání omezení a cílů výzkumu.

Pokud ještě není nastaven žádný vault, řekni: „Nenalezen wiki vault. Nejprve spusť `/wiki` pro nastavení."

Po dokončení výzkumu aktualizuj `wiki/index.md`, `wiki/log.md` a `wiki/hot.md`.

Nahlas, kolik stránek bylo vytvořeno a jaké jsou klíčové nálezy.
