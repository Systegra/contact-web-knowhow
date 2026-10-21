# 04. Správa uživatelů (Tabulka USERSET), přístupová práva a konfigurační logika

Tento dokument detailně popisuje vnitřní databázovou strukturu pro správy uživatelských účtů v NexusDB (tabulka `USERSET`), systém skupinových oprávnění, roli administrátora profilu (`SUPERVISOR`) a konfigurační logiku INI bloků.

---

## 1. Fyzická databázová struktura tabulky USERSET

Uživatelské účty a jejich komplexní chování v celé aplikaci jsou uloženy v centrální tabulce **`USERSET`**.

### Klíčová databázová pole:
* **`DOMNAME`:** Unikátní identifikátor uživatelského účtu v rámci domény/profilu (např. `1_1SPRÁVCE`, `34_PETERKA`, `20_HANKA`).
* **`LOGNAME` & `FULLNAME`:** Přihlašovací jméno uživatele a jeho celé jméno.
* **`ACCESS_TOTAL`:** Řídicí prvek aktivity uživatelského účtu (`0` = aktivní uživatel, vyšší hodnoty určují neaktivitu či omezení přihlášení).
* **`NICKNAME` (Oddělení):** Historicky pojmenované pole, které v moderní architektuře slouží jako **Oddělení** (např. *1. Zákaznická podpora*, *3. Vývoj*).
  * **Účel:** Slouží k efektivnímu filtrování výpovědních seznamů v uživatelských dodátcích/tabulkách (místo celého stromu uživatelů systém nabízí pouze pracovníky konkrétního oddělení).
* **`USERCOLOR` (Uživatelská barva):**
  * Jedinečné celé číslo (Integer) určující barvu uživatele.
  * V systému se barva používá jako **primární vizuální identifikátor** namísto fotografií.
  * Tvoří podkladové barevné schéma v kalendáři (Diář / Aktivity) a uživatelských agendách pro okamžitou týmovou orientaci.
* **`SUPERVISOR` (Administrátor profilu):**
  * Boolean příznak určující práva k administraci klientského profilu.
  * Umožňuje zakládat a upravovat účty, měnit konfigurační nastavení a spravovat oprávnění ostatním uživatelům.
  * **UI Indikace:** V seznamu uživatelů je supervisor vizuálně označen ikonou **medaile**.

---

## 2. Klonování při zakládání účtu (Vzorový uživatel)

Vzhledem k rozsahu konfigurace systému se při vytváření nového uživatele využívá princip **Vzorového uživatele**:

1. **Výběr vzoru:** Administrátor zvolí existujícího uživatele se stejnou pozicí nebo ze stejného oddělení.
2. **Klonování konfigurace:** Systém automaticky překopíruje veškeré rozsáhlé INI konfigurační bloky, skupinová členství i detailní přístupová práva.
3. **Doplnění osobních údajů:** Vyplňují se pouze unikátní parametry (přihlašovací jméno, heslo, e-mail, mobil a vlastní `USERCOLOR`).

---

## 3. Skupiny uživatelů a bezpečnostní model

Skupiny uživatelů (např. *Obchod*, *Podpora*, *Vývoj*) slouží k hromadnému řízení přístupových práv a viditelnosti dat:

* **Oprávnění nad entitami:** Definice úrovně přístupu (Plná práva, Pouze pro čtení) nad agendami (Firmy, Osoby, Projekty) včetně oprávnění editovat statusy, kategorie či poznámky.
* **Práva na základě vlastnictví:** Zajišťuje sdílení a viditelnost dat podle pravidla *"Uživatel vidí/edituje záznamy, které vlastní kterýkoliv člen jeho uživatelské skupiny"*.

---

## 4. Konfigurační INI BLOB pole (Příklad SET_FIRM)

Rozsáhlé možnosti přizpůsobení aplikace jsou uloženy v konfiguračních BLOB Wide Memo polích. Příkladem je **`SET_FIRM`**, který řídí chování karty Organizace:

```ini
[BASE_SET]
APPENDFORM=quick                 ; Režim zakládání záznamu (rychlý formulář)

[EASYAPPEND_SET]
PROPERTY_ICO=1                  ; Aktivace validace/načítání podle IČO
PROPERTY_CPCOBPA1=4109           ; Přednastavení výchozího zatřídění

[LISTS]
Tabs=LINK,NOTES_HOTLINE,EVENT,TABLE_96_6,EMAIL,DOCUMENT,AI  ; Zobrazené spodní záložky
Details=CONNECT,PERSONFIRM       ; Postranní detailní panely
Inspector="FIRMA","PFORMA","ULICE","MISTO","PSC","ICO","DIC","ADDG_Licence" ; Pole formuláře
Grid=SELECTION,X,FIRMA+PFORMA,ULICE,MISTO+PSC,Y              ; Sloupce přehledové mřížky
ToolBar=TREE,SEL,FILTER,VIEWPRINTACT,CATALOG                 ; Tlačítka v nástrojové liště
Views=96_46,96_20,96_88          ; Dostupné uživatelské pohledy
