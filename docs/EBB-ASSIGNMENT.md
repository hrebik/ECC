# EBB: vyplněné zadání nové platformy

Datum založení: 2026-10-16. Stav: **rozpracováno, vyplňuje se postupně s uživatelem.**
Šablona: [EBB-ASSIGNMENT-TEMPLATE.md](EBB-ASSIGNMENT-TEMPLATE.md).
Provozní kontext: [EBB-BRIEF.md](../EBB-BRIEF.md),
[integrační plán](EBB-CODEX-INTEGRATION-PLAN.md).

Štítky: **FIXED** (závazné) · **PREFERENCE** (lze změnit s odůvodněním) ·
**PROPOSE** (má navrhnout agent) · **OPEN** (čeká na rozhodnutí).
Nevyplněná pole zůstávají OPEN; žádné pole se nedoplňuje odhadem.

## 0. Execution topology (kdo pracuje, ne co se staví)

Tato sekce není v původní šabloně. Je zde proto, že uživatel rozhoduje
o rozdělení rolí mezi modely; produktové zadání tím není dotčeno.
Rozhodnuto 2026-10-16. Promítne se do `EBB-BRIEF.md` a jako autorizační
pravidlo do §7.

| Role | Model / runtime | Odpovědnost | Stav |
|---|---|---|---|
| Hlavní (lead) | Claude Code (`agentIdentity: claude`) | Plán, rozhodnutí, git, syntéza, komunikace s uživatelem | **FIXED** |
| Oponent (reviewer) | Codex (`agentIdentity: codex`, Orca OAuth účet) | Nezávislá oponentura návrhů leada, audit, hledání vad | **FIXED** |
| Worker | Muse Spark `muse-spark-1.3` (`agentIdentity: muse`) | Dílčí práce: rešerše, dokumentace, čtení velkých souborů, testovací skripty | **FIXED** |
| GLM 5.3 Flash | OpenCode / OpenRouter | původní jediný subworker dle briefu | **nahrazen** — terminál `term_0adb62a7-…` už v Orce neexistuje |

Pravidla, která platí bez ohledu na obsazení rolí:
- Oponent nesmí být tatáž relace jako lead; review je vždy samostatný krok.
- Worker dostává samostatně srozumitelný dispatch (vidí jen poslaný text).
- Žádný worker nemá autorizaci k instalaci, změně oprávnění, push ani deploymentu.
- Lead nepřebírá nálezy oponenta ani workera bez porovnání se zdrojovými soubory.

Zjištěno v prostředí 2026-10-16 (fakta, nikoli rozhodnutí):
- Stejná trojice claude/codex/muse už běží v worktree parrotfish, Simulation-Lab,
  IGF-Backtest a TradingView-Lab. Topologie EBB je tedy konzistentní s ostatními projekty.
- V worktree Ebb-Development neběží zatím žádný Orca terminál.
- Codex má v Orce systémový OAuth účet; `claude`, `opencode` a `devin` nemají aktivní účet.
- `~/.config/opencode/opencode.jsonc` je prázdná kostra bez providerů a modelů.

**OPEN (malé):** přesný spouštěcí příkaz Muse Spark pro
`orca terminal create --worktree active --command "<příkaz>" --title "Muse - worker"`.
Potřebné až ve chvíli prvního dispatche, neblokuje vyplňování zadání.

**Vyřešeno 2026-10-16:** mise ECC je **sdílená vrstva nad oběma harnessy** —
ECC dodává skills, pravidla a instrukce, které čte Claude Code i Codex,
není náhradní runtime. `EBB-BRIEF.md` je podle toho zrevidován.
Zbývá zrevidovat `EBB-CODEX-INTEGRATION-PLAN.md`, který je psaný s Codexem
jako primárním harnessem; jeho auditní zjištění zůstávají platná.

## 1. Identita, vlastník a místo práce

- Název platformy a stručná definice: **OPEN**
- Vlastník produktu a rozhodující osoba: **FIXED** — `jiris`
- Paperclip project/task ID: **OPEN** — dosud nepropojeno
- Git repo, větev a pracovní adresář: **FIXED** — `hrebik/ECC`,
  větev `hrebik/Ebb-Development`,
  `C:\Users\jiris\orca\workspaces\ECC\Ebb-Development`
