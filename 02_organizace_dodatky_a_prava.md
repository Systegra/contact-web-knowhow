# 02. Organizace, Dodatky (ADDSET) a Přístupová práva

Tento dokument popisuje konfiguraci agendy Organizací (Firem), architekturu vlastních polí (Dodatků) a logiku přístupových práv v systému **Databox CONTACT Web**.

---

## 1. Koncept Organizace a Vlastníka (Obchodník)

### Terminologie
V systému se dříve používal pojem *Firma*, nicméně pro pokrytí univerzálních potřeb zákazníků (školy, úřady, neziskové organizace, státní instituce) se pracuje s obecným konceptem **Organizace / Subjekt**. Názvosloví v UI je plně konfigurovatelné.

### Vlastník Organizace (`DOMNAME`)
Každá Organizace (stejně jako Osoba, Projekt nebo Událost) má právě jednoho primárního vlastníka:
* **Identifikace:** V databázi je vlastník reprezentován pomocí pole **`DOMNAME`** (přihlašovací doména uživatele), což je hlavní uživatelský index.
* **Pojmenování v UI:** V rozhraní se pole zobrazuje typicky jako *Obchodník*, *Správce kontaktu* nebo *Vlastník*.

---

## 2. Řízení přístupových práv a viditelnosti podle Vlastníka

Pole vlastníka (`DOMNAME`) slouží jako primární filtr pro řízení přístupu a viditelnost záznamů:

1. **Globální viditelnost:** Uživatel vidí a upravuje všechny Organizace v databázi.
2. **Omezení edice:** Uživatel vidí všechny Organizace, ale upravovat může pouze ty, kde je zapsán jako vlastník.
3. **Striktní privátní režim:** Uživatel vidí výhradně Organizace, u kterých je přihlášen jako vlastník (`DOMNAME`). Cizí organizace jsou v adresáři skryté.

### Příklady INI konfigurace práv:
* **`PROFILSET.SET_FIRM -> [ICON_SECURITY]`**:
  * `Secured=2,5,3` – Zabezpečené ikony/statusy (přístup mají pouze uživatelé s nastavením v `USERSET`).
  * `Confidential=4,6,5` – Důvěrné ikony (přístup má pouze vlastník firmy nebo člen povolené skupiny).
* **`PROFILSET.SET_FIRM -> [RIGHT]`**:
  * `Enabled=1`, `HideUnauthorized=1` – Skrytí neautorizovaných firem z přehledu.
* **`USERSET.SET_FIRM -> [RIGHTS]`**:
  * `BasicEdit=1` – Povoluje úpravu statusu, kategorie a poznámky i v režimu čtení.
* **`USERSET.SET_USER -> [RIGHTS]`**:
  * `HeadOfGroups=1000_3343` – Povoluje uživateli měnit vlastníka firmy, pokud stávající vlastník patří do dané skupiny.

---

## 3. Architektura Dodatků (Tabulka `ADDSET`)

Dodatky představují custom doplňková pole pro různé agendy.

### Struktura tabulky `ADDSET`:
* **`ID` (Určení subjektu):**
  * `41` = **Firmy / Organizace**
  * `42` = Osoby
  * `43` = Události
  * `44` = Projekty / Bloky
  * `47` = Uživatelé
* **`CPCM` + `CM` (Složený klíč):** `CPCM` odkazuje na číslo profilu (`PROFIL_CP`, např. `1000`), `CM` je unikátní pořadové číslo dodatku v rámci profilu.
* **`CPCC` + `CC` (Vazba na číselník):** Pro doplňková pole typu výběrový seznam tvoří relaci do tabulky `CATEGORYTREE`.
* **`DESC`:** Název doplňkového pole v UI (např. *Servisní smlouva*, *KAM*, *Počet strojů*).
* **`TYP`:** Datový formát a ovládací prvek (text, číslo, datum, seznam, checkbox, poznámka apod.).
* **`VISIBLE` / `READONLY`:** Příznaky viditelnosti a zamezení editace.
* **`PORADI`:** Pořadí zobrazení na formuláři.

### Detailní nastavení v INI poli `BASE_SET` (tabulka `ADDSET`):
* **`[BASE]`**: `Hint=`, `Required=1`, `SumEnabled=0`, `DefValue=`, `Hypertext=1`.
* **`[EXTENDED]`**: `PswGen=1` (generování hesla), `EmailFromConnect=1` (převzetí e-mailu ze spojení), `TableViewer=...` (zobrazení navázané tabulky).
* **`[STORED_FUNCTION]`**: `Name=...` (vypočítávaná hodnota ze zadané funkce nebo suma z podtabulky).
* **`[WEB_VALIDATION]`**: `Rule=>0` (validace vstupní hodnoty).

---

## 4. Zobrazení Dodatků v Uživatelském Rozhraní

1. **Uživatelská viditelnost:** V nastavení uživatele (`USERSET`) lze definovat, které konkrétní dodatky se danému uživateli zobrazují na hlavní kartě adresáře.
2. **Seskupení do Akordeonů:** Dodatky lze řadit do skupin (definovaných v tabulce kategorií), které v UI tvoří samostatné rozbalovací panely (akordeony).
3. **Výchozí panel „Dodatečná pole“:** Pokud dodatek není zařazen do žádné vlastní skupiny, systém jej automaticky vykreslí ve výchozím panelu s názvem **Dodatečná pole** (např. *Dodatečná pole (7/15)*).
