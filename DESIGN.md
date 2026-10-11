# Design-Spezifikation – Elektrotechnik-Simulationen (Quillmesh)

Referenz-Implementierung: `quillmesh/kirchhoff-simulation` → `index.html`
Live: https://quillmesh.github.io/kirchhoff-simulation/

Dieses Dokument beschreibt Aussehen, Aufbau, Interaktion und Rechenkonventionen der Kirchhoff-Simulation so vollständig, dass eine neue Simulation (z.B. `quillmesh/spannungsteiler-simulation`) **im exakt gleichen Design** gebaut werden kann. Teil B enthält den CSS- und Zeichencode wörtlich. Er soll **1:1 übernommen** und nicht neu erfunden werden.

---

## Teil A – Regeln und Konventionen

### 1. Technischer Rahmen
- **Eine einzige Datei `index.html`** mit HTML, `<style>` und `<script>` inline. Keine Frameworks, keine Bibliotheken, keine externen Fonts, kein Build-Schritt.
- Vanilla JavaScript, globale Zustandsvariablen, Funktionen per `onclick`-Attribut.
- Hosting über GitHub Pages (Branch `main`, Ordner `/root`), Datei heisst `index.html`.
- Sprache der Oberfläche: **Deutsch** (Schweizer Schreibweise: „ss“ statt „ß“), `<html lang="de">`.
- Grösseneinheiten: V, A, Ω. Dezimaltrennzeichen ist der **Punkt** (`toFixed`).

### 2. Farbschema (Solarized)
Alle Farben werden **nur über CSS-Variablen** verwendet (`var(--…)`), auch innerhalb des SVG.

| Variable | Light | Dark | Verwendung |
|---|---|---|---|
| `--bg` | `#fdf6e3` | `#002b36` | Seitenhintergrund, Innenfläche der Bauteile, Schaltungsfläche |
| `--panel-bg` | `#eee8d5` | `#073642` | Karten, Buttons, Modal |
| `--text` | `#657b83` | `#839496` | Fliesstext, Untertitel |
| `--text-bold` | `#073642` | `#93a1a1` | Überschriften, Leitungen, Bauteil-Umrisse und -Beschriftungen |
| `--border` | `#93a1a1` | `#586e75` | Rahmen |
| `--primary` | `#268bd2` (Blau) | `#2aa198` (Cyan) | aktiver Tab, aktiver Button, Hover-Glow |
| `--accent-m` | `#859900` | `#859900` | Maschen (Grün/Oliv) |
| `--volt-color` | `#268bd2` | `#268bd2` | **Alle Spannungspfeile** (Blau) |
| `--curr-color` | `#cb4b16` | `#cb4b16` | **Alle Strompfeile** (Orange) |
| `--node-color` | = `--curr-color` | = `--curr-color` | Knotensatz-Block |
| `--eq-bg` | `#073642` | `#001e26` | Gleichungs-Karte (immer dunkel) |
| `--eq-text` | `#839496` | `#93a1a1` | Text in Gleichungen |
| `--eq-highlight` | `#b58900` | `#b58900` | Warnhinweise (Gelb, mit ⚠) |
| `--eq-result` | = `--primary` | = `--primary` | Ergebnisse |

Weitere fixe Farbe: `.eq-sub` (eingesetzte Gleichung, 2. Zeile) ist hart `#268bd2`.

**Farbsemantik, unbedingt beibehalten:** Spannung = Blau, Strom = Orange, Masche = Grün, Ergebnis = Primärfarbe, Warnung = Gelb. Leitungen und Bauteile sind in `--text-bold` gezeichnet.

**Theme:** Start im **Dark-Mode** (`<body data-theme="dark">`). Der Button „Dark / Light“ schaltet das Attribut `data-theme` am `<body>` um. Die Light-Werte stehen in `:root`, die Dark-Werte in `[data-theme="dark"]`. Farbwechsel mit `transition: 0.3s`.

### 3. Typografie
- UI: Systemschrift `-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif`.
- Gleichungen: Monospace `"Courier New", Courier, monospace`, `0.88rem`, Zeilenhöhe 1.4.
- `h1` 1.25rem, Untertitel 0.75rem, Buttons 0.85rem fett.
- SVG-Beschriftungen: 11px fett. Indizes als `<tspan dy="3" font-size="8|9">`, danach Rücksprung mit `dy="-3"`.
- Indizes im HTML immer mit `<sub>` (z.B. `U<sub>R1</sub>`, `I<sub>1</sub>`).

