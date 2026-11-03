## 1. Projekt áttekintése

A projektben egy aszimmetrikus átfolyású ipari tartály vízszintszabályozásának teljes körű automatizálását és optimális szabályozási architektúráját valósítottam meg **CODESYS V3.5** környezetben. A tartály feltöltését egy töltőszelep, ürítését pedig egy gravitációs lefolyású szelep végzi. A kiömlő oldal nagyobb kapacitású a töltőoldalnál, a lefolyás sebessége pedig a tartályban lévő vízoszlop magasságával (a víznyomással) arányosan változik.

A berendezés célja a megadott vízszint-alapjel pontos lekövetése a tényleges tartályszinttel, valamint a lefolyási oldalon fellépő hirtelen terhelésváltozások (zavarások) azonnali kompenzálása.

---

### Fő funkciók és tulajdonságok

* **Négyfajta üzemmód (Kézi és Automata vezérlés):** A rendszer rugalmasan kapcsolható a HMI felületről és a fizikai bemenetekről:
  * **Hand PID (`iControlMode = 0`):** A vízszint alapjelét egy fizikai potencióméter ($0\text{--}10\text{ V} \rightarrow 0\text{--}300\text{ cm}$) adja meg, a szabályozást a PID kör végzi.
  * **HMI Manual (`iControlMode = 1`):** A töltő- és ürítőszelepek közvetlen kézi vezérlése a HMI-ről ($0\text{ V}$ / $10\text{ V}$ állapotok között).
  * **HMI PID (`iControlMode = 2`):** Automatikus szintszabályozás a HMI-n megadott digitális alapjel alapján.
  * **HMI FB+FF (`iControlMode = 3`):** Fejlett Visszacsatolt + Előrecsatolt (PI + Feedforward) szabályozás, ahol az ürítőszelep mint bemeneti zavarás szerepel, a töltőszelep pedig automatikusan kompenzál.

### Megvalósított rendszerösszetevők

- **CODESYS V3.5 PLC Vezérlőprogram:** A teljes szabályozási architektúra, jelskálázások, illesztések és az üzemmód-választó logika lefejlesztése.
- **Ignition HMI (Mobil / Android kompatibilis):** Modern, reszponzív ipari kezelőfelület, amellyel a rendszer távolról, akár okostelefonról vagy táblagépről is teljes körűen felügyelhető és irányítható.
- **Factory I/O Digitális Iker (Digital Twin):** A tartályrendszer valós idejű 3D-s szimulációja a fizikai folyamatok modellezésére és a szabályozási algoritmusok éles tesztelésére.

## 2. HMI Kezelőfelület (Ignition HMI)

A rendszer felügyeletét és irányítását egy Ignition alapon megvalósított, mobil- és táblagép-kompatibilis reszponzív HMI felület biztosítja. A kezelőfelület két fő nézetre oszlik: **MainView** (Főnézet) és **TrendView** (Trendgrafikonos nézet).

---

### MainView (Főnézet)
<img width="787" height="773" alt="{9FB7FA17-B848-4164-A4E1-689B645A708F}" src="https://github.com/user-attachments/assets/b3deec7b-cdbd-48d0-9835-23ce842dab4a" />

A MainView a folyamat valós idejű megjelenítésére, az üzemmódok közötti váltásra, az alapjelek megadására és a kézi beavatkozásra szolgál.

#### Navigáció
* **Chart gomb (bal fent):** Átnavigál a TrendView (grafikonos) felületre.

#### Folyamatjelzők és vizuális visszacsatolás
* **Animált tartálygrafika:** Százalékos formában ($0\text{--}100\%$) vizuálisan jeleníti meg a tartály feltöltöttségi szintjét.
* **Fill Valve indicator:** A töltőszelep pillanatnyi nyitottsági állapotát mutatja százalékban ($0\text{--}100\%$).
* **Discharge Valve indicator:** Az ürítőszelep pillanatnyi nyitottsági állapotát mutatja százalékban ($0\text{--}100\%$).
* **Discharge Flow Rate indicator:** A kiömlő oldali átfolyásmérő által mért pilanatnyi tömegáramot jelzi ki ($m^3/s$).
* **Set Point display:** Az aktuálisan érvényben lévő vízszint-alapjel kijelzése fizikai mértékegységben ($cm$).
* **Actual Level display:** A tartályban lévő mért vízszint ($PV$) kijelzése fizikai mértékegységben ($cm$).

