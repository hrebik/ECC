# EBB: ECC s Codexem jako primárním harness

Datum: 2026-09-27. Stav: audit a návrh, bez instalace či změn runtime konfigurace.
Zdroj: EBB-BRIEF.md, následné upřesnění uživatele a lokální repozitář 2.2.2.
Paperclip projekt/úkol ani Herdr workspace nebyly v této relaci propojeny;
dokument je datovaný podklad, nikoli autorita pro aktuální pracovní stav.

## Závěr a doložené hranice

Codex může být primární engineering orchestrátor a runtime pro ECC workflow.
ECC dodává instrukce, skills a pomocné mechanismy; není náhradní runtime.
Nelze tvrdit plnou automatizační paritu s Claude Code ani z počtu skills
odvozovat, že všechny závislosti jsou dostupné v konkrétní relaci.

| Oblast | Co je nativně použitelné | Co není plná parita |
|---|---|---|
| Instrukce | Root AGENTS.md, explicitně přečtený .codex/AGENTS.md supplement | CLAUDE.md je Claude-specific; nelze spoléhat na jeho automatické načtení Codexem. Supplement v .codex není univerzálně automaticky root instrukcí |
| Skills | Native plugin odkazuje na skills/; repo discovery na .agents/skills/ | Lokálně 292 SKILL.md vs 39 Codex-facing kopií; dostupný katalog není záruka funkčních externích nástrojů či shellů |
| MCP | Nativní plugin deklaruje chrome-devtools v .mcp.json | .codex/config.toml deklaruje jiný set šesti serverů; nekopírovat baseline slepě ani neslibovat jejich připojení |
| Agenti | Codex multi-agent role explorer, reviewer, docs_researcher | 68 Claude-style promptů v agents/ není 68 automaticky registrovaných Codex rolí; model/tool metadata se musí přizpůsobit |
| Commands | Workflow lze vykonat přes skill či přečtené instrukce | Claude slash-command shimy nejsou automaticky Codex příkazy |
| Hooks | hooks/codex-hooks.json obsahuje pouze SessionStart bootstrap | Žádná plná projekce Claude Pre/PostToolUse, Stop, PreCompact či SessionEnd; čtyři Claude hook profily nejsou Codex profily |
| Memory/instincts | Existující obsah lze číst a využít; SessionStart obsahuje načítání kontextu | Automatické pozorování, učení a zápisy z dalších Claude hooků tím nejsou zapojené. Explicitní memory workflow je samostatná možnost |
| Bezpečnost | Instrukce a skutečné Codex runtime policy; hook trust spravuje Codex | Instrukce nejsou tvrdé vynucení. Repo reference workspace-write/on-request neodpovídá této relaci s unrestricted/never; konfigurace nebyla měněna |

