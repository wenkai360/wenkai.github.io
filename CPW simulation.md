---
layout: null
title: Interactive Gapped CPW Simulator
---

<script src="https://cdn.plot.ly/plotly-2.35.2.min.js"></script>

<style>

    :root {
      --bg: #f6f7fb;
      --panel: #ffffff;
      --text: #1f2937;
      --muted: #6b7280;
      --border: #d1d5db;
      --accent: #2563eb;
      --shadow: 0 8px 24px rgba(15, 23, 42, 0.08);
    }
    body {
      margin: 0;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
      background: var(--bg);
      color: var(--text);
    }
    header {
      padding: 18px 22px;
      background: #111827;
      color: white;
    }
    header h1 {
      margin: 0 0 6px 0;
      font-size: 22px;
      font-weight: 700;
    }
    header p {
      margin: 0;
      color: #d1d5db;
      font-size: 14px;
    }
    main {
      display: grid;
      grid-template-columns: 360px minmax(0, 1fr);
      gap: 16px;
      padding: 16px;
      box-sizing: border-box;
    }
    .controls, .plots, .readout {
      background: var(--panel);
      border: 1px solid var(--border);
      border-radius: 14px;
      box-shadow: var(--shadow);
    }
    .controls {
      padding: 14px;
      height: calc(100vh - 120px);
      overflow-y: auto;
    }
    details {
      border: 1px solid var(--border);
      border-radius: 10px;
      margin-bottom: 10px;
      background: #fff;
      overflow: hidden;
    }
    summary {
      cursor: pointer;
      padding: 10px 12px;
      font-weight: 700;
      background: #f9fafb;
      user-select: none;
    }
    .control {
      padding: 10px 12px 12px 12px;
      border-top: 1px solid #eef0f4;
    }
    .row {
      display: flex;
      align-items: center;
      justify-content: space-between;
      gap: 10px;
      margin-bottom: 6px;
    }
    label {
      font-size: 13px;
      line-height: 1.2;
    }
    output {
      font-family: ui-monospace, SFMono-Regular, Menlo, Consolas, monospace;
      color: var(--accent);
      font-size: 12px;
      white-space: nowrap;
    }
    input[type="range"] {
      width: 100%;
    }
    input[type="number"], select {
      width: 100%;
      box-sizing: border-box;
      padding: 7px 8px;
      border: 1px solid var(--border);
      border-radius: 8px;
      font-size: 13px;
      background: white;
    }
    button {
      width: 100%;
      padding: 10px 12px;
      margin: 6px 0 12px 0;
      border: none;
      border-radius: 10px;
      color: white;
      background: var(--accent);
      font-weight: 700;
      cursor: pointer;
    }
    button.secondary {
      background: #4b5563;
    }
    .plots {
      padding: 10px;
      min-width: 0;
    }
    .grid {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 10px;
    }
    .plot {
      height: calc((100vh - 168px) / 2);
      min-height: 285px;
      border: 1px solid #edf0f5;
      border-radius: 10px;
    }
    .readout {
      margin-top: 10px;
      padding: 10px 12px;
      font-family: ui-monospace, SFMono-Regular, Menlo, Consolas, monospace;
      font-size: 12px;
      color: #111827;
      overflow-x: auto;
      white-space: pre-wrap;
    }
    .hint {
      color: var(--muted);
      font-size: 12px;
      line-height: 1.4;
      padding: 8px 12px 10px 12px;
    }
    @media (max-width: 1100px) {
      main { grid-template-columns: 1fr; }
      .controls { height: auto; max-height: none; }
      .grid { grid-template-columns: 1fr; }
      .plot { height: 380px; }
    }
  