#### Üzemmód-választó gombok
Az éppen aktív üzemmódot a HMI zöld kiemeléssel jelzi:
* **Hand PID:** Kiválasztja a potencióméteres kézi alapjel-beállítású automatikus PID szabályozást (`iControlMode = 0`).
* **HMI Manual:** Átkapcsol a közvetlen szelepműködtetésű kézi üzemmódra (`iControlMode = 1`).
* **HMI PID:** Aktiválja a HMI alapjellel működő automatikus PID szabályozást (`iControlMode = 2`).
* **HMI FB+FF:** Aktiválja a fejlett Visszacsatolt + Előrecsatolt (PI + Feedforward) zavarkompenzáló szabályozást ('iControlMode = 3').

#### Beavatkozó elemek és beviteli mezők
* **Fill gomb (balra):** HMI Manual üzemmódban a töltőszelep nyitása és zárása ($0\text{ V} / 10\text{ V}$).
* **Discharge gomb (balra):** HMI Manual üzemmódban az ürítőszelep nyitása és zárása ($0\text{ V} / 10\text{ V}$).
* **Set Point Input (jobbra lent):** Numerikus beviteli mező a kívánt vízszint-alapjel ($0\text{--}300\text{ cm}$) megadására HMI PID és HMI FB+FF üzemmódokban.

---

### TrendView (Grafikonos nézet)
<img width="783" height="704" alt="{D4CED08C-BE83-43E8-88B4-1FE098CE39F5}" src="https://github.com/user-attachments/assets/866a7348-85f8-44f7-8614-0d40237f0050" />

A TrendView a folyamatdinamika valós idejű nyomon követésére és a szabályozási kör beállási tulajdonságainak elemzésére szolgál.

#### Navigáció
* **Back gomb (fent középen):** Visszanavigál a MainView főnézetre.

#### Grafikonos megjelenítő és adatelemzés
* **Időtengely-beállítás:** A grafikon feletti időablak-választóval (pl. `Last 1 minutes`) szabályozható a megjelenített időtartomány.
* **Zöld görbe (`rlevel_pv`):** A tartályban mért tényleges vízszint alakulása az idő függvényében.
* **Narancssárga görbe (`rsetpoint_sp`):** A beállított vízszint-alapjel alakulása.
* **Pen Table (Alsó adattáblázat):** A regisztrált jelek részletes statisztikai adatit jeleníti meg:
  * **Current Value:** Pillanatnyi érték.
  * **Minimum:** A vizsgált időablakban mért legkisebb érték.
  * **Maximum:** A vizsgált időablakban mért legnagyobb érték.
  * **Average:** Átlagérték az adott időintervallumban.

## 3. PLC Kód és Vezérlési Logika

A PLC program CODESYS V3.5 környezetben készült. A szabályozási és átkapcsolási feladatokat egy fő programblokk (`PLC_PRG`), egy saját skálázó funkcióblokk (`FC_Scale`), egy egyedileg megírt PID szabályozó blokk (`FB_PID_MAN`), valamint a CODESYS beépített `PID` könyvtári funkcióblokkja valósítják meg.

---

### Változódeklarációk (`VAR`)

A programban használt be- és kimeneti, belső, valamint paraméterező változók struktúrája a kód alapján:

