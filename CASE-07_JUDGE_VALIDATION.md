# CASE-07: Kvaliteedivärava valideerimine

**Testitav süsteem:** `turunduskirjutaja` skill v3, sisemine automaatne ostjakriitik
**Periood:** 22.08.2026 kuni 04.09.2026
**Staatus:** avatud defekt. Kandidaatparandus on mõõdetud, paigaldamata.
**Testija:** Allan Noor

| Jooks | Roll | Terviklus |
|---|---|---|
| `TURUNDUSKIRJUTAJA-QA-20260822-008` | sertifitseerimisjooks, 6 juhtumit, 13 assertion'it, 20 tõendit | `VALID` |
| `ICP-PROXY-QA-20260904-001` | kandidaatparanduse mõõtmine samal valimil | `SELF_REPORTED_NOT_INDEPENDENTLY_VALIDATED` |

---

## Kokkuvõte

Skill kirjutab eestikeelseid müügipostitusi. Teksti kvaliteedi otsustab skilli sees eraldi komponent: pime automaatne ostjakriitik, kes vastab viiele küsimusele `JAH` või `EI`.

Sertifitseerimisel mõõtsin kirjutajat ja hindajat eraldi. Kirjutaja läbis. Hindaja kukkus läbi.

Automaatne kriitik langes inimhindajaga kokku 7 korral 20-st. Ta lükkas tagasi 11 teksti, mille inimene kiitis heaks, ja kiitis heaks ühe teksti, mille inimene ainsana tagasi lükkas.

Kandidaatparandus, mis vahetab viie küsimuse kontroll-lehe välja esimeses isikus lugejareaktsiooni vastu, andis samal valimil 18 kokkulangevust 20-st ja tabas ka selle ühe tagasilükatud teksti. Parandus on skilli sisse viimata, sest aus kordus vajab uut inimese pimetesti.

---

## 1. Miks hindajat üldse eraldi testiti

Skilli leping eraldab kaks küsimust:

- **allikakontroll** küsib, kas väide on tõendatud;
- **ostjakontroll** küsib, kas lugeja peatub, saab aru ja usub.

Allikaga kooskõla tõendab väite õigsust. Lugeja käitumise kohta ta midagi ei ütle. Nendele küsimustele vastavad eri hindajad ja neid ei liideta üheks skooriks.

Sellest järeldub, et automaatne ostjahindaja on iseseisev komponent. Iseseisev komponent vajab oma vastuvõtukriteeriumi. Selle kriteerium sai olla ainult üks: kui hästi ta ennustab päris inimese otsust.

---

## 2. Seadistus

Viis sünteetilist brändi, 20 kandidaatteksti. Kaks hindajat said sama valimi teineteisest sõltumatult.

**Inimhindaja (Allan).** Pimetest. Iga teksti kohta viis küsimust, tekst saab `PASS` ainult siis, kui kõik viis vastust on `JAH`. Värav oli 16/20 ehk 80%.

**Automaatne kriitik.** Sai ainult lugeja olukorra, pakkumise nime, kanali ja ühe teksti. Talle jäid andmata allikaledger, kirjutaja põhjendused, oodatud vastus, kvaliteedireeglite loend ja eelmise kontrolli tulemus. Viis küsimust: `STOP`, `OUTCOME`, `WHY_THIS_PROVIDER`, `NO_REPETITION`, `HUMAN_VOICE`. Värav oli kokkulangevus inimesega 18/20 ehk 90%.

Allani `INCOMPLETE` loeti kokkulangevuse arvutuses konservatiivselt mittekokkulangevuseks.

---

## 3. Tulemus: kirjutaja läbis

| Näitaja | Väärtus |
|---|---|
| `PASS` | 18 |
| Selge `FAIL` | 1 (C02, põhjus: `STOP` oli `EI`) |
| `INCOMPLETE` | 1 (C15, `STOP` vastus jäi andmata) |
| Värav | 16/20 |
| Tulemus | läbitud, staatus `HUMAN_VALIDATED` |

Allani sõnastus 24.08.2026: "Minu hinnangul täitsa arusaadavad tekstid, konkreetsed ja ilma mullita. Ma arvan, et skilli saab kasutada küll."

---

## 4. Tulemus: hindaja kukkus läbi

TC-05 `FAIL`.

| Näitaja | Väärtus |
|---|---|
| Kokkulangevus inimesega | 7/20 (35%) |
| Nõutud | 18/20 (90%) |
| Vale-negatiivid (kriitik `FAIL`, inimene `PASS`) | 11 |
| Vale-positiivid (kriitik `PASS`, inimene `FAIL`) | 1 |