</style>


  <header>
    <h1>Interactive hBN/TMD/hBN Gapped CPW Simulator</h1>
    <p>Adjust CPW, hBN gap, TMD inductance/capacitance/resistance, and transition frequency in real time. Watch S11 / S21 / phase / group delay / effective impedance update instantly.</p>
  </header>

  <main>
    <section class="controls">
      <button onclick="updateAll()">Recalculate / Update</button>
      <button class="secondary" onclick="resetDefaults()">Reset defaults</button>

      <details open>
        <summary>Model selection</summary>
        <div class="control">
          <label>Sandwich topology</label>
          <select id="topology" onchange="updateAll()">
            <option value="series" selected>Series gap inserted in CPW</option>
            <option value="shunt">Shunt sandwich branch to ground</option>
          </select>
          <div class="hint">Series corresponds to Ms = series_impedance_abcd(Zgap) in the current script. Shunt can be used to compare with a grounded sandwich resonator.</div>
        </div>
        <div class="control">
          <label>Frequency-state convention</label>
          <select id="stateConvention" onchange="updateAll()">
            <option value="scBelow" selected>f &lt; fc: superconducting state; f ≥ fc: normal state</option>
            <option value="scAbove">f &lt; fc: normal state; f ≥ fc: superconducting state</option>
          </select>
        </div>
      </details>

      <details open>
        <summary>Frequency sweep</summary>
        <div id="freqControls"></div>
      </details>

      <details open>
        <summary>CPW / Port parameters</summary>
        <div id="cpwControls"></div>
      </details>

      <details open>
        <summary>hBN / Gap capacitance</summary>
        <div class="hint">You can adjust Chbn1/Chbn2 directly, or enable automatic calculation from geometry.</div>
        <div class="control">
          <label><input type="checkbox" id="useGeometry" onchange="toggleGeometry(); updateAll();"> Calculate Chbn from geometry</label>
        </div>
        <div id="gapControls"></div>
        <div id="geometryControls"></div>
      </details>

      <details open>
        <summary>TMD material parameters</summary>
        <div id="tmdControls"></div>
      </details>

      <details>
        <summary>Parasitic capacitances</summary>
        <div id="parasiticControls"></div>
      </details>

      <details open>
        <summary>Readout</summary>
        <div class="control">
          <label>Readout frequency f_readout (GHz)</label>
          <input type="number" id="fReadout" min="0.001" max="300" step="0.1" value="40" onchange="updateAll()" />
        </div>
      </details>
    </section>

    <section>
      <div class="plots">
        <div class="grid">
          <div id="magPlot" class="plot"></div>
          <div id="phasePlot" class="plot"></div>
          <div id="delayPlot" class="plot"></div>
          <div id="zPlot" class="plot"></div>
        </div>
      </div>
      <div id="readout" class="readout"></div>
    </section>
  </main>

<script>
const eps0 = 8.854187817e-12;
const c0 = 299792458;