```pascal
PROGRAM PLC_PRG
VAR
    // Fizikai kimenetek (Szelep pozícióparancsok 0-10V)
    rFillValveCmd      : REAL;
    rDischargeValveCmd : REAL;
    
    // Üzemmódválasztó regiszter
    iControlMode       : INT;
    
    // Nyers és skálázott analóg jelek
    rLevel_Raw         : REAL;
    rLevel_PV          : REAL;
    rPot_Raw           : REAL;
    rSetpoint_SP       : REAL;
    rScaledPot         : REAL;
    rFlowMeterRaw      : REAL;
    rFlowMeterScaled   : REAL;
    
    // Vezérlő és engedélyező bit-jelek
    bAutoMode          : BOOL := TRUE;
    bEnableFFFB        : BOOL;
    bHMIControl        : BOOL := FALSE;
    rHMISetPOint       : REAL;
    bHMI_FillBtn       : BOOL;
    bHMI_DischargeBtn  : BOOL;
    
    // PID_1 paraméterek (Hand PID / HMI PID)
    rKp                : REAL := 1.0;
    rTi                : REAL := 5.0;
    rTd                : REAL;
    rTs                : REAL := 0.02;
    
    // PID_2 és Feedforward paraméterek (FB+FF)
    rFKp               : REAL := 0.7;
    rFTi               : REAL := 15.0;
    rFTd               : REAL := 0;
    rFKff              : REAL := 0;
    rKDisc             : REAL := 2;
    
    // Hibaértékek és holtsáv
    rError             : REAL;
    rDeadband          : REAL := 1.5;
    
    // Szabályozási kimenetek és köztes számítási változók
    rPID_Out           : REAL;
    rFBPID_Out         : REAL;
    rFFProduct         : REAL;
    rNoLimitOut        : REAL;
    rLimitedOut        : REAL;
    
    // Funkcióblokk példányosítások
    FB_PID_MAN_1       : FB_PID_MAN;
    FB_PID_MAN_2       : PID;
END_VAR
```

---

### Üzemmód-választási logika (Networks 1–4)

Az üzemmódok közötti váltást az `iControlMode` egész változó értéke határozza meg, amelynek alapján a program automatikusan kiválasztja a megfelelő alapjelet és szabályozási ágat:

* **Network 1 (`iControlMode = 0`):** Az `EQ` blokk összehasonlítja az `iControlMode` értékét 0-val. Egyezés esetén a `MOVE` blokk átmásolja a potencióméter skálázott értékét (`rScaledPot`) az aktív alapjelre (`rSetpoint_SP`).
* **Network 2 (`iControlMode = 1`):** Amikor az `iControlMode` értéke 1 (HMI Manual), a bontó (N.C.) logika lekapcsolja a `bAutoMode` engedélyező bitet (`FALSE`), ezzel tiltva az automatikus PID szabályozók működését.
* **Network 3 (`iControlMode = 2`):** Az `iControlMode = 2` feltétel teljesülésekor a `MOVE` blokk a HMI-n megadott digitális értéket (`rHMISetPOint`) írja be az aktív alapjelre (`rSetpoint_SP`).
* **Network 4 (`iControlMode = 3`):** Az `iControlMode = 3` beállítás esetén a kimeneti tekercs aktiválja a `bEnableFFFB` bitet (`TRUE`), amely engedélyezi a kombinált Feedback + Feedforward szabályozási kört.

### Egyedi PID Funkcióblokk (`FB_PID_MAN`)

Az 1-es (Hand PID) és 3-as (HMI PID) üzemmódokban nem könyvtári elemet, hanem saját fejlesztésű, diszkrét idejű PID funkcióblokkot (`FB_PID_MAN`) használtam Structured Text (ST) nyelven. A blokk tartalmazza a kimeneti határolást és az integrál-túltelítődés elleni védelmet (Clamping Anti-Windup).

#### A funkcióblokk ST forráskódja

