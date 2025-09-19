<script lang="ts">
  import { onMount, afterUpdate } from 'svelte';

  /* ─────────────  定数 / ユーティリティ  ──────────── */
  const FONT_FAMILY = '"Dosis", "M PLUS Rounded 1c", sans-serif';
  const FONT_DECL    = `700 92px ${FONT_FAMILY}`;
  const QUERY        = typeof window !== 'undefined' ? new URLSearchParams(location.search) : null;
  const NON_ASCII_RE = /[^\x00-\x7F]/;
  const isWebKit     = () => navigator.vendor?.includes('Apple') ?? false;

  const getParam = (key: string, def = '') =>
    QUERY?.get(key) || def;

  const sanitize = (s: string) =>
    s.replace(/[^a-zA-Z0-9\u3040-\u309F\u30A0-\u30FF\u4E00-\u9FAF]/g, '').slice(0, 20);

  /* ─────────────  状態  ──────────── */
  let text1 = getParam('text1', 'VR');
  let text2 = getParam('text2', 'CHAT');

  let canvas: HTMLCanvasElement;
  let ctx: CanvasRenderingContext2D;
  let fontReady = false;
  let copyFeedback = '';

  /* ────────────  ライフサイクル  ─────────── */
  onMount(async () => {
    cleanQuery();
    await loadFonts();
    ctx = canvas.getContext('2d', { alpha: true })!;
    fontReady = true;
    draw();          // 初回
  });

  /* text1/text2 が変わったら自動再描画 */
  afterUpdate(() => fontReady && draw());

  /* ─────────────  関数群  ──────────── */

  async function loadFonts() {
    await document.fonts.ready;
    await Promise.all([
      document.fonts.load(`700 10px "Dosis"`),
      document.fonts.load(`700 10px "M PLUS Rounded 1c"`)
    ]);
  }

  function cleanQuery() {
    if (!QUERY) return;
    ['text1', 'text2'].forEach(key => QUERY.delete(key));
    history.replaceState({}, '', location.pathname);
  }

  /* 各種計算をまとめたオブジェクトを返す */
  function calcLayout() {
    ctx.font = FONT_DECL;

    const t1Met = ctx.measureText(text1);
    const t2Met = ctx.measureText(text2);
    const spacing        = 6;
    const leftPadding    = text1 ? 20 : 15;
    const rightPadding   = text2 ? -12 : -25;
    const bottomPadding  = -30;
    const topPadding     = NON_ASCII_RE.test(text2) ? 55 : 45;

    const totalTextW = t1Met.width + t2Met.width + spacing;
    const totalWidth = totalTextW + leftPadding + rightPadding + 42;
    const totalHeight = 100 + topPadding + bottomPadding;

    /* 返却 */
    return {
      t1Met, t2Met, spacing,
      leftPadding, chatOffsetX: leftPadding + t1Met.width + spacing,
      chatBoxH: NON_ASCII_RE.test(text2) ? totalHeight / 1.35 : totalHeight / 1.45,
      totalWidth, totalHeight,
      offsetY: -20
    };
  }

  function draw({ forExport = false } = {}) {
    if (!ctx) return;

    /* ---- レイアウト計算 ---- */
    const L = calcLayout();

    /* ---- canvas サイズ決定 ---- */
    canvas.width  = L.totalWidth + 80;
    canvas.height = 200;

    /* 背景クリア & 塗りつぶし (プレビュー時のみ白背景) */
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    if (!forExport) {
      ctx.fillStyle = '#fff';
      ctx.fillRect(0, 0, canvas.width, canvas.height);
    }

    /* ---- 吹き出し本体 ---- */
    const x = (canvas.width  - L.totalWidth)  / 2;
    const y = (canvas.height - L.totalHeight) / 2 + L.offsetY;
    const r = 18;
    const tailH = 30;
    const tailX = 5;
    const bubbleCenterY = y + L.totalHeight / 2 + 4;
    ctx.font = FONT_DECL;

    ctx.beginPath();
    // 上辺
    ctx.moveTo(x + r, y);
    ctx.lineTo(x + L.totalWidth - r, y);
    ctx.quadraticCurveTo(x + L.totalWidth, y, x + L.totalWidth, y + r);
    // 右辺 & 右下角
    ctx.lineTo(x + L.totalWidth, y + L.totalHeight - r);
    ctx.quadraticCurveTo(x + L.totalWidth, y + L.totalHeight, x + L.totalWidth - r, y + L.totalHeight);
    // 吹き出し尻尾
    ctx.lineTo(x + L.totalWidth - r - 5 + tailX, y + L.totalHeight);
    ctx.lineTo(x + L.totalWidth - r - 5 + tailX, y + L.totalHeight + tailH + 5);
    ctx.quadraticCurveTo(
      x + L.totalWidth - r - 5 + tailX,
      y + L.totalHeight + tailH + 20,
      x + L.totalWidth - r - 5 - 15 + tailX,
      y + L.totalHeight + tailH
    );
    ctx.lineTo(x + L.totalWidth - r - 45 + tailX, y + L.totalHeight);
    // 左辺
    ctx.lineTo(x + r, y + L.totalHeight);
    ctx.quadraticCurveTo(x, y + L.totalHeight, x, y + L.totalHeight - r);
    ctx.lineTo(x, y + r);
    ctx.quadraticCurveTo(x, y, x + r, y);
    ctx.closePath();

    ctx.fillStyle   = '#fff';
    ctx.strokeStyle = '#000';
    ctx.lineWidth   = 8;
    ctx.fill();
    ctx.stroke();

    /* ---- 文字列描画 ---- */
    const txtOffsetY = isWebKit() ? -6 : 0;
    ctx.textAlign = 'left';
    ctx.textBaseline = 'middle';

    if (text1) {
      ctx.fillStyle = '#000';
      ctx.fillText(text1, x + L.leftPadding, bubbleCenterY + txtOffsetY);
    }

    /* CHAT ボックス */
    if (text2) {
      const boxX = x + L.chatOffsetX;
      const boxW = L.t2Met.width + 8;
      const boxY = bubbleCenterY - L.chatBoxH / 2 - 5;
      const br   = 12;

      roundedRect(boxX, boxY, boxW, L.chatBoxH, br, '#000');
      ctx.fillStyle = '#fff';
      ctx.fillText(text2, boxX + 4, bubbleCenterY + txtOffsetY);
    }
  }

  /* 角丸矩形を塗りつぶし */
  function roundedRect(x: number, y: number, w: number, h: number, r: number, color: string) {
    ctx.beginPath();
    ctx.moveTo(x + r, y);
    ctx.lineTo(x + w - r, y);
    ctx.quadraticCurveTo(x + w, y, x + w, y + r);
    ctx.lineTo(x + w, y + h - r);
    ctx.quadraticCurveTo(x + w, y + h, x + w - r, y + h);
    ctx.lineTo(x + r, y + h);
    ctx.quadraticCurveTo(x, y + h, x, y + h - r);
    ctx.lineTo(x, y + r);
    ctx.quadraticCurveTo(x, y, x + r, y);
    ctx.closePath();
    ctx.fillStyle = color;
    ctx.fill();
  }

  /* --------- DL / シェア --------- */
  function downloadImage() {
    draw({ forExport: true });
    const a = document.createElement('a');
    a.download = `${sanitize(text1 || 'text1')}_${sanitize(text2 || 'text2')}.png`;
    a.href      = canvas.toDataURL('image/png', 1.0);
    a.click();
    draw(); // 戻す
  }

  function shareUrl() {
    const url = new URL(location.href);
    url.searchParams.set('text1', text1);
    url.searchParams.set('text2', text2);

    const shareDone = (msg: string) => {
      copyFeedback = msg;
      setTimeout(() => (copyFeedback = ''), 2000);
    };

    if (navigator.share) {
      navigator.share({
        title: 'VRSNS風ロゴジェネレーター',
        text : `${text1} ${text2}のロゴを作ったよ！`,
        url  : url.toString()
      }).then(() => shareDone('シェアしました！'))
        .catch(e => e.name !== 'AbortError' && fallback());
    } else {
      fallback();
    }

    function fallback() {
      navigator.clipboard.writeText(url.toString())
        .then(() => shareDone('URLをコピーしました！'))
        .catch(()  => shareDone('シェアに失敗しました'));
    }
  }