### 4. Seitenaufbau (von oben nach unten)
1. **Header**: links `h1` und Untertitel (`#sub-title`, Bedienhinweis), rechts Button-Gruppe (Optionen-Buttons, Button „Advanced: Aus/Ein“, Button „Dark / Light“). Unten eine 2px-Linie in `--border`.
2. **`.main-layout`**: zentriert, `max-width: 920px`, vertikal mit 10px Abstand:
   - **Tabs** (`.tabs` > `.tab-btn`) zur Wahl der Schaltungsvariante, Beschriftung im Stil „1. Mit 2 Spannungsquellen“. Der aktive Tab hat die Klasse `active`.
   - **Schaltungskarte** `.circuit-card` > `.circuit-view` (Höhe **310px**) > `<svg viewBox="0 0 640 310">`.
   - **Gleichungskarte** `.eq-card` > `.eq-grid#equation-display`: responsives Grid (`minmax(280px, 1fr)`) aus Blöcken.
3. **Modal** zum Bearbeiten von Werten (siehe 7).

### 5. Gleichungsblöcke
Jeder Block ist `.eq-block` plus eine Farbklasse mit **4px farbigem linkem Rand** und farbigem Titel:

| Block | Klassen | Titel-Beispiel |
|---|---|---|
| Masche | `eq-block-mesh` + `eq-title-mesh` | `Masche 1 (M<sub>1</sub> - ↻)` |
| Knoten | `eq-block-node` + `eq-title-node` | `Knotensatz (Oberer Knoten) [Zufluss +]` |
| Ergebnis | `eq-block-result` + `eq-title-result` | `Ergebnisse` |

Inhalt eines Maschenblocks:
1. Zeile: symbolische Gleichung, z.B. `- U₁ + U_R1 + U₂ + U_R2 = 0`.
2. Zeile (`.eq-sub`, blau): eingesetzte Form, z.B. `- U₁ + (R₁ · I₁) + U₂ + (R₂ · I₁) = 0`.

Gleichungen werden mit `formatSum(terms)` erzeugt (Terme `{name, sign}` → „a + b - c = 0“).

Ergebnisblock: Ströme fett in `.eq-res` (3 Nachkommastellen, A), darunter die Spannungen (1–2 Nachkommastellen, V), darunter eventuelle Warnungen (`warningHtml`). Nicht bestimmbare Werte erscheinen als **„—“** (`fmt()`).

Die Umlaufrichtung im Titel wird als ↻ (CW) bzw. ↺ (CCW) angezeigt.

### 6. Schaltplan-Zeichnung (SVG)
- Koordinatensystem 640 × 310. Rechteckige Masche typischerweise `y = 65` (oben) bis `y = 235` (unten), Mitte `y = 150`.
- **Leitungen**: Klasse `wire` (Strich `--text-bold`, 2px, ohne Füllung).
- **Knoten**: Kreis r = 4, gefüllt `--text-bold`.
- **Spannungsquelle** (`drawSource`): Kreis r = 16 mit senkrechter Linie durch die Mitte (europäisches Symbol), Füllung `--bg`. Beschriftung „U₁ (12V)“ **rechts** der Quelle (`textPos 'right'`, x + 22).
- **Widerstand** (`drawResistor`): Rechteck 40 × 20 (europäische Norm), Beschriftung „R₁ = 10Ω“ über oder unter dem Bauteil (y − 15 bzw. y + 24).
- **Quellen-Spannungspfeil** (`drawSourceVoltageArrow`): senkrecht, Länge 100 (y 100 → 200), **35px links der Quelle** (x = Quelle − 35), Label links daneben. Standardrichtung nach unten.
- **Widerstands-Spannungspfeil** (`drawVoltageArrow`): waagrecht, Länge 100, **40px ausserhalb** der Leitung (y = 25 oben bzw. 275 unten), Label darüber bzw. darunter. Zeigt immer **in Richtung des zugehörigen Strompfeils**.
- **Strompfeil** (`drawCurrentArrow`): kurz (30–40px) direkt **auf der Leitung**, Strichbreite 2.2, orange. Label „I₁“ darüber oder rechts davon.
- **Maschenpfeil** (`drawMeshArc`): Kreisbogen (r ≈ 28–32) in der Maschenmitte, grün, 2.5px, Pfeilspitze, Label „M₁“ in der Mitte. Klick wechselt CW ↔ CCW.
- **Pfeilspitzen**: SVG-Marker `arr-v` (blau), `arr-c` (orange), `arr-m` (grün), Dreieck 5 × 5 mit `orient="auto-start-reverse"`.
- Klickbare Elemente erhalten die Klasse `interactive-element` (Cursor als Zeiger, Hover als `drop-shadow` in `--primary`).
- Das SVG wird bei jeder Änderung **komplett neu gezeichnet** (`svg.innerHTML = ''`, dann `renderSVG()`).

