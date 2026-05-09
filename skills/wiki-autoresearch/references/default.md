# Výzkumný program (default)

Tento soubor konfiguruje smyčku autoresearch. Upravte jej tak, aby odpovídal vaší doméně a stylu výzkumu. Skill autoresearch jej načítá před každým spuštěním.

---

## Cíle vyhledávání

Výchozí cíle pro každou výzkumnou relaci:

- Najít autoritativní zdroje (preferovat: .edu, peer-reviewed články, oficiální dokumentaci, primární zdroje, zavedené publikace)
- Extrahovat klíčové entity (osoby, organizace, produkty, nástroje)
- Extrahovat klíčové koncepty a frameworky
- Zaznamenat rozpory mezi zdroji
- Identifikovat otevřené otázky a mezery ve výzkumu
- Preferovat zdroje z posledních 2 let, pokud není téma fundamentální

---

## Hodnocení míry jistoty

U každého tvrzení při ukládání označte míru jistoty:

- **high**: více nezávislých autoritativních zdrojů se shoduje
- **medium**: jeden dobrý zdroj nebo částečná shoda zdrojů
- **low**: spekulace, názorový text, jeden neformální zdroj nebo neověřené tvrzení

U faktických tvrzení vždy uvádějte datum zdroje. Tvrzení ze zdrojů starších než 3 roky označte jako potenciálně zastaralá.

---

## Omezení smyčky

- Maximální počet kol vyhledávání na téma: **3**
- Maximální počet wiki stránek vytvořených za relaci: **15**
- Maximální počet zdrojů stažených v jednom kole: **5**
- Pokud je dosaženo maxima stránek před dokončením smyčky: založte, co máte, a co bylo vynecháno, poznamenejte do sekce Open Questions

---

## Styl výstupu

- Oznamovací způsob, přítomný čas
- Každé netriviální tvrzení opatřete citací: `(Source: [[Page]])`
- Krátké stránky: pod 200 řádků. Delší rozdělte.
- Žádné vyhýbavé formulace („zdá se", „možná", „pravděpodobně")
- Nejistotu označujte explicitně: `> [!gap] Toto tvrzení vyžaduje ověření.`

---

## Poznámky k doméně

[Zde doplňte instrukce specifické pro vaši doménu. Příklady:]

Pro výzkum v oblasti AI/tech:
- Preferovat: arXiv, oficiální GitHub repozitáře, oficiální produktovou dokumentaci, diskuze na Hacker News s vysokou karmou
- Pozor: benchmarky LLM bývají často „ohýbány" — tvrzení z leaderboardů považujte za málo důvěryhodná, dokud nejsou nezávisle ověřena

Pro byznysový a tržní výzkum:
- Preferovat: výroční zprávy a regulatorní podání společností (company filings), Crunchbase, Bloomberg, ověřené oborové reporty
- Flagovat: tiskové zprávy jako málo důvěryhodné bez nezávislého ověření

Pro lékařský a zdravotnický výzkum:
- Preferovat: PubMed, Cochrane reviews, peer-reviewed klinické studie
- Vždy poznamenat: velikost vzorku, typ studie (RCT vs. observační) a recentnost

---

## Vyloučení

Jako zdroje s vysokou mírou jistoty necitujte:
- Příspěvky na Redditu nebo fórech (využívejte je pouze jako odrazový můstek k primárním zdrojům)
- Příspěvky na sociálních sítích
- Nedatované webové stránky
- Zdroje, které samy necitují, odkud čerpají
