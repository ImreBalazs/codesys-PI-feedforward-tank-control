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

* **Fejlett szabályozási architektúra (PI + Feedforward):** 
  * A visszacsatolt PI kör ($K_p = 0.7, T_i = 15\text{ s}$) a pontos alapjel-követést biztosítja.
  * A kiömlőoldali átfolyásmérőre alapozott Előrecsatolás ($K_{ff} = 1.2$) holtidő nélkül rányit a töltőszelepre az ürítőszelep elmozdulásakor, így a zavarást még a vízszint beszakadása előtt elhárítja.

* **Negatív PID tartomány ($Y_{MIN} = -10.0$) és Anti-Windup:** 
  * A PID kimeneti alsó korlátjának $-10.0\text{ V}$-ra állításával a szabályozó képes kioltani az előrecsatolási offsetet túllépés esetén, garantálva a töltőszelep teljes lezárását ($0\%$) és a gyors ürülést/hűtést.

* **Jelillesztés és skálázás (`FC_Scale`):** 
  * A nyers $0\text{--}10\text{ V}$-os analóg jelek (`rLevel_Raw`, `rPot_Raw`, `rFlowMeterRaw`) szabványos fizikai mértékegységekké ($0\text{--}300\text{ cm}$, átfolyás) alakítása saját skáláz