const defaults = {
  fStartGHz: {label: 'f_start (GHz)', value: 0.001, min: 0.001, max: 10, step: 0.001, group: 'freqControls'},
  fStopGHz: {label: 'f_stop (GHz)', value: 100, min: 1, max: 300, step: 1, group: 'freqControls'},
  nPts: {label: 'N points', value: 2000, min: 300, max: 6000, step: 100, group: 'freqControls'},
  Z0: {label: 'Z0 VNA reference impedance (Ω)', value: 50, min: 1, max: 200, step: 1, group: 'cpwControls'},
  Zc: {label: 'Zc CPW characteristic impedance (Ω)', value: 50, min: 1, max: 200, step: 1, group: 'cpwControls'},
  epsEff: {label: 'eps_eff', value: 6, min: 1, max: 20, step: 0.1, group: 'cpwControls'},
  alpha: {label: 'alpha attenuation constant (Np/m)', value: 0, min: 0, max: 50, step: 0.1, group: 'cpwControls'},
  l1mm: {label: 'Left CPW length l1 (mm)', value: 4, min: 0, max: 30, step: 0.1, group: 'cpwControls'},
  l2mm: {label: 'Right CPW length l2 (mm)', value: 4, min: 0, max: 30, step: 0.1, group: 'cpwControls'},
  Chbn1fF: {label: 'Chbn1 / top gap capacitance (fF)', value: 584.38, min: 1, max: 5000, step: 1, group: 'gapControls'},
  Chbn2fF: {label: 'Chbn2 / bottom gap capacitance (fF)', value: 584.38, min: 1, max: 5000, step: 1, group: 'gapControls'},
  epsHbn: {label: 'eps_hBN_perp', value: 3.3, min: 1, max: 10, step: 0.05, group: 'geometryControls'},
  Wum: {label: 'W_overlap (µm)', value: 10, min: 0.1, max: 100, step: 0.1, group: 'geometryControls'},
  Lum: {label: 'L_overlap (µm)', value: 10, min: 0.1, max: 100, step: 0.1, group: 'geometryControls'},
  d1nm: {label: 'Top hBN thickness d1 (nm)', value: 5, min: 0.5, max: 100, step: 0.1, group: 'geometryControls'},
  d2nm: {label: 'Bottom hBN thickness d2 (nm)', value: 5, min: 0.5, max: 100, step: 0.1, group: 'geometryControls'},
  fcGHz: {label: 'fc transition frequency (GHz)', value: 10, min: 0.1, max: 300, step: 0.1, group: 'tmdControls'},
  Rs: {label: 'Rs superconducting residual resistance (Ω)', value: 1, min: 0.001, max: 100, step: 0.1, group: 'tmdControls'},
  Rn: {label: 'Rn normal-state resistance (Ω)', value: 1000, min: 1, max: 10000, step: 10, group: 'tmdControls'},
  Lg_pH: {label: 'Lg geometric inductance (pH)', value: 1, min: 0, max: 500, step: 1, group: 'tmdControls'},
  Lk_pH: {label: 'Lk kinetic inductance (pH)', value: 50, min: 0, max: 2000, step: 1, group: 'tmdControls'},
  Cg_fF: {label: 'Cg TMD capacitance (fF)', value: 20, min: 0, max: 5000, step: 1, group: 'tmdControls'},
  Cp1fF: {label: 'Cp1 left parasitic capacitance to ground (fF)', value: 0, min: 0, max: 1000, step: 1, group: 'parasiticControls'},
  Cp2fF: {label: 'Cp2 right parasitic capacitance to ground (fF)', value: 0, min: 0, max: 1000, step: 1, group: 'parasiticControls'}
};

function makeControl(id, cfg) {
  const wrap = document.createElement('div');
  wrap.className = 'control';
  wrap.innerHTML = `
    <div class="row"><label for="${id}">${cfg.label}</label><output id="${id}_out"></output></div>
    <input type="range" id="${id}" min="${cfg.min}" max="${cfg.max}" step="${cfg.step}" value="${cfg.value}">
  `;
  document.getElementById(cfg.group).appendChild(wrap);
  const input = document.getElementById(id);
  input.addEventListener('input', () => { updateOutput(id); debouncedUpdate(); });
  updateOutput(id);
}

for (const [id, cfg] of Object.entries(defaults)) makeControl(id, cfg);

toggleGeometry();

let debounceTimer = null;
function debouncedUpdate() {
  clearTimeout(debounceTimer);
  debounceTimer = setTimeout(updateAll, 120);
}

function updateOutput(id) {
  const el = document.getElementById(id);
  const out = document.getElementById(id + '_out');
  out.value = Number(el.value).toPrecision(5);
}

function val(id) { return Number(document.getElementById(id).value); }

function resetDefaults() {
  for (const [id, cfg] of Object.entries(defaults)) {
    document.getElementById(id).value = cfg.value;
    updateOutput(id);
  }
  document.getElementById('topology').value = 'series';
  document.getElementById('stateConvention').value = 'scBelow';
  document.getElementById('useGeometry').checked = false;
  document.getElementById('fReadout').value = 40;
  toggleGeometry();
  updateAll();
}

function toggleGeometry() {
  const useGeom = document.getElementById('useGeometry').checked;
  for (const id of ['Chbn1fF', 'Chbn2fF']) {
    document.getElementById(id).disabled = useGeom;
  }
}

function linspace(start, stop, n) {
  const arr = new Array(n);
  const step = (stop - start) / (n - 1);
  for (let i = 0; i < n; i++) arr[i] = start + step * i;
  return arr;
}