```pascal
FUNCTION_BLOCK FB_PID_MAN
VAR_INPUT
    PV_k   : REAL; // Mért process érték (Level PV)
    SP_k   : REAL; // Alapjel (Setpoint)
    Kp     : REAL; // Arányos erősítés
    Ti     : REAL; // Integrálási idő [s]
    Td     : REAL; // Deriválási idő [s]
    Ts     : REAL; // Mintavételezési idő [s]
    u_min  : REAL; // Kimeneti alsó korlát (-10.0V / 0.0V)
    u_max  : REAL; // Kimeneti felső korlát (10.0V)
END_VAR
VAR
    bFirstScan : BOOL := TRUE;
    e_k        : REAL;
    P_k        : REAL;
    I_kTemp    : REAL;
    I_k_1      : REAL;
    Dk         : REAL;
    PV_k_1     : REAL;
    u_raw      : REAL;
END_VAR
VAR_OUTPUT
    u_k        : REAL; // Korlátozott beavatkozó jel
END_VAR

// 1. Hibaképzés és Arányos (P) tag
e_k := SP_k - PV_k;
P_k := Kp * e_k;

// Első futás inicializálása (deriváló tüske elkerülése indításkor)
IF bFirstScan THEN
    PV_k_1 := PV_k;
    bFirstScan := FALSE;
END_IF;

// 2. Integrál (I) tag számítása (Diszkrét Euler-módszer)
IF Ti > 0.0 THEN
    I_kTemp := I_k_1 + ((Kp / Ti) * e_k * Ts);
ELSE
    I_kTemp := 0.0;
END_IF;

// 3. Derivált (D) tag számítása a mért érték (PV) alapján
IF Td > 0.0 AND Ts > 0.0 THEN
    Dk := -Kp * Td * (PV_k - PV_k_1) / Ts;
ELSE
    Dk := 0.0;
END_IF;

// 4. Összegzés és kimeneti telítés/Anti-Windup kezelése
u_raw := P_k + I_kTemp + Dk;

IF u_raw >= u_max THEN
    u_k := u_max;
ELSIF u_raw <= u_min THEN
    u_k := u_min;
ELSE
    u_k := u_raw;
    I_k_1 := I_kTemp; // Integrál frissítése csak akkor, ha a kimenet nincs telítésben
END_IF;

PV_k_1 := PV_k;
```

#### Algoritmus főbb tulajdonságai

* **Deriválás a PV alapján (Derivative on PV):** A deriváló tag nem a hibajelből ($e_k$), hanem a mért érték változásából ($(PV_k - PV_{k-1})$) számol. Ez megakadályozza az alapjel hirtelen módosításakor fellépő úgynevezett deriválási tüskét (Derivative Kick).
* **Feltételes integrálás (Clamping Anti-Windup):** Ha a nyers kimenet (`u_raw`) átlépi a beállított $[u_{min}, u_{max}]$ határokat, az integrál belső állapota (`I_k_1`) nem frissül. Ez megelőzi a beavatkozó szerv telítődésekor fellépő integrál-túltöltődést és az abból adódó jelentős túllövést.

---

### Üzemmódok részletes megvalósítása

#### 1. Hand PID üzemmód (`iControlMode = 0`)
<img width="904" height="747" alt="{703A11B6-3EC2-4F42-AA3D-C69E5754C7AC}" src="https://github.com/user-attachments/assets/d735c619-fc21-49e1-98ca-dcf60d6cf137" />
<img width="1183" height="801" alt="{5BF4796C-DA99-4C39-A239-573CF8D13D15}" src="https://github.com/user-attachments/assets/9990edd0-e625-4045-9809-e8ab561a36c6" />
<img width="481" height="373" alt="{82ABEC6C-FF17-41FE-9271-5E0D4FF72B8F}" src="https://github.com/user-attachments/assets/d0113354-bdac-4363-a1c4-b30bbc2f3117" />


* **Működési elv:** A vízszint alapjelét a fizikai potencióméter adja meg. A nyers analóg bemenetet (`rPot_Raw`) az `FC_Scale` blokk konvertálja $0\text{--}300\text{ cm}$ fizikai tartományra (`rScaledPot`), ami az 1-es hálózatban átmásolódik az `rSetpoint_SP` változóba.
* **Szabályozási logika (Networks 5, 6, 7):**
  * A `bAutoMode = TRUE` jel engedélyezi a saját `FB_PID_MAN_1` szabályozóblokk működését.
  * A szabályozó $u_k$ kimenete (`rPID_Out`) alapján a program eldönti a szelepek mozgatását:
    * **Ha `rPID_Out > 0`:** A töltőszelep kap nyitóparancsot (`rFillValveCmd = rPID_Out`), az ürítőszelep zárva marad (`rDischargeValveCmd = 0`).
    * **Ha `rPID_Out < 0`:** Az ürítőszelep nyit az előjelváltás után (`rDischargeValveCmd = rPID_Out * -1`), a töltőszelep pedig lezár (`rFillValveCmd = 0`).
    * **Ha a hibajel a holtsávon belül van ($|rError| < rDeadband$):** Mindkét szelep parancsa 0 értéket vesz fel a felesleges szelepműködtetés elkerülése érdekében.

