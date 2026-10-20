# Lina AI - Analisis Saham (Teknikal + Fundamental + Berita)

Aplikasi web single-page untuk analisis saham pemula dengan 3 lapis analisis:
**Teknikal + Fundamental + Berita** yang menghasilkan rekomendasi **BUY / SELL / HOLD**.

## ⚠️ Penting - Setup API

### 1. FMP API Key (WAJIB untuk US stocks - free 250 req/hari)
Daftar di https://site.financialmodelingprep.com/register
- Tanpa FMP: hanya data teknikal Yahoo (jika proxy up) + chart meta (terbatas)
- Dengan FMP: data fundamental LENGKAP (PE, PBV, ROE, ROA, PEG, Graham Number, EV/Sales, EV/FCF, dll) + analyst estimates (forecast 4 tahun)

### 2. Custom CORS Proxy URL (opsional, untuk IDX/Crypto + berita)
Public CORS proxy (allorigins, codetabs, dll) sering down/rate-limited. Untuk reliability penuh, deploy Cloudflare Worker sendiri (GRATIS, 100k req/hari):

#### Cara Deploy Cloudflare Worker sebagai CORS Proxy:

1. Buat akun gratis di https://dash.cloudflare.com/sign-up
2. Klik **Workers & Pages** → **Create application** → **Create Worker**
3. Beri nama (misal: `cors-proxy`) → **Deploy**
4. Klik **Edit code** → copy-paste kode di bawah → **Save and deploy**

```javascript
// Cloudflare Worker - CORS Proxy untuk Yahoo Finance
export default {
  async fetch(request) {
    const url = new URL(request.url);
    let targetUrl = url.searchParams.get('url') || url.pathname.replace('/', '');
    
    if (!targetUrl) {
      return new Response('CORS Proxy Ready. Usage: ?url=<target-url> or /<target-url>', { 
        status: 200,
        headers: { 'Access-Control-Allow-Origin': '*' }
      });
    }
    
    if (!targetUrl.startsWith('http')) {
      targetUrl = 'https://' + targetUrl;
    }
    
    try {
      const response = await fetch(targetUrl, {
        method: request.method,
        headers: { 'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64)' }
      });
      
      const body = await response.text();
      return new Response(body, {
        status: response.status,
        headers: {
          'Access-Control-Allow-Origin': '*',
          'Access-Control-Allow-Methods': 'GET, POST, OPTIONS',
          'Access-Control-Allow-Headers': '*',
          'Content-Type': response.headers.get('Content-Type') || 'application/json'
        }
      });
    } catch (e) {
      return new Response(JSON.stringify({ error: e.message }), {
        status: 500,
        headers: { 'Access-Control-Allow-Origin': '*', 'Content-Type': 'application/json' }
      });
    }
  }
};
```

5. Setelah deploy, dapatkan Worker URL: `https://cors-proxy.<username>.workers.dev`
6. Di aplikasi Lina AI, klik ⚙️ Settings → paste URL di "Custom CORS Proxy URL"
7. Format URL: `https://cors-proxy.username.workers.dev/?url=`

## Fitur Utama