- Herdr workspace a vztah k Orca worktree: **OPEN**
- Má Ebb Development obsahovat produktový kód, nebo pouze návrh/ECC integraci?
  **OPEN** — uživatel rozhodne po vyplnění §2
- Verze zadání, datum, odkazy na zdroje: v0.1, 2026-10-16, zdroje viz hlavička

## 2.–6. a 8.–9. Produktové sekce (odloženo)

**ODLOŽENO 2026-10-16.** (§7 níže zůstává v platnosti — bezpečnost a autorizace platí i pro integrační práci.) Uživatel potvrdil, že nový produkt v hlavě je,
ale zatím není ostrý. Sekce 2–6 a 8–9 se nevyplňují, dokud nebude.
Do té doby běží integrační práce a pilot, které mají hodnotu nezávisle na produktu.

**Poznámka k pojmu „platforma“ — zdroj nedorozumění.** `EBB-BRIEF.md` mluvil
o „naší platformě“ ve významu *existující agent stack* (Orca → agents → projekty).
V `EBB-CODEX-INTEGRATION-PLAN.md` a v šabloně se z toho stala *„nová platforma“*,
tedy dosud neexistující produkt, a na ten vznikla 228řádková produktová šablona
a 12 otázek. Ta premisa nebyla s uživatelem ověřena. Pojem „platforma“ se nadále
používá jen ve významu, který je na místě použití výslovně uveden.

## 7. Constraints, bezpečnost a oprávnění

- Akce vyžadující člověka: **FIXED** (převzato z briefu a integračního plánu) —
  credentials, změny oprávnění, veřejné/LAN vystavení, destruktivní a systémové
  změny, produkční deployment, zásadní architektura, `git push`.
- Žádný force-push; commit lokálně, push až po souhlasu uživatele.
- Nic se nespouští na produkční platformě (`~/.config/opencode`, Hermes config,
  plánované úlohy) bez schválení; vše se testuje izolovaně v tomto worktree.
- Zbytek sekce: **OPEN**

## 10. Dodávka, rozhodnutí a otevřené otázky

- Požadovaný výstup teď: **FIXED** — discovery / vyplnění zadání.
  Implementace nezačala.
- Zbytek sekce: **OPEN**

## Běžící práce (nezávislá na produktu)

### Sdílené zdroje skills

Zaregistrovány jako Claude Code marketplaces v `~/.claude` (2026-10-16).
Registrace je pouze ukazatel: nic neinstaluje a nespotřebovává kontext.
Vratné přes `claude plugin marketplace remove <name>`.

| Marketplace | Repo | ★ | Skills |
|---|---|---|---|
| `superpowers-dev` | obra/superpowers | 300 611 | 15, multi-harness (claude/codex/cursor/devin/agents) |
| `ecc` | affaan-m/ECC | 281 373 | 311 — upstream tohoto forku |
| `anthropic-agent-skills` | anthropics/skills | 181 584 | 19 oficiálních + spec a template |
| `agentic-awesome-skills` | sickn33/agentic-awesome-skills | 47 727 | 1000, nutná kurace |

Neinstalovatelné (pouze seznamy, bez `marketplace.json`): ComposioHQ 77 249,
VoltAgent 35 846, vercel-labs 32 531 (vlastní distribuce), travisvn 15 436
(bez pushe od dubna 2026).

**Zjištěné problémy:**
- Upstream se registroval pod názvem **`ecc`**. Přidání vlastního forku jako
  lokálního marketplace narazí na kolizi názvu — přesně jak varoval integrační plán.
- Fork má 292 skills, upstream 311. **Jsme o 19 pozadu.**

### Automatické commitování (2026-10-17)

Dvě naplánované úlohy, obě pod uživatelským účtem, obě ověřené ostrým během.

| Úloha | Co dělá | Kadence |
|---|---|---|
| `Obsidian Vault Backup (Jir_KA)` | `vault_backup.py` — scoped záloha vaultu, commit i push | 30 min |
| `Agent Autocommit (all repos)` | `multiagent-orchestration/scripts/autocommit.py` — 18 repozitářů | 30 min, posun 15 min |

