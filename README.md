# Gold Sentiment Index (GSI)

Der **Gold Sentiment Index (GSI)** ist ein kompaktes Forschungsprojekt, das
Finanz‑ und Makro‑Nachrichten in einen einzigen **Sentiment‑Index von 0–100**
für Gold überführt:

- **0 = Extrem bärisch** gegenüber Gold  
- **50 = Neutral**  
- **100 = Extrem bullisch** / sehr positiver Ton

Dies geschieht in folgenden Schritten:

1. **Abruf von Nachrichtenartikeln** zu Gold, Makroökonomie und Geopolitik über die **NewsAPI**.
2. **Sentiment‑Analyse mit einem finanzmarktspezifischen Modell (FinBERT)** für jeden Artikel.
3. **Gewichtung** jedes Artikels nach **Aktualität** und **makroökonomischem Impact**
   (Fed, Krisen, BRICS etc.).
4. Aggregation aller Informationen zu einem **Gold Sentiment Index** und
5. Generierung eines kompakten **HTML‑Dashboards** mit einem Gauge (0–100) und
   der aktuellen Regime‑Bezeichnung  
   („Extreme Fear“, „Fear“, „Neutral“, „Greed“, „Extreme Greed“).

Ziel ist es, eine schnelle, modellbasierte Sicht darauf zu geben, wie der
**jüngste Newsflow** für Gold tendiert (Fear vs. Greed) – vom Ansatz her ähnlich
dem Fear & Greed Index von CNN, jedoch fokussiert auf Gold und
makroökonomische Schlagzeilen.

---

## Projektstruktur

Wichtigste Dateien und ihre Funktionen:

- `run.py`  
  - Zentrale **Command‑Line‑Schnittstelle**.  
  - Koordiniert News‑Fetch, Sentiment‑Analyse und Aufbau des Dashboards.

