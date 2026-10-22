# 01. Vize systému, architektura a 5 primárních entit (Subjektů)

Tento dokument popisuje základní koncepci, architekturu a entitní model webové platformy **Databox CONTACT Web**.

---

## 1. Hlavní vize a koncept platformy

Platforma **Databox CONTACT Web** je navržena jako vysoce flexibilní a modularizovaný znalostní a operativní systém (CRM / ERP nadstavba) postavený na databázovém jádru **NexusDB**. 

### Hlavní pilíře architektury:
* **Minimalistické jádro, vysoká extenzibilita:** Základní databázové tabulky primárních entit obsahují pouze nezbytné systémové a identifikační sloupce. Veškerá zákaznická kustomizace se realizuje pomocí dynamických polí (**`ADDSET`**) a uživatelských podřízených tabulek (**`TABSET`**).
* **Konfigurace pomocí INI bloků:** Chování UI, validace, skripty, tiskové sestavy a automaty se definují pomocí strukturovaného nastavení (`BASE_SET`, `PROFILSET`, `USERSET`) uloženého přímo v databázi.
* **Procesní řízení (Workflow):** Řízení životního cyklu záznamů, zamykání formulářů, spouštění SQL procedur a podmíněná viditelnost polí jsou navázány na stavová pole (např. **`COLOR`**).

---

## 2. Pět primárních entit (Subjektů) a ID mapping

Celý systém je vnitřně provázán přes jednotný číselník entitních typů (**`ID`**). Tento identifikátor určuje, se kterým typem objektu databáze i aplikační logika pracuje:

| ID | Entita (Subjekt) | Popis a význam v systému |
| :---: | :--- | :--- |
| **1** | **Organizací / Firmy (`FIRM`)** | Právnické i fyzické osoby, úřady, školy a instituce. Tvoří centrální uzel adresáře. |
| **2** | **Osoby (`PERSON`)** | Kontaktní osoby, zaměstnanci klientů i fyzické osoby navázané na organizace či projekty. |
| **3** | **Události (`DIAR` / `TIME`)** | Aktivity, kalendářové záznamy, zápisy z jednání, úkoly, e-maily a plánované akce. |
| **4** | **Projekty / Bloky (`NOTEBLOCK`)** | Zakázky, smlouvy, stroje, obchody, servisní případy a jakékoliv procesní spisy. |
| **7** | **Uživatelé (`USERS`)** | Interní uživatelé systému, pracovníci a nositelé přístupových práv a rolí. |

---

## 3. Složené relace a křížové propojení (CPCx / Cx)

Všechny primární entity i uživatelské tabulky využívají pro identifikaci a vzájemné propojování koncept **složeného klíče**:

* **`CPCX` (Číslo profilu / Klientského klastru):** Identifikátor profilu nebo databázového klastru (např. `CPCF`, `CPCP`, `CPCN`, `CPCT`).
* **`CX` (Unikátní ID záznamu):** Unikátní číslo záznamu v rámci daného profilu (např. `CF`, `CP`, `CN`, `CT`).

### Křížové obousměrné vazby:
Díky univerzálnímu navazování přes trojici polí **`ID1`** (typ subjektu 1, 2, 4), **`CPCX1`** a **`CX1`** dokáže systém propojovat jakýkoliv záznam (např. fakturu nebo servisní výkaz) s libovolnou primární entitou. Vzniklé vazby jsou plně **obousměrné** a přístupné jak z karty firmy, tak z detailu smlouvy či projektu.

---

## 4. Kompletní architektura
Profil (Název - většinou podle jména firmy a číslo profilu)
├── 01_Uživatel/
│   ├── Login, Barva, Oddělení, Skupina, Práva, Zobrazení adresáře, Nastavení diáře
│   ├── Informace o uživateli = Dodatky -> Datum narození, adresa, zaměstnanecké číslo a podobně
│   └── Uživatelské agendy = Tabulky -> řádky se váží na uživatele a primárně každý uživatel vidí pouze svoje řádky - Práva na sdílení řádků.
├── 02_Organizace/Firmy
│   ├── 01_Kontakty (Vazba na osoby - Priorita, Funkce, Oddělení) + Poznámka 
│   ├── 02_Dodatky + Skupiny dodatků
│   ├── 03_formulas_events.md    # Syntaxe výpočtů a spouštění procedur
│   └── 04_master_detail.md      # Jak se staví 1:N tabulky
└── 03_user_guides/
    ├── 01_servis_a_stroje.md    # Proces servisu, MTH, výkazů
    ├── 02_kniha_jizd.md         # Proces evidování jízd a vyúčtování
    └── 03_marketing_emails.md   # Šablony, odosílání, statistiky
