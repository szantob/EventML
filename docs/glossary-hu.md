# Magyar szójegyzék · Hungarian glossary

The EventML specification is written in English. This file maps every term to Hungarian, for the people who
actually build these systems and talk about them in Hungarian on site.

**Where the Hungarian trade uses the English word unchanged, that is recorded rather than corrected.** A
Hungarian sound engineer says *stagebox*, *snake*, *patchelés* and *Dante flow*; inventing *színpadi
csatlakozódoboz* would be a translation nobody uses and would make the glossary useless for its purpose.
The rule: give the term that is actually spoken, and add a literal or descriptive form only where it helps
someone who has not heard it before.

A specifikáció angolul íródik. Ez a fájl minden fogalmat magyarra képez le — annak, aki ezeket a
rendszereket ténylegesen építi, és a helyszínen magyarul beszél róluk. **Ahol a magyar szakma az angol szót
használja változatlanul, ott az szerepel**, nem egy senki által nem használt fordítás.

---

## 1. Metamodell · Metamodel

| English | Magyar | Megjegyzés |
|---|---|---|
| model | modell | |
| definition (`def`) | definíció, katalógustípus | A `lib/`-ben élő újrahasznosítható típus |
| usage | példány, használat | Egy projekten belüli előfordulás |
| `ItemDef` | jeltípus-definíció | Az, ami folyik: jel, tápfeszültség, adat |
| `PortDef` / `Port` | portdefiníció / port | A szakma a *port* és a *csatlakozó* szót is használja; a *port* a logikai, a *csatlakozó* a fizikai |
| `PartDef` / `Part` | eszköztípus / eszköz, elem | L2-n *funkcióblokk*, L3-on *eszköz* |
| `InterfaceDef` | kapcsolattípus | A két port közti link típusa — kábel, médium, kapacitás |
| `Connection` | kapcsolat, kábelezés | Az, hogy *létezik* a kábel |
| `Flow` | jelfolyam | Az, hogy *mi fut rajta és milyen útvonalon* |
| `RequirementDef` / `Requirement` | követelménysablon / követelmény | |
| `ConstraintDef` | megkötés, szabály | Számolható korlát: terhelés, csatornaszám, kábelhossz |
| internal traversability | belső átjárhatóság | Melyik bemenet melyik kimenetre jut el az eszközön belül |
| transform | átalakítás | Amit az eszköz a jellel csinál útközben (A/D, erősítés, keverés) |
| catalogue ID | katalógusazonosító | `<domain>.<kind>.<name>` |
| instance ID | példányazonosító | Rövid, pont nélküli, projekten belül egyedi |
| redundancy | redundancia | `primary` / `secondary` — *elsődleges* / *másodlagos* |

## 2. Rétegek · Layers

| English | Magyar | Megjegyzés |
|---|---|---|
| L0 Brief | L0 Igény, ügyféligény | Amit az ügyfél és a közönség csinál, a saját szavaival |
| L1 Requirement | L1 Követelmény | Amit a rendszernek teljesítenie kell, technológiafüggetlenül |
| L2 Logical | L2 Logikai | Funkcióblokkok és jelutak, technológiaválasztás nélkül |
| L3 Physical | L3 Fizikai | Konkrét eszközök, portok, kapcsolatok |
| brief | ügyféligény, briefing | A szakma a *brief* szót is használja változatlanul |
| audience area | közönségtér, nézőtér | |
| load-in | bepakolás, beépítés | A szakma a *load-in* szót is használja |
| soundcheck | hangpróba | *Soundcheck* ugyanolyan gyakori |
| changeover | átállás | Két programpont közti technikai átállás |

## 3. Relációk és bizonytalanság · Relations and uncertainty

| English | Magyar | Megjegyzés |
|---|---|---|
| `refine` | finomít | L0 → L1: egy igényből követelmény lesz |
| `derive` | levezet | L1 → L1: követelmény felbontása alkövetelményekre |
| `satisfy` | kielégít, teljesít | L2/L3 → L1: ez az elem teljesíti azt a követelményt |
| `allocate` | leképez, hozzárendel | L2 → L3: a logikai blokkot ez a konkrét eszköz valósítja meg |
| `trace` | visszavezet | Bármi → forrás: melyik ügyfélmondat, döntés vagy alapérték |
| `stated` | megadott | Az ügyfél mondta; van forrása |
| `derived` | levezetett | Szabályból vagy másik értékből következik |
| `assumed` | feltételezett | Mi tettük oda, hogy haladni lehessen; van indoklása |
| `unknown` | ismeretlen | Hiányzik, és tudjuk, hogy hiányzik |
| `conflicting` | ellentmondó | Két forrás mást mond |
| `ask` | megkérdezendő | Kerüljön be a kérdéslistába |
| question list | kérdéslista | A modellből **levezetett**, nem kézzel írt lista |
| `blocks` | blokkol, hány követelményt | A rangsor alapja: hány követelmény függ tőle |