Zdroje: [platform matrix](../README.md#platform-support),
[navigation guide](CODEX-NAVIGATION-GUIDE.md),
[Codex supplement](../.codex/AGENTS.md),
[manifest](../.codex-plugin/plugin.json),
[native plugin notes](../.codex-plugin/README.md),
[hook projection](../hooks/codex-hooks.json).
Také [oficiální OpenAI dokumentace](https://developers.openai.com/plugins/build/plugins)
potvrzuje, že instalace pluginu neuděluje trust jeho hookům.
Review/trust musí zůstat Codex-owned; žádný ECC trust bypass.

## Dry-run a skutečný runtime

- `codex --version`: `codex-cli 0.157.1` na Windows.
- `npx --no-install ecc-universal install --guided --harness codex --dry-run`
  nejprve skončil CODEX_NOT_FOUND. Stejnou chybu reprodukoval lokální
  `node scripts/ecc.js install --guided --harness codex --dry-run`.
- PowerShell našel `C:/Users/jiris/bin/codex.cmd`; instalační knihovna používá
  execFile('codex', ..., shell:false), což zde shim nerozpoznalo.
- Po přidání adresáře již existujícího codex.exe pouze do PATH procesu prošel
  tentýž npx dry-run. Žádný trvalý PATH, profil, auth ani sandbox se neměnil.
- Lokální JSON dry-run potvrdil `would-install`, `would-add`, plugin `ecc@ecc`,
  `restartRequired: true`. To je plán, nikoli provedená instalace.
- `--no-install` zabránilo npx stahovat chybějící balíček. Příkaz byl spuštěn
  v tomto repozitáři; závěr se nevztahuje na libovolnou budoucí npm verzi.
- Aktuální guided cesta je pevně navázaná na `affaan-m/ECC` a kontroluje jeho
  Git původ. Náš remote je `hrebik/ECC`; tento dry-run není plán instalace forku.
- Výsledek platí pro dostupný Windows runtime, nikoli inventuru kanonického
  WSL `/home/jirka/.codex`. Tu je nutné před budoucí integrací ověřit samostatně.

## Připravený integrační plán

1. **Vymezit novou platformu a její umístění.** Vyplnit
   [zadání](EBB-ASSIGNMENT-TEMPLATE.md); rozhodnout, zda je ECC worktree místem
   produktového kódu nebo integračního návrhu. Doména ani stack se nepředpokládá.
2. **Inventura a governance.** Dohledat existující Paperclip/Git/Herdr vazby,
   zjistit skutečné spouštění Codexu a aktivní home ve WSL/Windows. Inventarizovat
   pluginy, skills a legacy sync bez auth výpisů. Nezakládat další framework.
3. **Sjednotit instrukce a dokumentaci.** AGENTS.md pro společné konvence;
   CLAUDE.md pouze Claude doplněk; Codex supplement explicitně dosažitelný.
   Upravit navigation guide a README tak, aby oddělily plugin, repo discovery
   a legacy sync, popsaly jediný SessionStart hook a rozlišily nativní načtení
   od funkčnosti workflow. Opravit zastaralé 281 skills, experimental/stable
   tvrzení a rozdílné MCP baselines. Změny teprve připravit v samostatném diffu.
4. **Zvolit distribuční zdroj.** Preferovat nativní plugin, nekombinovat ho
   s legacy sync. Pro fork nejdříve prověřit lokální marketplace cestu popsanou
   v plugin README a konflikt názvu ecc; upstream-only guided flow neměnit
   obcházením kontroly původu. Případnou podporu forku navrhnout explicitně.
5. **Izolované ověření po autorizaci instalace.** Bez kopírování credentials
   ověřit plugin cache, skill discovery, relevantní MCP, role a SessionStart.
   Trust ručně přes Codex /hooks; enablement přes /plugins je jiná operace.
   Otestovat čistou instalaci, opakování, update, změnu hook definice,
   odmítnutý trust a uninstall; chránit existující uživatelské soubory.
6. **Malý pilot.** Vybraný inkrement nové platformy projde
   planner/architect → implementer/TDD → tester → samostatný reviewer → Codex
   integrace → context curator. Paměť a handoff nejprve explicitně do Git;
   automatické continuous-learning neslibovat. Runtime workers podle Herdr
   politiky, dokud uživatel neurčí jinak.
7. **Teprve potom schválené připojení.** Předložit konkrétní diff, výsledky,
   cílový home, source revision, rollback a rozsah oprávnění. Zachovat sdílený
   WSL Codex home, nevytvářet další login. Napojit výsledek na Paperclip.

Windows-native observer/memory omezení uvedená v platform matrix jsou důvodem
ověřit příslušné cesty na reálném runtime; nejsou důvodem automatické migrace.

## Ověření a GLM handoff

- Lead: docs navigation test 3/3 PASS; native plugin setup test 23/23 PASS
  (včetně inventory-only dry-run, kontroly původu a fail-closed chování).
- GLM hlásí codex-config 5/5 PASS; scripts/codex-hooks test se nespustil kvůli
  chybějícímu `@iarna/toml`. Závislost se neinstalovala; plné hook ověření
  proto není prokázané. Dokumentační testy neověřují pravdivost všech tvrzení.
- GLM přes orca-cli potvrdil přijetí a předal read-only audit EBB-CODEX-AUDIT.
  Změněné soubory/rozhraní: žádné. Zjištění: rozdíly katalogů/MCP,
  jediný SessionStart, zastaralé počty a multi-agent status. Lead je porovnal
  se zdrojovými soubory. Absolutní tvrzení „Codex nepíše paměť“ se nepřebírá:
  chybí automatické zapojení konkrétních hooků, nikoli schopnost explicitního zápisu.
- GLM dokončil EBB-INPUT-CONTRACT: vstupy planner/architect/TDD a tři rizika
  (nejasné rozhodování, přebírání ukázkového stacku, TDD bez kontraktu).
  Bez změn souborů/rozhraní a bez testů. Doporučení jsou zapracována do šablony.
  Lead nepřebírá navržené povinné schvalování každého handoffu: lidské rozhodnutí
  je nutné pro skutečně otevřené produktové/architektonické otázky a akce
  vymezené politikou, nikoli pro každý běžný již autorizovaný krok.
- Existující `.opencode/package.json`, `.opencode/package-lock.json` a
  neversionovaný EBB-BRIEF.md nebyly měněny. Žádný commit/push/deployment.
- Kontrola lokálních odkazů a Markdown fences obou nových dokumentů PASS;
  kontrola whitespace bez nálezů. Lead provedl obsahovou kontrolu proti
  přečteným promptům. Dodatečný EBB-DOCS-REVIEW byl přijat terminálem, ale jeho
  spuštění/výsledek není potvrzené: GLM současně odpovídá uživateli. Neuvádíme
  proto finální nezávislé schválení dokumentů a jeho relaci nepřerušujeme.
- WORKERS_SPAWNED=0; WORKERS_CLOSED=0; WORKERS_PRESERVED=1
  (PRESERVE: uživatelem dodaný GLM terminál, nevytvořený tímto HEAD).
  CLEANUP_COMPLETE=true pro pracovníky vytvořené tímto HEAD (žádní).