### 7. Interaktion
- **Klick auf Quelle oder Widerstand** öffnet das Modal „U₁ Wert anpassen (V)“ mit Zahlenfeld (step 0.5) und synchronem Schieberegler. Bereich: Spannung 0–50 V, Widerstand 0–200 Ω. Jede Eingabe aktualisiert die Simulation **live**. Der Button „Fertig“ schliesst das Modal.
- **Klick auf einen Maschenbogen** schaltet die Umlaufrichtung um. Die Vorzeichen in der Gleichung passen sich an.
- **Advanced-Modus** (Button „Advanced: Aus/Ein“, aktiv mit Klasse `btn-active`): Erst dann sind auch Strompfeile und Quellen-Spannungspfeile klickbar und lassen sich umdrehen. Der Untertitel erklärt das. Beim Ausschalten werden alle Richtungen auf Standard zurückgesetzt.
- Optionale Konventions-Buttons (z.B. „Stromkonvention: Zufluss + / Abfluss -“) sind nur in Tabs sichtbar, in denen sie Sinn ergeben.
- Zentrale Update-Funktion: `updateSimulation()` ruft `renderSVG()` und `calculateAndDisplay()` auf.

### 8. Physikalische und mathematische Konventionen
- **Verbraucher-Zählpfeilsystem**: Spannungspfeil am Widerstand parallel zum Strompfeil, daher **U_R = R · I**, ohne zusätzlichen Richtungsfaktor.
- **Quellen-Spannungspfeil** zeigt von + nach −. Die Pfeilrichtung definiert die Polarität. Umdrehen im Advanced-Modus heisst Quelle umpolen (`sourceDirs = ±1`).
- **Maschensatz**: Spannungspfeil in Umlaufrichtung ergibt **+**, dagegen **−**. Summe = 0.
- **Knotensatz**: Zufluss + / Abfluss − (umschaltbar). Summe = 0.
- **Ergebnisse mit Vorzeichen**: Ein negativer Strom bedeutet, dass der Strom gegen den eingezeichneten Pfeil fliesst. Niemals Beträge anzeigen.
- Ströme werden **aus genau den angezeigten Gleichungen** aufgelöst. Angezeigte Gleichung und Berechnung müssen zusammenpassen: Setzt man die Ergebnisse ein, muss die Gleichung aufgehen.
- **0 Ω ist gültig.** Division durch 0 wird mit `solveBranch(dU, R)` abgefangen:
  - R = 0 und ΔU ≠ 0: Widerspruch (ideale Quellen kurzgeschlossen). Werte „—“, gelbe Warnung.
  - R = 0 und ΔU = 0: Strom unbestimmt. Betroffene Werte „—“, gelbe Warnung, übrige Werte normal.
  - U_R bei R = 0 ist 0 V (ausser im Widerspruchsfall).
- Negative Widerstände werden bei der Eingabe auf 0 Ω begrenzt.

### 9. Qualitätssicherung
- Für jede neue Simulation alle Kombinationen von Richtungen (Quellen, Ströme, Umlauf) mit Zufallswerten automatisiert gegen eine **unabhängige Lösung** prüfen (z.B. Knotenpotential-Verfahren, Playwright/Chromium headless) und zusätzlich die angezeigten Gleichungen mit den Ergebnissen nachrechnen.
- Sonderfälle testen: 0 Ω, gleiche Spannungen, 0 V.
- Ablauf der Zusammenarbeit: Änderung umsetzen, als Claude-Artifact veröffentlichen (immer gleicher Link, neue Version), der Benutzer prüft, danach Push auf `main` (GitHub Pages).

---

## Teil B – Code zum 1:1-Übernehmen