## 4. Hang · Audio

| English | Magyar | Megjegyzés |
|---|---|---|
| Analogue microphone level | analóg mikrofonszint | |
| Analogue line level | analóg vonalszint | |
| Loudspeaker level | hangfalszint, teljesítményszint | |
| AES3 digital audio | AES3 digitális audio | *AES/EBU* néven is |
| MADI multichannel audio | MADI többcsatornás audio | |
| Dante audio flow | Dante flow | Változatlanul angolul; *flow* = folyam, de senki nem fordítja |
| Wireless audio RF link | vezeték nélküli RF-átvitel | A szakma szava: *wireless*, *bezsi* |
| Main PA | fő hangrendszer, fő PA | *PA* változatlanul |
| Delay line | delay vonal, késleltetett sor | *Delay* változatlanul |
| Front fill | front fill | Változatlanul; leíró magyar: *első sor kitöltés* |
| Monitor world | monitorvilág, színpadi monitorozás | |
| Microphone pool | mikrofonkészlet | |
| Playback source | lejátszóforrás, playback | |
| Mix position | keverőállás, FOH | *FOH* (front of house) változatlanul |
| Microphone | mikrofon | |
| DI box | DI doboz | Változatlanul |
| Digital stage box | digitális stagebox | Változatlanul; leíró magyar: *színpadi bekötődoboz* |
| Mixing console | keverőpult, pult | |
| DSP processor | DSP processzor, hangprocesszor | |
| Power amplifier | végfok, erősítő | *Végfok* a bevett szakszó |
| Loudspeaker | hangfal, hangsugárzó | |
| Powered loudspeaker | aktív hangfal | |
| Wireless receiver | vezeték nélküli vevő, vevőegység | |
| XLR 3-pin female | XLR 3 pólusú anya (papa/mama) | A szakma: *XLR mama*, *3 pólusú anya* |
| XLR 3-pin male | XLR 3 pólusú apa | |
| XLR 3-pin male, microphone level | XLR 3 pólusú apa, mikrofonszint | |
| 6.35 mm TRS jack | 6,35 mm-es jack, sztereó jack | |
| speakON NL4 output | speakON NL4 kimenet | *speakON* változatlanul |
| speakON NL4 input | speakON NL4 bemenet | |
| speakON NL8 | speakON NL8 | |
| BNC MADI | BNC MADI | |
| XLR 3-pin female, AES3 | XLR 3 pólusú anya, AES3 | Fizikailag azonos az analóggal, elektromosan nem |
| BNC antenna input | BNC antennabemenet | 50 ohmos, nem a 75 ohmos MADI-BNC |
| RJ45 Dante | RJ45 Dante | |
| Analogue line cable | analóg vonalkábel | |
| Microphone cable | mikrofonkábel | |
| Loudspeaker cable, NL4 | hangfalkábel, NL4 | |
| MADI coaxial link | MADI koax link | |
| Analogue run length | analóg kábelhossz-korlát | |
| Input capacity | bemeneti kapacitás, csatornaszám | |
| Speech intelligibility | beszédérthetőség | Szabványos szakszó, STI-vel mérve |
| Music reproduction | zenei reprodukció, zenei átvitel | |
| Stage monitoring | színpadi monitorozás | |
| Coverage uniformity | lefedettség egyenletessége | |
| Weather protection | időjárás elleni védelem | |

## 5. Erősáram · Power

