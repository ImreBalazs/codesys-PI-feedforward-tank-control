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

<img width="707" height="613" alt="{5D6B2EB1-0D85-46C6-8CB6-936CC67DFDBD}" src="https://github.com/user-attachments/assets/96414f7e-cbfc-4332-ab0a-0a2a293e104e" />