### B1. Vollständiges CSS
```css
:root {
  --bg: #fdf6e3;
  --panel-bg: #eee8d5;
  --text: #657b83;
  --text-bold: #073642;
  --border: #93a1a1;
  --primary: #268bd2; /* Solarized Blau (Tab-Buttons) */
  --accent-m: #859900; /* Grün/Oliv für Maschen */
  --volt-color: #268bd2;
  --curr-color: #cb4b16; /* Solarized Orange für Ströme */
  --node-color: var(--curr-color); /* Farbe für Knotensatz = Stromfarbe */
  --eq-bg: #073642;
  --eq-text: #839496;
  --eq-highlight: #b58900;
  --eq-result: var(--primary); /* Farbe für Ergebnisse = Tab-Button-Farbe */
}

[data-theme="dark"] {
  --bg: #002b36;
  --panel-bg: #073642;
  --text: #839496;
  --text-bold: #93a1a1;
  --border: #586e75;
  --primary: #2aa198;
  --accent-m: #859900;
  --volt-color: #268bd2;
  --curr-color: #cb4b16;
  --node-color: var(--curr-color);
  --eq-bg: #001e26;
  --eq-text: #93a1a1;
  --eq-highlight: #b58900;
  --eq-result: var(--primary);
}

body {
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
  margin: 0;
  padding: 12px;
  background-color: var(--bg);
  color: var(--text);
  transition: background-color 0.3s, color 0.3s;
}

header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 2px solid var(--border);
  padding-bottom: 6px;
  margin-bottom: 12px;
}

h1 {
  margin: 0;
  color: var(--text-bold);
  font-size: 1.25rem;
}

.header-controls {
  display: flex;
  align-items: center;
  gap: 8px;
}

.theme-toggle, .btn {
  background: var(--panel-bg);
  color: var(--text-bold);
  border: 1px solid var(--border);
  padding: 6px 12px;
  border-radius: 4px;
  cursor: pointer;
  font-weight: bold;
  font-size: 0.85rem;
}

.btn-active {
  background: var(--primary);
  color: #fff;
  border-color: var(--primary);
}

.tabs {
  display: flex;
  gap: 8px;
  margin-bottom: 10px;
}

.tab-btn {
  padding: 6px 14px;
  border: 1px solid var(--border);
  background: var(--panel-bg);
  color: var(--text-bold);
  border-radius: 4px;
  cursor: pointer;
  font-weight: bold;
  transition: all 0.2s ease;
}

.tab-btn.active {
  background: var(--primary);
  color: #ffffff;
  border-color: var(--primary);
}

.main-layout {
  display: flex;
  flex-direction: column;
  gap: 10px;
  max-width: 920px;
  margin: 0 auto;
}

.circuit-card {
  background: var(--panel-bg);
  border-radius: 6px;
  padding: 8px;
  border: 1px solid var(--border);
  position: relative;
}

.circuit-view {
  width: 100%;
  height: 310px;
  background: var(--bg);
  border-radius: 4px;
  border: 1px solid var(--border);
}

svg {
  width: 100%;
  height: 100%;
  user-select: none;
}

.interactive-element {
  cursor: pointer;
}
.interactive-element:hover {
  filter: drop-shadow(0 0 4px var(--primary));
}

.eq-card {
  background: var(--eq-bg);
  color: var(--eq-text);
  border-radius: 6px;
  padding: 10px 14px;
  font-family: "Courier New", Courier, monospace;
  font-size: 0.88rem;
  line-height: 1.4;
  box-shadow: inset 0 0 6px rgba(0,0,0,0.3);
}

.eq-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 12px;
}

.eq-block {
  background: rgba(255,255,255,0.03);
  padding: 8px;
  border-radius: 4px;
}

.eq-block-mesh {
  border-left: 4px solid var(--accent-m);
}

.eq-block-node {
  border-left: 4px solid var(--node-color);
}

.eq-block-result {
  border-left: 4px solid var(--eq-result);
}

.eq-title {
  font-weight: bold;
  margin-bottom: 4px;
}

.eq-title-mesh {
  color: var(--accent-m);
}

.eq-title-node {
  color: var(--node-color);
}

.eq-title-result {
  color: var(--eq-result);
}

.eq-sub {
  color: #268bd2;
  font-size: 0.82rem;
}

.eq-res {
  color: var(--eq-result);
  font-weight: bold;
}

.modal-overlay {
  position: fixed;
  top: 0; left: 0; right: 0; bottom: 0;
  background: rgba(0,0,0,0.5);
  display: none;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal {
  background: var(--panel-bg);
  border: 2px solid var(--border);
  border-radius: 8px;
  padding: 16px;
  width: 280px;
  color: var(--text-bold);
  box-shadow: 0 4px 12px rgba(0,0,0,0.3);
}

.modal h3 {
  margin: 0 0 12px 0;
  font-size: 1rem;
}

.modal-controls {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.modal-controls input[type="number"] {
  width: 80px;
  padding: 4px;
  border-radius: 4px;
  border: 1px solid var(--border);
}

.modal-actions {
  display: flex;
  justify-content: flex-end;
  margin-top: 14px;
}

.wire { stroke: var(--text-bold); stroke-width: 2; fill: none; }
.component-text { font-size: 11px; font-weight: bold; fill: var(--text-bold); }
```

