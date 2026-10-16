# EBB — Brief pro vedoucího agenta (Claude Code)

Datum: 26. 9. 2026 • Prostor: Orca workspace **Ebb Development** (repo `ECC`, fork `hrebik/ECC` ← affaan-m/ECC)

## Mise

Revidováno 2026-10-16. Původní znění viz sekce *Revize* na konci dokumentu.

Přeměnit tento fork **ECC** (agent harness: skills, instincts, memory, security,
research-first vývoj) na **vývojové centrum naší platformy** — provozovat ho jako
**sdílenou vrstvu nad oběma harnessy** v našem agent stacku (Orca → agents →
projekty). ECC dodává skills, pravidla a instrukce, které čte Claude Code
i Codex; není náhradní runtime ani náhrada jednoho harnessu druhým.

## Role (revidováno 2026-10-16)

| Role | Kdo | Odpovědnost |
|---|---|---|
| **Vedoucí (lead)** | Claude Code | Plán, rozhodnutí, git, syntéza, komunikace s uživatelem |
| **Oponent** | Codex | Nezávislá oponentura návrhů leada, audit, hledání vad |
| **Worker** | Muse Spark (`muse-spark-1.3`) | Rešerše, dokumentace, čtení velkých souborů, testovací skripty |

- Lead plánuje a řídí integraci ECC do naší platformy.
- Oponenta i workera zakládáš jako samostatné terminály v tomto worktree:
  ```bash
  orca terminal create --worktree active --title "Codex - oponent" --command "codex" --json
  orca terminal create --worktree active --title "Muse - worker" --command "<muse>" --json
  ```
- Dispatch a čtení výsledků přes Orca CLI:
  ```bash
  orca terminal send --terminal <handle> --text "<úkol>" --enter --json
  orca terminal read --terminal <handle> --json
  ```
  Než pracuješ dál, čekáš na potvrzení (read → odpověď).
- Klíčové úlohy (plán, rozhodnutí, git) dělá lead sám. Dílčí práce deleguje
  na workera. Každý významný návrh projde oponenturou u Codexu.
- Lead nepřebírá nálezy oponenta ani workera bez porovnání se zdrojovými soubory.

## První kroky (doporučené, rozhoduješ ty)

1. Přečti `README.md`, `docs/`, `AGENTS.md`, `CLAUDE.md`, `config/`, `hooks/`
   (velké dávky můžeš delegovat na workera se strukturovaným shrnutím).
2. Zjisti, co ECC skutečně instaluje a jak se integruje (skills, memory,
   security, hooks) — a co z toho je kompatibilní s naším stackem
   (Orca + OpenCode + Codex + lokální Hermes).
3. Navrhni plán integrace „ECC místo Claude Code" — fáze, rizika, přesné
   kroky. Plán nejdřív napiš, než něco instaluješ.
4. Nic nespouštěj na produkční platformě (žádné změny v `~/.config/opencode`,
   Hermes configu, plánovaných úlohách) bez schválení uživatele —
   vše odtestuj izolovaně zde v worktree.

## Pravidla

- User je `jiris`. Nikdy nezobrazuj sekrety/klíče.
- Gity: žádný force-push; commituj lokálně, push jen po souhlasu uživatele.
- Jaderná konvence: každý dispatch workerovi i oponentovi musí být samostatný
  a srozumitelný (vidí jen text, který mu pošleš).
- Na konci každé větší etapy vrať uživateli souhrn.

## Revize

**2026-10-16 — rozhodnutí uživatele `jiris`:**

1. *Mise.* Původní znění bylo „rozběhnout ho **na místě Claude Code**".
   Změněno na sdílenou vrstvu nad oběma harnessy. Důvod: audit
   [integračního plánu](docs/EBB-CODEX-INTEGRATION-PLAN.md) doložil, že Codex
   dnes nemá paritu v hoocích, skills ani MCP, a ECC sám sebe definuje jako
   vrstvu nad harnessy, nikoli jako runtime.
2. *Role.* Lead byl Codex, jediný subworker GLM 5.3 Flash
   (`term_0adb62a7-3ffd-45f9-99fb-9aa13cef4c17`). Nově Claude Code lead,
   Codex oponent, Muse Spark worker. Stejná trojice už běží v worktree
   parrotfish, Simulation-Lab, IGF-Backtest a TradingView-Lab.
3. *GLM.* Uvedený terminál v Orce již neexistuje; worker je nahrazen.

**Dopad, který ještě není proveden:** `docs/EBB-CODEX-INTEGRATION-PLAN.md`
je psaný s Codexem jako primárním harnessem a runtimem. Jeho závěry o paritě,
dry-runech a instalaci zůstávají platné jako audit, ale rámec „Codex jako
primární harness" je nutné zrevidovat na sdílenou vrstvu. Revize zatím neproběhla.

Produktové zadání nové platformy se vyplňuje v
[docs/EBB-ASSIGNMENT.md](docs/EBB-ASSIGNMENT.md).