| English | Magyar | Megjegyzés |
|---|---|---|
| AC 230 V single phase | 230 V egyfázisú váltóáram | A szakma: *egyfázis*, *fázis* |
| AC 400 V three phase | 400 V háromfázisú váltóáram | *Háromfázis*, *erőátam*, *ipari áram* |
| DC 48 V | 48 V egyenáram | |
| Power source | betáplálás, áramforrás | |
| Power distribution | áramelosztás | |
| Load | fogyasztó, terhelés | |
| Power distribution unit | elosztó, power distro | *Distro* változatlanul is |
| Cable drum | kábeldob | Feltekerve nem terhelhető a névleges áramra — ez valós szabály |
| Residual current device | Fi-relé, áram-védőkapcsoló | *Fi-relé* a bevett szó |
| Uninterruptible power supply | szünetmentes tápegység, UPS | |
| Generator | aggregátor, generátor | *Aggregátor* a bevett szó a helyszíni gépre |
| Schuko outlet (CEE 7/3) | schuko aljzat, hagyományos konnektor | |
| Schuko inlet (CEE 7/4 plug side) | schuko dugó | |
| powerCON TRUE1 inlet | powerCON TRUE1 bemenet | Változatlanul; *true1* |
| powerCON TRUE1 outlet | powerCON TRUE1 kimenet | |
| CEE 16 A five-pin outlet | 16 A-es ötpólusú CEE aljzat | A szakma: *16-os piros*, *ipari aljzat* |
| CEE 16 A five-pin inlet | 16 A-es ötpólusú CEE dugó | |
| CEE 32 A five-pin outlet | 32 A-es ötpólusú CEE aljzat | *32-es piros* |
| CEE 32 A five-pin inlet | 32 A-es ötpólusú CEE dugó | |
| Harting 6-pole multipin | Harting 6 pólusú multipin | Változatlanul |
| Single-phase power cable | egyfázisú tápkábel | |
| Three-phase power cable | háromfázisú tápkábel | |
| Harting multicore loom | Harting multikábel, loom | |
| Voltage drop | feszültségesés | |
| Phase balance | fázisszimmetria, fáziskiegyenlítés | |
| Supply headroom | tartalék a betáplálásban | |
| Supply capacity | betáplálási kapacitás | |
| Residual current protection | Fi-védelem, érintésvédelem | |
| Silent supply | csendes betáplálás | Az a követelmény, ami eldönti, hogy aggregátor egyáltalán szóba jöhet-e |

## 6. Hálózat · Network

| English | Magyar | Megjegyzés |
|---|---|---|
| Gigabit Ethernet | gigabites Ethernet | |
| 10 Gigabit Ethernet | 10 gigabites Ethernet | |
| OM3 multimode fibre | OM3 multimódusú optika | *Optika*, *üveg* |
| Dante transport | Dante transzport | Változatlanul |
| PTP clock | PTP óra, órajel | *Clock*, *órajel* |
| sACN | sACN | Változatlanul |
| Art-Net | Art-Net | Változatlanul |
| NMOS IS-04 registration | NMOS IS-04 regisztráció | |
| Network backbone | hálózati gerinc | |
| Network edge | hálózati végpont, edge | |
| Clock master | órajelmester, clock master | Változatlanul is |
| Managed switch | menedzselt switch | *Switch* változatlanul; *kapcsoló* nem használatos |
| Unmanaged switch | nem menedzselt switch | |
| Media converter | médiakonverter | |
| Wireless access point | hozzáférési pont, AP | *AP* változatlanul |
| RJ45 | RJ45 | |
| etherCON, primary | etherCON, elsődleges | *etherCON* változatlanul |
| etherCON, secondary | etherCON, másodlagos | |
| SFP cage | SFP foglalat | |
| LC duplex fibre | LC duplex optika | |
| RJ45 with PoE | RJ45 PoE-val | *PoE* változatlanul |
| Cat6a copper link | Cat6a rézkábeles link | |
| Cat6a copper link, 10G | Cat6a rézkábeles link, 10G | Ugyanaz a kábel, rövidebb hatótáv |
| OM3 fibre link | OM3 optikai link | |
| Maximum copper run | maximális rézkábel-hossz | |
| Bandwidth headroom | sávszélesség-tartalék | |
| Single clock master | egyetlen órajelmester | |
| Clock stability | órajel-stabilitás | |
| Network redundancy | hálózati redundancia | |
| Traffic segregation | forgalom szétválasztása | VLAN-nal vagy fizikailag |

## 7. Kép · Video