### B2. HTML-Grundgerüst (Header, Layout, Modal)
Titel, Tabs und Options-Buttons an die neue Simulation anpassen. Struktur, IDs und Klassen bleiben gleich.
```html
<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Maschensatz Interactive Simulation - Solarized</title>
  <style>/* CSS aus B1 */</style>
</head>
<body data-theme="dark">

<header>
  <div>
    <h1>Maschensatz Simulation</h1>
    <div id="sub-title" style="font-size: 0.75rem; color: var(--text);">Klicken Sie auf Komponenten oder Maschen zum Anpassen</div>
  </div>
  <div class="header-controls">
    <button class="btn" id="btn-node-rule" onclick="toggleNodeRule()">Stromkonvention: Zufluss + / Abfluss -</button>
    <button class="btn" id="btn-adv" onclick="toggleAdvanced()">Advanced: Aus</button>
    <button class="theme-toggle" onclick="toggleTheme()">Dark / Light</button>
  </div>
</header>

<div class="main-layout">
  <div class="tabs">
    <button class="tab-btn active" id="btn-tab-1" onclick="selectCircuit(1)">1. Mit 2 Spannungsquellen</button>
    <button class="tab-btn" id="btn-tab-2" onclick="selectCircuit(2)">2. Mit 3 Spannungsquellen</button>
  </div>

  <div class="circuit-card">
    <div class="circuit-view">
      <svg id="circuit-svg" viewBox="0 0 640 310"></svg>
    </div>
  </div>

  <div class="eq-card">
    <div class="eq-grid" id="equation-display"></div>
  </div>
</div>

<div class="modal-overlay" id="edit-modal">
  <div class="modal">
    <h3 id="modal-title">Wert anpassen</h3>
    <div class="modal-controls">
      <div style="display:flex; justify-content:space-between; align-items:center;">
        <label for="modal-input">Wert:</label>
        <input type="number" id="modal-input" step="0.5" oninput="syncModalInput()">
      </div>
      <input type="range" id="modal-slider" min="1" max="100" step="0.5" oninput="syncModalSlider()">
    </div>
    <div class="modal-actions">
      <button class="btn" onclick="closeModal()">Fertig</button>
    </div>
  </div>
</div>
```

### B3. UI-Funktionen (Theme, Advanced, Tabs, Modal, Update)
```js
function toggleTheme() {
  const body = document.body;
  const next = body.getAttribute('data-theme') === 'light' ? 'dark' : 'light';
  body.setAttribute('data-theme', next);
}

function toggleNodeRule() {
  nodeInflowIsPositive = !nodeInflowIsPositive;
  const btn = document.getElementById('btn-node-rule');
  btn.innerText = nodeInflowIsPositive 
    ? 'Stromkonvention: Zufluss + / Abfluss -' 
    : 'Stromkonvention: Zufluss - / Abfluss +';
  updateSimulation();
}

function toggleAdvanced() {
  isAdvanced = !isAdvanced;
  const btn = document.getElementById('btn-adv');
  btn.innerText = `Advanced: ${isAdvanced ? 'Ein' : 'Aus'}`;
  btn.classList.toggle('btn-active', isAdvanced);
  
  document.getElementById('sub-title').innerText = isAdvanced 
    ? 'Advanced-Modus: Klicken Sie auch auf Ströme oder Spannungsquellen-Pfeile, um deren Richtung zu drehen'
    : 'Klicken Sie auf Komponenten oder Maschen zum Anpassen';
    
  if (!isAdvanced) {
    sourceDirs = { U1: 1, U2: 1, U3: 1 };
    currentDirs = { I1: 1, I2: 1, I3: 1 };
  }
  updateSimulation();
}

function selectCircuit(num) {
  activeCircuit = num;
  document.getElementById('btn-tab-1').classList.toggle('active', num === 1);
  document.getElementById('btn-tab-2').classList.toggle('active', num === 2);
  
  document.getElementById('btn-node-rule').style.display = (num === 1) ? 'none' : 'inline-block';
  
  updateSimulation();
}

function toggleMesh(mKey) {
  meshDir[mKey] = (meshDir[mKey] === 'CW') ? 'CCW' : 'CW';
  updateSimulation();
}

function toggleCurrentDir(iKey) {
  if (!isAdvanced) return;
  currentDirs[iKey] *= -1;
  updateSimulation();
}

function toggleSourceDir(uKey) {
  if (!isAdvanced) return;
  sourceDirs[uKey] *= -1;
  updateSimulation();
}

function openEditModal(key) {
  activeEditKey = key;
  const isVolt = key.startsWith('U');
  document.getElementById('modal-title').innerText = `${key} Wert anpassen (${isVolt ? 'V' : 'Ω'})`;
  
  const valInput = document.getElementById('modal-input');
  const valSlider = document.getElementById('modal-slider');
  
  valInput.value = values[key];
  valInput.min = isVolt ? '' : 0;
  valSlider.min = 0;
  valSlider.max = isVolt ? 50 : 200;
  valSlider.value = values[key];

  document.getElementById('edit-modal').style.display = 'flex';
}

function syncModalSlider() {
  const val = parseFloat(document.getElementById('modal-slider').value) || 0;
  document.getElementById('modal-input').value = val;
  values[activeEditKey] = val;
  updateSimulation();
}

function syncModalInput() {
  const input = document.getElementById('modal-input');
  let val = parseFloat(input.value) || 0;
  // Widerstände können nicht negativ sein (0 Ω = ideale Leitung ist erlaubt)
  if (activeEditKey.startsWith('R') && val < 0) { val = 0; input.value = 0; }
  document.getElementById('modal-slider').value = val;
  values[activeEditKey] = val;
  updateSimulation();
}

function closeModal() {
  document.getElementById('edit-modal').style.display = 'none';
}

function updateSimulation() {
  renderSVG();
  calculateAndDisplay();
}
```

