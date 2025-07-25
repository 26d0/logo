<svelte:head>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin="anonymous">
  <link href="https://fonts.googleapis.com/css2?family=Dosis:wght@700&family=M+PLUS+Rounded+1c:wght@700&display=swap" rel="stylesheet">
  <title>Logo Generator</title>
  <link rel="shortcut icon" href="/favicon.ico">
</svelte:head>

<script lang="ts">
  import { onMount } from 'svelte';
  
  // Get initial values from URL query parameters with smart defaults
  const getQueryParam = (name: string, defaultValue: string): string => 
    typeof window !== 'undefined' 
      ? new URLSearchParams(window.location.search).get(name) || defaultValue 
      : defaultValue;
  
  let text1 = getQueryParam('text1', 'VR');
  let text2 = getQueryParam('text2', 'CHAT');
  let canvas: HTMLCanvasElement;
  let ctx: CanvasRenderingContext2D | null;
  let fontLoaded = false;
  let copyFeedback = '';

  // detect use of webkit
  function isWebKit(): boolean {
    return navigator.vendor ? navigator.vendor.includes('Apple') : false;
  }

  // Clean URL parameters to remove text1 and text2
  function cleanUrlParameters(): void {
    if (typeof window !== 'undefined') {
      const url = new URL(window.location.href);
      const hasText1 = url.searchParams.has('text1');
      const hasText2 = url.searchParams.has('text2');
      
      if (hasText1 || hasText2) {
        url.searchParams.delete('text1');
        url.searchParams.delete('text2');
        window.history.replaceState({}, document.title, url.toString());
      }
    }
  }

  onMount(async () => {
    // Clean URL parameters if they exist
    cleanUrlParameters();
    
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

  function generateLogo(forExport: boolean = false): void {
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

  function downloadImage(): void {
    generateLogo(true);
    const link = document.createElement('a');
    
    // Generate dynamic filename based on text inputs
    const sanitizeText = (text: string): string => text.replace(/[^a-zA-Z0-9\u3040-\u309F\u30A0-\u30FF\u4E00-\u9FAF]/g, '').slice(0, 20);
    const fileName1 = text1 ? sanitizeText(text1) : 'text1';
    const fileName2 = text2 ? sanitizeText(text2) : 'text2';
    
    link.download = `${fileName1}_${fileName2}.png`;
    link.href = canvas.toDataURL('image/png', 1.0);
    link.click();
    generateLogo(false);
  }

  function shareUrl(): void {
    const url = new URL(window.location.href);
    url.searchParams.set('text1', text1);
    url.searchParams.set('text2', text2);
    
    // Check if Web Share API is available
    if (navigator.share) {
      navigator.share({
        title: 'VRSNS風ロゴジェネレーター',
        text: `"${text1}" と "${text2}" のロゴを作ったよ！`,
        url: url.toString()
      }).then(() => {
        copyFeedback = 'シェアしました！';
        setTimeout(() => {
          copyFeedback = '';
        }, 2000);
      }).catch(err => {
        // Fallback to clipboard copy if share is cancelled or fails
        if (err.name !== 'AbortError') {
          fallbackCopyUrl(url.toString());
        }
      });
    } else {
      // Fallback to clipboard copy for browsers without Web Share API
      fallbackCopyUrl(url.toString());
    }
  }

  function fallbackCopyUrl(url: string): void {
    navigator.clipboard.writeText(url).then(() => {
      copyFeedback = 'URLをコピーしました！';
      setTimeout(() => {
        copyFeedback = '';
      }, 2000);
    }).catch(err => {
      console.error('Failed to copy URL: ', err);
      copyFeedback = 'シェアに失敗しました';
      setTimeout(() => {
        copyFeedback = '';
      }, 2000);
    });
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
        class:hidden={!fontLoaded}
        class="max-w-lg w-full md:w-auto rounded-lg"
      >
      </canvas>
    </div>
    <div class="flex flex-col items-center gap-2">
      <div class="flex gap-2">
        <button 
          on:click={downloadImage}
          class="bg-green-600 hover:bg-green-700 text-white px-4 py-2 rounded-md text-sm font-medium transition-colors duration-200 flex items-center gap-2"
        >
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M12 16L7 11L8.4 9.55L11 12.15V4H13V12.15L15.6 9.55L17 11L12 16Z" fill="currentColor"/>
            <path d="M5 20V18H19V20H5Z" fill="currentColor"/>
          </svg>
          画像をダウンロード
        </button>
        <button 
          on:click={shareUrl}
          class="bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded-md text-sm font-medium transition-colors duration-200 flex items-center gap-2"
        >
          <svg width="16" height="16" viewBox="0 0 24 24" fill="none" xmlns="http://www.w3.org/2000/svg">
            <path d="M18 16.08C17.24 16.08 16.56 16.38 16.04 16.85L8.91 12.7C8.96 12.47 9 12.24 9 12S8.96 11.53 8.91 11.3L15.96 7.19C16.5 7.69 17.21 8 18 8C19.66 8 21 6.66 21 5S19.66 2 18 2 15 3.34 15 5C15 5.24 15.04 5.47 15.09 5.7L8.04 9.81C7.5 9.31 6.79 9 6 9C4.34 9 3 10.34 3 12S4.34 15 6 15C6.79 15 7.5 14.69 8.04 14.19L15.16 18.34C15.11 18.55 15.08 18.77 15.08 19C15.08 20.61 16.39 21.92 18 21.92S20.92 20.61 20.92 19C20.92 17.39 19.61 16.08 18 16.08Z" fill="currentColor"/>
          </svg>
          リンクをシェア
        </button>
      </div>
      {#if copyFeedback}
        <p class="text-sm text-green-400 animate-pulse">{copyFeedback}</p>
      {/if}
    </div>
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
