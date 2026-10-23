09. Databázové bloky a Projekty (`NOTEBLOCK` / NOTE ID 4)

Tento dokument detailně popisuje architekturu, datový model, uživatelskou konfiguraci a obchodní logiku Databázových bloků a Projektů v systému Databox CONTACT Web.
---
1. Architektura a koncepce Databázových bloků
Databázové bloky (zkráceně Bloky) představují nejvyšší úroveň přizpůsobitelných procesních databází v systému Databox CONTACT Web. Stojí na stejné úrovni jako klíčové entitní agendy (Firmy, Osoby).
Rozhodovací logika: Uživatelská tabulka vs. Databázový blok
Uživatelská tabulka: Volí se pro jednoduché, ploské evidence s několika základními atributy (např. jednoduchá evidence vydaných nabídek obsahující pouze datum, částku a stav).

Databázový blok: Volí se pro rozsáhlé, komplexní a procesně orientované agendy, které vyžadují celý životní cyklus, navázané pod-tabulky (režim Master-Detail), dokumenty, galerii, časový výkaz, události v Diáři a vlastní automatizované workflows (např. řízení obchodního případu od poptávky po realizaci, servisní činnosti, správa strojního parku).

Typické agendy řešené přes Databázové bloky
Projektové a Obchodní agendy:
Obchodní případy / Příležitosti
Zakázky a Etapy
Projekty (SW/HW vývoj, stavby)
Marketingové kampaně
Výběrová řízení

Servisní a Majetkové agendy:
Stroje a zařízení
Servisní zakázky / Hotline
Nájemní a prodejní zakázky
Příslušenství a vybavení

---
2. Databázový model a struktura tabulek
Architektura bloků je zkonstruována minimalisticky. Místo neustálého přidávání pevných databázových sloupců staví na kombinaci základních stavových atributů v hlavní tabulce, doplňkových dynamických parametrů (Dodatků) a navázaných relací.
```
+------------------+         1 : N         +----------------------+
|    NOTEBLOCK     | --------------------> |         NOTE         |
| (Definice bloku) |                       | (Záznamy / Projekty) |
+------------------+                       +----------------------+
                                                      |
                                       +--------------+--------------+
                                       |              |              |
                                  1:N  v         1:N  v         1:N  v
                                 +----------+   +----------+   +----------+
                                 |  TABLES  |   |   LINK   |   | ADDITION |
                                 | (Tabulky)|   | (Vazby)  |   | (Dodatky)|
                                 +----------+   +----------+   +----------+
```
A. Tabulka `NOTEBLOCK` (Definice bloku)
Slouží pro registraci a globální konfiguraci dané agendy (bloku).

Field Name	Type	Description

`BLOCKNAME`	WideString(21)	Systémový identifikátor bloku

`NAZEV`	WideString(50)	Zobrazovaný název agendy v UI

`IDENT`	WideString(10)	Kódové označení agendy

`PUBLIC`	Boolean	Příznak veřejné/globální dostupnosti

`BASE_SET`	BLOB Wide Memo	INI konfigurace globálního chování, UI, akcí a tabulek

`FIELD_SET`	BLOB Wide Memo	INI konfigurace vlastností jednotlivých formulářových polí

`CREATOR / LASTEDIT`	WideString(21)	Auditní informace o autorovi a posledním editorovi

`CRDATE / EDDATE`	DateTime	Datum vytvoření a poslední úpravy

B. Tabulka `NOTE` (Záznam / Projekt)
Základní datovou jednotkou jakéhokoliv databázového bloku je Projekt (uložený v tabulce `NOTE`).

Field Name	Type	Description

`UID`	GUID	Unikátní identifikátor záznamu

`CPCN` / `CN`	Int32	ID profilu (`CPCN`) a číslo záznamu (`CN`)

`IDENT`	WideString(30)	Číslo zakázky / identifikátor (např. 26-008)

`NAZEV`	WideString(250)	Název projektu / zakázky

`STARTDATE` / `ENDDATE`	Date	Časové ohraničení projektu (datum zahájení a ukončení)

`ICONTYPE`	Int16	Status projektu (odkazuje na ikonku statusu z profilu)

`PRIORITY`	Int16	Kategorie projektu (barva a klasifikace záznamu)

`ID1`, `CPCX1`, `CX1`	Int16 / Int32	Primary Vazba ID 1 na hlavního klienta / firmu / osobu

`NOTES_OWNER`	WideString(21)	Odpovědný vlastník / garanta (obchodník, projektant, technik)

`MEMO` / `MEMOTYPE`	BLOB Memo / Int16	Hlavní textová poznámka / detail (HTML nebo plain text)

`MEMOOWNER`	WideString(21)	Vlastník textové poznámky

---
3. Vizuální identita a UX
Barevné kódování agend: Každý Databázový blok má přiřazenou svou unikátní barvu v rozhraní (propisuje se do hlaviček, bočních lišt a stromového menu). Vzhledem k unifikovanému vzhledu formulářů a mřížek napříč systémem slouží barevné odlišení jako okamžitý vizuální kotvící bod (např. Stroje a zařízení = Fialová, Projekty/Zakázky = Červeno-hnědá).
360° Pohled u klienta: Na kartě konkrétní firmy nebo osoby se v záložce Projekty zobrazují všechny navázané blokové záznamy včetně jejich aktuálních statusů, kategorií, časového průběhu a vykázaného času.

