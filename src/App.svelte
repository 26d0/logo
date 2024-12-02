<svelte:head>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Dosis:wght@700&family=M+PLUS+Rounded+1c:wght@700&display=swap" rel="stylesheet">
  <title>Logo Generator</title>
  <link rel="shortcut icon" href="/favicon.ico">
</svelte:head>

<script>
  import { onMount } from 'svelte';
  let text1 = 'VR';
  let text2 = 'CHAT';
  let canvas;
  let ctx;
  let fontLoaded = false;

  // detect use of webkit
  function isWebKit() {
    return navigator.vendor && navigator.vendor.includes('Apple');
  }

  onMount(async () => {
    // フォントの読み込みを待つ
    await document.fonts.ready;
    // 両方のフォントが読み込まれるのを確認
    await Promise.all([
      document.fonts.load('700 10px "Dosis"'),
      document.fonts.load('700 10px "M PLUS Rounded 1c"')
    ]);
    
    fontLoaded = true;
    ctx = canvas.getContext('2d', { alpha: true });
    generateLogo(false);
  });

  function generateLogo(forExport = false) {
    if (!ctx || !fontLoaded) return;

    ctx.font = '700 92px "Dosis", "M PLUS Rounded 1c", sans-serif';
    const text1Metrics = ctx.measureText(text1);
    const text2Metrics = ctx.measureText(text2);
    const spacing = 6;
    
    // detect use of non-ASCII characters to avoid overflow
    const containsNonAscii_text1 = /[^\x00-\x7F]/.test(text1);
    const containsNonAscii_text2 = /[^\x00-\x7F]/.test(text2);

    const leftPadding = (text1 == "")?15:20;       
    const rightPadding = (text2 == "")?-25:-12;   
    const bottomPadding = -30;  
    const topPadding = containsNonAscii_text2?55:45;     
    const totalTextWidth = text1Metrics.width + text2Metrics.width + spacing;
    const totalWidth = totalTextWidth + leftPadding + rightPadding + 42;
    
    canvas.width = totalWidth + 80;
    canvas.height = 200;

    ctx.clearRect(0, 0, canvas.width, canvas.height);

    if (!forExport) {
        ctx.fillStyle = '#FFFFFF';
        ctx.fillRect(0, 0, canvas.width, canvas.height);
    }

    // overall offset
    const offsetY = -20;

    // text offset (for fixing the visual error in webkit)
    const textOffsetY = isWebKit()?-6:0;

    const cornerRadius = 18;
    const textHeight = 100 + topPadding + bottomPadding;
    const x = (canvas.width - totalWidth) / 2;
    const y = (canvas.height - textHeight) / 2 + offsetY;
    const bubbleCenterY = y + textHeight / 2 + 4;

    ctx.font = '700 92px "Dosis", "M PLUS Rounded 1c", sans-serif';

    // left top to right bottom
    ctx.beginPath();
    ctx.moveTo(x + cornerRadius, y);
    ctx.lineTo(x + totalWidth - cornerRadius, y);
    ctx.quadraticCurveTo(x + totalWidth, y, x + totalWidth, y + cornerRadius);
    ctx.lineTo(x + totalWidth, y + textHeight - cornerRadius);
    ctx.quadraticCurveTo(x + totalWidth, y + textHeight, x + totalWidth - cornerRadius, y + textHeight);
    
    // Drawing the tail
    const tailHeight = 30;
    const tailBeginX = 5;

    ctx.lineTo(x + totalWidth - cornerRadius - 5 + tailBeginX, y + textHeight);
    ctx.lineTo(x + totalWidth - cornerRadius - 5 + tailBeginX, y + textHeight + tailHeight + 5);
    //  Rounding the tail
    ctx.quadraticCurveTo(x + totalWidth - cornerRadius - 5 + tailBeginX, y + textHeight + tailHeight + 20,x + totalWidth - cornerRadius - 5 -15 + tailBeginX, y + textHeight + tailHeight);
    ctx.lineTo(x + totalWidth - cornerRadius - 45 + tailBeginX, y + textHeight);
    
    // tail to left bottom
    ctx.lineTo(x + cornerRadius, y + textHeight);
    ctx.quadraticCurveTo(x, y + textHeight, x, y + textHeight - cornerRadius);
    ctx.lineTo(x, y + cornerRadius);
    ctx.quadraticCurveTo(x, y, x + cornerRadius, y);

    ctx.closePath();

    ctx.fillStyle = '#FFFFFF';
    ctx.fill();
    ctx.lineWidth = 8;
    ctx.strokeStyle = '#000000';
    ctx.stroke();

    ctx.fillStyle = '#000000';
    ctx.textAlign = 'left';
    ctx.textBaseline = 'middle';
    ctx.fillText(text1, x + leftPadding, bubbleCenterY + textOffsetY);

    const chatX = x + leftPadding + text1Metrics.width + spacing;
    const chatBoxHeight = containsNonAscii_text2?(textHeight/1.35):(textHeight/1.45);
    const chatY = bubbleCenterY - chatBoxHeight/2 - 5;
    const chatBoxWidth = text2Metrics.width + 8;
    const chatCornerRadius = 12;

    if(text2 != ""){
      ctx.beginPath();
      
      ctx.moveTo(chatX + chatCornerRadius, chatY);
      ctx.lineTo(chatX + chatBoxWidth - chatCornerRadius, chatY);
      ctx.quadraticCurveTo(chatX + chatBoxWidth, chatY, chatX + chatBoxWidth, chatY + chatCornerRadius);
      ctx.lineTo(chatX + chatBoxWidth, chatY + chatBoxHeight - chatCornerRadius);
      ctx.quadraticCurveTo(chatX + chatBoxWidth, chatY + chatBoxHeight, chatX + chatBoxWidth - chatCornerRadius, chatY + chatBoxHeight);
      ctx.lineTo(chatX + chatCornerRadius, chatY + chatBoxHeight);
      ctx.quadraticCurveTo(chatX, chatY + chatBoxHeight, chatX, chatY + chatBoxHeight - chatCornerRadius);
      ctx.lineTo(chatX, chatY + chatCornerRadius);
      ctx.quadraticCurveTo(chatX, chatY, chatX + chatCornerRadius, chatY);
      ctx.closePath();
    
      ctx.fillStyle = '#000000';
      ctx.fill();

      ctx.fillStyle = '#FFFFFF';
      ctx.fillText(text2, chatX + 4, bubbleCenterY + textOffsetY);
    }
    
    
  }

  function downloadImage() {
    generateLogo(true);
    const link = document.createElement('a');
    link.download = 'vrchat-logo.png';
    link.href = canvas.toDataURL('image/png', 1.0);
    link.click();
    generateLogo(false);
  }

  $: if (ctx && fontLoaded && (text1 || text2)) generateLogo(false);
