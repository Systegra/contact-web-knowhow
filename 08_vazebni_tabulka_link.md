# 08. Centrální vazební tabulka LINK

Tento dokument detailně popisuje architekturu centrální vazební tabulky **`LINK`**, která zajišťuje veškeré M:N relace, křížové vazby a propojení mezi všemi agendami v systému **Databox CONTACT Web**.

---

## 1. Databázová struktura a primární klíč

Tabulka `LINK` funguje jako univerzální databázový most propojující různé subjekty (Firmy, Osoby, Události, Projekty, E-maily) bez nutnosti modifikovat schémata jednotlivých agend[cite: 13].

### Klíčové databázové atriibuty:
* **Složený primární klíč:** Jednoznačná identifikace záznamu je určena kombinací polí **`ID`**, **`CPCL`** (ID profilu) a **`CL`** (ID vazebního záznamu)[cite: 13, 14].
* **Zdrojová strana (`CPCX` & `CX`):** Identification profilu (`CPCX`) a ID konkrétního záznamu (`CX`) na levé straně relace[cite: 13, 14].
* **Cílová strana (`CPCY` & `CY`):** Identifikátor profilu (`CPCY`) a ID konkrétního záznamu (`CY`) na pravé straně relace[cite: 13, 14].
* **Typologie a charakter vztahu:**
  * **`FUNCTION` (Funkce / Role):** Specifikuje roli vztahu (např. *Jednatel*, *Fakturační kontakt*, *Garant projektu*)[cite: 13].
  * **`SECTION` (Sekce / Úsek):** Bližší zařazení či sekce daného vztahu[cite: 13].
  * **`PRIORITA` (Priorita):** Celočíselné určení váhy nebo pořadí vazby[cite: 13].
* **Auditní pole:** `CREATOR`, `LASTEDIT`, `CRDATE`, `EDDATE`, `SYNDATE` pro sledování historie vzniku a úprav[cite: 13].

---

## 2. Číselník typů vazeb podle pole `ID`

Pole **`ID`** určuje, mezi jakými tabulkami/agendami vazba vzniká:

| ID | Typ vazby | Zdroj (`CPCX, CX`) | Cíl (`CPCY, CY`) | Popis / Využití |
| :--- | :--- | :--- | :--- | :--- |
| **`10`** | **Firma – Osoba** | `FIRM` (51) | `PERSON` (52) | Základní vazba kontaktní osoby na firmu[cite: 13]. |
| **`11–14`** | *Interní konstanty* | — | — | Pouze konstanty v kódu aplikace, v datech se nevyskytují[cite: 13]. |
| **`15`** | **Firma – Firma** | `FIRM` (51) | `FIRM` (51) | Vztahy mezi organizacemi (mateřské/dceřiné firmy, partnerství)[cite: 13]. |
| **`16`** | **Osoba – Osoba** | `PERSON` (52) | `PERSON` (52) | Vztahy a vazby mezi fyzickými osobami[cite: 13]. |
| **`19`** | **Firma – Archiv. osoba** | `FIRM` (51) | `PERSON` (52) | Obdoba ID 10 pro archivované či neaktivní osoby[cite: 13]. |
| **`21`** | **Událost – Firma** | `TIMELINE` | `FIRM` (51) | Navázání události z Diáře na konkrétní firmu[cite: 13]. |
| **`22`** | **Událost – Osoba** | `TIMELINE` | `PERSON` (52) | Navázání události z Diáře na konkrétní osoba[cite: 13]. |
| **`23`** | **Událost – Událost** | `TIMELINE` | `TIMELINE` | Propojení souvisejících událostí v Diáři[cite: 13]. |
| **`24`** | **Událost – Projekt** | `TIMELINE` | `NOTEBLOCK` (4) | Navázání události z Diáře na Projekt/Blok[cite: 13]. |
| **`111`** | **Projekt – Firma** | `FIRM` (51) | `NOTEBLOCK` (4) | Vazba projektu na firmu[cite: 13]. |
| **`112`** | **Projekt – Osoba** | `PERSON` (52) | `NOTEBLOCK` (4) | Vazba projektu na osobu[cite: 13]. |
| **`113`** | **Projekt – Událost** | `TIMELINE` | `NOTEBLOCK` (4) | Alternativní vazba události na projekt (často nahrazováno ID 24)[cite: 13]. |
| **`114`** | **Projekt – Projekt** | `NOTEBLOCK` (4) | `NOTEBLOCK` (4) | Hierarchické propojení projektů (podprojekty, etapy)[cite: 13]. |
| **`26`** | **E-mail – Firma** | `FIRM` (51) | `EMAIL` | Navázání e-mailu na firmu[cite: 13]. |
| **`27`** | **E-mail – Osoba** | `PERSON` (52) | `EMAIL` | Navázání e-mailu na osobu[cite: 13]. |
| **`29`** | **E-mail – Projekt** | `NOTEBLOCK` (4) | `EMAIL` | Navázání e-mailu na projekt[cite: 13]. |

---

## 3. Význam pro architekturu a AI integraci

* **360° Pohled na klienta:** Díky tabulce `LINK` dokáže systém z libovolného bodu (např. z karty Osoby) okamžitě dohledat mateřskou firmu, všechny související projekty, e-maily i veškeré události v Diáři[cite: 13].
* **Klíčový podklad pro AI:** Při vytváření kontextových promptů a analyzování dat AI modely se tabulka `LINK` používá jako primární mapa grafu vztahů, která propojuje strukturovaná data s textovými záznamy v databázi.