</script>

<svelte:head>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="anonymous">
  <link href="https://fonts.googleapis.com/css2?family=Dosis:wght@700&family=M+PLUS+Rounded+1c:wght@700&display=swap" rel="stylesheet">
  <title>Logo Generator</title>
  <link rel="shortcut icon" href="/favicon.ico">
</svelte:head>

<main class="flex flex-col items-center justify-center gap-8">
  <p class="text-2xl md:text-5xl text-center font-semibold main-title">VRSNS風ロゴジェネレーター</p>

  <div class="flex flex-col md:flex-row gap-2 md:gap-6 md:max-w-lg w-full">
    <input bind:value={text1} placeholder="VR"   class="input" />
    <input bind:value={text2} placeholder="CHAT" class="input" />
  </div>

  <div class="md:w-auto">
    {#if !fontReady}
      <div class="loading">フォントを読み込み中...</div>
    {/if}
    <canvas bind:this={canvas} class="max-w-lg w-full md:w-auto rounded-lg" class:hidden={!fontReady}></canvas>
  </div>

  <div class="flex flex-col items-center gap-2">
    <div class="flex gap-2">
      <button class="btn green" on:click={downloadImage}>画像をダウンロード</button>
      <button class="btn blue"  on:click={shareUrl}>リンクをシェア</button>
    </div>
    {#if copyFeedback}
      <p class="feedback-text">{copyFeedback}</p>
    {/if}
  </div>
</main>

<style>
  :global(body) {
    margin: 0;
    font-family: "Dosis", "M PLUS Rounded 1c", sans-serif;
    font-weight: 700;
  }
  .input {
    background-color: white;
    margin-top: 0.25rem;
    padding: 0.5rem 0.75rem;
    border: 1px solid #d1d5db;
    box-shadow: 0 1px 2px 0 rgba(0, 0, 0, 0.05);
    border-radius: 0.375rem;
    width: 100%;
    display: block;
    font-size: 0.875rem;
    color: #111827;
    transition: border-color 0.15s ease-in-out, box-shadow 0.15s ease-in-out;
  }
  .input:focus {
    outline: none;
    border-color: #3b82f6;
    box-shadow: 0 0 0 1px #3b82f6;
  }
  @media (prefers-color-scheme: dark) {
    .input {
      background-color: #374151;
      border-color: #4b5563;
      color: #f9fafb;
    }
    .input:focus {
      border-color: #3b82f6;
    }
  }
  .btn { 
    padding: 0.5rem 1rem;
    border-radius: 0.375rem;
    font-size: 0.875rem;
    font-weight: 500;
    display: flex;
    align-items: center;
    gap: 0.5rem;
    color: white;
    transition: filter 0.15s ease-in-out;
    border: none;
    cursor: pointer;
  }
  .btn:hover {
    filter: brightness(1.1);
  }
  .green { background: #16a34a; }
  .blue { background: #2563eb; }
  .main-title {
    color: #213547;
  }
  @media (prefers-color-scheme: dark) {
    .main-title {
      color: rgba(255, 255, 255, 0.87);
    }
  }
  .feedback-text {
    font-size: 0.875rem;
    color: #059669;
    animation: pulse 2s cubic-bezier(0.4, 0, 0.6, 1) infinite;
  }
  @media (prefers-color-scheme: dark) {
    .feedback-text {
      color: #34d399;
    }
  }
</style>
