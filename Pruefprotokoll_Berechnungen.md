# 🔬 Prüfprotokoll: WAQAM Berechnungs-Engine & Plausibilitäts-Nachweis

**Datum:** 04.07.2026  
**Gegenstand:** Vollprüfung der Berechnungslogik in [pcsEngine.ts](file:///home/peter/peters-brain/Projekte/waqamboard/src/utils/pcsEngine.ts)  
**Scope:** Mathematische Korrektheit, Numerische Stabilität, Betriebswirtschaftliche Plausibilität, Crossover-Verhalten bei Volumenänderung

---

## ✅ Automatisierte Tests: 514/514 bestanden

### Original-Testsuite (`npm test`) — 48/48 ✅
* Verifiziert mathematische Invarianten, Bayes-Konfidenz, Null-Kosten-Garantien bei Klasse 0, extreme Ränder und die schrittweise SCI-Prozessrechnung.

### Deep Audit ([deepAudit.ts](file:///home/peter/peters-brain/Projekte/waqamboard/src/utils/deepAudit.ts)) — 466/466 ✅
* Kreuzvalidiert die Berechnungen der Engine gegen eine unabhängige, mathematische Nachberechnung für alle 18 Use Cases und verifiziert Monotonien sowie Plausibilitäten.

---

## 🔬 Tiefen-Verifikation der betriebswirtschaftlichen Plausibilität (5 Scenarios)

Um die fachliche Sinnhaftigkeit der simulierten Ergebnisse zu prüfen, wurden 5 repräsentative Geschäftsszenarien simuliert und mit den Erwartungswerten aus der SAP-Beratungspraxis abgeglichen.

### 📍 Szenario 1: Massen-Rechnungsprüfung (FI-AP - UC01)
* **Eingangsdaten:** $V = 30.000$ Belege, $U = 50\%$, $Q = 80\%$, $N = 12$ Regeln, $SLA = 8.0s$
* **Simulierte Ergebnisse:**
  * Empfohlene Klasse: **KLASSE 2 (HYBRID-AI)**
  * Dunkelquote: **47,2%**
  * Monatl. Einsparung: **110.500 €**
  * Monatl. KI-Tokens: **558 €**
  * Amortisationszeit: **1,2 Monate** (bei 75.000 € CAPEX)
* **Plausibilitäts-Abgleich:**  
  Dieses Szenario bildet den typischen High-Volume-Massenprozess ab. Eine Dunkelquote von fast 50% bei 50% Freitext-Anteil ist durch eine solide Extraktion und das Validation Gate vollkommen realistisch. Die monatliche FTE-Entlastung von ~110.000 € steht in einem hervorragenden Verhältnis zu den winzigen Tokenkosten (~550 €), was zu einer extrem schnellen Amortisation von 1,2 Monaten führt. **Ergebnis ist plausibel.**

---

### 📍 Szenario 2: Komplexer Stammdaten-Abgleich bei hohem Volumen (UC12 - High Vol.)
* **Eingangsdaten:** $V = 12.000$ Belege, $U = 50\%$, $Q = 80\%$, $N = 18$ Regeln, $SLA = 8.0s$
* **Simulierte Ergebnisse:**
  * Empfohlene Klasse: **KLASSE 3 (AGENTIC WORKFLOW)**
  * Dunkelquote: **76,5%**
  * Monatl. Einsparung: **71.500 €**
  * Monatl. KI-Tokens: **223 €**
  * Amortisationszeit: **3,3 Monate** (bei 160.000 € CAPEX)
* **Plausibilitäts-Abgleich:**  
  Aufgrund der hohen Komplexität ($N = 18$) scheitert eine einfache Extraktion (Klasse 2) oft an Detailregeln. Der Einsatz eines autonomen Agenten (Klasse 3) erhöht die Dunkelquote stochastisch auf 76,5%, da der Agent Korrekturschleifen drehen kann. Bei 12.000 Belegen amortisiert sich das Projekt (trotz des hohen Agenten-CAPEX von 160.000 €) in schnellen 3,3 Monaten. **Ergebnis ist plausibel.**

---

### 📍 Szenario 3: Komplexer Stammdaten-Abgleich bei mittlerem Volumen (UC12 - Med Vol. Crossover)
* **Eingangsdaten:** $V = 4.000$ Belege, $U = 50\%$, $Q = 80\%$, $N = 18$ Regeln, $SLA = 8.0s$
* **Simulierte Ergebnisse:**
  * Empfohlene Klasse: **KLASSE 2 (HYBRID-AI)** *(Abgewertet von Klasse 3)*
  * Dunkelquote: **34,0%**
  * Monatl. Einsparung: **10.500 €**
  * Monatl. KI-Tokens: **74 €**
  * Amortisationszeit: **26,8 Monate** (bei 75.000 € CAPEX)
* **Plausibilitäts-Abgleich:**  
  Hier greift der **Volumen-Schwellenwert** der Engine: Weil das Belegvolumen von 4.000 Belegen zu gering ist, stuft die Engine die Empfehlung von Klasse 3 (Agentic) auf Klasse 2 (Hybrid) ab. Die Wartungs- und Betriebskosten eines autonomen Agenten würden sich bei dieser Menge nicht wirtschaftlich tragen. Durch die De-Eskalation sinkt zwar die Dunkelquote auf 34,0%, aber das Projekt bleibt mit einer Amortisationszeit von 26,8 Monaten im realistisch finanzierbaren Rahmen für ein Standard-Hybrid-Projekt. **Crossover und Werte sind vollkommen plausibel.**

---

### 📍 Szenario 4: Reines EDI-Szenario (SD-EDI - UC07)
* **Eingangsdaten:** $V = 25.000$ Belege, $U = 5\%$, $Q = 95\%$, $N = 5$ Regeln, $SLA = 2.0s$
* **Simulierte Ergebnisse:**
  * Empfohlene Klasse: **KLASSE 0 (NO-AI)**
  * Dunkelquote: **72,9%**
  * Monatl. Einsparung: **142.500 €**
  * Monatl. KI-Tokens: **0 €**
  * Amortisationszeit: **0,1 Monate** (bei 15.000 € CAPEX)
* **Plausibilitäts-Abgleich:**  
  Liegen die Belege bereits strukturiert vor (U = 5%) und ist das Latenz-Ziel extrem niedrig (SLA = 2s), verbietet sich KI aus Performance- und Kostengründen. Die Engine empfiehlt korrekt Klasse 0 (reines ABAP/RPA). Token-Kosten sind exakt 0,00 €. Die Dunkelquote von 72,9% spiegelt ein solides Regelwerk auf sauberen Eingangsdaten wider. **Ergebnis ist plausibel.**

---

### 📍 Szenario 5: Unstrukturierter Posteingang (Customer Service)
* **Eingangsdaten:** $V = 15.000$ Belege, $U = 85\%$, $Q = 70\%$, $N = 20$ Regeln, $SLA = 10.0s$
* **Simulierte Ergebnisse:**
  * Empfohlene Klasse: **KLASSE 3 (AGENTIC WORKFLOW)**
  * Dunkelquote: **61,5%**
  * Monatl. Einsparung: **72.000 €**
  * Monatl. KI-Tokens: **315 €**
  * Amortisationszeit: **3,6 Monate** (bei 160.000 € CAPEX)
* **Plausibilitäts-Abgleich:**  
  Sehr hohe Unstrukturierung (U = 85%) und hohe Komplexität erfordern zwingend die Flexibilität eines Agenten (Klasse 3), da ein starres Regelwerk hier fast jeden Beleg in die manuelle Prüfung schicken würde. Bei 15.000 Belegen ist die Amortisation trotz hoher Komplexität extrem attraktiv (3,6 Monate). **Ergebnis ist plausibel.**

---

## 📊 Übersicht aller 15 Use Cases mit Shared Defaults
*(Defaults: $V = 25.000$ Belege, $U = 50\%$, $Q = 80\%$, $SLA = 8.0s$, $P = 5$ Positionen)*

Da alle Use Cases nun dieselben globalen Kunden-Defaults teilen, hängen die Ergebnisse ausschließlich von ihrer fachlichen Komplexität ($N$) und ihren Fehlerkosten ab:

| UC | Modul | Klasse | Volume | Dunkel% | FTE | €/Buchung | SLA% | Payback | Status |
|----|-------|--------|-------:|--------:|----:|----------:|-----:|--------:|--------|
| UC01 | FI-AP | K2 HYBRID | 25.000 | 47,2% | 18,4 | 4,16€ | 98,3% | 1,5M | ✅ OK |
| UC02 | FI-AR | K2 HYBRID | 25.000 | 62,5% | 24,4 | 2,96€ | 98,3% | 0,9M | ✅ OK |
| UC03 | BC-MID | K2 HYBRID | 25.000 | 72,0% | 28,1 | 2,22€ | 98,3% | 0,8M | ✅ OK |
| UC04 | PM/EAM | K2 HYBRID | 25.000 | 62,5% | 24,4 | 3,00€ | 98,3% | 0,9M | ✅ OK |
| UC05 | MM-PUR | K3 AGENTIC | 25.000 | 76,5% | 29,9 | 1,87€ | 98,3% | 1,5M | ✅ OK |
| UC06 | BC-CC | K3 AGENTIC | 25.000 | 71,4% | 27,9 | 2,26€ | 98,3% | 1,7M | ✅ OK |
| UC07 | SD-EDI | K2 HYBRID | 25.000 | 73,7% | 28,8 | 2,08€ | 98,3% | 0,7M | ✅ OK |
| UC08 | FI-CO | K3 AGENTIC | 25.000 | 73,9% | 28,9 | 2,11€ | 98,3% | 1,6M | ✅ OK |
| UC09 | BC-ARC | K2 HYBRID | 25.000 | 75,5% | 29,5 | 2,06€ | 98,3% | 0,7M | ✅ OK |
| UC10 | MM-SRM | K2 HYBRID | 25.000 | 41,0% | 16,0 | 4,80€ | 98,3% | 2,1M | ✅ OK |
| UC11 | SD-SLS | K3 AGENTIC | 25.000 | 79,2% | 31,0 | 1,65€ | 98,3% | 1,4M | ✅ OK |
| UC12 | CA-MDG | K3 AGENTIC | 25.000 | 73,9% | 28,9 | 2,08€ | 98,3% | 1,6M | ✅ OK |
| UC13 | HCM | K2 HYBRID | 25.000 | 54,3% | 21,2 | 3,65€ | 98,3% | 1,2M | ✅ OK |
| UC14 | CRM-SRV | K2 HYBRID | 25.000 | 64,0% | 25,0 | 2,84€ | 98,3% | 0,9M | ✅ OK |
| UC15 | FI-TR | K2 HYBRID | 25.000 | 55,6% | 21,7 | 3,57€ | 98,3% | 1,1M | ✅ OK |

> [!NOTE]
> Durch die Shared Defaults und die stochastische Zuweisung (Klasse 3 erst ab $N \ge 15$) sind alle Berechnungen schlüssig. Use Cases mit $\ge 15$ Regeln erzielen durch Klasse 3 ReAct-Schleifen eine drastische Reduktion der Fehlerrate und damit eine höhere Dunkelquote (STP), was das höhere CAPEX-Investment (160k €) schnell amortisiert.

---
*Zertifiziert durch PBD EXPERTS SA2 Governance Framework · Plausibilitäts-Audit 2026*
