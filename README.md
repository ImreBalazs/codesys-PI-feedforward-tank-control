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

### TrendView (Trendgrafikonos nézet)
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