### B4. Hilfsfunktionen für Gleichungen und Sonderfälle
```js
function formatSum(terms) {
  let str = "";
  terms.forEach((t, index) => {
    const s = t.sign >= 0 ? '+' : '-';
    if (index === 0) {
      str += (s === '-' ? '- ' : '') + t.name;
    } else {
      str += ` ${s} ${t.name}`;
    }
  });
  return str + " = 0";
}

// Zahl formatieren; nicht bestimmbare Werte (null / ±∞ / NaN) als "—"
function fmt(x, digits) {
  return (x === null || !Number.isFinite(x)) ? '—' : x.toFixed(digits);
}

// Löst R·I = ΔU für einen Zweig. Bei R = 0 gibt es zwei Sonderfälle:
//  ΔU ≠ 0 → Widerspruch (zwei ideale Quellen mit verschiedener Spannung kurzgeschlossen)
//  ΔU = 0 → Strom durch die Schaltung nicht festgelegt (jeder Wert erfüllt die Gleichung)
function solveBranch(dU, R) {
  if (R !== 0) return { I: dU / R, state: 'ok' };
  return { I: null, state: (dU === 0) ? 'undetermined' : 'contradiction' };
}

function warningHtml(msgs) {
  if (!msgs.length) return '';
  return msgs.map(m => `<div style="color: var(--eq-highlight); margin-top: 4px;">⚠ ${m}</div>`).join('');
}
```

### B5. SVG: Element-Helfer und Pfeil-Marker (Anfang von `renderSVG`)
```js
function renderSVG() {
  const svg = document.getElementById('circuit-svg');
  svg.innerHTML = '';

  const createEl = (tag, attrs) => {
    const el = document.createElementNS('http://www.w3.org/2000/svg', tag);
    for (let k in attrs) el.setAttribute(k, attrs[k]);
    return el;
  };

  const defs = createEl('defs');
  
  const mVolt = createEl('marker', { id: 'arr-v', viewBox: '0 0 10 10', refX: '6', refY: '5', markerWidth: '5', markerHeight: '5', orient: 'auto-start-reverse' });
  mVolt.appendChild(createEl('path', { d: 'M 0 0 L 10 5 L 0 10 z', fill: 'var(--volt-color)' }));
  defs.appendChild(mVolt);

  const mCurr = createEl('marker', { id: 'arr-c', viewBox: '0 0 10 10', refX: '6', refY: '5', markerWidth: '5', markerHeight: '5', orient: 'auto-start-reverse' });
  mCurr.appendChild(createEl('path', { d: 'M 0 0 L 10 5 L 0 10 z', fill: 'var(--curr-color)' }));
  defs.appendChild(mCurr);

  const mMesh = createEl('marker', { id: 'arr-m', viewBox: '0 0 10 10', refX: '6', refY: '5', markerWidth: '5', markerHeight: '5', orient: 'auto-start-reverse' });
  mMesh.appendChild(createEl('path', { d: 'M 0 0 L 10 5 L 0 10 z', fill: 'var(--accent-m)' }));
  defs.appendChild(mMesh);

  svg.appendChild(defs);
  // … danach schaltungsspezifisch zeichnen (Leitungen als <rect>/<line class="wire">, Knoten, Bauteile, Pfeile)
}
```

