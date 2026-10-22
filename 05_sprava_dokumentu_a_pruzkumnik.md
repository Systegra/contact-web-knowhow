# 05. Správa dokumentů u subjektů, architektura úložiště a Průzkumník souborů

Tento dokument detailně popisuje architekturu ukládání dokumentů v systému **Databox CONTACT Web**, bezpečnostní model skládání fyzických cest na back-endu (`ServerIS`), databázovou evidenci v NexusDB, klientské otevírání souborů skrze pomocnou aplikaci `ShellIS.exe` a dva základní režimy práce s dokumenty u primárních entit.

---

## 1. Databázová architektura a uložení dokumentů

V systému existuje striktní oddělení mezi obecnou správou dokumentů primárních entit a přílohami k dynamickým uživatelským tabulkám:

### A. Centrální registratura dokumentů (`DOCUMENT`)
Tabulka `DOCUMENT` v databázi NexusDB tvoří hlavního spráce vazeb a metadat pro 5 primárních subjektů. 

#### Databázová struktura tabulky `DOCUMENT`:
* **`ID` (Typ primární entity / subjektu):**
  * `51` = Firma / Organizaci
  * `52` = Osoba
  * `53` = Projekt / Blok (`NOTEBLOCK`)
  * *(a další primární subjekty)*
* **`CPCX` & `CX`:** Složený klíč identifikující konkrétní záznam daného subjektu. `CPCX` určuje ID profilu (např. `1000`) a `CX` je unikátní ID konkrétního záznamu (např. ID firmy nebo ID projektu).
* **`CPCT` & `CT`:** Kladné identifikátory pro vazbu na typy, kategorie či hlubší vazby v rámci struktury.
* **`DESC` (Název / Popis):** Uživatelský název souboru nebo jeho stručný popis (např. `Sellier & Bellot a.s._231001_JT2.pdf`).
* **`PATH` (Relativní cesta):** Relativní umístění souboru pod hlavním root adresářem (např. `D:\MAG\SELLIER & BELLOT\Sellier & Bellot a.s._231001_JT2.pdf`).
* **`CATEGORY` & `MEMO`:** Uživatelské zatřídění (např. *Smlouvy*, *Nabídky*, *Certifikáty*) a detailní víceřádkový textový popis dokumentu.
* **`CREATOR` & `LASTEDIT`:** Auditní pole uchovávající přihlašovací jméno autora vložení a uživatele, který prováděl poslední úpravu.
* **`CRDATE`, `EDDATE`, `SYNDATE`:** Časová razítka vytvoření záznamu, poslední editace a synchronizace.

### B. Soubory uživatelských tabulek (`TABFILES`)
Tabulka `TABFILES` slouží výhradně jako úložiště pro souborové přílohy navázané na jednotlivé řádky dynamických tabulek (`TABROW`). Nepletou se tedy s obecnými spisy firem či projektů v tabulce `DOCUMENT`.

---

## 2. Bezpečnostní architektura a dynamické skládání cest (`ServerIS`)

Z bezpečnostních důvodů klientská aplikace (**CONTACT Web**) nikdy nepracuje s absolutními fyzickými cestami na serveru. Reálná struktura disků je pro front-end zcela skrytá a zná ji pouze back-end server `ServerIS`.