| English | Magyar | Megjegyzés |
|---|---|---|
| 3G-SDI | 3G-SDI | Változatlanul |
| 12G-SDI | 12G-SDI | |
| HDMI 2.0 | HDMI 2.0 | |
| DisplayPort 1.4 | DisplayPort 1.4 | |
| NDI stream | NDI stream | Változatlanul |
| ST 2110-20 video essence | ST 2110-20 videó esszencia | |
| Image source | képforrás | |
| Image processing | képfeldolgozás | |
| Image display | képmegjelenítés | |
| Camera | kamera | |
| Media server | médiaszerver | |
| Video switcher | képkeverő, switcher | *Képkeverő* a bevett szó |
| Scaler | scaler, képméretező | Változatlanul is |
| LED processor | LED processzor | |
| LED panel | LED panel, LED csempe | |
| Projector | projektor | |
| Display screen | kijelző, monitor, vászon | *Vászon* csak projektorhoz |
| BNC SDI input | BNC SDI bemenet | |
| BNC SDI output | BNC SDI kimenet | |
| BNC 12G-SDI output | BNC 12G-SDI kimenet | |
| HDMI Type A input | HDMI A típusú bemenet | |
| HDMI Type A output | HDMI A típusú kimenet | |
| DisplayPort | DisplayPort | |
| RJ45 NDI | RJ45 NDI | |
| LC duplex fibre, video | LC duplex optika, videó | |
| SDI coaxial link | SDI koax link | |
| HDMI cable | HDMI kábel | Passzívan 10 m felett nem megbízható |
| NDI over Cat6a | NDI Cat6a-n | |
| SDI run length | SDI kábelhossz-korlát | |
| HDMI run length | HDMI kábelhossz-korlát | |
| Display legibility | olvashatóság | Képmagasság és nézőtávolság aránya dönti el |
| Image legibility | képolvashatóság | |
| Source switching | forrásváltás | |
| Screen coverage | rálátás a kijelzőre | |
| Remote participation | távoli részvétel, hibrid részvétel | |

## 8. Fény · Lighting

| English | Magyar | Megjegyzés |
|---|---|---|
| DMX512-A | DMX512-A | Változatlanul; *DMX* |
| RDM | RDM | Változatlanul |
| DMX over sACN | DMX sACN-en | |
| DMX over Art-Net | DMX Art-Neten | |
| Light output | fénykibocsátás, megvilágítás | Luxban mérve a tárgynál |
| Key light | kulcsfény, key light | |
| Wash light | wash, általános fény | *Wash* változatlanul |
| Effect light | effektfény | |
| Control surface | vezérlőfelület, pult | |
| DMX distribution | DMX elosztás | |
| Moving head fixture | mozgófejes lámpa, movinghead | *Movinghead* változatlanul |
| PAR fixture | PAR lámpa | |
| Profile fixture | profillámpa | |
| Lighting console | fénypult | |
| DMX node | DMX node | Változatlanul |
| Dimmer | dimmer, fényerőszabályzó | *Dimmer* változatlanul |
| XLR 5-pin female, DMX in | XLR 5 pólusú anya, DMX be | |
| XLR 5-pin male, DMX out | XLR 5 pólusú apa, DMX ki | |
| XLR 3-pin female, DMX in | XLR 3 pólusú anya, DMX be | Szabványtalan, de mindenhol előfordul |
| XLR 3-pin male, DMX out | XLR 3 pólusú apa, DMX ki | |
| RJ45 sACN | RJ45 sACN | |
| DMX cable | DMX kábel | 110 ohmos, nem mikrofonkábel |
| sACN over Cat6a | sACN Cat6a-n | |
| DMX run limit | DMX lánchossz-korlát | 32 eszköz, 300 m, lezárással |
| Universe capacity | univerzum-kapacitás | 512 csatorna univerzumonként |
| Face lighting | arcmegvilágítás | Az a követelmény, amit senki nem kér, és ami nélkül a felvétel használhatatlan |
| Stage wash | színpadi általános fény | |
| Control universes | vezérelt univerzumok | |
| House light control | nézőtéri fény vezérlése | Általában helyszíni egyeztetés, nem eszközkérdés |

---

## A fordítás elve

Három szabály, amit ez a szójegyzék követ:

1. **Ha a szakma az angol szót használja, az szerepel.** *Stagebox*, *movinghead*, *distro*, *front fill*,
   *FOH* — ezek magyar szavak abban az értelemben, hogy magyar mondatokban hangzanak el.
2. **Ahol van bevett magyar szakszó, az az elsődleges.** *Végfok*, nem *teljesítményerősítő*. *Fi-relé*,
   nem *maradékáram-működtetésű megszakító*. *Aggregátor*, nem *áramfejlesztő generátor*.
3. **Az ügyfélnek szóló kérdések nyelve nem ez.** A `lib/` `asks` mezőiben álló kérdések angolul íródnak és
   a kliens nyelvére lokalizálódnak — de nem ezekkel a szakszavakkal. Egy ügyfél nem tudja, mi a *front
   fill*; azt tudja, hogy állnak-e emberek közvetlenül a színpad előtt. A lokalizálás a tooling dolga, nem
   a nyelvé — lásd `spec/04-uncertainty.md` §5.