---
4. Konfigurační model (INI soubory v `NOTEBLOCK`)
Konfigurace chování a vzhledu každého bloku je uložena přímo v záznamu tabulky `NOTEBLOCK` ve dvou klíčových paměťových polích `BASE_SET` a `FIELD_SET`.

A. Globální nastavení: `BASE_SET`
Řídí globální strukturu UI, záložky, použité číselníky, napojené moduly a skriptovací akce.
`[ICONS]` & `[PRIORITY]`: Definují povolenou sadu ikonek (statusů) a priorit (kategorií) z databáze `PROFILESET` a jejich výchozí hodnoty (`DefIconType`, `DefPriority`).
`[BASE]`: Systémová nastavení agendy, včetně odkazu na spouštěné databázové procedury po uložení (`WEB_AfterPost=Fce_SYST_ProjektDur`) a ikonu agendy (`Agenda4.png`).
`[LISTS]`:
`Tabs`: Navázané záložky na kartě (např. `LINK`, `FIRM`, `PERSON`, `DOCUMENT`, `EMAIL`, `TABLE_*`, `EVENT`, `ADDITION`).
`Details`: Aktivní rozbalovací widgety na hlavní kartě (např. výkazy, události, dokumenty).
`Inspector`: Výčet polí a skupin dodatků zobrazených v pravém/detailním panelu.
`Grid`: Specifikace sloupců a jejich pořadí v hlavním přehledovém seznamu projektů.
`[DOCUMENT]`: Definice cesty a názvové konvence pro automatický export a ukládání dokumentů (např. `ExportDir=Projekty\[Projekt.Subjekt1]\[Projekt.Název]`).
`[IDENT]`: Pravidla pro automatické generování číselných řad (např. `Expression=[YY]-[N.3]` vytváří kód 26-008).
`[NOTIF_*]` & `[RIGHT]`: Matice přístupových práv, úprav a automatických notifikací pro vlastníky a uživatele při změnách v projektu či navázaných podsystémech.

B. Nastavení polí: `FIELD_SET`
Definuje chování konkrétních polí formuláře a podmíněnou logiku zobrazení.
Parametry atributů: U jednotlivých polí (např. `NAZEV`, `STARTDATE`, `MEMOOWNER`) nastavuje vlastnosti jako:
`Required=1` (povinné pole)
`ReadOnly=1` (pouze pro čtení)
`BoldValue=1` (tučné zvýraznění hodnota)
`AlarmDayPrev` / `AlarmDayNext` (upozornění před/po datu)
Podmíněné zobrazení dodatků (`[ADD_*_CONDITION]`): Řídí dynamické zviditelnění doplňkových polí podle hodnot v jiných polích (např. zobrazení dodatku `ADD_96_80` pouze v případě, že je `Projekt.PrioritaID` v množině `0, 1, 5, 4, 7`).

---
5. Uživatelské Akce a Makrojazyk (`[ACTION_*]`)
V sekcích `[ACTION_Název]` v souboru `BASE_SET` lze definovat naprogramovatelná tlačítka. Tato tlačítka se v uživatelském rozhraní automaticky vykreslují v levém ovládacím panelu pod žlutým tlačítkem akcí.
Příklad definice akce v INI:
```ini
[ACTION_Dokončit]
GetParam=ActualSubject;Není vybrán projekt
o[projekt].Create(Typ=Project)
o[dodatky].Create(Typ=AdditionList)
o[value].Create(Typ=Value; ValueType=text)
o[podminka-text].Create(Typ=Condition; ValueType=text)

o[projekt].Load(CN=actual)
o[dodatky].Load(Source=o[projekt])

Dataset=NOTE.Edit
SetField=NOTE.ICONTYPE=53
SetField=NOTE.ENDDATE=0
Dataset=NOTE.Post

// Validace povinného vyplnění dodatku před dokončením
o[value].Value=o[dodatky].96_105   // Dodatek: Odsouhlaseno
o[podminka-text].Load(Source1=o[value].Value; Operator==; Value2=)
o[podminka-text].Validate(OnTrue=break; ErrMessage=Položka Odsouhlaseno musí být vyplněna.)

Program=GotoSubject
```
Funkce tlačítek akcí:
Změna stavu/statusu (ikony) a automatické nastavení koncových dat.
Validace datových podmínek před provedením operace.
Automatické generování záznamů v navázaných uživatelských tabulkách (Workflow) a spárování s Diářem (`NOTE.LINKEVENT`).
Změna vlastníka projektu (např. předání na modul `HOTLINE`).
---
6. Připravenost pro AI Integraci
Strukturované workflows: Díky striktně definovaným statusům (`ICONTYPE`), kategoriím (`PRIORITY`) a INI akcím může AI snadno interpretovat, v jaké fázi se daný obchodní či servisní případ nachází.
Přístup k celému grafu: Přes relace v tabulce `LINK` a navázané tabulky má AI model okamžitý přístup ke všem e-mailům, výkazům práce, dokumentům i záznamům v Diáři spojeným s daným projektem.
Analýza nestrukturovaných dat: Pole `MEMO` v tabulce `NOTE` v kombinaci s navázanými textovými dodatky představují ideální vstup pro LLM modely k automatické sumarizaci stavu projektu, generování nabídek nebo návrhu dalších kroků.
```
