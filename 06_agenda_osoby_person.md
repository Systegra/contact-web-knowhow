# 06. Agenda Osoby (PERSON / ID 52), statusy, kategorie a B2B/B2C kontexty

Tento dokument popisuje databázovou architekturu agendy Osoby, její vazby na firmy a dokumenty, způsob kategorizace skrze pole `X` a `Y` a různé business kontexty použití v systému **Databox CONTACT Web**.

---

## 1. Databázová architektura a struktura tabulky `PERSON`

Všechny fyzické osoby v systému jsou evidovány v centrální tabulce **`PERSON`** (ID entity **`52`**) v databázi NexusDB[cite: 10].

### Klíčová databázová pole:
* **`UID` (GUID):** Unikátní globální identifikátor záznamu osoby[cite: 10].
* **`CPCO` & `CO`:** Identifikátor profilu (`CPCO`) a unikátní ID osoby (`CO`) v rámci profilu[cite: 10].
* **Jmenné a titulové atributy:**
  * **`FNAME` / `SNAME`:** Jméno a příjmení osoby[cite: 10].
  * **`FTITUL` / `STITUL`:** Akademické tituly před a za jménem (např. *Ing.*, *Bc.*, *Ph.D.*)[cite: 10].
* **Adresní atributy:** `ULICE`, `ULICE2`, `MISTO`, `PSC`, `REGION`, `STAT` pro kompletní evidenci adresního sídla či bydliště[cite: 10].
* **Kategorizace a Status (`X` & `Y`):**
  * **`X` (Status osoby):** Celočíselná hodnota určující provozní stav (např. *řešitel*, *významný*, *kontaktovat*, *nekontaktovat*, *nekomunikuje*, *smazat*)[cite: 10, 11].
  * **`Y` (Kategorie / Role):** Celočíselná hodnota určující profesní zatřídění či roli (např. *obchodník*, *IT správce*, *konzultant*, *manager*, *majitel*, *programátor*)[cite: 10, 11].
* **Auditní stopa a poznámky:** `POZN` (BLOB pole pro libovolnou poznámku), `CREATOR`, `LASTEDIT`, `CRDATE`, `EDDATE` a `SYNDATE`[cite: 10].

---

## 2. Business kontexty použití agendy Osoby

Flexibilita tabulky `PERSON` umožňuje její nasazení ve dvou odlišných obchodních modelech:

1. **B2B sféra (Kontaktní osoby u organizací):**
   * Osoby fungují jako zástupci, zaměstnanci nebo kontakty navázané na primární firmy (`FIRM` / ID 51)[cite: 10, 11, 12].
   * V tabulkách, formulářích a dokladech (nabídky, zakázky, aktivity) vystupují často jako vazba typu **`CX1`** (určení konkrétní kontaktní/přebírající osoby)[cite: 10, 11, 12].
2. **B2C sféra a specifické profily (Samostatní klienti):**
   * U zákazníků využívajících **CONTACT Web** pro koncový trh (např. realitní makléři, pojišťovací poradci, finanční konzultanti) tvoří osoby hlavní primární agendu.
   * V těchto profilech firma jako subjekt buď nefiguruje vůbec, nebo plní pouze sekundární roli.

---

## 3. Uživatelské rozhraní v CONTACT Web

### A. Mřížka a filtrování (Seznam osob)
* **Tagy a filtrovací lišta:** Umožňuje okamžité filtrování podle rolí a statusů pomocí barevných odznáčků (např. *obchodník*, *manager*, *majitel*)[cite: 11].
* **Rychlé změny statusu:** Umožňuje přímo z mřížky měnit stav osoby (např. příznak *řešitel* nebo ikona *významný*)[cite: 11].
* **Přehled vazeb:** Sloupce zobrazují jméno, titul, přiřazenou firmu, oddělení a funkci[cite: 11].

### B. Detailní karta Osoby
* **Hlavička:** Zobrazuje jméno s tituly, přiřazené tagy role a statusu a jméno odpovědného pracovníka[cite: 12].
* **Kontakty & Vazba na organizaci:** Zobrazuje propojenou firmu (např. `NAVY MAX s.r.o.`), adresy a strukturované komunikační kanály (telefony, e-maily s odlišením typů jako *Web - uživatel*, *fakturační*)[cite: 12].
* **Dodatečná pole:** Obsahuje dynamická pole pro lingvistické skloňování (např. *Oslovení 5. pád* $\rightarrow$ *Karásková*, *Oslovení* $\rightarrow$ *paní*)[cite: 12].
* **Návazné záložky a historie:** Součástí karty jsou podzáložky pro **Galerii** (fotografie/média), **Dokumenty**, **E-maily** a **Události/Aktivity** vázané přímo k dané osobě[cite: 12].