### Mechanismus skládání cesty na back-endu:
1. **Definice kořenového adresáře (`ServerIS.ini`):**
   Na úrovni API serveru je v konfiguračním souboru definován pevný root adresář profilu:
   ```ini
   [SystegraIS]
   DocumentDir=D:\MAG\

-------------------------

1.Dynamická šablona cesty (PROFILSET -> SET_FIRM):
V konfiguračním INI bloku dané agendy (v databázi PROFILSET) je v sekci [DOCUMENT] definována pravidlo-šablona s využitím dynamických makro-značek:Ini, TOML[DOCUMENT]
ExportDir=\Projekty\[Firma.Firma]
Vyhodnocení makra na serveru:
Při požadavku na zobrazení či uložení dokumentu back-end server ServerIS nahradí dynamické makro [Firma.Firma] reálnou hodnotou názevu firmy z databáze (např. ANVI TRADE).Složení absolutní cesty:
Server sloučí root složku s vyhodnocenou dynamickou cestou:D:\MAG\ + \Projekty\ANVI TRADE\ $\rightarrow$ D:\MAG\Projekty\ANVI TRADE\Hierarchické mapování u Projektů (NOTEBLOCK):Složky projektů nevznikají izolovaně, ale skládají se dynamicky s ohledem na nadřazeného zákazníka podľa klíče:[Root]\Projekty\[Zákazník]\[Název projektu] (např. D:\MAG\Projekty\ANVI TRADE\Nabídky pro ABRA).Křížová viditelnost (Dědičnost náhledu): Jelikož je složka projektu fyzicky umístěna jako podadresář v adresáři firmy, u karty Firmy v Průzkumníku souborů uživatel vidí složky všech jejích projektů. Naopak na kartě Projektu přistupuje přímým výřezem do konkrétní podřazené složky.

3. Dva způsoby práce s dokumenty v UI (CONTACT Web)
V detailním pohledu karty subjektu (Firma, Osoba, Projekt) má uživatel k dispozici dvě záložky pro práci s dokumenty:

A. Režim „DOKUMENTY“ (Plochý seznam příloh)
Zobrazuje lineární seznam souborů připojených k danému záznamu z tabulky DOCUMENT.

Připomíná práci s přílohami v e-mailovém klientovi.

Prvky rozhraní:

Tlačítko + (Plus) pro ruční nahrání nového dokumentu.

Mřížka zobrazující ikonu typu souboru, název, název kategorie, datum, čas a jméno autora.

Akční tlačítka u každého řádku: Náhled (ikona lupy/dalekohledu), Stáhnout (šipka dolů) a Smazat (ikona koše).

Stránkovací lišta na spodním okraji (volba 10, 25, 50 položek na stránku).

B. Režim „PRŮZKUMNÍK SOUBORŮ“ (Síťový adresářový strom)
Zprostředkovává přímý živý pohled do fyzické adresářové struktury na síťovém disku serveru (např. \\10.180.200.20\Systegra\Projekty\ANVI TRADE).

Obousměrná transparentnost:

Soubor přidaný přes běžný Průzkumník ve Windows přímo na síťový disk se ihned zobrazí ve webovém rozhraní CONTACT Web.

Soubor vložený ve webovém rozhraní (přes tlačítko Nahrát soubory nebo přetažením Drag & Drop) se fyzicky zapíše do složky na síťovém disku.

Práce s adresáři: Umožňuje vytváření podadresářů (např. Nabídky, Newsletter, Objednávky na sklad, Šablony) a zachovává přehlednou stromovou strukturu spisu.

4. Přímé otevírání dokumentů z prohlížeče (ShellIS.exe)
Aby uživatelé při práci s webovým rozhraním nemuseli soubory neustále stahovat do lokální složky Stažené soubory a manuálně je otevírat, využívá systém pomocnou klientskou aplikaci.

SYSTEGRA Information Services Shell (ShellIS.exe):

Lokálně nainstalovaná utilita na PC uživatele, která do OS Windows zaregistruje vlastní URI protokol (Custom Protocol Scheme).

Průběh akce:

Uživatel v CONTACT Web klikne na otevření dokumentu (Word, Excel, PDF, CAD atd.).

Webový prohlížeč zachytí volání vlastního protokolu a zobrazí systémové okno: "Spustit aplikaci SYSTEGRA Information Services Shell?".

Po potvrzení přebere aplikaci ShellIS.exe příkaz s argumenty, přistoupí k souboru na síťové cestě a rovnou jej otevře v příslušné asociované aplikaci (MS Word, MS Excel atd.) bez mezipaměti v adresáři stažených souborů.




