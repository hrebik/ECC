# EBB: zadání nové platformy pro Codex a ECC

Datum: 2026-09-27. Stav: návrh k vyplnění, nikoli schválené zadání implementace.
Zdroj: požadavky uživatele v této relaci a aktuální instrukce ECC uvedené níže.
Paperclip ID projektu/úkolu: dosud nespojeno. Tento dokument není náhradou živého
stavu v Paperclip. Doménu, produkt i rozsah nové platformy určuje uživatel.

## Co ECC agenti skutečně očekávají

ECC nemá jeden povinný vstupní JSON ani univerzální parser zadání. Soubory
`agents/*.md` definují role, postupy a očekávané výstupy. Následující tabulka
odvozuje potřebné vstupy z těchto postupů; je to doporučený kontrakt pro EBB,
nikoli tvrzení, že ECC má validátor všech těchto polí.

| Role a zdroj | Vstup potřebný pro smysluplnou práci | Očekávaný výstup |
|---|---|---|
| [planner](../agents/planner.md) | Cíl, požadavky a priority, hranice rozsahu, kritéria úspěchu, omezení, dostupný repozitář a konvence; neznámé věci explicitně označené | Fáze a konkrétní kroky, dotčené soubory, závislosti, rizika, testovací strategie, otevřené otázky |
| [architect](../agents/architect.md) | Funkční požadavky, objemy a kvalitativní cíle, datové toky, integrace, bezpečnost, provozní omezení, existující komponenty nebo výslovné greenfield | Varianty s kompromisy, doporučení, hranice komponent, datové a API kontrakty, ADR ke schválení |
| [tdd-guide](../agents/tdd-guide.md) | Jeden vymezený implementační úkol, rozhodnuté chování, akceptační kritéria, kontrakty a chybové stavy, testovací prostředí a příkazy, povolené soubory | RED → GREEN → refaktor, unit/integration/E2E pro relevantní chování, výsledky a coverage; instrukce požadují nejméně 80 % větví, funkcí, řádků a statements |
| [code-reviewer](../agents/code-reviewer.md) | Zadání, diff/base commit, okolní kód, rozhodnutí, testovací výsledky a známé limity | Doložené nálezy s konkrétním spouštěčem a místem; samostatný review verdikt |
| [security-reviewer](../agents/security-reviewer.md) | Role a oprávnění, hranice důvěry, citlivá data, vstupy, integrace, změněný kód a závislosti | Konkrétní bezpečnostní nálezy a ověření kontrol; nikdy nepotřebuje tajné hodnoty v zadání |
| [e2e-runner](../agents/e2e-runner.md) | Kritické uživatelské scénáře, očekávané výsledky, testovací URL, způsob spuštění/resetu, syntetická data a bezpečné testovací identity | Spustitelné scénáře, výsledky, důkazy selhání a artefakty |

Pod Codexem jsou tyto Claude-style prompty podklady pro role. Jejich názvy,
`model: opus/sonnet`, nástroje ani automatické spouštění se nepřenášejí samy.
Repo má tři explicitní Codex role: explorer, reviewer, docs_researcher.
Codex z vyplněného zadání sestaví omezené worker úkoly; nemusíš psát prompty
pro každého specialistu ani navrhnout tabulky, endpointy a názvy souborů.

Pro přechod od produktového záměru ke kontraktům navazuje tato šablona na
[product-capability](../skills/product-capability/SKILL.md) a jeho
[základní šablonu](examples/product-capability-template.md).

## Jaké zadání funguje a jaké ne

- **Funguje pro discovery:** „Pro roli X chceme vyřešit problém Y. MVP musí
  umožnit A a B, C je mimo rozsah. Stack není rozhodnutý; porovnej dvě varianty
  v rozpočtu Z. Nejprve vrať otázky a návrh, bez implementace.“
- **Funguje pro implementaci:** „V dohodnutém modulu přidej operaci A. Vstup,
  výstup, oprávnění a chybové stavy jsou v kontraktu. Splň AC-01 až AC-04,
  uprav pouze uvedené soubory, ověř určenými testy.“
- **Nestačí pro implementaci:** „Postav moderní platformu jako X se vším.“
  Chybí uživatel, prioritní tok, hranice a způsob rozhodnutí, zda je hotovo.
- **Nestačí:** seznam technologií bez uživatelského výsledku; screenshot bez
  chování a dat; „rychlé a bezpečné“ bez měřitelných nebo ověřitelných podmínek;
  současně „offline-only“ a povinná externí služba bez rozhodnutí o konfliktu.
- **Není nutné mít všechno rozhodnuté:** `OPEN — má navrhnout architect` je
  správný vstup pro návrh. Není to implicitní souhlas s jeho nasazením.

## Šablona k vyplnění

