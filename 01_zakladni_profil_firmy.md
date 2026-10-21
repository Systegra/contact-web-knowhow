# 01. Základní profil firmy, databázový fundament a kategorizace (X/Y)

Tento dokument popisuje počáteční fázi implementace systému **Databox CONTACT Web**, nastavení profilu držitele licence v databázi NexusDB a princip vizuální kategorizace subjektů.

---

## 1. Správa profilu v uživatelském rozhraní
V sekci **Nastavení profilu -> Obecné** se definují základní identifikátory a kontaktní údaje vlastní firmy (držitele licence):
* **Firma / Název:** Oficiální název společnosti.
* **Adresní údaje:** Ulice, Adresa 2, Město, PSČ, Kraj, Stát.
* **Kontaktní údaje:** WWW adresa, e-mail / fax.

---

## 2. Databázová úroveň: Tabulka `PROFILSET` a klíč `PROFIL_CP`

Konstrukce celého prostředí vychází z databázové tabulky **`PROFILSET`**, která zpravidla obsahuje **jediný záznam (jednu větu)** definující globální parametry instance:

* **`PROFIL_NAME`:** Název profilu / aplikace.
* **`PROFIL_CP` (Database ID):** Zásadní číselný identifikátor databáze/profilu. Z tohoto čísla se odvozují křížové indexy v navázaných tabulkách (`CPCX`, `CPCF`, `CPCO` atd.).

### Přehled INI konfiguračních polí v `PROFILSET`:
1. **`SET_COMMON`:**
   * `[ADDRESS]`: Identifikační a adresní údaje vlastní firmy.
   * `[DUPLICITY_CHECK]`: Nastavení kontroly duplicit.
   * `[EMAIL]`: Konfigurace SMTP a odesílání pošty.
   * `[PROFILBAR]`: Systémové nastavení URL aplikace (`ContactWeb=...`) a modulů.
   * `[DOCUMENT]`: Globální pravidla pro práci s dokumenty.
2. **`SET_FIRM`:** Konfigurace agendy Firem (názvosloví, cesty pro exporty, definice ikon, barev a výchozích stavů).
3. **`SET_PERSON`:** Konfigurace agendy Osob (názvosloví, pravidla přebírání adres z firem, šablony oslovení, definice rolí).
4. **`SET_NOTE`:** Globální číselník ikon, priorit a přístupových práv pro Databázové bloky.
5. **`SET_DIAR`:** Nastavení rychlého zadávání záznamů (`[EASYAPPEND]`) a formulářů.

---

## 3. Logika Statusů (X) a Kategorií (Y)

Pro operativní rozdělení a přehlednost Firem i Osob systém používá dvoudimenzionální klasifikaci označenou jako **X** a **Y**:

| Parametr | Dřívější termín v INI | Vizuální prvek | Význam a použití |
| :--- | :--- | :--- | :--- |
| **Status (X)** | `ICONS` / `IconsID` | **Ikona** (obrázek `0.png`–`80.png`) | Vyjadřuje aktuální stav (např. *Aktivní provozka*, *Nekontaktovat*, *V likvidaci*). |
| **Kategorie (Y)** | `PRIORITY` / `PriorityID` | **Barva** štítku / pilulky | Vyjadřuje typ nebo vztah k firmě (např. *Dodavatel*, *Partner*, *Potencionální*, *Pasivní*). |

### Vizuální navázání v aplikaci:
* **Ikony Statusu (X):** Jsou uloženy v číselné řadě (`0.png` až `80.png` + `mapMarker.png`). V INI konfiguraci (`SET_FIRM` / `SET_PERSON`) se do pole `IconsID` zadávají přímo tato čísla oddělená čárkami, která odpovídají textovým popisům v `IconsDesc`.
* **Kategorie (Y):** V UI vystupují jako barevné filtrovací pilulky v záhlaví seznamu s počty záznamů. V INI konfiguraci se definují přes `PriorityID`, `PriorityDesc` a číselné kódy barev v `PriorityColor`.