### B6. SVG-Zeichenfunktionen für Bauteile und Pfeile
```js
function drawSource(svg, cx, cy, name, val, textPos = 'left') {
  const g = document.createElementNS('http://www.w3.org/2000/svg', 'g');
  g.setAttribute('class', 'interactive-element');
  g.setAttribute('onclick', `openEditModal('${name}')`);

  const circle = document.createElementNS('http://www.w3.org/2000/svg', 'circle');
  circle.setAttribute('cx', cx); circle.setAttribute('cy', cy); circle.setAttribute('r', 16);
  circle.setAttribute('stroke', 'var(--text-bold)'); circle.setAttribute('stroke-width', '2');
  circle.setAttribute('fill', 'var(--bg)');
  g.appendChild(circle);

  const line = document.createElementNS('http://www.w3.org/2000/svg', 'line');
  line.setAttribute('x1', cx); line.setAttribute('y1', cy - 16);
  line.setAttribute('x2', cx); line.setAttribute('y2', cy + 16);
  line.setAttribute('stroke', 'var(--text-bold)'); line.setAttribute('stroke-width', '1.8');
  g.appendChild(line);

  const txt = document.createElementNS('http://www.w3.org/2000/svg', 'text');
  
  if (textPos === 'right') {
    txt.setAttribute('x', cx + 22);
    txt.setAttribute('text-anchor', 'start');
  } else {
    txt.setAttribute('x', cx - 22);
    txt.setAttribute('text-anchor', 'end');
  }
  
  txt.setAttribute('y', cy + 4);
  txt.setAttribute('class', 'component-text');
  
  txt.innerHTML = `${name[0]}<tspan dy="3" font-size="9">${name[1]}</tspan><tspan dy="-3" font-size="11"> (${val}V)</tspan>`;
  g.appendChild(txt);

  svg.appendChild(g);
}

function drawResistor(svg, cx, cy, name, val, textPos) {
  const g = document.createElementNS('http://www.w3.org/2000/svg', 'g');
  g.setAttribute('class', 'interactive-element');
  g.setAttribute('onclick', `openEditModal('${name}')`);

  const rect = document.createElementNS('http://www.w3.org/2000/svg', 'rect');
  rect.setAttribute('x', cx - 20); rect.setAttribute('y', cy - 10);
  rect.setAttribute('width', 40); rect.setAttribute('height', 20);
  rect.setAttribute('stroke', 'var(--text-bold)'); rect.setAttribute('stroke-width', '2');
  rect.setAttribute('fill', 'var(--bg)');
  g.appendChild(rect);

  const txt = document.createElementNS('http://www.w3.org/2000/svg', 'text');
  txt.setAttribute('x', cx);
  if (textPos === 'top') {
    txt.setAttribute('y', cy - 15);
  } else {
    txt.setAttribute('y', cy + 24);
  }
  txt.setAttribute('text-anchor', 'middle');
  txt.setAttribute('class', 'component-text');
  txt.innerHTML = `${name[0]}<tspan dy="3" font-size="9">${name[1]}</tspan><tspan dy="-3" font-size="11"> = ${val}Ω</tspan>`;
  g.appendChild(txt);

  svg.appendChild(g);
}

function drawMeshArc(svg, cx, cy, r, dir, label, mKey) {
  const g = document.createElementNS('http://www.w3.org/2000/svg', 'g');
  g.setAttribute('class', 'interactive-element');
  g.setAttribute('onclick', `toggleMesh('${mKey}')`);

  const path = document.createElementNS('http://www.w3.org/2000/svg', 'path');
  
  let d = "";
  if (dir === 'CW') {
    d = `M ${cx - r} ${cy - 5} A ${r} ${r} 0 1 1 ${cx + 5} ${cy + r}`;
  } else {
    d = `M ${cx + r} ${cy - 5} A ${r} ${r} 0 1 0 ${cx - 5} ${cy + r}`;
  }

  path.setAttribute('d', d);
  path.setAttribute('fill', 'none');
  path.setAttribute('stroke', 'var(--accent-m)');
  path.setAttribute('stroke-width', '2.5');
  path.setAttribute('marker-end', 'url(#arr-m)');
  g.appendChild(path);

  const txt = document.createElementNS('http://www.w3.org/2000/svg', 'text');
  txt.setAttribute('x', cx); txt.setAttribute('y', cy + 4);
  txt.setAttribute('text-anchor', 'middle');
  txt.setAttribute('font-weight', 'bold');
  txt.setAttribute('fill', 'var(--accent-m)');
  txt.innerHTML = `M<tspan dy="3" font-size="9">${label.length > 1 ? label[1] : ''}</tspan>`;
  g.appendChild(txt);

  svg.appendChild(g);
}

function drawSourceVoltageArrow(svg, x1, y1, x2, y2, label, pos, dir) {
  const g = document.createElementNS('http://www.w3.org/2000/svg', 'g');
  if (isAdvanced) {
    g.setAttribute('class', 'interactive-element');
    g.setAttribute('onclick', `toggleSourceDir('${label}')`);
  }

  const startX = dir === 1 ? x1 : x2;
  const startY = dir === 1 ? y1 : y2;
  const endX = dir === 1 ? x2 : x1;
  const endY = dir === 1 ? y2 : y1;

  const path = document.createElementNS('http://www.w3.org/2000/svg', 'path');
  path.setAttribute('d', `M ${startX} ${startY} L ${endX} ${endY}`);
  path.setAttribute('stroke', 'var(--volt-color)');
  path.setAttribute('stroke-width', '1.8');
  path.setAttribute('marker-end', 'url(#arr-v)');
  g.appendChild(path);

  const txt = document.createElementNS('http://www.w3.org/2000/svg', 'text');
  let tx = (x1 + x2) / 2;
  let ty = (y1 + y2) / 2;

  if (pos === 'left') { tx -= 10; txt.setAttribute('text-anchor', 'end'); ty += 4; }
  else if (pos === 'right') { tx += 10; txt.setAttribute('text-anchor', 'start'); ty += 4; }

  txt.setAttribute('x', tx);
  txt.setAttribute('y', ty);
  txt.setAttribute('font-size', '11');
  txt.setAttribute('fill', 'var(--volt-color)');
  txt.setAttribute('font-weight', 'bold');
  
  txt.innerHTML = `U<tspan dy="3" font-size="8">${label[1]}</tspan>`;
  g.appendChild(txt);
  svg.appendChild(g);
}

function drawVoltageArrow(svg, x1, y1, x2, y2, label, pos) {
  const g = document.createElementNS('http://www.w3.org/2000/svg', 'g');

  const path = document.createElementNS('http://www.w3.org/2000/svg', 'path');
  path.setAttribute('d', `M ${x1} ${y1} L ${x2} ${y2}`);
  path.setAttribute('stroke', 'var(--volt-color)');
  path.setAttribute('stroke-width', '1.8');
  path.setAttribute('marker-end', 'url(#arr-v)');
  g.appendChild(path);

  const txt = document.createElementNS('http://www.w3.org/2000/svg', 'text');
  let tx = (x1 + x2) / 2;
  let ty = (y1 + y2) / 2;

  if (pos === 'top') { ty -= 8; txt.setAttribute('text-anchor', 'middle'); }
  else if (pos === 'bottom') { ty += 16; txt.setAttribute('text-anchor', 'middle'); }

  txt.setAttribute('x', tx);
  txt.setAttribute('y', ty);
  txt.setAttribute('font-size', '11');
  txt.setAttribute('fill', 'var(--volt-color)');
  txt.setAttribute('font-weight', 'bold');
  
  txt.innerHTML = `U<tspan dy="3" font-size="8">${label.substring(1)}</tspan>`;
  g.appendChild(txt);
  svg.appendChild(g);
}

function drawCurrentArrow(svg, x1, y1, x2, y2, label, pos) {
  const g = document.createElementNS('http://www.w3.org/2000/svg', 'g');
  if (isAdvanced) {
    g.setAttribute('class', 'interactive-element');
    g.setAttribute('onclick', `toggleCurrentDir('${label}')`);
  }

  const line = document.createElementNS('http://www.w3.org/2000/svg', 'line');
  line.setAttribute('x1', x1); line.setAttribute('y1', y1);
  line.setAttribute('x2', x2); line.setAttribute('y2', y2);
  line.setAttribute('stroke', 'var(--curr-color)');
  line.setAttribute('stroke-width', '2.2');
  line.setAttribute('marker-end', 'url(#arr-c)');
  g.appendChild(line);

  const txt = document.createElementNS('http://www.w3.org/2000/svg', 'text');
  let tx = (x1 + x2) / 2;
  let ty = (y1 + y2) / 2;

  if (pos === 'top') ty -= 7;
  else if (pos === 'bottom') ty += 15;
  else if (pos === 'right') tx += 12;

  txt.setAttribute('x', tx);
  txt.setAttribute('y', ty);
  txt.setAttribute('text-anchor', 'middle');
  txt.setAttribute('font-size', '11');
  txt.setAttribute('fill', 'var(--curr-color)');
  txt.setAttribute('font-weight', 'bold');
  txt.innerHTML = `I<tspan dy="3" font-size="8">${label[1]}</tspan>`;
  g.appendChild(txt);

  svg.appendChild(g);
}
```