Každá sekce musí mít odpověď, `N/A + důvod`, nebo `OPEN + kdo rozhodne`.
Používej štítky **FIXED** (závazné), **PREFERENCE** (lze změnit s odůvodněním),
**PROPOSE** (má navrhnout Codex), **OPEN** (čeká na rozhodnutí).
Otevřené implementační blokery nebrání read-only discovery.

### 1. Identita, vlastník a místo práce

- Název platformy a stručná definice:
- Vlastník produktu a člověk rozhodující sporné otázky:
- Paperclip project/task ID nebo `OPEN — dohledat existující`:
- Git repo, větev a pracovní adresář:
- Herdr workspace a vztah k aktuálnímu Orca worktree:
- Má Ebb Development obsahovat produktový kód, nebo pouze návrh/ECC integraci?
- Verze zadání, datum, odkazy na zdroje:

### 2. Problém, uživatelé a cíle

- Kdo platformu používá, co dnes dělá a kde naráží:
- Jaký nový výsledek mu umožníme:
- Hlavní role a jejich odlišná oprávnění:
- Měřitelný cíl MVP a způsob měření:
- Proč je tato priorita důležitá:

### 3. Scope a non-goals

- MUST pro první verzi, seřazeno:
- LATER po MVP:
- Výslovně mimo rozsah:
- Nejmenší samostatně použitelný end-to-end tok:
- Co lze zatím simulovat a co musí být skutečná integrace:

### 4. Scénáře a funkční požadavky

Pro každý MUST vyplň:

```text
REQ-ID:
Aktér a předpoklady:
Spouštěč / vstup:
Hlavní průchod:
Pozorovatelný výsledek:
Neplatný vstup / zamítnuté oprávnění / nedostupná závislost:
Povolené stavy a přechody:
Invarianty (co se nikdy nesmí stát):
Opakování, souběh a obnova po selhání, pokud relevantní:
Navazující AC-ID:
```

### 5. Data a integrace

- Hlavní entity, vlastník dat a zdroj pravdy pro každou oblast:
- Příklady vstupů/výstupů se syntetickými daty:
- Existující API/systémy a odkazy na kontrakty:
- Povolené čtení/zápis a očekávané vedlejší účinky:
- Retence, export, mazání, audit, případná migrace:
- Dostupnost testovacího prostředí a chování při výpadku:
- Přístupové údaje pouze jako reference na schválený mechanismus, nikdy hodnoty:

### 6. Tech stack a co znovu použít

- FIXED technologie/verze/platformy a důvod:
- PREFERENCE a důvody:
- PROPOSE oblasti, pro které má Codex porovnat možnosti:
- Existující komponenty, služby, repozitáře a reusable knihovny:
- Zakázané technologie nebo dodavatelé:
- Cílový OS/runtime, distribuce a provozní prostředí:

Stack může být celý PROPOSE. Codex před větší volbou provede
DISCOVER → EVALUATE → ADOPT / ADAPT / BUILD, doloží kompromisy a předloží
významná architektonická rozhodnutí člověku. Příklady stacku v promptu
`architect.md` nejsou automaticky doporučením pro tuto platformu.

### 7. Constraints, bezpečnost a oprávnění

- Izolace uživatelů/tenantů a autorizační pravidla:
- Citlivost dat, lokální/cloudové hranice a povolené destinace:
- Akce vyžadující člověka a akce povolené autonomně:
- Rozpočtový strop, termín, kapacita týmu a licence:
- Závazná kompatibilita a provozní omezení:
- Co se nesmí měnit v existující infrastruktuře:

Pro EBB platí stávající politika: Paperclip vlastní pracovní stav, Git trvalé
artefakty, Codex vede engineering, Herdr zajišťuje runtime; Hermes je pomocník.
Nedostupné spojení se eviduje, nenahrazuje novým řídicím systémem. Credentials,
změny oprávnění, veřejné/LAN vystavení, destruktivní změny, systémové změny,
produkční deployment a zásadní architektura vyžadují explicitní souhlas.
Současný rozsah je pouze návrh a dokumentace, bez instalací.

### 8. UX a kvalitativní/provozní cíle

- Povrchy: web / desktop / API / CLI / jiný:
- Jazyky, přístupnost, zařízení, reference návrhu:
- Očekávaný počet uživatelů, souběh a objem dat:
- Latence: limit + percentil + operace + testovací podmínky:
- Dostupnost, zálohy, obnova a tolerovaná ztráta dat:
- Logování/metryky, kdo řeší chyby a jak:

### 9. Akceptační kritéria a ověření

| ID | REQ-ID | Given / When / Then | Jak ověřit | Důkaz dokončení |
|---|---|---|---|---|
| AC-01 | REQ-01 | [výchozí stav] / [akce] / [pozorovatelný výsledek] | unit / integration / E2E / ruční | [výsledek testu/artefakt] |
| AC-02 | REQ-01 | [chybný vstup nebo nepovolená role] / [akce] / [bezpečné odmítnutí, bez nežádoucí změny] | [metoda] | [důkaz] |

