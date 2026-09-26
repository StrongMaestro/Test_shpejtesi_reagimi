<!DOCTYPE html>
<html>
<head>
<title>Testi i shpejtësisë së reagimit</title>
<style>
  :root {
    --bg: #0f1115;
    --panel: #1a1d24;
    --text: #f2f2f2;
    --muted: #9aa0aa;
    --red: #e5484d;
    --green: #46a758;
    --yellow: #e5a000;
  }
  @media (prefers-color-scheme: light) {
    :root:not([data-theme="dark"]) {
      --bg: #f5f6f8;
      --panel: #ffffff;
      --text: #1a1d24;
      --muted: #5b6472;
    }
  }
  :root[data-theme="dark"] {
    --bg: #0f1115;
    --panel: #1a1d24;
    --text: #f2f2f2;
    --muted: #9aa0aa;
  }
  * { box-sizing: border-box; }
  html, body {
    height: 100%;
    margin: 0;
  }
  body {
    background: var(--bg);
    color: var(--text);
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    padding: 24px;
    padding-top: calc(24px + env(safe-area-inset-top, 0px));
    padding-bottom: calc(24px + env(safe-area-inset-bottom, 0px));
    min-height: 100%;
    text-align: center;
  }
  h1 {
    font-size: 1.4rem;
    margin: 0 0 6px 0;
  }
  p.sub {
    color: var(--muted);
    margin: 0 0 28px 0;
    font-size: 0.95rem;
  }
  #pad {
    width: min(90vw, 420px);
    height: min(90vw, 420px);
    max-width: 420px;
    max-height: 420px;
    border-radius: 24px;
    border: none;
    cursor: pointer;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    gap: 8px;
    font-size: 1.3rem;
    font-weight: 600;
    color: white;
    background: var(--red);
    transition: background 0.15s ease;
    user-select: none;
    -webkit-tap-highlight-color: transparent;
  }
  #pad.waiting { background: var(--yellow); }
  #pad.go { background: var(--green); }
  #pad .big {
    font-size: 2.4rem;
    font-weight: 800;
  }
  #pad .small {
    font-size: 1rem;
    font-weight: 400;
    opacity: 0.9;
    max-width: 80%;
  }
  #stats {
    margin-top: 28px;
    display: flex;
    gap: 24px;
    color: var(--muted);
    font-size: 0.9rem;
    flex-wrap: wrap;
    justify-content: center;
  }
  #stats div strong {
    display: block;
    color: var(--text);
    font-size: 1.2rem;
    margin-top: 2px;
  }
  #reset {
    margin-top: 20px;
    background: none;
    border: 1px solid var(--muted);
    color: var(--muted);
    padding: 8px 16px;
    border-radius: 8px;
    cursor: pointer;
    font-size: 0.85rem;
  }
  #reset:hover { color: var(--text); border-color: var(--text); }
</style>
</head>
<body>
  <h1>Testi i shpejtësisë së reagimit</h1>
  <p class="sub">Shtypni butonin. Prisni të bëhet jeshil. Shtypeni sa më shpejt të mundeni.</p>

  <button id="pad" class="idle">
    <span class="big" id="mainText">Shtyp për të filluar</span>
    <span class="small" id="subText"></span>
  </button>

  <div id="stats">
    <div>Prova e fundit<strong id="lastVal">–</strong></div>
    <div>Prova më e mirë<strong id="bestVal">–</strong></div>
    <div>Mesatarja<strong id="avgVal">–</strong></div>
    <div>Numri i provave<strong id="countVal">0</strong></div>
  </div>

  <button id="reset">Zero rezultatet</button>

<script>
(function () {
  const pad = document.getElementById('pad');
  const mainText = document.getElementById('mainText');
  const subText = document.getElementById('subText');
  const lastVal = document.getElementById('lastVal');
  const bestVal = document.getElementById('bestVal');
  const avgVal = document.getElementById('avgVal');
  const countVal = document.getElementById('countVal');
  const resetBtn = document.getElementById('reset');

  let state = 'idle';
  let timeoutId = null;
  let startTime = null;
  let results = [];
  let best = null;

  function setState(s) {
    state = s;
    pad.className = s;
  }

  function startRound() {
    clearTimeout(timeoutId);
    setState('waiting');
    mainText.textContent = 'Prit për jeshilen...';
    subText.textContent = '';
    delay = 1000 + Math.random() * 3000; // 1-4s
    timeoutId = setTimeout(() => {
      setState('go');
      startTime = performance.now();
      mainText.textContent = 'Shtyp tani!';
      subText.textContent = '';
    }, delay);
  }

  function recordResult(ms) {
    results.push(ms);
    lastVal.textContent = ms + ' ms';
    if(best === null){
        best = results[0];
    }else{
        best = Math.min(best, results[results.length-1]);
    }
    const avg = Math.round(results.reduce((a, b) => a + b, 0) / results.length);
    bestVal.textContent = best + ' ms';
    avgVal.textContent = avg + ' ms';
    countVal.textContent = results.length;
  }

  function tooSoon() {
    clearTimeout(timeoutId);
    setState('idle');
    mainText.textContent = 'Duhet të prisni për jeshilen!';
    subText.textContent = '';
  }

  function showResult(ms) {
    setState('idle');
    mainText.textContent = ms + ' ms';
    subText.textContent = 'Provo përsëri';
    recordResult(ms);
  }

  pad.addEventListener('click', () => {
    if (state === 'idle') {
      startRound();
    } else if (state === 'waiting') {
      tooSoon();
    } else if (state === 'go') {
      const elapsed = Math.round(performance.now() - startTime);
      showResult(elapsed);
    }
  });

  resetBtn.addEventListener('click', () => {
    results = [];
    lastVal.textContent = '–';
    bestVal.textContent = '–';
    avgVal.textContent = '–';
    countVal.textContent = '0';
    best = null;
  });
})();
</script>
</body>
</html>
