# 03. Architektura uživatelských tabulek (Subsystém TABSET) a Master-Detail

Tento dokument detailně popisuje vnitřní databázovou architekturu NexusDB pro tvorbu dynamických uživatelských tabulek (subsystém `TABSET`), jejich Master-Detail relace, řízení workflow pomocí pole `COLOR` a obousměrné provázání subjektů.

---

## 1. Databázová struktura subsystému TABSET

Uživatelské tabulky umožňují rozšiřovat hlavní entitu Organizace (i další entitní typy) o strukturované 1:N agendy (např. *Firemní flotila*, *Fakturace*, *Slevy / Individuální ceníky*, *Kontrakty*).

### Přehled klíčových databázových tabulek:
* **`TABSET` (Definice tabulky):** 
  * Zakládací tabulka uchovávající metadata o tabulce.
  * Složený unikátní klíč: **`CPCT`** (číslo profilu, např. `1000`) + **`CT`** (unikátní číslo tabulky v rámci profilu).
  * **`ID` (Identifikátor entity):** Určuje, ke které agendě tabulka patří (`1` = Firmy/Organizace, `2` = Osoby, `3` = Události, `4` = Projekty/Bloky, `7` = Uživatelé).
  * **`BASE_SET`:** Konfigurační INI blok (BLOB Wide Memo) obsahující pravidla, chování, tiskové šablony a skripty.
* **`TABDEF` (Definice sloupců):**
  * Obsahuje definice jednotlivých polí (sloupců) uživatelské tabulky.
  * Určuje název (`NAME`), typ (`TYP`), pořadí (`PORADI`), viditelnost (`VISIBLE`), editovatelnost (`READONLY`) a rozšířené podmínky.
* **`TABROW` (Core datová tabulka – Optimalizovaný řádek):**
  * Základní řídicí tabulka pro fyzické ukládání dat.
  * Obsahuje fixní sadu předpřipravených generických sloupců (`TEXT1`, `TEXT2`, `NUMBER1`, `NUMBER2`, `DATE1`, `DATE2`, `COLOR`, `IDENT` atd.), které se na aplikační úrovni na základě `TABDEF` logicky přejmenovávají.
* **`TABDATA` (Rozšíření datových polí):**
  * Přetékající/rozšiřující tabulka pro dynamická pole, která se nevejdou do fixní struktury `TABROW`.
  * Hodnoty se ukládají jako textový řetězec (`VALUE` - VARCHAR) a na aplikační úrovni se přetypovávají (castejí) podle formátu z definice.
  * Vazba na konkrétní řádek je realizována přes relaci `CPCR` / `CR`.
* **`TABFILES` & `TABMEMO` (Přílohy a dlouhé texty):**
  * **`TABFILES`:** Ukládá soubory a dokumenty vázané k řádku tabulky (`FILE_PATH`, `FILE_NAME`, `FILE_SIZE`).
  * **`TABMEMO` / `TIMEMEMO`:** Uchovává rozsáhlé textové zápisy, poznámky a HTML obsahy vázané na řádek `TABROW`.

---

## 2. Hierarchie Master-Detail a podřízené tabulky

Uživatelské tabulky podporují vícenásobné zanoření (např. Nadřízená tabulka *Fakturace* -> Podřízená tabulka *Položky faktury*).

### Technická vazba rodič-dítě:
* Podřízená tabulka má ve svém záznamu v tabulce **`TABSET`** vypleštěná pole **`PARENT_CPCT`** a **`PARENT_CT`**, která směřují na nadřízenou tabulku.
* **Uživatelské rozhraní (CONTACT Web):** V horní části modálního okna/formuláře se zobrazuje a edituje hlavičkový záznam (Master), zatímco v dolní části je přímo integrovaná mřížka s podřízenými řádky (Detail), kde lze položky okamžitě přidávat nebo upravovat.

---

## 3. Význam a pokročilá logika pole `COLOR` (Stav)

Pole **`COLOR`** uchovává stav řádku tabulky (např. *Založená*, *Připravená do Pohody*, *Vyúčtovaná v Pohodě*, *Storno*) a v systému zastává 3 klíčové funkce:

1. **Vizuální rozlišení a status:** Barevně rozlišuje řádky v přehledových mřížkách (gridech).
2. **Zamčení záznamu (`LOCK_WEB` / INI):** 
   * Umožňuje definovat stavy, při kterých se formulář automaticky uzamkne proti změnám (např. schválená či vyúčtovaná faktura).
   * Lze definovat výjimky pro konkrétní uživatelské skupiny, které mohou data upravovat i v zamčeném stavu.
3. **Podmíněné zobrazování polí:** 
   * Umožňuje dynamicky skrývat nebo zobrazovat ostatní pole formuláře v závislosti na zvoleném stavu (např. pole *Datum ukončení* se zobrazí pouze tehdy, je-li stav přepnut na *Ukončený*).

---

## 4. Obousměrná vazba přes pole Subjekt (`ID1`, `CPCX1`, `CX1`)

Pro provázání tabulkového řádku s jinou entitou (Firma, Osoba, Projekt/Smlouva) se používá trojice polí v `TABROW`:
* **`ID1`:** Typ cílového subjektu (`1` = Firma, `2` = Osoba, `4` = Projekt/Blok).
* **`CPCX1` + `CX1`:** Profil a ID konkrétního záznamu cílového subjektu.

### Princip obousměrné viditelnosti (Bidirectional Link):
Díky této architektuře je vytvořený vztah viditelný z obou stran:
* **Z pohledu Organizace:** V detailu firmy vidím přehled všech jejích faktur a u každé faktury odkaz na příslušnou smlouvu.
* **Z pohledu Projektu / Smlouvy:** Při otevření detailu konkrétní smlouvy systém automaticky načte a zobrazí všechny faktury, které na tuto smlouvu přes pole `CX1` odkazují.
