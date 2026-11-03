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