- `scraping/newsapi.py`  
  - Anbindung an die **NewsAPI** (https://newsapi.org/).  
  - Formuliert Suchanfragen mit Fokus auf **Gold**, **Makro**, **Zinsen**, **BRICS**
    und **Geopolitik**.  
  - Ruft Artikel ab, normalisiert sie und speichert sie im Projekt‑Root in
    `news.json`.

- `processing/sentiment.py`  
  - Lädt Artikel aus `news.json`.  
  - Nutzt FinBERT zur Berechnung von **positiven/negativen/neutralen**
    Sentiment‑Scores pro Artikel.  
  - Wendet **Recency‑Gewichte** und **Macro‑Impact‑Gewichte** auf jeden Artikel an.  
  - Ruft `processing/index_calc.py` auf, um daraus einen einzelnen GSI‑Wert zu
    berechnen.  
  - Speichert eine strukturierte Auswertung in `sentiment_results.json` sowie
    einen kompakten Snapshot in `gsi_value.json`.

- `processing/index_calc.py`  
  - Verdichtet alle artikelbezogenen Scores zu einem **Gold Sentiment Index**.  
  - Leitet daraus zwei Kenngrößen ab:
    - `nw`   – Netto‑Sentiment in [-1, 1]  
    - `nw_norm` / `gsi` – normierter Index in [0, 100]  
  - Klassifiziert den Index in fünf Sentiment‑Regime:
    - 0–25   → **Extremely Bearish**  
    - 25–45  → **Bearish**  
    - 45–55  → **Neutral**  
    - 55–75  → **Bullish**  
    - 75–100 → **Extremely Bullish**

- `processing/dashboard.py`  
  - Liest `sentiment_results.json`.  
  - Erzeugt ein **interaktives HTML‑Dashboard** `dashboard.html` mit:
    - einem maßgeschneiderten Gauge (0–100) mit farbigen Regime‑Bändern,  
    - aktuellem numerischem GSI‑Wert,  
    - Regime‑Label (Extremely Bearish / Bearish / Neutral / Bullish / Extremely Bullish),  
    - Zeitstempel des letzten Updates.

- `models/finbert_gold.py`  
  - Wrapper um das **FinBERT‑Finanzsentiment‑Modell**
    (`yiyanghkust/finbert-tone`).  
  - Stellt eine einfache API bereit:
    - `analyze_text(text) -> SentimentScores`  
    - `analyze_batch(texts) -> List[SentimentScores]`

- Generierte Datendateien (entstehen beim Ausführen der Pipeline):  
  - `news.json`  
    - Alle abgerufenen Artikel (nach URL dedupliziert), sortiert von neu nach alt.  
  - `sentiment_results.json`  
    - Vollständiges Analyse‑Ergebnis (Scores pro Artikel plus Index‑Komponenten).  
  - `gsi_value.json`  
    - Minimaler Payload mit nur dem aktuellen GSI‑Wert und der Klassifikation.  
  - `dashboard.html`  
    - Eigenständiges Dashboard, das in jedem Browser geöffnet werden kann.

> Hinweis: `news_new.json` stammt aus älteren Versionen und wird **nicht mehr verwendet**.  
> Die Pipeline nutzt ausschließlich `news.json`. `news_new.json` kann für einen
> aufgeräumten Workspace problemlos gelöscht werden.

---

## Abhängigkeiten und Anforderungen

- **Python:** empfohlen wird Version 3.10 oder neuer.  
- **Bibliotheken:** in `requirements.txt` aufgeführt:
  - `requests`
  - `torch`
  - `transformers`
  - `pandas`
  - `numpy`
  - `python-dotenv`
  - `peft`
  - `pytz`

Zusätzlich erforderlich:

- Ein **NewsAPI‑Account** (Free Tier reicht für Experimente).  
- Ein System, auf dem PyTorch lauffähig ist – **CPU genügt**, GPU beschleunigt
  lediglich.

---

## Installation (Schritt für Schritt)

Diese Schritte beziehen sich auf ein frisch geklontes Repository.

### 1. Repository klonen

```bash path=null start=null
git clone <your-fork-or-repo-url>
cd gold-sentiment-index-main
```
> Wenn das Projekt als ZIP von GitHub geladen wurde: Archiv entpacken und mit
> `cd` in den Projektordner wechseln.

### 2. Python‑Abhängigkeiten installieren

Im Projektverzeichnis (ggf. mit aktivierter virtueller Umgebung):

```bash
pip install -r requirements.txt
# oder äquivalent (explizit):
# pip install requests torch transformers pandas numpy python-dotenv peft pytz
```
Damit werden u.a. PyTorch, Transformers, `pytz` und Hilfsbibliotheken installiert.

---

## API‑Keys & Konfiguration

Die **einzige** externe API, die dieses Projekt benötigt, ist die **NewsAPI**.

### 1. NewsAPI‑Key anfordern

1. Auf https://newsapi.org/ registrieren.  
2. Im Dashboard den **API‑Key** kopieren.

### 2. Vorhandene `.env` bearbeiten

Im Projekt‑Root liegt bereits eine Template‑Datei `.env`.  
Diese in einem Editor öffnen und den Platzhalterwert ersetzen:

```ini
NEWSAPI_KEY=your_real_newsapi_key_here
```
durch den echten Key, z.B.:

```ini
NEWSAPI_KEY=abcdef1234567890
```
- Keine Anführungszeichen setzen.  
- Dieser Key wird in `scraping/newsapi.py` über `python-dotenv` als
  Umgebungsvariable `NEWSAPI_KEY` eingelesen.  
- Es existiert **kein hart codierter API‑Key** im Source Code; alles läuft über
  die Umgebung.

Ist `NEWSAPI_KEY` nicht gesetzt oder noch auf dem Platzhalter, bricht der Code
mit einer klaren Fehlermeldung ab:

> `RuntimeError: NEWSAPI_KEY is not set in .env`

---

## Projekt ausführen

Der Haupteinstiegspunkt ist `run.py`, das unter dem Kommando `sentiment` eine
kleine CLI bereitstellt.

Im Projekt‑Root (mit aktivierter Umgebung):

```bash
python run.py sentiment update
```
> Der FinBERT‑Schritt kann – abhängig von Anzahl der Artikel in `news.json`
> und Hardware (CPU/GPU) – von wenigen Sekunden bis zu mehreren Minuten
> dauern. Beim ersten Lauf wird das Modell zusätzlich heruntergeladen.

### CLI‑Kommandos

`run.py` unterstützt folgende Subkommandos:

1. **News abrufen + Sentiment berechnen + Dashboard aktualisieren**

```bash
   python run.py sentiment update
```
   - Ruft `scraping.newsapi.run_cli()` auf und schreibt/merged neue Artikel in
     `news.json`.  
   - Führt FinBERT‑Sentimentanalyse über alle **aktuellen** Artikel aus.  
   - Berechnet den Gold Sentiment Index.  
   - Schreibt `sentiment_results.json` und `gsi_value.json`.  
   - Generiert das aktuelle `dashboard.html`.

2. **Nur News abrufen (ohne Analyse)**

```bash
   python run.py sentiment news
```
   - Ruft Artikel über die vordefinierten NewsAPI‑Queries ab.  
   - Merged sie in `news.json` (Deduplikation per URL).  
   - Führt **keine** Sentiment‑Analyse aus und verändert das Dashboard nicht.

3. **Nur vorhandene News analysieren (kein neuer API‑Call)**

```bash
   python run.py sentiment analyze
```
   - Liest die bestehende `news.json`.  
   - Führt FinBERT‑Analyse für alle relevanten Artikel durch.  
   - Erzeugt `sentiment_results.json`, `gsi_value.json` und aktualisiert
     `dashboard.html`.

4. **Nur Dashboard neu erzeugen (ohne neue Analyse)**

```bash
   python run.py sentiment dashboard
```
   - Holt **keine** neuen Daten.  
   - Führt **keine** erneute Sentiment‑Berechnung durch.  
   - Zeichnet lediglich `dashboard.html` auf Basis von
     `sentiment_results.json` neu.

Das ist nützlich, um z.B. API‑Limits zu schonen oder Anpassungen an
Modell‑Logik, Gewichtung oder Indexberechnung zu testen, ohne neue Daten zu
laden.

---

## Dashboard anzeigen

Nach Ausführung von

- `python run.py sentiment update` oder  
- `python run.py sentiment analyze`

liegt im Projekt‑Root eine Datei `dashboard.html`.

Diese kann im Browser wie folgt geöffnet werden:

- Doppelklick im Dateimanager, oder  
- Rechtsklick → „Öffnen mit …“ → Browser, oder  
- über die Shell:
  - macOS: `open dashboard.html`  
  - Linux: `xdg-open dashboard.html`  
  - Windows (PowerShell): `start dashboard.html`

Das Dashboard zeigt:

- Ein **Gauge** von 0 bis 100 mit farbigen Bändern:
  - 0–25: Extremely Bearish (rot)  
  - 25–45: Bearish (hellrot)  
  - 45–55: Neutral (grau)  
  - 55–75: Bullish (hellgrün)  
  - 75–100: Extremely Bullish (grün)
- Den **aktuellen GSI‑Wert** (gerundet).  
- Das zugehörige **Regime‑Label** (z.B. „Greed“).  
- Den **Zeitstempel des letzten Updates** (in die lokale Zeitzone des
  Betrachters konvertiert).

---

## Was die Skripte im Detail tun (End‑to‑End)

In diesem Abschnitt wird die gesamte Pipeline in einfachen Worten beschrieben.

### 1. News‑Erfassung (`scraping/newsapi.py`)

- Definiert mehrere umfangreiche **Suchqueries**, die u.a. abdecken:
  - Gold und Edelmetalle („gold“, „bullion“, „gold price“, „gold demand“),  
  - Geldpolitik und Zinsen („Fed“, „interest rates“, „rate hike“, „rate cut“),  
  - Makro‑Umfeld („inflation“, „recession“, „deflation“, „stagflation“),  
  - BRICS, Sanktionen, Geopolitik, Handelskonflikte etc.
- Ruft den `everything`‑Endpoint von NewsAPI mit u.a. folgenden Parametern auf:
  - `language=en`  
  - `sortBy=publishedAt`  
  - `pageSize=100`  
  - `page=1 .. max_pages` (unter Berücksichtigung der Free‑Tier‑Limits).
- Normalisiert jeden Artikel auf ein kompaktes JSON‑Schema:

```json
  {
    "title": "...",
    "description": "...",
    "content": "...",
    "url": "https://example.com/...",
    "timestamp": "2025-12-01T08:49:45+00:00"  // immer UTC ISO8601
  }
```
- Optionaler **ökonomisch/politischer Keyword‑Filter** (standardmäßig deaktiviert,
  um breitere Coverage zu haben).  
- Dedupliziert Artikel anhand der **URL** über mehrere Läufe hinweg, sodass
  derselbe Artikel nicht mehrfach gezählt wird.  
- Führt neue Artikel mit bestehenden zusammen und schreibt die aktualisierte
  Liste nach `news.json` (neueste zuerst).

### 2. Sentiment auf Dokumentebene (`processing/sentiment.py` + `models/finbert_gold.py`)

Für jeden Artikel in `news.json`:

1. Erzeugt einen Text‑Blob aus:
   - `title`  
   - `description`  
   - `content`
2. Berechnet ein **Recency‑Gewicht** basierend auf dem Artikel‑Zeitstempel:

   - 0–1 Tage alt   → 1,0 (sehr frisch, voller Impact)  
   - 1–3 Tage       → 0,8  
   - 3–7 Tage       → 0,6  
   - 7–14 Tage      → 0,3  
   - 14–30 Tage     → 0,1  
   - älter als 30 Tage → 0,0 (praktisch ignoriert)

3. Führt das FinBERT‑Modell auf dem Text aus und erhält Wahrscheinlichkeiten:

   - `positive` in [0, 1]  
   - `negative` in [0, 1]  
   - `neutral`  in [0, 1]

4. Berechnet ein **Impact‑Gewicht** pro Dokument, das:

   - auf der **Margin** `|positive - negative|` basiert → Konfidenz, ob die
     Headline klar bullisch oder bärisch ist,  
   - den Wert um den Faktor **3** erhöht, wenn bestimmte Makro‑Schlüsselwörter
     auftreten, z.B. „Powell“, „Federal Reserve“, „rate hike“, „recession“,
     „crisis“, „de-dollarization“, „BRICS“,  
   - eine nichtlineare Potenz (`gamma = 1.5`) anwendet, um starke Signale zu
     betonen und mehrdeutige zu dämpfen.

5. Multipliziert **Recency‑Gewicht × Impact‑Gewicht**, um ein
   **effektives Gewicht** für jeden Artikel zu erhalten.

Ergebnis:

- Eine Liste von `SentimentScores` (je Artikel)  
- Eine korrespondierende Liste von **Gewichten**, die Frische und
  makroökonomische Relevanz reflektieren.

### 3. Indexberechnung (`processing/index_calc.py`)

Auf Basis aller artikelbezogenen Scores und Gewichte:

1. Berechnet einen **gewichteten Durchschnitt** der positiven, negativen und
   neutralen Wahrscheinlichkeiten über alle Dokumente.  
2. Definiert ein Netto‑Sentiment **NW** als:

```text
   NW_raw = avg_positive - avg_negative   # Bereich [-1, 1]
   NW = clip(SENSITIVITY * NW_raw, -1, 1)
```
   wobei `SENSITIVITY` aktuell `2.2` beträgt und dafür sorgt, dass sich der
   Index stärker bewegt, wenn das Modell eine hohe Konfidenz hat.

3. Normiert `NW` von [-1, 1] auf [0, 100]:

```text
   nw_norm = (NW + 1.0) * 50.0
   gsi = nw_norm
```
4. Ordnet den resultierenden `gsi`‑Wert einem **Regime‑Label** zu:

   - `< 25`  → „Extremely Bearish“  
   - `< 45`  → „Bearish“  
   - `< 55`  → „Neutral“  
   - `< 75`  → „Bullish“  
   - `>= 75` → „Extremely Bullish“

Die finale Struktur `IndexComponents` enthält:

- `nw` – Netto‑Sentiment in [-1, 1]  
- `nw_norm` – Normalisierung auf [0, 100]  
- `gsi` – Alias für `nw_norm`  
- `classification` – Regime‑Label (Extremely Bearish / Bearish / Neutral /
  Bullish / Extremely Bullish)

### 4. Ergebnisablage und Dashboard (`processing/sentiment.py` & `processing/dashboard.py`)

- `processing/sentiment.save_results()` schreibt eine detaillierte JSON‑Struktur
  nach `sentiment_results.json`, inkl.:
  - Zeitstempel des Laufs,  
  - Sentiment‑Scores und Metadaten pro Artikel,  
  - Index‑Komponenten (`nw`, `nw_norm`, `gsi`, `classification`).  
- Zusätzlich wird ein kleines `gsi_value.json` erzeugt, das nur den aktuellen
  Index und die Klassifikation enthält (z.B. für Integrationen).  
- `processing/dashboard.generate_dashboard()`:
  - Liest `sentiment_results.json`.  
  - Generiert `dashboard.html` mit eingebettetem `<canvas>` und
    Custom‑Zeichenlogik (kein Backend‑Server nötig).  
  - Zeichnet:
    - Hintergrund‑Bänder (Extremely Bearish → Extremely Bullish),  
    - den Zeiger beim aktuellen GSI‑Wert,  
    - numerischen Wert und Regime‑Text,  
    - „Last updated“ im lokalen Datums‑/Zeitformat.

---

## Warum es `news.json` und `news_new.json` gab

Historisch schrieb das Skript:

- `news.json` – die **vollständig gemergte Historie** aller Artikel,  
- `news_new.json` – nur die **neu gefundenen** Artikel des letzten Laufs.

Die weitere Pipeline (Sentiment & Index) verwendete ausschließlich `news.json`.
`news_new.json` war nur Hilfsdatei und nicht notwendig.

Zur Vereinfachung wurde der Code angepasst, sodass **nur noch `news.json`
relevant** ist. `news_new.json` wird nicht mehr geschrieben oder genutzt; sie
kann bei Bedarf gelöscht werden.

---

## Rate Limits, Kosten und praktische Hinweise

- Kostenlose NewsAPI‑Accounts haben **begrenzte** Request‑Kontingente und
  Historienzugriff.  
- `python run.py sentiment news` sollte auf dem Free Tier sparsam eingesetzt
  werden.  
- Bei HTTP‑Fehlern von NewsAPI (z.B. 426 Upgrade Required) sollten
  Seitenanzahl und/oder Abruffrequenz reduziert werden.  
- Für historische Backfills können
  `scraping/newsapi.backfill_last_days()` und
  `scraping/newsapi.run_backfill_cli()` verwendet werden – hierbei ist jedoch
  zu beachten, dass Free‑Tier‑Limits leicht überschritten werden können.

---

## Troubleshooting

**1. `RuntimeError: NEWSAPI_KEY is not set in .env`**

- Prüfen, ob `.env` im Projekt‑Root existiert.  
- Sicherstellen, dass `NEWSAPI_KEY` auf den echten Key gesetzt ist:

```ini
  NEWSAPI_KEY=abcdef1234567890
```
- Terminal ggf. neu starten bzw. virtuelle Umgebung neu aktivieren, damit die
  Umgebungsvariablen eingelesen werden.

**2. `requests.exceptions.HTTPError` von NewsAPI**

- Die ausgegebene Fehlermeldung prüfen; typische Ursachen:
  - ungültiger API‑Key,  
  - Quota überschritten,  
  - Free‑Tier‑Beschränkungen (zu viele Seiten, zu breiter Datumsbereich).  
- `max_pages` und/oder das Datumsfenster anpassen.

**3. Modell‑Download langsam oder fehlerhaft**

- Beim ersten FinBERT‑Lauf lädt `transformers` das Modell aus dem Internet.  
- Stabile Internetverbindung sicherstellen.  
- Bei Speicherproblemen (Prozess wird beendet) ggf. auf Maschine mit mehr RAM
  oder Swap ausweichen.

**4. Dashboard ist permanent „Neutral“**

- Wenn `news.json` überwiegend nicht‑makrorelevante oder sehr kurze/ambigue
  Inhalte enthält, tendiert FinBERT zu neutralen Einschätzungen.  
- Keyword‑Filter in `scraping/newsapi.py` ggf. verschärfen.  
- Recency‑ und Impact‑Gewichtungen in `processing/sentiment.py` und
  `processing/index_calc.py` anpassen, um den Index sensibler oder
  weniger sensibel zu machen.

---

## Mögliche Erweiterungen

Ideen für zukünftige Ausbauten:

- Integration von **Twitter/X‑Sentiment** (separate API und Keys erforderlich;
  aktuell aus Einfachheitsgründen nicht implementiert).  
- Persistente **GSI‑Zeitreihe** (z.B. Tageswerte) speichern und als Chart unter
  dem Gauge visualisieren.  
- Weitere domänenspezifische Modelle oder speziell auf Gold feinjustierte
  Modelle ausprobieren.  
- Das Dashboard als kleine Web‑Applikation statt als rein statische Datei
  bereitstellen.

Der aktuelle Kern‑Workflow ist:

1. `.env` öffnen und `NEWSAPI_KEY` auf den NewsAPI‑Key setzen.  
2. `pip install -r requirements.txt`.  
3. `python run.py sentiment update`.  
4. `dashboard.html` im Browser öffnen.

Mehr ist nicht erforderlich, um von Rohdaten (Makro‑/Gold‑Headlines) zu einem
visuellen Gold Sentiment Index zu gelangen.
```


```