Kaks viga maksavad erinevalt.

**11 vale-negatiivi** tähendab hõõrdumist. Päris kasutuses jääks hea tekst avaldamata ja kirjutaja saadetaks parandusringi ilma põhjuseta. Kaks parandusringi on skilli lepingus maksimum, seega osa tekste jääks igavesse loopi.

**1 vale-positiiv on kallim.** C02 oli ainus tekst, mille Allan päriselt tagasi lükkas. Automaatne kriitik andis talle `PASS`. Just see tekst oleks kvaliteedivärava kaudu avaldamisse läinud.

---

## 5. Diagnoos

Kriitik vastas kontroll-lehele. Ta kontrollis, kas tekstis on olemas tunnused, mille kohta küsimused käivad.

Vale-negatiivide põhjused koondusid kolmele küsimusele: `HUMAN_VOICE`, `WHY_THIS_PROVIDER` ja `OUTCOME`. Kõigil kolmel juhul tõlgendas kriitik nõuet rangemalt kui päris lugeja. Tekst, mis inimesele tundus tavaline ja arusaadav, kukkus mudeli jaoks läbi, sest põhjus valida teenusepakkuja polnud sõnastatud piisavalt selgelt välja.

C02 langes teistpidi. Kriitik leidis tekstist kõik viis tunnust ja andis `PASS`. Allan luges sama teksti ja ei peatunud esimese rea juures.

Hüpotees: hindaja, kes otsib tunnuseid, mõõdab teksti pinda. Ostja otsus sõltub reaktsioonist, mida tunnuste loend kinni ei püüa.

---

## 6. Kandidaatparandus

04.09.2026 kirjutati hindaja ümber. Viie küsimusega `JAH`/`EI` kontroll-leht asendus ühe esimeses isikus lugejareaktsiooniga: kus tähelepanu püsib, kus libiseb, kus mõte kaob ja mida oodati, aga ei tulnud. Otsus loetakse reaktsiooni viimasest lausest.

Samad 20 kandidaati, sama inimese ground truth.

| Kandidaat | Inimene | Vana kriitik | Uus hindaja |
|---|---|---|---|
| C01 | PASS | FAIL | PASS |
| **C02** | **FAIL** | **PASS** | **FAIL** |
| C03 | PASS | FAIL | PASS |
| C04 | PASS | FAIL | PASS |
| C05 | PASS | FAIL | PASS |
| C06 | PASS | PASS | PASS |
| C07 | PASS | FAIL | PASS |
| C08 | PASS | PASS | PASS |
| C09 | PASS | PASS | PASS |
| C10 | PASS | FAIL | PASS |
| C11 | PASS | PASS | PASS |
| C12 | PASS | FAIL | PASS |
| C13 | PASS | FAIL | PASS |
| C14 | PASS | FAIL | PASS |
| C15 | INCOMPLETE | FAIL | EBASELGE |
| **C16** | **PASS** | **FAIL** | **FAIL** |
| C17 | PASS | PASS | PASS |
| C18 | PASS | PASS | PASS |
| C19 | PASS | PASS | PASS |
| C20 | PASS | FAIL | PASS |

| Näitaja | Vana kriitik | Uus hindaja |
|---|---|---|
| Kokkulangevus | 7/20 (35%) | 18/20 (90%) |
| Vale-negatiivid | 11 | 1 (C16) |
| Vale-positiivid | 1 (C02) | 0 |

Kaks kohta väärivad eraldi tähelepanu.

**C02.** Uus hindaja andis õige `FAIL`. See on ainus tekst valimis, mille vale otsus oleks päris avaldamiseni jõudnud.

**C15.** Inimene jättis vastuse andmata, uus hindaja jäi `EBASELGE`. Reegli järgi loetakse see mittekokkulangevuseks. Sisuliselt kõhklesid mõlemad hindajad sama teksti juures.

---

## 7. Avatud defekt

Parandus on skilli sisse viimata.

`SKILL.md` jaotis "Pime ostjakontroll" ja `references/reviewer-contract.md` jaotis B sisaldavad seisuga 06.09.2026 endiselt viie küsimusega automaatset kriitikut. Skill suunab lugejareaktsiooni töö eraldi skillile.

Praktiline tähendus: skill on kasutuskõlblik, kui hindaja on inimene. Autonoomse kvaliteediväravana ta ei kvalifitseeru.

---

## 8. Jääkriskid

Septembri number 90% ei ole sõltumatu tõend. Neli põhjust, miks:

1. **Pimedus oli rikutud.** Hindaja oli sama sessiooni jooksul juba näinud inimese otsuste maatriksit, enne kui reaktsioonid kirjutati. Reaktsioonid on tekstiga põhjendatavad. Tõestamata jääb, et päris pime jooks annaks sama tulemuse.
2. **Baasmäär moonutab läve.** Inimese valimis oli 18 `PASS` 20-st. Sellise jaotuse juures läbib strateegia "anna alati PASS" mehaaniliselt seatud läve ilma ühtegi teksti lugemata. Väärtus tuleb ainult C02 tabamisest, ja seda tuleb hinnata tabelist, mitte protsendist.
3. **Uus vale-negatiiv.** C16 on uus viga, mitte null. Odavam kui vana kriitiku 11, aga olemas.
4. **Jooksu terviklust ei valideeritud sõltumatult.** `ICP-PROXY-QA-20260904-001` ei läbinud `validate_run.py` kontrolli, sest ta ei kasuta standardset `plan.json` struktuuri. Augustijooks läbis.

Lisaks jääb kogu case'i piiriks:

- valim on 20 teksti ja viis sünteetilist brändi;
- allikakontroll on tõendatud fixture'itel, päris brändi väidetel mitte;
- `SALES_VALIDATED` puudub. Ühtegi kampaaniat ei käivitatud, päringuid ega müüki ei mõõdetud.

**Mida suletud ring nõuab:** 20 uut hindamata teksti ja uus inimese pimetest hindaja peal, kes seda valimit varem näinud pole.

---

## 9. Mida sellest õppida

**Automaatne hindaja on komponent ja vajab oma testi.** Süsteemis, kus AI kirjutab ja AI hindab, võib hindaja olla nõrgim lüli, ilma et see väljundis välja paistaks. Selles jooksus oli kirjutaja korras ja väravas oli viga.

**Hindaja vastuvõtukriteerium on kokkulangevus inimesega, mitte tema enda enesekindlus.** Kriitik andis iga otsuse juurde põhjenduse ja kõik põhjendused kõlasid mõistlikult. 13 neist 20-st olid inimese otsusest mööda.

**Vale-negatiiv ja vale-positiiv tuleb lugeda eraldi.** Üks protsendiarv peidab ära, kumba viga süsteem teeb, ja need kaks maksavad erinevalt.

**Kaldu baasmäära kontrollida.** Kui 90% valimist on `PASS`, siis kõrge kokkulangevusprotsent tuleb odavalt kätte. Üksik õigesti tabatud tagasilükkamine tõendab rohkem kui protsent.

**Reaktsiooni küsimine töötas selles valimis paremini kui tunnuste loendamine.** Sama mudel, sama tekst, teistsugune küsimus, kokkulangevus tõusis 35%-lt 90%-le. Üldistuseks on üks valim vähe, hüpoteesiks järgmiseks jooksuks piisav.

---

## 10. Tõendid

Tõendipakk on privaatses auditijäljes, sest ta sisaldab skilli sisemist lepingut ja hindamisjuhiseid. Selle case'i numbrid pärinevad järgmistest failidest.

Augustijooks `TURUNDUSKIRJUTAJA-QA-20260822-008`, terviklus `VALID`:

| Fail | Mida tõendab |
|---|---|
| `evidence/tc03/candidates.json` | 20 kandidaatteksti, viis sünteetilist briefi |
| `evidence/tc04/allan_pimetest_answers.json` | inimhindaja toorvastused, viis küsimust teksti kohta |
| `evidence/tc04/human_score.json` | 18 `PASS`, 1 `FAIL`, 1 `INCOMPLETE`, staatus `HUMAN_VALIDATED` |
| `evidence/tc05/automatic_buyer_reviews.json` | automaatse kriitiku otsused ja põhjendused |
| `evidence/tc05/agreement_matrix.json` | kokkulangevus 7/20, 11 vale-negatiivi, 1 vale-positiiv |
| `validation.json` | terviklus `VALID`, 6 juhtumit, 13 assertion'it, 20 tõendit |
| `RESIDUAL_RISKS.md` | jooksu enda piirangud |

Septembrijooks `ICP-PROXY-QA-20260904-001`, terviklus `SELF_REPORTED_NOT_INDEPENDENTLY_VALIDATED`:

| Fail | Mida tõendab |
|---|---|
| `evidence/icp_reactions.md` | 20 lugejareaktsiooni |
| `evidence/comparison_matrix.json` | kandidaadipõhine võrdlus, kokkulangevus 18/20 |
| `run.json` | meetod, tulemus ja terviklusmärge |
| `RESIDUAL_RISKS.md` | pimeduse rike, baasmäär, valimi suurus |

Auditijälje saab päringu peale läbi vaadata.