Režim: kódová repa **commitují lokálně a nepushují**; vaulty pushují.
Důvod rozdílu: lokální commit je záchranná síť, kterou nikdo nevidí; push
rozpracovaného kódu do sdílené větve je jiná operace s jinými důsledky.

**Proč brána, a ne `git add -A`.** Git obnovuje obsah snadno, ale odstraňuje
ho jen přepsáním historie. Commitnuté tajemství je tam napořád — „vrácení
o verzi zpátky“ ho neodstraní. Každá brána existuje proto, aby něco do historie
nepustila, ne aby bránila práci. Gate: denylist názvů, secret a conflict scan,
limit 20 MB, strop počtu souborů i mazání, zákaz detached HEAD a rozpracovaného
merge, explicitní pathspecy místo `add -A`, rescan ze stage a ne z pracovního stromu.

**Co to hned při prvním běhu zachytilo:** `hermes-ops/backups/cxweb-pre-20260926/auth.json`
s `OPENAI_API_KEY`, `tokens` a `id_token`. Untracked, bez `.gitignore`, nikdy
necommitnutý. Slepý autocommit by ten klíč zapsal do historie natrvalo.
`hermes-ops` žádný `.gitignore` neměl; byl doplněn.

První sweep: 369 souborů v 9 repozitářích.

### Pilot

[EBB-PILOT-INVENTORY.md](EBB-PILOT-INVENTORY.md) — inventura agentní vrstvy
napříč 29 worktree. Zadání ke schválení, implementace nezačala.

### Stav instalace ECC (ověřeno 2026-10-16)

ECC není nainstalováno nikde. `~/.claude/skills` obsahuje 2 skills (`cua-driver`,
`orca-cli`), `~/.codex/skills` 2 skills z YouTube projektu. WSL `/home/jirka/.codex`
neexistuje — „kanonický WSL home“ z integračního plánu na tomto stroji není.

## Log rozhodnutí

| Datum | Rozhodnutí | Kdo | Dopad |
|---|---|---|---|
| 2026-10-16 | Založen živý dokument zadání; vyplňuje se po sekcích | jiris | Šablona zůstává prázdným vzorem |
| 2026-10-16 | Topologie: Claude Code lead, Codex oponent, Muse Spark worker | jiris | Mění roli leada oproti EBB-BRIEF.md; vyžaduje revizi integračního plánu |
| 2026-10-16 | GLM 5.3 Flash nahrazen jako worker | jiris | Jeho terminál v Orce již neexistuje; brief se aktualizuje |
| 2026-10-16 | Umístění produktového kódu odloženo za §2 | jiris | §1 zůstává částečně OPEN |
| 2026-10-16 | Mise ECC = sdílená vrstva nad Claude Code i Codexem | jiris | EBB-BRIEF.md zrevidován; integrační plán revizi teprve čeká |
| 2026-10-16 | Produktové zadání (§2–9) odloženo — produkt není vyzrálý | jiris | Šablona zůstává k dispozici; mezitím běží integrace a pilot |
| 2026-10-16 | Zaregistrovány 4 skill marketplaces v ~/.claude | jiris | Jen ukazatele, nic nenainstalováno; kolize názvu `ecc` |
| 2026-10-16 | Pilot = inventura agentní vrstvy napříč worktree | jiris | Zadání v EBB-PILOT-INVENTORY.md, čeká na schválení |
| 2026-10-17 | Obsidian vault připojen do Orcy; registrovány i Hermes, NinjaTrader a multiagent-orchestration | jiris | Z 14 na 18 repozitářů |
| 2026-10-17 | Commitnuto a pushnuto hromadné přejmenování vaultu (.en/.cs) | jiris | Odblokovalo zálohu, která stála od 26. 9. |
| 2026-10-17 | Automatický commit na všech repozitářích, lokálně bez pushe | jiris | Dvě naplánované úlohy po 30 min |
| 2026-10-17 | MO: fast-forward main na 3309f65, konec detached HEAD | jiris | Záchranná větev rescue/detached-head-20261017 ponechána |
