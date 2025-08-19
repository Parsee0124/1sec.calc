<!doctype html>
<html lang="ja">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover" />
  <title>ポケモン簡易ダメージ計算 (Lv50・五捨五超入)</title>
  <style>
    :root {
      --bg: #0f1318;
      --card: #161b22;
      --text: #e6edf3;
      --muted: #9aa4b2;
      --accent: #3ea6ff;
      --danger: #ff6b6b;
      --ok: #37d67a;
      --bar-bg: #2a2f36;
      --bar-range: #ff7b7b;
      --input-bg: #0f141a;
      --btn: #212734;
      --btn-active: #2e384a;
      --btn-border: #303846;
    }
    * { box-sizing: border-box; }
    html, body { height: 100%; }
    body {
      margin: 0;
      font-family: system-ui, -apple-system, Segoe UI, Roboto, "Hiragino Kaku Gothic ProN", "Noto Sans JP", sans-serif;
      background: var(--bg);
      color: var(--text);
      -webkit-tap-highlight-color: transparent;
    }
    .app {
      display: flex;
      flex-direction: column;
      height: 100vh;
    }
    .display {
      padding: 12px 14px;
      background: var(--card);
      border-bottom: 1px solid var(--btn-border);
      height: 25vh;
      min-height: 160px;
    }
    .row { display: flex; gap: 8px; align-items: center; }
    .row + .row { margin-top: 8px; }
    .field {
      flex: 1;
      display: flex;
      flex-direction: column;
      gap: 6px;
    }
    label {
      font-size: 12px;
      color: var(--muted);
      letter-spacing: .03em;
    }
    input[type="text"] {
      width: 100%;
      padding: 12px 10px;
      font-size: 18px;
      color: var(--text);
      background: var(--input-bg);
      border: 1px solid var(--btn-border);
      border-radius: 10px;
      outline: none;
    }
    .results {
      margin-top: 8px;
      padding: 10px;
      background: var(--input-bg);
      border: 1px solid var(--btn-border);
      border-radius: 10px;
    }
    .results .nums {
      display: flex; justify-content: space-between; gap: 10px; font-variant-numeric: tabular-nums;
    }
    .hpbar {
      position: relative;
      margin-top: 10px;
      height: 20px;
      background: var(--bar-bg);
      border-radius: 999px;
      overflow: hidden;
      border: 1px solid var(--btn-border);
    }
    .hpbar .range {
      position: absolute;
      top: 0; bottom: 0;
      background: var(--bar-range);
      opacity: 0.9;
    }
    .hpbar .label {
      position: absolute;
      width: 100%;
      text-align: center;
      top: 0; bottom: 0;
      display: flex; align-items: center; justify-content: center;
      font-size: 12px; color: var(--text);
      text-shadow: 0 1px 2px rgba(0,0,0,.6);
      pointer-events: none;
    }

    .keypad {
      height: 75vh;
      padding: 12px;
      display: grid;
      grid-template-rows: repeat(5, 1fr);
      gap: 8px;
    }
    .keyrow {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 8px;
    }
    .btn {
      background: var(--btn);
      border: 1px solid var(--btn-border);
      color: var(--text);
      font-size: 20px;
      border-radius: 14px;
      display: flex; align-items: center; justify-content: center;
      user-select: none;
      -webkit-user-select: none;
      touch-action: manipulation;
      padding: 10px 6px;
      min-height: 54px;
    }
    .btn:active { background: var(--btn-active); }
    .btn--accent { border-color: var(--accent); }
    .btn--calc { border-color: var(--ok); font-weight: 700; }
    .btn--toggle { font-size: 16px; }
    .btn--on { box-shadow: inset 0 0 0 2px var(--accent); background: #1b2433; }
    .meta {
      display: flex; gap: 10px; align-items: center; flex-wrap: wrap;
      margin-bottom: 6px; color: var(--muted); font-size: 12px;
    }
    .pill {
      border: 1px solid var(--btn-border); padding: 4px 8px; border-radius: 999px;
      background: var(--input-bg);
    }
    .hint { color: var(--muted); font-size: 11px; margin-top: 6px; }
  </style>
</head>
<body>
  <div class="app">
    <!-- 上部表示領域（1/4） -->
    <div class="display">
      <div class="row">
        <div class="field">
          <label>攻撃</label>
          <input type="text" inputmode="numeric" pattern="[0-9]*" id="atk" placeholder="例) 200" />
        </div>
        <div class="field">
          <label>防御</label>
          <input type="text" inputmode="numeric" pattern="[0-9]*" id="def" placeholder="例) 150" />
        </div>
      </div>
      <div class="row">
        <div class="field">
          <label>HP</label>
          <input type="text" inputmode="numeric" pattern="[0-9]*" id="hp" placeholder="例) 175" />
        </div>
        <div class="field">
          <label>技威力</label>
          <input type="text" inputmode="numeric" pattern="[0-9]*" id="pow" placeholder="例) 90" />
        </div>
      </div>

      <div class="results">
        <div class="meta">
          <span class="pill">Lv 50 固定</span>
          <span class="pill">補正は <b>五捨五超入</b></span>
          <span class="pill">乱数 0.85〜1.00（16段階）</span>
        </div>
        <div class="nums">
          <div>最小: <b id="minDmg">—</b></div>
          <div>最大: <b id="maxDmg">—</b></div>
        </div>
        <div class="hpbar" aria-label="ダメージ割合バー">
          <div class="range" id="range" style="left:0%; width:0%;"></div>
          <div class="label" id="barLabel">—</div>
        </div>
        <div class="hint">※ 簡易版：STAB/ハチマキ/珠のみ対応（後で特性・天候・フィールドを追加予定）</div>
      </div>
    </div>

    <!-- 下部キーパッド（3/4） -->
    <div class="keypad">
      <div class="keyrow">
        <div class="btn" data-key="1">1</div>
        <div class="btn" data-key="2">2</div>
        <div class="btn" data-key="3">3</div>
      </div>
      <div class="keyrow">
        <div class="btn" data-key="4">4</div>
        <div class="btn" data-key="5">5</div>
        <div class="btn" data-key="6">6</div>
      </div>
      <div class="keyrow">
        <div class="btn" data-key="7">7</div>
        <div class="btn" data-key="8">8</div>
        <div class="btn" data-key="9">9</div>
      </div>
      <div class="keyrow">
        <div class="btn btn--accent" id="next">次へ</div>
        <div class="btn" data-key="0">0</div>
        <div class="btn btn--calc" id="calc">計算</div>
      </div>
      <div class="keyrow">
        <div class="btn btn--toggle" id="stab">STAB</div>
        <div class="btn btn--toggle" id="band">ハチマキ</div>
        <div class="btn btn--toggle" id="orb">珠</div>
      </div>
    </div>
  </div>

  <script>
    // --------------- 五捨五超入（round half down） ----------------
    // ※ 正の値を想定（ダメージや補正）。ちょうど x.5 は切り捨て。
    function roundHalfDown(x) {
      const f = Math.floor(x);
      const frac = x - f;
      if (frac > 0.5) return f + 1;
      return f;
    }

    // --------------- 入力管理 ----------------
    const fields = [document.getElementById('atk'), document.getElementById('def'),
                    document.getElementById('hp'), document.getElementById('pow')];
    let activeIndex = 0;
    fields[activeIndex].classList.add('active');
    function focusField(i) {
      activeIndex = (i + fields.length) % fields.length;
      fields[activeIndex].focus();
    }
    // タップでフォーカスを移す
    fields.forEach((el, i) => {
      el.addEventListener('focus', () => { activeIndex = i; });
    });

    // 数字入力（末尾に追加）
    function appendDigit(d) {
      const el = fields[activeIndex];
      el.value = (el.value || '') + d;
    }

    // 次へ
    document.getElementById('next').addEventListener('click', () => focusField(activeIndex + 1));

    // 数字キー
    document.querySelectorAll('.btn[data-key]').forEach(btn => {
      btn.addEventListener('click', () => appendDigit(btn.dataset.key));
    });

    // トグル（STAB / ハチマキ / 珠）
    const toggles = {
      stab: false,
      band: false, // 簡易版ではダメージ補正1.5倍として適用
      orb:  false  // いのちのたま 1.3倍
    };
    function toggleBtn(id) {
      const btn = document.getElementById(id);
      toggles[id] = !toggles[id];
      btn.classList.toggle('btn--on', toggles[id]);
    }
    document.getElementById('stab').addEventListener('click', () => toggleBtn('stab'));
    document.getElementById('band').addEventListener('click', () => toggleBtn('band'));
    document.getElementById('orb').addEventListener('click',  () => toggleBtn('orb'));

    // --------------- ダメージ計算（Lv50固定・簡易版） ----------------
    const minD = document.getElementById('minDmg');
    const maxD = document.getElementById('maxDmg');
    const rangeEl = document.getElementById('range');
    const barLabel = document.getElementById('barLabel');

    function parseIntSafe(v) {
      const n = parseInt(v, 10);
      return isNaN(n) ? 0 : n;
    }

    function calcOnce(base, modifiers) {
      // 補正は順番に乗算し、その都度「五捨五超入」を適用
      // modifiers: Array<number>
      let dmg = base;
      for (const m of modifiers) {
        dmg = roundHalfDown(dmg * m);
      }
      return dmg;
    }

    function compute() {
      const atk = parseIntSafe(fields[0].value);
      const def = parseIntSafe(fields[1].value);
      const hp  = parseIntSafe(fields[2].value);
      const pow = parseIntSafe(fields[3].value);
      if (atk <= 0 || def <= 0 || hp <= 0 || pow <= 0) {
        minD.textContent = '—';
        maxD.textContent = '—';
        rangeEl.style.left = '0%';
        rangeEl.style.width = '0%';
        barLabel.textContent = '—';
        return;
      }

      const Lv = 50;

      // 公式形（除算ごとに切り捨て）：
      // base = (((((2*Lv)/5 + 2) * pow * atk / def) / 50) + 2)
      // -> 各除算直後に floor
      let step1 = Math.floor((2 * Lv) / 5) + 2;
      let step2 = step1 * pow * atk;           // ここは整数のまま
      let step3 = Math.floor(step2 / def);     // /def の直後で切り捨て
      let step4 = Math.floor(step3 / 50);      // /50 の直後で切り捨て
      let base = step4 + 2;

      // 簡易版の補正（順序）：乱数 → STAB → ハチマキ → 珠（※公式順とは若干異なるが後で精密化）
      const randVals = Array.from({length: 16}, (_, i) => (85 + i) / 100);

      let minDamage = Infinity;
      let maxDamage = -Infinity;
      for (const r of randVals) {
        const mods = [r];
        if (toggles.stab) mods.push(1.5);
        if (toggles.band) mods.push(1.5);
        if (toggles.orb)  mods.push(1.3);

        const dmg = calcOnce(base, mods);
        if (dmg < minDamage) minDamage = dmg;
        if (dmg > maxDamage) maxDamage = dmg;
      }

      // 表示更新
      minD.textContent = String(minDamage);
      maxD.textContent = String(maxDamage);

      const minPct = Math.max(0, Math.min(100, (minDamage / hp) * 100));
      const maxPct = Math.max(0, Math.min(100, (maxDamage / hp) * 100));
      const left = Math.min(minPct, maxPct);
      const width = Math.max(0, Math.abs(maxPct - minPct));

      rangeEl.style.left = left + '%';
      rangeEl.style.width = width + '%';

      const pctStr = (v)=> (Math.round(v*10)/10).toFixed(1) + '%';
      barLabel.textContent = `${pctStr(minPct)} 〜 ${pctStr(maxPct)} / HP${hp}`;
    }

    document.getElementById('calc').addEventListener('click', compute);

    // Enterで次へ、数値キー直接打鍵にも軽く対応（任意）
    document.addEventListener('keydown', (e) => {
      if (e.key === 'Enter') { compute(); }
      if (/[0-9]/.test(e.key) && !e.ctrlKey && !e.metaKey) {
        appendDigit(e.key);
        e.preventDefault();
      }
      if (e.key === 'Tab') {
        focusField(activeIndex + 1);
        e.preventDefault();
      }
    });
  </script>
</body>
</html>