function cadd(a,b){return [a[0]+b[0], a[1]+b[1]];}
function csub(a,b){return [a[0]-b[0], a[1]-b[1]];}
function cmul(a,b){return [a[0]*b[0]-a[1]*b[1], a[0]*b[1]+a[1]*b[0]];}
function cdiv(a,b){const d=b[0]*b[0]+b[1]*b[1]; return [(a[0]*b[0]+a[1]*b[1])/d, (a[1]*b[0]-a[0]*b[1])/d];}
function cscale(a,s){return [a[0]*s, a[1]*s];}
function cinv(a){const d=a[0]*a[0]+a[1]*a[1]; return [a[0]/d, -a[1]/d];}
function cabs(a){return Math.hypot(a[0], a[1]);}
function carg(a){return Math.atan2(a[1], a[0]);}
function csinh(a){return [Math.sinh(a[0]) * Math.cos(a[1]), Math.cosh(a[0]) * Math.sin(a[1])];}
function ccosh(a){return [Math.cosh(a[0]) * Math.cos(a[1]), Math.sinh(a[0]) * Math.sin(a[1])];}

function matmul(M, N) {
  return [
    [cadd(cmul(M[0][0], N[0][0]), cmul(M[0][1], N[1][0])), cadd(cmul(M[0][0], N[0][1]), cmul(M[0][1], N[1][1]))],
    [cadd(cmul(M[1][0], N[0][0]), cmul(M[1][1], N[1][0])), cadd(cmul(M[1][0], N[0][1]), cmul(M[1][1], N[1][1]))]
  ];
}

function transmissionLine(gamma, Zc, length) {
  const gl = cscale(gamma, length);
  const ch = ccosh(gl);
  const sh = csinh(gl);
  return [[ch, cscale(sh, Zc)], [cscale(sh, 1/Zc), ch]];
}
function seriesZ(Z) { return [[[1,0], Z], [[0,0], [1,0]]]; }
function shuntY(Y) { return [[[1,0], [0,0]], [Y, [1,0]]]; }

function unwrapPhase(ph) {
  const out = new Array(ph.length);
  out[0] = ph[0];
  let offset = 0;
  for (let i = 1; i < ph.length; i++) {
    let dp = ph[i] - ph[i-1];
    if (dp > Math.PI) offset -= 2*Math.PI;
    if (dp < -Math.PI) offset += 2*Math.PI;
    out[i] = ph[i] + offset;
  }
  return out;
}

function gradient(y, x) {
  const n = y.length;
  const g = new Array(n);
  g[0] = (y[1] - y[0]) / (x[1] - x[0]);
  for (let i = 1; i < n - 1; i++) g[i] = (y[i+1] - y[i-1]) / (x[i+1] - x[i-1]);
  g[n-1] = (y[n-1] - y[n-2]) / (x[n-1] - x[n-2]);
  return g;
}