#### Analóg jelillesztés és skálázás (`FC_Scale`)

A nyers $0\text{--}10\text{ V}$-os analóg bemeneti jelek — mint a tartály vízszintje (`rLevel_Raw`), a fizikai potencióméter (`rPot_Raw`), valamint a kiömlőoldali átfolyásmérő (`rFlowMeterRaw`) — szabványos fizikai mértékegységekbe ($0\text{--}300\text{ cm}$, átfolyás) történő átszámítását a saját megírású `FC_Scale` funkcióblokk végzi lineáris interpolációval:

$$\text{rScaledOut} = \frac{\text{rIn} - \text{rInMin}}{\text{rInMax} - \text{rInMin}} \cdot (\text{rOutMax} - \text{rOutMin}) + \text{rOutMin}$$

##### Blokk interface deklarációja

```pascal
FUNCTION FC_Scale : REAL
VAR_INPUT
    rIn      : REAL; // Nyers analóg bemenet (pl. rPot_Raw: 0.0 - 10.0 V)
    rInMin   : REAL; // Bemeneti skála minimuma (0.0 V)
    rInMax   : REAL; // Bemeneti skála maximuma (10.0 V)
    rOutMin  : REAL; // Fizikai tartomány minimuma (0.0 cm)
    rOutMax  : REAL; // Fizikai tartomány maximuma (300.0 cm)
END_VAR
VAR
    rInDiff       : REAL;
    rInSpan       : REAL;
    rOutSpan      : REAL;
    rRatio        : REAL;
    rScaledOffset : REAL;
END_VAR
VAR_OUTPUT
    rScaledOut    : REAL; // Skálázott fizikai érték (rScaledPot)
END_VAR
```
<img width="707" height="613" alt="{5D6B2EB1-0D85-46C6-8CB6-936CC67DFDBD}" src="https://github.com/user-attachments/assets/96414f7e-cbfc-4332-ab0a-0a2a293e104e" />

#### PID Szabályozó Hangolási Folyamata (Tuning)

A saját `FB_PID_MAN` szabályozóblokk paramétereinek ($K_p$, $T_i$, $T_d$) meghatározása manuális, kísérleti úton történt a szimulációs környezetben. A hangolás során lépésről lépésre vizsgáltuk az egyes tagok hatását a beállási folyamatra:

##### 1. Arányos tag ($K_p$) beállítása
* Első lépésként az integráló és deriváló tagokat kikapcsoltuk , és kizárólag a $K_p$ arányos erősítést növeltük.
* **Tapasztalat:** A $K_p$ növelésével a rendszer válaszideje gyorsult, azonban tisztán arányos szabályozással maradó szabályozási eltérés (offset) keletkezett. A tartály gravitációs lefolyási sajátossága miatt a kimenet nem tudta teljesen elérni a kívánt alapjelet ($SP$).

##### 2. Integráló tag ($T_i$) bevezetése
* A maradó eltérés megszüntetésére bekapcsoltuk a $T_i$ integrálási időt.
* **Tapasztalat:** A $T_i$ fokozatos beállításával az integráló tag megszüntette az állandósult hibát, és a tartályszint pontosan beállt az alapjelre. A hangolás során olyan egyensúlyt kerestünk, ahol a beállási idő optimális, de a túllövés ($overshoot$) még nem okoz instabilitást.