</script>

<main>
  <div class="flex flex-col items-center justify-center gap-8">
    <div>
      <p class="text-2xl md:text-5xl text-center font-semibold">VRSNS風ロゴジェネレーター</p>
    </div>
    <div class="flex flex-col md:flex-row gap-2 md:gap-6 md:max-w-lg w-full">
      <input 
        type="text" 
        bind:value={text1} 
        placeholder="VR"
        class="bg-color-0 mt-1 px-3 py-2 border shadow-sm border-color-1 focus:outline-none focus:border-color-1 focus:ring-color-1 w-full block rounded-md sm:text-sm focus:ring-1"
      >
      <input 
        type="text" 
        bind:value={text2} 
        placeholder="CHAT"
        class="bg-color-0 mt-1 px-3 py-2 border shadow-sm border-color-1 focus:outline-none focus:border-color-1 focus:ring-color-1 w-full block rounded-md sm:text-sm focus:ring-1"
      >
    </div>
    <div class="md:w-auto ">
      {#if !fontLoaded}
      <div class="loading">フォントを読み込み中...</div>
      {/if}
  
      <canvas 
        bind:this={canvas} 
        on:click={downloadImage}
        title="クリックしてダウンロード"
        class:hidden={!fontLoaded}
        class="max-w-lg w-full md:w-auto rounded-lg"
      >
      </canvas>
    </div>
    <p class="text-sm md:text-xl">↑<br>キャンバスをクリックするとPNG画像をダウンロードできます</p>
  </div>
</main>

<style>
  :global(body) {
    background-color: #1a1b26;
    color: #a9b1d6;
    margin: 0;
    padding: 0;
  }

  :global(body) {
    font-family: "Dosis", "M PLUS Rounded 1c", sans-serif;
    font-optical-sizing: auto;
    font-weight: 700;
    font-style: normal;
    color: #a9b1d6;
  }
</style>