function calculate() {
  const nPts = Math.round(val('nPts'));
  const f = linspace(val('fStartGHz')*1e9, val('fStopGHz')*1e9, nPts);
  const w = f.map(x => 2*Math.PI*x);
  const Z0 = val('Z0'), Zc = val('Zc'), epsEff = val('epsEff'), alpha = val('alpha');
  const l1 = val('l1mm')*1e-3, l2 = val('l2mm')*1e-3;
  const fc = val('fcGHz')*1e9;
  const Rs = val('Rs'), Rn = val('Rn');
  const Lg = val('Lg_pH')*1e-12, Lk = val('Lk_pH')*1e-12, Cg = val('Cg_fF')*1e-15;
  const Cp1 = val('Cp1fF')*1e-15, Cp2 = val('Cp2fF')*1e-15;
  const useGeom = document.getElementById('useGeometry').checked;
  let Chbn1, Chbn2;
  if (useGeom) {
    const A = val('Wum')*1e-6 * val('Lum')*1e-6;
    Chbn1 = eps0 * val('epsHbn') * A / (val('d1nm')*1e-9);
    Chbn2 = eps0 * val('epsHbn') * A / (val('d2nm')*1e-9);
  } else {
    Chbn1 = val('Chbn1fF')*1e-15;
    Chbn2 = val('Chbn2fF')*1e-15;
  }
  const topology = document.getElementById('topology').value;
  const scBelow = document.getElementById('stateConvention').value === 'scBelow';

  const S11 = [], S21 = [], Zgap = [], Ysand = [], Ytmd = [];
  for (let i = 0; i < nPts; i++) {
    const fi = f[i], wi = w[i];
    const Zhbn1 = [0, -1/(wi*Chbn1)];
    const Zhbn2 = [0, -1/(wi*Chbn2)];
    const isSC = scBelow ? (fi < fc) : (fi >= fc);
    let Yt;
    if (isSC) {
      const Zrl = [Rs, wi*(Lg+Lk)];
      Yt = cadd(cinv(Zrl), [0, wi*Cg]);
    } else {
      const Zrl = [Rn, wi*Lg];
      Yt = cadd(cinv(Zrl), [0, wi*Cg]);
    }
    const Zt = cinv(Yt);
    const Zs = cadd(cadd(Zhbn1, Zt), Zhbn2);
    const Ys = cinv(Zs);
    Zgap.push(Zs); Ysand.push(Ys); Ytmd.push(Yt);

    const beta = wi * Math.sqrt(epsEff) / c0;
    const gamma = [alpha, beta];
    const Mtl1 = transmissionLine(gamma, Zc, l1);
    const Mtl2 = transmissionLine(gamma, Zc, l2);
    const Mp1 = shuntY([0, wi*Cp1]);
    const Mp2 = shuntY([0, wi*Cp2]);
    const Mcenter = topology === 'series' ? seriesZ(Zs) : shuntY(Ys);
    let M = matmul(matmul(matmul(matmul(Mtl1, Mp1), Mcenter), Mp2), Mtl2);
    const A = M[0][0], B = M[0][1], C = M[1][0], D = M[1][1];
    const den = cadd(cadd(A, cscale(B, 1/Z0)), cadd(cscale(C, Z0), D));
    const s11num = csub(cadd(A, cscale(B, 1/Z0)), cadd(cscale(C, Z0), D));
    S11.push(cdiv(s11num, den));
    S21.push(cdiv([2,0], den));
  }

  const phaseS11 = unwrapPhase(S11.map(carg));
  const phaseS21 = unwrapPhase(S21.map(carg));
  const gd = gradient(phaseS21, w).map(x => -x);
  return {f, w, S11, S21, Zgap, Ysand, Ytmd, phaseS11, phaseS21, gd, params: {Chbn1, Chbn2, fc, Lg, Lk, Cg}};
}

function updateAll() {
  const data = calculate();
  const fGHz = data.f.map(x => x/1e9);
  const S21dB = data.S21.map(z => 20*Math.log10(Math.max(cabs(z), 1e-300)));
  const S11dB = data.S11.map(z => 20*Math.log10(Math.max(cabs(z), 1e-300)));
  const phaseS21deg = data.phaseS21.map(x => x*180/Math.PI);
  const phaseS11deg = data.phaseS11.map(x => x*180/Math.PI);
  const gdPs = data.gd.map(x => x*1e12);
  const zRe = data.Zgap.map(z => z[0]);
  const zIm = data.Zgap.map(z => z[1]);
  const yMag = data.Ysand.map(y => cabs(y));
  const fcGHz = data.params.fc/1e9;

  const commonLayout = (title, ytitle) => ({
    title: {text: title, font: {size: 14}},
    margin: {l: 62, r: 18, t: 42, b: 48},
    xaxis: {title: 'Frequency (GHz)', showgrid: true},
    yaxis: {title: ytitle, showgrid: true},
    legend: {orientation: 'h', x: 0, y: 1.12},
    shapes: [{type: 'line', x0: fcGHz, x1: fcGHz, y0: 0, y1: 1, yref: 'paper', line: {dash: 'dash', width: 1}}]
  });
  const cfg = {responsive: true, displaylogo: false};

  Plotly.react('magPlot', [
    {x: fGHz, y: S21dB, name: 'S21', type: 'scatter', mode: 'lines'},
    {x: fGHz, y: S11dB, name: 'S11', type: 'scatter', mode: 'lines'}
  ], commonLayout('(a) Magnitude response', 'Magnitude (dB)'), cfg);

  Plotly.react('phasePlot', [
    {x: fGHz, y: phaseS21deg, name: '∠S21', type: 'scatter', mode: 'lines'},
    {x: fGHz, y: phaseS11deg, name: '∠S11', type: 'scatter', mode: 'lines'}
  ], commonLayout('(b) Phase response', 'Phase (degrees)'), cfg);

  Plotly.react('delayPlot', [
    {x: fGHz, y: gdPs, name: 'Group delay', type: 'scatter', mode: 'lines'}
  ], commonLayout('(c) Group delay', 'Group delay (ps)'), cfg);

  Plotly.react('zPlot', [
    {x: fGHz, y: zRe, name: 'Re(Zgap)', type: 'scatter', mode: 'lines'},
    {x: fGHz, y: zIm, name: 'Im(Zgap)', type: 'scatter', mode: 'lines'},
    {x: fGHz, y: yMag, name: '|Ysand| (S)', type: 'scatter', mode: 'lines', yaxis: 'y2'}
  ], {
    ...commonLayout('(d) Effective gap/sandwich response', 'Impedance (Ω)'),
    yaxis2: {title: '|Ysand| (S)', overlaying: 'y', side: 'right', showgrid: false},
    margin: {l: 62, r: 62, t: 42, b: 48}
  }, cfg);

  updateReadout(data, S11dB, S21dB, gdPs);
}

