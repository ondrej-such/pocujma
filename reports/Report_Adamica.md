## Obsah
- [Právny Rámec pre Nahrávanie Rádiového Vysielania](#Právny-Rámec-pre-Nahrávanie-Rádiového-Vysielania)
- [Technická Realizácia Nahrávania](#Technická-Realizácia-Nahrávania)
- [Prieskum Veľkých ASR Modelov](#Prieskum-Veľkých-ASR-Modelov)
- [Tvorba Trénovacej Množiny pre Detekciu Kľúčových Slov (KWS)](#Tvorba-Trénovacej-Množiny-pre-Detekciu-Kľúčových-Slov-KWS)

***

### Právny Rámec pre Nahrávanie Rádiového Vysielania

Nahrávanie a využitie rádiového vysielania pre výskum je na Slovensku legálny, ak sú dodržané podmienky stanovené kľúčovými právnymi predpismi.

#### **Kľúčová Legislatíva**

-   **Autorský zákon (č. 185/2015 Z. z.):** Najmä novela č. 71/2022 Z. z., ktorá upravuje výnimku pre textové a dátové ťaženie (TDM) na účely výskumu.
    
-   **Smernica EÚ (2019/790):** Harmonizuje pravidlá pre TDM v EÚ.
    
-   **GDPR (Nariadenie EÚ 2016/679):** Upravuje ochranu osobných údajov (napr. hlasov) vo výskume.
    
#### **Základné Podmienky Projektu**

**1. Účel a Zdroj:**  
Projekt musí byť striktne **vedecký a nekomerčný**, realizovaný oprávnenou **výskumnou inštitúciou**. Zdroj dát musí byť legálny, čo voľne dostupné rádio spĺňa.

**2. Bezpečnosť a Prístup:**  
Nahrávky sa musia **bezpečne uchovávať**. Prístup k nim smie mať **iba výskumný tím** a je zakázané ich akokoľvek zverejňovať.

**3. Výstupy a Ochrana Dát:**  
Výsledný model nesmie umožniť rekonštrukciu pôvodných nahrávok. Osobné údaje (hlasy, mená) je potrebné chrániť, minimalizovať a podľa možnosti anonymizovať.

**4. Ukončenie Projektu:**  
Po skončení výskumu je nutné **všetky nahrávky zmazať**.

***

### Technická Realizácia Nahrávania

Systém na nahrávanie rádiového vysielania je navrhnutý ako robustná, kontajnerizovaná aplikácia, ktorá zabezpečuje nepretržitý záznam šiestich zvolených staníc.

#### **Použité Technológie**

-   **Docker:** Celé riešenie beží v izolovanom kontajneri (na báze **python:3.11-slim**), čo zaručuje konzistentné prostredie a jednoduché nasadenie.
    
-   **FFmpeg:** Kľúčový nástroj na spracovanie multimédií, ktorý zabezpečuje samotné pripojenie k streamom, konverziu a ukladanie zvuku.
    
-   **Python:** Skript, ktorý spravuje procesy nahrávania a zabezpečuje odolnosť voči chybám.
    
#### **Funkcionalita Systému**

**1. Paralelné Nahrávanie:**  
Python skript využíva knižnicu **multiprocessing** na spustenie samostatného procesu pre každú zo 6 rádiových staníc. Tým je zaručené, že výpadok jedného streamu neovplyvní ostatné.

**2. Segmentácia a Formát:**  
Nahrávanie sa neukladá do jedného veľkého súboru, ale je automaticky delené na menšie celky:

-   **Dĺžka segmentu:** 10 minút (pre ľahšiu správu a spracovanie).
    
-   **Formát:** Opus pri bitrate 128 kbps (úsporný a kvalitný formát pre reč a hudbu).
    
**3. Odolnosť a Monitoring:**  
Hlavný proces pravidelne (každých 60 sekúnd) kontroluje stav nahrávacích procesov. V prípade, že niektorý proces zlyhá (napr. výpadok streamu), systém ho automaticky reštartuje.

**4. Časová Synchronizácia:**  
Kontajner má nastavenú časovú zónu Europe/Bratislava, čo zabezpečuje korektný čas.

***

### Prieskum Veľkých ASR Modelov

Moderné hlasové asistenty (Google Assistant, Amazon Alexa, Apple Siri) využívajú pokročilé ASR (Automatic Speech Recognition) modely založené na hlbokých neurónových sieťach. Všetky fungujú na princípe neustále aktívneho, lokálneho detektora (zvyčajne DNN) pre aktivačné slovo ("wake-word"), ktorý spúšťa výkonnejší systém na rozpoznávanie reči.

#### **Google Assistant**

*   **Model:** Využíva end-to-end architektúru **RNN-Transducer (RNN-T)**, ktorá transkribuje reč znak po znaku v reálnom čase. Tento model beží priamo na zariadení (napr. v telefóne), čo znižuje odozvu a umožňuje offline použitie.
*   **Optimalizácia:** Pôvodne veľký model (450 MB) bol agresívnou kvantizáciou zmenšený na približne **80 MB**, čo mu umožňuje efektívny beh aj na mobilných zariadeniach.
*   **Tréningové dáta:** Modely sú trénované na obrovských, anonymizovaných súboroch dát, napríklad jeden z modelov bol trénovaný na približne **18 000 hodinách** hlasových záznamov.
*   **Aktivačné slová:** Používa frázy "OK Google" alebo "Hey Google". Tieto slová sú foneticky bohaté a v bežnej reči menej časté, čo znižuje riziko falošných aktivácií.

#### **Amazon Alexa**

*   **Model:** Podobne ako Google, aj Amazon využíva moderné end-to-end modely, vrátane architektúry **RNN-T**. Systém aktívne využíva kontext z predchádzajúcich konverzácií na spresnenie rozpoznávania.
*   **Nasadenie:** Historicky sa rozpoznávanie dialo v cloude, no Amazon presúva čoraz viac funkcionality priamo na zariadenia ("on-device"), aby znížil odozvu.
*   **Tréningové dáta:** Modely sú trénované na tisícoch hodín (5 v uvedenom experimente) reálnych, anonymizovaných interakcií s Alexou.
*   **Aktivačné slová:** Štandardné slovo je "Alexa", no používatelia majú na výber aj iné alternatívy alebo vlastné slová. Systém používa jednoduchší lokálny model na detekciu a následne druhý, presnejší model (často v cloude) na overenie.

#### **Apple Siri**

*   **Model:** Systém je výrazne zameraný na súkromie a funguje kompletne na zariadení. Využíva dvojfázový prístup:
    1.  Malá, neustále bežiaca neurónová sieť s extrémne nízkou spotrebou energie deteguje potenciálne aktivačné slová.
    2.  Výkonnejší **Conformer** model následne overí, či išlo o správnu frázu, čím zabezpečí vysokú presnosť.
*   **Tréningové dáta:** Apple detaily nezverejňuje.
*   **Aktivačné slová:** Používa frázy "Hey Siri" a od nedávna aj samotné "Siri". Systém navyše obsahuje model na rozpoznanie hlasu majiteľa, aby sa zabránilo aktivácii cudzími osobami.

#### Zdroje

1.  **Google:** [An All-Neural On-Device Speech Recognizer](https://research.google/blog/an-all-neural-on-device-speech-recognizer/)
2.  **Apple:** [Voice Trigger System for Siri - Apple Machine Learning Research](https://machinelearning.apple.com/research/voice-trigger)
3.  **Amazon:** [Amazon’s new research on automatic speech recognition - Amazon Science](https://www.amazon.science/blog/amazons-new-research-on-automatic-speech-recognition)

***

### Tvorba Trénovacej Množiny pre Detekciu Kľúčových Slov (KWS)

Kvalita KWS modelu priamo závisí od zloženia a vyváženosti trénovacích dát. Cieľom je naučiť model ignorovať všetko okrem definovaného kľúčového slova.

#### **Zloženie a Pomer Dát**

Trénovacia množina je zámerne **extrémne nevyvážená**, aby odrážala reálne použitie, kde sa kľúčové slovo vysloví len zriedka.

-   **Pomer Pozitívnych a Negatívnych Vzoriek:**  
    V tréningu sa typicky používa pomer negatívnych vzoriek voči pozitívnym v rozmedzí **10:1 až 20:1**.
    
-   **Zloženie Negatívnej Množiny:**  
    Kľúčom k úspechu je rôznorodá negatívna množina, ktorá učí model odolnosti. Jej typické zloženie je:
    
    -   **~60% Bežná Reč:** Nahrávky reči, ktoré neobsahujú kľúčové slovo. Zabraňujú aktivácii na bežnú konverzáciu.
        
    -   **~25% Podobné Slová ("Hard Negatives"):** Slová alebo frázy, ktoré znejú foneticky podobne ako kľúčové slovo. Sú kritické pre zníženie počtu falošných aktivácií.
        
    -   **~15% Hluk a Ticho:** Zvuky prostredia (hudba, ruch ulice, kancelársky šum) a segmenty ticha.
        
#### **Augmentácia Dát**

Na umelé zvýšenie variability dát a robustnosti modelu sa používajú nasledujúce techniky:

-   **Pridávanie šumu:** Do nahrávok sa miešajú rôzne typy pozadia (hluk, hudba).
    
-   **Simulácia ozveny (Reverb):** Napodobnenie akustiky rôznych miestností.
    
-   **Zmena rýchlosti a výšky hlasu:** Simulácia variability medzi hovorcami.


***