- Příkazy pro spuštění, testy a reset prostředí, nebo PROPOSE:
- Syntetická data/fixtures a integrační sandboxy:
- Kritické scénáře a coverage politika:
- Definice hotovo: kritéria splněna, relevantní testy úspěšné, samostatný
  review, známé limity zapsané, dokumentace a handoff předané:
- Kdo akceptuje produktový výsledek:

### 10. Dodávka, rozhodnutí a otevřené otázky

- Požadovaný výstup teď: discovery / návrh / prototyp / implementace:
- Milníky a priorita jednotlivých inkrementů:
- Povolený rozsah úprav a zakázané cesty:
- Deployment/rollback plán nebo PROPOSE; autorizace nasazení samostatně:
- Otázky: ID, rozhodovatel, zda blokuje discovery/architekturu/implementaci:
- Které předpoklady jsou potvrzené a které čekají:

## Otázky, které nyní potřebuji od uživatele

Pro první předání plannerovi/architectovi stačí odpovědět na otázky 1–6;
u zbytku lze uvést OPEN/PROPOSE. Před implementací vyřešíme všechny otázky,
které ovlivňují daný inkrement.

1. Jakou platformu vytváříme: pro koho, jaký problém řeší a co má uživatel
   po první verzi dokázat? Je produkt nezávislý na ECC, nebo rozšiřuje ECC?
2. Jaké jsou tři nejdůležitější funkce MVP a co výslovně do první verze nepatří?
3. Popiš jeden prioritní uživatelský tok od začátku do konce. Podle čeho
   prakticky poznáš, že funguje správně?
4. Má produktový kód vznikat přímo v tomto ECC worktree? Jaké existující
   Paperclip/Git/Herdr přiřazení máme použít a kdo vlastní produktové rozhodnutí?
5. Co je pevně dané: technologie, OS, lokální/cloudový provoz, termín,
   rozpočet a povinné integrace? Co mám naopak sám navrhnout?
6. Má další krok skončit návrhem architektury, prototypem, nebo implementací
   prvního inkrementu? Jaký rozsah změn bude povolen?
7. Jaké role, oprávnění a případní tenanti existují? Které operace musí vždy
   schválit člověk?
8. S jakými daty pracujeme, kdo je vlastní, co je citlivé a kam smějí odcházet?
9. Které služby či komponenty už existují a můžeme je znovu použít? Jsou
   dostupné kontrakty, testovací sandboxy a syntetická data?
10. Jaké jsou očekávané objemy, rychlost, dostupnost a požadavky na obnovu?
11. Jaké UI/povrchy, zařízení, jazyky a designové reference potřebujeme?
12. Kdo bude provozovat a akceptovat výsledek a jaké důkazy/testy chce vidět?

## Jak z toho Codex vytvoří práci pro ECC role

1. Zkontroluje odpovědi a oddělí potvrzená fakta, předpoklady a blokery.
2. Reconciliuje Paperclip project ↔ Git repo ↔ Herdr workspace; při chybějícím
   propojení zachová práci v autorizovaném návrhovém rozsahu.
3. Planner sestaví fáze; architect navrhne sporné kontrakty a významné volby.
4. Po rozhodnutí o příslušném návrhu předá malé inkrementy implementerovi/TDD.
5. Tester a jiný reviewer ověří výsledek; Codex integruje a kurátor uloží
   rozhodnutí, testy a handoff do Git a naváže je na Paperclip.

Každý worker dostane jen relevantní kontext a tento kontrakt:

```text
TASK_ID: [Paperclip ID; při discovery explicitně nepřiřazeno]
ROLE / GOAL:
SOURCES: [verze zadání, kontrakty, relevantní soubory]
OWNERSHIP: [povolené soubory/modul; kdo souběžně pracuje]
IN_SCOPE / OUT_OF_SCOPE:
INPUTS / INTERFACES / INVARIANTS:
ACCEPTANCE_CRITERIA:
CONSTRAINTS / AUTHORIZATION:
TEST_COMMANDS / EXPECTED_EVIDENCE:
BLOCKERS / STOP_CONDITIONS:
RETURN: TASK_ID, GOAL, WORK_COMPLETED, FILES_CHANGED,
        INTERFACES_CHANGED, DECISIONS, TESTS_RUN, TEST_RESULTS,
        KNOWN_ISSUES, NEXT_STEP
```

Výchozí limity budoucí engineering mise: nejvýše 4 child workery, hloubka 2,
nejvýše 3 implementační/testovací opravné cykly. Pro aktuální audit je jediný
povolený pomocník existující GLM přes Orca; nová implementační mise nezačala.