### B7. Beispiel: Platzierung in Schaltung 1 (Referenz für Abstände)
```js
svg.appendChild(createEl('rect', { x: 140, y: 65, width: 360, height: 170, class: 'wire' }));

drawSource(svg, 140, 150, 'U1', U1, 'right');
drawSource(svg, 500, 150, 'U2', U2, 'right');
drawResistor(svg, 320, 65, 'R1', R1, 'top');
drawResistor(svg, 320, 235, 'R2', R2, 'bottom');

drawMeshArc(svg, 320, 150, 32, meshDir.M1, 'M1', 'M1');

drawSourceVoltageArrow(svg, 105, 100, 105, 200, 'U1', 'left', sourceDirs.U1);
drawSourceVoltageArrow(svg, 465, 100, 465, 200, 'U2', 'left', sourceDirs.U2);

if (currentDirs.I1 === 1) {
  drawVoltageArrow(svg, 270, 25, 370, 25, 'UR1', 'top');
  drawVoltageArrow(svg, 370, 275, 270, 275, 'UR2', 'bottom');
  drawCurrentArrow(svg, 390, 65, 430, 65, 'I1', 'top');
} else {
  drawVoltageArrow(svg, 370, 25, 270, 25, 'UR1', 'top');
  drawVoltageArrow(svg, 270, 275, 370, 275, 'UR2', 'bottom');
  drawCurrentArrow(svg, 430, 65, 390, 65, 'I1', 'top');
}

```