##### 3. Deriváló tag ($T_d$) vizsgálata
* Utolsó lépésként kísérleteztem a $T_d$ deriválási idő beállításával a beállási folyamat csillapításának érdekében.
* **Tapasztalat:** A $T_d$ bevezetése során a fő problémát a töltőszelep felőli beömlés okozta. A tartályba érkező víz a felszínen locsogást és hullámzást váltott ki, ami átmeneti zajt és ingadozást eredményezett a szintmérő mérésében. A deriváló tag ezt a gyors mérésbeli tüskét drasztikusan felerősítette, ami a szelepek felesleges, rángatózó mozgásához (a beavatkozó szervek túlterheléséhez) vezetett, anélkül, hogy érdemben javította volna a beállási dinamikát.

##### Végeredmény
A fizikai mérészaj és a felszíni hullámzás felerősödése miatt a deriváló tag elhagyásra került ($T_d = 0\text{ s}$), így a végleges, stabil alapjel-követést biztosító szabályozó egy tisztán PI struktúra lett az alábbi paraméterekkel:
* $K_p = 1.0$
* $T_i = 5.0\text{ s}$
* $T_d = 0.0\text{ s}$

##### Működési teszt és beállási dinamika ($SP = 142\text{ cm}$)

A Hand PID üzemmód működésének és a felhangolt PI szabályozó hatékonyságának szemléltetésére a fizikai potencióméterrel egy $142\text{ cm}$-es alapjel-ugrást hoztunk létre.

###### Demonstrációs videó
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/3e12dfdd-6587-46df-bae8-4fd2046a9c1a" />

Az alábbi felvételen látható a potencióméter beállítása és a tartály szintjének automatikus reagálása a 3D-s szimulációs környezetben:
https://www.youtube.com/watch?v=E4ijaTnyNM0

#### 2. HMI Manual üzemmód (`iControlMode = 1`)

* **Működési elv:** Ez a nyílt hurkú (Open-loop) kézi beavatkozási üzemmód, ahol az automatikus zárt hurkú PID szabályozás teljesen ki van kapcsolva. Az operátor az Ignition HMI felületen elhelyezett nyomógombok segítségével közvetlenül, diskrét módon (0 V vagy 10 V, azaz teljesen zárva vagy 100%-ig nyitva) tudja működtetni a töltő- és ürítőszelepeket.
  
<img width="533" height="552" alt="image" src="https://github.com/user-attachments/assets/20056c58-7a69-4db4-ae98-7217e727f85d" />
 
* **Kézivezérlési logika (Networks 11–12):**
  * Mindkét hálózat elején egy bontó (N.C.) kontaktus található (`bAutoMode`), amely szavatolja, hogy a manuális szelepparancsok kizárólag kikapcsolt automatika (`bAutoMode = FALSE`) mellett léphessenek érvénybe.
  * **Network 11 (Ürítőszelep vezérlése):** A `SEL` (Select) blokk a `bHMI_DischargeBtn` HMI nyomógomb állapota alapján választ:
    * Ha a gomb aktív (`G = TRUE`), a `SEL` blokk az `IN1` bemenet értékét ($10.0\text{ V}$) írja ki a `rDischargeValveCmd` kimenetre (szelep fully open).
    * Ha a gomb inaktív (`G = FALSE`), az `IN0` bemenet értéke ($0.0\text{ V}$) kerül a kimenetre (szelep fully closed).
  * **Network 12 (Töltőszelep vezérlése):** A `SEL` blokk a `bHMI_FillBtn` HMI nyomógomb jele alapján hasonlóan működik:
    * Gombnyomásra (`G = TRUE`) $10.0\text{ V}$-ot ad a `rFillValveCmd` kimenetre.
    * Elengedett gomb esetén (`G = FALSE`) $0.0\text{ V}$ parancsot továbbít a töltőszelepnek.
* **Alkalmazási terület:** Karbantartás, beüzemelési tesztek, valamint a tartály kézi előtöltése és leürítése a szabályozási körök indítása előtt.

##### Működési demonstráció
<img width="1919" height="1079" alt="image" src="https://github.com/user-attachments/assets/c91ade07-ead8-47ec-8cbc-65d4bb4532ad" />
Az üzemmód gyakorlati működését, a szelepek HMI nyomógombokkal történő közvetlen nyitását és zárását az alábbi felvétel szemlélteti:
https://youtu.be/ntr-lD2tA1s