function updateReadout(data, S11dB, S21dB, gdPs) {
  const fRead = Number(document.getElementById('fReadout').value)*1e9;
  let idx = 0, best = Infinity;
  for (let i = 0; i < data.f.length; i++) {
    const d = Math.abs(data.f[i] - fRead);
    if (d < best) { best = d; idx = i; }
  }
  const Ch1 = data.params.Chbn1*1e15;
  const Ch2 = data.params.Chbn2*1e15;
  const Cseries = 1/(1/data.params.Chbn1 + 1/data.params.Chbn2);
  const Lsc = data.params.Lg + data.params.Lk;
  const fLC_hbn = Lsc > 0 && Cseries > 0 ? 1/(2*Math.PI*Math.sqrt(Lsc*Cseries))/1e9 : NaN;
  const fLC_tmd = Lsc > 0 && data.params.Cg > 0 ? 1/(2*Math.PI*Math.sqrt(Lsc*data.params.Cg))/1e9 : NaN;
  const z = data.Zgap[idx];
  const y = data.Ysand[idx];
  const text = [
    `Chbn1 = ${Ch1.toFixed(3)} fF, Chbn2 = ${Ch2.toFixed(3)} fF, Chbn-series = ${(Cseries*1e15).toFixed(3)} fF`,
    `Approx. f_LC using (Lg+Lk) and hBN-series C = ${isFinite(fLC_hbn) ? fLC_hbn.toFixed(3) : 'N/A'} GHz`,
    `Approx. f_LC using (Lg+Lk) and Cg_TMD = ${isFinite(fLC_tmd) ? fLC_tmd.toFixed(3) : 'N/A'} GHz`,
    '',
    `Readout at f = ${(data.f[idx]/1e9).toFixed(3)} GHz`,
    `Zgap = ${z[0].toExponential(4)} ${z[1] >= 0 ? '+' : '-'} j${Math.abs(z[1]).toExponential(4)} Ω`,
    `Ysand = ${y[0].toExponential(4)} ${y[1] >= 0 ? '+' : '-'} j${Math.abs(y[1]).toExponential(4)} S`,
    `|Ysand * Z0| = ${(cabs(y)*val('Z0')).toExponential(4)}`,
    `S11 = ${S11dB[idx].toFixed(3)} dB, S21 = ${S21dB[idx].toFixed(3)} dB`,
    `Group delay = ${gdPs[idx].toFixed(4)} ps`
  ].join('\n');
  document.getElementById('readout').textContent = text;
}

window.addEventListener('resize', () => {
  for (const id of ['magPlot', 'phasePlot', 'delayPlot', 'zPlot']) Plotly.Plots.resize(id);
});

updateAll();
</script>

