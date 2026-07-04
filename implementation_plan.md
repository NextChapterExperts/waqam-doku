# Implementation Plan: Step-by-Step Complexity & Platform TCO Simulator

Dieses Dokument beschreibt die geplante Implementierung der schrittweisen Komplexitätsrechnung (Step-level SCI), der technologieagnostischen Implementierungstypen pro Schritt und des stochastischen Validierungssicherheitsnetzes im WAQAM Simulator.

## User Review Required

> [!IMPORTANT]
> **Mathematische Dämpfungswerte (De-rating):**
> Die Konfidenzdämpfung für komplexe kognitive Schritte ($C_i$) und Schnittstellen ($S_i$) wurde auf $-5\,\%$ (pro Punkt über $C=1$) bzw. $-3\,\%$ (pro Punkt über $S=1$) festgelegt. Diese Werte basieren auf historischen Modellerfahrungen und können im Simulator angepasst werden.

> [!WARNING]
> **Kompatibilitäts-Fallback:**
> Wenn ein Use Case keine Schrittdetails oder stochastischen Dimensionen deklariert, greift der Simulator automatisch auf das bewährte globale Modell (basierend auf `rules_count`) zurück. So wird die Abwärtskompatibilität gewahrt.

---

## Proposed Changes

### 1. Types & Data Structures

#### [MODIFY] [types.ts](file:///home/peter/peters-brain/Projekte/waqamboard/src/types.ts)
*   Erweiterung des `StepDetail` Interfaces um optionale 4D-Komplexitätsindizes ($I, R, S, C$) und den Implementierungstyp (`implType`).
*   Definition der unterstützten Implementierungstypen:
    ```typescript
    export interface StepDetail {
      desc: string;
      tech: string;
      cfo: string;
      I?: number; // Information Density (1-5)
      R?: number; // Rules Complexity (1-5)
      S?: number; // System Integration (1-5)
      C?: number; // Cognitive Discretion (1-5)
      implType?: 'deterministic' | 'ai' | 'agentic';
    }
    ```

#### [MODIFY] [useCases.ts](file:///home/peter/peters-brain/Projekte/waqamboard/src/data/useCases.ts)
*   Zuweisung standardisierter $I, R, S, C$-Werte und `implType`-Vorgaben für die einzelnen Schritte der wichtigsten Use-Cases (inklusive `EXT01`, `EXT02`, `EXT03` und Standard-Use-Cases).
*   *Beispiel für einen Schritt:*
    ```typescript
    "Stammdaten-Extraktion (KI)": {
      desc: "...",
      tech: "...",
      cfo: "...",
      I: 3, R: 3, S: 2, C: 4,
      implType: 'ai'
    }
    ```

---

### 2. Math & Simulation Engine

#### [MODIFY] [pcsEngine.ts](file:///home/peter/peters-brain/Projekte/waqamboard/src/utils/pcsEngine.ts)
*   Erweiterung der Funktionssignatur von `runPcsWaqamSimulation` um `steps?: string[]` und `stepDetails?: Record<string, StepDetail>`.
*   Implementierung der Schleife über alle Einzelschritte zur Berechnung der schrittweisen Durchlasswahrscheinlichkeit:
    *   **Dämpfung:** `mean_confidence_step = mean_confidence_base * (1.0 - (C - 1) * 0.05 - (S - 1) * 0.03)`
    *   **Fehler-Modifikator:** Anwendung der Klassen-Multiplikatoren auf den Schritt (Klasse 3 erhält Self-Correction-Vorteil für Agentic Steps).
    *   **Skalierung:** Multiplikation der Regeln mit `pos_safe` falls $I \ge 5$.
*   Berechnung der Gesamt-Dunkelquote als Produkt der Einzelschritte.
*   Einbau eines robusten Fallbacks für UCs ohne Detailangaben.

---

### 3. Verification & Validator Suite

#### [NEW] [pcsEngine.test.ts](file:///home/peter/peters-brain/Projekte/waqamboard/src/utils/pcsEngine.test.ts) (oder temporary validator)
*   Implementierung einer Test-Suite zur mathematischen Qualitätssicherung (QA):
    *   **Invarianten-Test:** Validierung, dass $V = \text{Auto} + \text{HITL} + \text{Errors}$ für alle Eingaben gilt.
    *   **Boundary-Test:** Verprüfung der mathematischen Ränder (Datenqualität = 100% / 0%).
    *   **Monotonie-Test:** Prüfung, ob eine Verschlechterung der Eingangswerte zu einer monoton sinkenden Kurve führt.
    *   **TCO-Plausibilität:** Validierung, dass die Klassen-Kosten konsistent bleiben.

---

### 4. UI & Visualisierung

#### [MODIFY] [Slide3Simulator.tsx](file:///home/peter/peters-brain/Projekte/waqamboard/src/components/Slide3Simulator.tsx)
*   Anpassung des Aufrufs `runPcsWaqamSimulation` unter Übergabe von `useCase.steps` und `useCase.step_details`.
*   Anzeige einer kleinen interaktiven Tabelle, die dem Benutzer die berechneten SCI-Scores pro Schritt transparent auflistet.

#### [MODIFY] [Slide4Certificate.tsx](file:///home/peter/peters-brain/Projekte/waqamboard/src/components/Slide4Certificate.tsx)
*   Dynamisches Zeichnen der Schritte als horizontale Boxen-Kette.
*   Farbliche und textuelle Kodierung der Boxen basierend auf der aktiven Klasse im Simulator:
    *   **Klasse 0 (No-AI):** Alle Schritte grau/blau (`[Deterministisch - Standard-Code]`).
    *   **Klasse 1 (Point-AI):** Extraktionstyp-Schritte werden grün (`[Point-AI OCR]`), andere grau.
    *   **Klasse 2 (Hybrid-AI):** Automatisierter Mix (grau = deterministisch, grün = AI-Extraktion mit Validation Gate).
    *   **Klasse 3 (Agentic):** Alle Schritte violett (`[Autonomous Agent - ReAct Loop]`).

---

## Verification Plan

### Automated Verification
1.  **TypeScript & Build Check:** Ausführung von `npm run build` zur Sicherstellung fehlerfreier Typisierung.
2.  **Validator Script:** Ausführen des stochastischen Testskripts und Kontrolle der Log-Ausgaben auf Invarianten-Verletzungen.

### Manual Verification
*   Wechseln zwischen den Klassen 0, 1, 2 und 3 im Simulator-Dashboard und visuelle Kontrolle, dass:
    *   die Boxen-Farben auf Folie 4 sofort umschalten,
    *   die Dunkelquote und Latenz plausibel springen (Klasse 3 erhöht die Latenz und Tokenkosten, senkt aber das Fehlerrisiko).
