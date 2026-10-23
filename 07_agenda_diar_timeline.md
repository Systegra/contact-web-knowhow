# 07. Agenda Diář (TIMELINE & TIMEMEMO), události, aktivity, úkoly a vazební tabulka LINK

Tento dokument detailně popisuje architekturu agendy **Diář** v systému **Databox CONTACT Web**, koncept událostí, rozdělení na aktivity a úkoly, systém sledování plnění, databázová pole v NexusDB a univerzální vazební mechanismus realizovaný přes tabulku `LINK`.

---

## 1. Koncept agendy Diář vs. Kalendář

V názvosloví systému **CONTACT Web** existuje jasné funkční rozlišení mezi Diářem a Kalendářem:

* **Kalendář (Zobrazení a navigace):** Slouží výhradně jako roční/měsíční vizuální mřížka a nástroj pro listování v čase.
* **Diář (Časový zápisník a agenda):** Pracovní prostor pro vytváření, evidenci a řízení časových záznamů. Každý uživatel systému má svůj **vlastní osobní diář** (určený vlastnictvím `DIAROWNER`). Diáře lze mezi uživateli libovolně sdílet a plánovat do diářů kolegeneric.
* **Hlavní entita (Událost):** Základní datovou jednotkou Diáře je **Událost**, která se dělí na dva typy:
  1. **Aktivity:** Časově ohraničené události, které začínají a končí v konkrétní čas a datum (např. schůzka od 9:00 do 10:30).
  2. **Úkoly:** Vícedenní nebo celodenní události, které nemají pevně stanovený přesný čas od–do, ale mají stanovený termín splnění.

---

## 2. Parametry události a klíčový princip plnění

Každá událost je v systému charakterizována dvěma základními atributy a striktní logikou dokončení:

### A. Klasifikace události
1. **Ikona (`ICONTYPE`):** Určuje základní druh akce (např. *Telefonát / Call*, *Jednání / Schůzka*, *Úkol / Udělat*, *Konzultace*)[cite: 8, 9].
2. **Kategorie / Stav (`EVENTTYPE` / `PRIORITA`):** Specifikuje charakter či způsob realizace:
   * **Směr / Místo:** *Příchozí*, *Odchozí*, *Mimo firmu*.
   * **Forma:** *Online*, *MS Teams* (plánované online schůzky)[cite: 9].
   * **Důležitost:** *Důležitá*, *Informativní*.

### B. Princip plnění a sledování nesplněných událostí
* **„Události jsou od toho, aby se plnily.“** – Základní filozofie řízení agendy.
* **Záznam minulosti:** Při zpětném vkládání proběhlého jednání se událost rovnou označuje jako **Splněno** (`DONE` = 1)[cite: 9].
* **Plánování do budoucna:** Nově naplánovaná událost zůstává nezaškrtnutá.
* **Seznam nesplněných událostí:** Pokud má jakákoliv nezaškrtnutá událost datum konce v minulosti, systém ji automaticky propaguje do speciálního přehledu **Nesplněné události** (notifikační seznam upozorňující uživatele na skluz v agendě)[cite: 8, 11].

---

## 3. Databázová struktura (`TIMELINE` & `TIMEMEMO`)

Data Diáře jsou v NexusDB rozdělena do dvou úzce propojených tabulek pro optimalizaci výkonu a kapacit:

### A. Hlavní tabulka událostí (`TIMELINE`)
Uchovává strukturované řádkové atributy, časová razítka a stavové příznaky události[cite: 10].

* **`DIAROWNER` & `EVENTOWNER`:** Identifikace vlastníka diáře a autora/vlastníka události (uživatelské doménové jméno)[cite: 10].
* **`STARTDAY` & `STARTTIME` / `DURATION`:** Datum a přesný čas začátku události a její trvání[cite: 10].
* **`DISPLAYFROM` & `DISPLAYTO`:** Rozsah zobrazení události v mřížce Diáře[cite: 10].
* **`ICONTYPE` & `EVENTTYPE`:** Číselné kódy vybrané ikony a kategorie/stavu[cite: 10].
* **`DONE`:** Stav splnění (0 = nesplněno, 1 = splněno)[cite: 10].
* **`PRIVATED`:** Příznak privátní události (skryté před ostatními uživateli s přístupem ke sdílenému diáři)[cite: 10].
* **`ALARM` & `REPEATED`:** Nastavení notifikací/připomínek a opakování události[cite: 10].
* **`CPCE`, `CE`, `CW` & `ID`:** Složené klíče a unikátní identifikátory záznamu[cite: 10].
* **`CREATOR`, `LASTEDIT`, `CRDATE`, `EDDATE`, `SYNDATE`:** Kompletní auditní stopa vzniku a úprav[cite: 10].

### B. Textová a HTML paměť (`TIMEMEMO`)
Obsahuje rozsáhlé textové popisy události a bohatý obsah[cite: 12].

* **`MEMO` (BLOB Wide Memo):** Samotný tělo zápisu[cite: 12]. Popisek podporuje **plný HTML formát**, což z události dělá primární registr komunikace (vkládání celých formátovaných e-mailů, zápisů z jednání, detailních požadavků a analýz)[cite: 9, 12].
* **Zjednodušené CRM:** U zákazníků, kteří nepoužívají komplexní agendu Obchodních případů, slouží HTML pole `MEMO` v událostech jako kompletní kronika vývoje vztahu se zákazníkem.

---

## 4. Provázanost entit a univerzální tabulka `LINK`

Události v Diáři neexistují izolovaně, ale jsou úzce propojeny s ostatními moduly systému (Firma, Osoba, Projekt, Dokument atd.)[cite: 9, 10].

### A. Křížové relace přes tabulku `LINK`
Veškeré křížové vztahy mezi událostmi a okolními entitami spravuje centrální vazební tabulka **`LINK`**.
* **Jedna událost může mít současně vazbu na:**
  * **Firmu** (`FIRM` / ID 51)[cite: 9]
  * **Kontaktní osobu** (`PERSON` / ID 52)[cite: 9]
  * **Projekt / Blok** (`NOTEBLOCK` / ID 4)[cite: 9]
  * **Dokumenty** (`DOCUMENT`)[cite: 9]

### B. Univerzálnost tabulky `LINK`
Tabulka `LINK` neslouží pouze pro Diář, ale realizuje **jakékoliv M:N relace** napříč celým systémem CONTACT Web:
* Vazba **Firma $\rightarrow$ Firma** (např. mateřská/dceřiná společnost, partnerství).
* Vazba **Projekt $\rightarrow$ Projekt** (podprojekty, etapy).
* Vazba **Firma $\rightarrow$ Projekt** nebo **Osoba $\rightarrow$ Projekt**.
* Konkrétní typ a význam relace v tabulce `LINK` je vždy definován příslušným ID typu vazby.

---

## 5. Standardní klientské a kolaborativní funkce

Diář CONTACT Web disponuje kompletní sadou standardních funkcí elektronických kalendářových systémů:
* **Pozvánky a schůzky:** Odesílání pozvánek na schůzky dalším účastníkům.
* **E-mailová integrace:** Přímé odesílání e-mailů z detailu události.
* **Notifikace a alarmy:** Systémové a e-mailové připomínky nadcházejících událostí.
* **Týmové plánování a sdílení:** Přepínání náhledů mezi diáři jednotlivých kolegů či celých oddělení (např. *1. Zákaznická podpora*, *3. Vývoj*)[cite: 8].
