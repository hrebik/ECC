# EBB pilot: inventura agentní vrstvy napříč worktree

Datum: 2026-10-16. Stav: **zadání ke schválení.** Implementace nezačala.
Nadřazené dokumenty: [EBB-ASSIGNMENT.md](EBB-ASSIGNMENT.md), [EBB-BRIEF.md](../EBB-BRIEF.md).

Účel pilotu je dvojí: získat reálnou inventuru a zároveň otestovat sestavu
Claude Code (lead) / Codex (oponent) / Muse Spark (worker) na kódu, ne na dokumentaci.

## 1. Identita a místo práce

- Název: `ebb-inventory`
- Vlastník: `jiris`
- Repo a větev: `hrebik/ECC`, `hrebik/Ebb-Development`
- Umístění kódu: **PROPOSE** — `tools/ebb-inventory/` v tomto worktree.
  Důvod pro samostatný adresář místo `scripts/`: tento repozitář je fork
  upstreamu `affaan-m/ECC`, který se bude chtít mergovat. Vlastní kód
  v namespacovaném adresáři přežije merge bez konfliktů; `scripts/` ne.
- Paperclip ID: **OPEN** — nepropojeno

## 2. Problém a cíl

Napříč 29 Orca worktree ve 13 repozitářích dnes neexistuje přehled o tom,
jaké skills, agenty, hooks a MCP který z nich má. Před jakýmkoli zaváděním
sdílené ECC vrstvy je to nutný vstup: bez něj nelze říct, co chybí,
co se duplikuje a co by se rozbilo.

Měřitelný cíl: jedním příkazem vznikne strojově čitelná i lidsky čitelná
inventura všech worktree, kterou lze použít jako podklad rozhodnutí.

## 3. Scope

**MUST**
1. Enumerovat worktree a jejich cesty.
2. Pro každý zjistit přítomnost a obsah agentní konfigurace.
3. Vypsat výsledek jako JSON a jako čitelnou tabulku.
4. Porovnat proti definovanému baseline a označit mezery.

**LATER** — historie v čase, návrh sjednocení, automatické doplňování chybějícího.

**Mimo rozsah** — jakýkoli zápis mimo vlastní výstupní soubor. Nástroj je read-only.
Neinstaluje, needituje cizí worktree, nevolá síť, nečte `auth.json` ani credentials.

## 4. Funkční požadavky

```text
REQ-01: Enumerace worktree
Vstup: `orca worktree list --json`; při nedostupnosti Orcy explicitní seznam cest z configu.
Výsledek: seznam {displayName, path, branch, repo}.
Chyba: Orca neběží → srozumitelná hláška + návod na fallback, exit != 0. Nikdy tichý prázdný seznam.

REQ-02: Sken agentní konfigurace jednoho worktree
Zjišťuje: .claude/ (skills, agents, commands, hooks, settings.json),
.codex/ (config.toml, AGENTS.md, skills), .agents/, .cursor/,
kořenové AGENTS.md a CLAUDE.md, .mcp.json a MCP servery z .codex/config.toml.
Výsledek: počty + názvy položek per kategorie.
Chyba: nedostupná cesta nebo odepřené právo → položka označena jako `unreadable`
s důvodem, sken pokračuje dál. Jeden nečitelný worktree neshodí celý běh.

REQ-03: Výstup
`--json` strojový výstup na stdout; bez přepínače čitelná tabulka.
Invariant: oba výstupy pocházejí z téhož skenu, nikdy se nerozcházejí.

REQ-04: Porovnání proti baseline
Baseline je deklarativní soubor (seznam očekávaných skills/hooks/souborů).
Výsledek: per worktree seznam chybějícího a přebývajícího; duplicity napříč worktree.
Chyba: chybějící nebo nevalidní baseline → běh bez porovnání + varování, ne pád.

REQ-05: Globální kontext
Načíst i ~/.claude a ~/.codex (skills, plugins, marketplaces) jako referenční řádek.
Invariant: nikdy nečíst ani nevypisovat obsah auth.json, tokenů či klíčů.
```

## 5. Tech stack

- **FIXED** Node.js >= 18, CommonJS, bez TypeScript a bez transpilace —
  konvence `.claude/rules/node.md` tohoto repozitáře.
- **FIXED** Testy v `node:test`, spouštěné stejným způsobem jako zbytek repa.
- **FIXED** Žádné nové runtime závislosti. Pouze stdlib + volání `orca` CLI.
- **FIXED** Cross-platform cesty (Windows je primární runtime).

## 6. Akceptační kritéria

| ID | REQ | Given / When / Then | Jak ověřit | Důkaz |
|---|---|---|---|---|
| AC-01 | REQ-01 | Orca běží / spustím nástroj / vrátí všech 29 worktree s cestou a větví | integration | výstup vs `orca worktree list` |
| AC-02 | REQ-01 | Orca neběží / spustím nástroj / srozumitelná chyba, nenulový exit, žádný prázdný úspěch | unit, mock | test |
| AC-03 | REQ-02 | Worktree s `.claude/skills` / sken / správné počty i názvy | unit, fixture | test |
| AC-04 | REQ-02 | Jeden worktree je nečitelný / sken / označen `unreadable`, ostatní dokončeny | unit, fixture | test |
| AC-05 | REQ-03 | Tentýž sken / `--json` i tabulka / stejná data v obou | unit | test |
| AC-06 | REQ-04 | Baseline chybí / spuštění / varování a běh bez porovnání, exit 0 | unit | test |
| AC-07 | REQ-05 | Jakýkoli běh / kontrola výstupu / nikde se neobjeví obsah auth.json ani token | unit + review | test + oponentura |
| AC-08 | REQ-02 | Reálný běh na 29 worktree / spuštění / dokončí bez pádu | ruční | uložený výstup |

Coverage: ≥ 80 % větví, funkcí, řádků a statements — požadavek `tdd-guide`.
Definice hotovo: AC-01 až AC-08 splněna, testy zelené, oponentura od Codexu
proběhla a její nálezy jsou vypořádané, známé limity zapsané.

## 7. Dělba práce a autorizace

| Krok | Kdo | Výstup |
|---|---|---|
| 1. Rešerše formátů konfigurace napříč harnessy | Muse (worker) | strukturovaný přehled, co kde hledat |
| 2. Plán a rozhodnutí o rozhraní | Claude Code (lead) | kontrakt modulů |
| 3. Oponentura zadání a kontraktu | Codex (oponent) | nálezy před psaním kódu |
| 4. RED → GREEN → refaktor | Claude Code (lead) | kód + testy |
| 5. Nezávislý review hotové implementace | Codex (oponent) | review verdikt |
| 6. Reálný běh a uložení výstupu | Claude Code (lead) | inventura 29 worktree |

Autorizace: nástroj je read-only. Commit lokálně; **push až po souhlasu `jiris`**.
Žádné změny v cizích worktree, v `~/.config/opencode`, Hermes configu ani
v plánovaných úlohách.

## 8. Otevřené otázky

- OQ-01: Co přesně má být v baseline? Blokuje REQ-04, neblokuje REQ-01 až REQ-03.
  Návrh: první verze baseline vznikne až *z* inventury, ne před ní.
- OQ-02: Má nástroj zahrnout i 13 `main` worktree mimo Orca workspaces,
  nebo jen těch 16 pracovních? Návrh: zahrnout vše, filtrovat přepínačem.