### 📈 Analisis Teknikal
Indikator lengkap dengan formula yang benar:
- **EMA 50 & 200** (tren harga)
- **RSI 14** (Wilder's smoothing - kejenuhan)
- **MACD** (12, 26, 9) - momentum
- **Bollinger Bands** (20, 2) - volatilitas
- **Stochastic Oscillator** (14, 3) - osilator
- **ATR 14** (Average True Range) - volatilitas harga
- **VWAP** (dari FMP historical-price-eod)

### 📊 Analisis Fundamental
Data dari FMP `/stable/` endpoints (6 endpoint paralel, langsung tanpa proxy):
- `/stable/profile` - info perusahaan
- `/stable/ratios-ttm` - 30+ rasio finansial
- `/stable/key-metrics-ttm` - PEG, Graham, EV metrics
- `/stable/income-statement` - laporan laba rugi (4 tahun)
- `/stable/balance-sheet-statement` - neraca (4 tahun)
- `/stable/cash-flow-statement` - arus kas (4 tahun)
- `/stable/analyst-estimates` - forecast revenue/earnings 4 tahun

Rasio yang dihitung & dinilai (color-coded: green/yellow/red):
- **Valuasi**: P/E, P/B, EV/EBITDA, Dividend Yield, P/S, P/FCF, P/OCF
- **Valuasi Lanjutan**: PEG Ratio, Graham Number, EV/Sales, EV/FCF
- **Profitabilitas**: ROE, ROA, Net Margin, Operating Margin, Gross Margin, EBITDA Margin
- **Pertumbuhan**: Revenue Growth, Earnings Growth (YoY), EPS, Free Cash Flow
- **Kesehatan Finansial**: Debt/Equity, Current Ratio, Quick Ratio, Beta, Payout Ratio

### 📰 Analisis Berita & Sentimen
- **Berita saham terkait**: Yahoo Finance search (perlu CORS proxy)
- **Fallback**: Hacker News algolia (CORS-friendly, untuk tech stocks)
- **Berita makro/ekonomi**: Yahoo search index (^GSPC untuk US, ^JKSE untuk IDX)
- **Sentiment analysis** keyword-based (Indonesia + English):
  - Kamus positif: naik, untung, profit, surge, growth, bullish, dll.
  - Kamus negatif: turun, rugi, loss, fall, crash, bearish, dll.
  - Skor per artikel: -3 sampai +3, dinormalisasi ke -10..+10

### 🎯 Rekomendasi Akhir (Gabungan)
Weighted scoring:
- **Teknikal: 40%** (skor -10..10 dari 6 indikator)
- **Fundamental: 35%** (skor 0..10 dari 15+ rasio)
- **Berita: 25%** (skor -10..10 dari sentiment analysis)

Output:
- 🟢 STRONG BUY (>= 80/100)
- 🟩 BUY (60-79)
- ⚪ HOLD (40-59)
- 🟥 SELL (20-39)
- 🔴 STRONG SELL (< 20)

## Cara Pakai

1. Buka file `Lina AI Saham TV.html` di browser (Chrome/Firefox/Edge)
2. Klik tombol ⚙️ di header → masukkan FMP API key (wajib) + Custom CORS Proxy URL (opsional)
3. Pilih market (NASDAQ / IHSG / CRYPTO)
4. Ketik kode saham (contoh: AAPL, BBCA, BTCUSD)
5. Klik tombol **Analisis**
6. Lihat hasil di 3 tab: Teknikal, Fundamental, Berita
7. Rekomendasi final muncul di panel kanan atas

## Performance Tips

- **US stocks (AAPL, TSLA, NVDA, dll)**: Pakai FMP API → super cepat (~500ms per call, 6 parallel calls)
- **IDX/Crypto**: Butuh CORS proxy untuk Yahoo Finance. Tanpa proxy, hanya data chart Yahoo (kalau proxy up)
- **Berita**: Yahoo Finance search butuh CORS proxy. Untuk US tech stocks, otomatis fallback ke Hacker News (CORS-friendly)
- **Screener**: Auto-pause saat main analysis jalan (anti-proxy-overload)

## Markets yang Didukung

- **NASDAQ** (US stocks): AAPL, TSLA, NVDA, AMZN, MSFT, META, dll - **BEST experience dengan FMP**
- **IDX (IHSG)**: BBCA, BBRI, TLKM, GOTO, ASII, BMRI, dll - butuh CORS proxy
- **CRYPTO**: BTCUSD, ETHUSD, BNBUSD, SOLUSD, XRPUSD, DOGEUSD - butuh CORS proxy

## Teknologi

- Single HTML file (vanilla JS, Tailwind CSS via CDN)
- TradingView widget untuk chart real-time
- Lightweight Charts untuk chart teknikal
- **FMP API** (langsung via fetchDirect - CORS-supported, no proxy)
- Yahoo Finance API (via CORS proxy - allorigins/codetabs/custom Cloudflare Worker)
- Tanpa backend - 100% client-side

## Troubleshooting

### "Gagal fetch setelah 3x percobaan" di IDX/Crypto
Yahoo Finance butuh CORS proxy. Public proxy gratis sering down. **Solusi**: Deploy Cloudflare Worker (lihat instruksi di atas).

### Fundamental data tidak muncul / "Data fundamental terbatas"
- Pastikan FMP API key sudah dimasukkan di Settings (⚙️)
- Pastikan API key valid di https://financialmodelingprep.com/dashboard
- FMP free plan hanya support US stocks. Untuk IDX (BBCA.JK) butuh premium plan.

### TradingView chart error: "[object Object]"
Sudah diperbaiki di versi ini - duplicate studies dihapus.

### Berita tidak muncul
Yahoo Finance search butuh CORS proxy. Untuk US tech stocks, otomatis fallback ke Hacker News. Untuk saham non-tech, butuh Cloudflare Worker.

## Limitations

- FMP free plan: 250 request/hari, US stocks only (untuk fundamental lengkap)
- Yahoo Finance: butuh CORS proxy (public proxy sering down)
- Berita HN: hanya untuk tech stocks (AAPL, GOOGL, MSFT, dll)
- **Bukan saran investasi** - hanya alat bantu analisis

## Lisensi

MIT - bebas dipakai & dimodifikasi
