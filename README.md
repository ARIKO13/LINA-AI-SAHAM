# Lina AI - Analisis Saham (Teknikal + Fundamental + Berita)

Aplikasi web single-page untuk analisis saham pemula dengan 3 lapis analisis:
**Teknikal + Fundamental + Berita** yang menghasilkan rekomendasi **BUY / SELL / HOLD**.

## Fitur Utama

### 📈 Analisis Teknikal
Indikator lengkap dengan formula yang benar:
- **EMA 50 & 200** (tren harga)
- **RSI 14** (Wilder's smoothing - kejenuhan)
- **MACD** (12, 26, 9) - momentum
- **Bollinger Bands** (20, 2) - volatilitas
- **Stochastic Oscillator** (14, 3) - osilator
- **ATR 14** (Average True Range) - volatilitas harga

### 📊 Analisis Fundamental
Mengambil laporan keuangan dari:
- Yahoo Finance QuoteSummary (jika tersedia tanpa auth)
- **Financial Modeling Prep (FMP)** - 5 endpoint parallel:
  - Profile, Ratios TTM, Income Statement, Balance Sheet, Cash Flow
- Fallback: Yahoo Chart meta data (informasi dasar)

Rasio yang dihitung & dinilai:
- **Valuasi**: P/E, P/B, EV/EBITDA, Dividend Yield
- **Profitabilitas**: ROE, ROA, Net Margin, Operating Margin
- **Pertumbuhan**: Revenue Growth, Earnings Growth, EPS, Free Cash Flow
- **Kesehatan Finansial**: Debt/Equity, Current Ratio, Quick Ratio, Beta

### 📰 Analisis Berita & Sentimen
- **Berita saham terkait**: Yahoo Finance search (symbol)
- **Berita makro/ekonomi/politik**: Yahoo Finance search index (^GSPC untuk US, ^JKSE untuk IDX)
- **Sentiment analysis** keyword-based (Indonesia + English):
  - Kamus positif: naik, untung, profit, surge, growth, bullish, dll.
  - Kamus negatif: turun, rugi, loss, fall, crash, bearish, dll.
  - Skor per artikel: -3 sampai +3, dinormalisasi ke -10..+10

### 🎯 Rekomendasi Akhir (Gabungan)
Weighted scoring:
- **Teknikal: 40%** (skor -10..10)
- **Fundamental: 35%** (skor 0..10)
- **Berita: 25%** (skor -10..10)

Output:
- 🟢 STRONG BUY (>= 80/100)
- 🟩 BUY (60-79)
- ⚪ HOLD (40-59)
- 🟥 SELL (20-39)
- 🔴 STRONG SELL (< 20)

## Cara Pakai

1. Buka file `Lina AI Saham TV.html` di browser (Chrome/Firefox/Edge)
2. Pilih market (NASDAQ / IHSG / CRYPTO)
3. Ketik kode saham (contoh: AAPL, BBCA, BTCUSD)
4. Klik tombol **Analisis**
5. Lihat hasil di 3 tab: Teknikal, Fundamental, Berita
6. Rekomendasi final muncul di panel kanan atas

### Untuk Data Fundamental Lengkap
1. Daftar gratis di https://site.financialmodelingprep.com/register (250 request/hari)
2. Klik tombol ⚙️ di header
3. Masukkan FMP API key
4. Klik Simpan, lalu klik Analisis lagi

## Markets yang Didukung

- **NASDAQ**: AAPL, TSLA, NVDA, AMZN, MSFT, META, dll.
- **IDX (IHSG)**: BBCA, BBRI, TLKM, GOTO, ASII, BMRI, dll.
- **CRYPTO**: BTCUSD, ETHUSD, BNBUSD, SOLUSD, XRPUSD, DOGEUSD

## Teknologi

- Single HTML file (vanilla JS, Tailwind CSS via CDN)
- TradingView widget untuk chart real-time
- Lightweight Charts untuk chart teknikal
- Yahoo Finance API (via CORS proxy: allorigins, codetabs, thingproxy)
- FMP API untuk fundamental data lengkap
- Tanpa backend - 100% client-side

## Catatan

- Yahoo Finance API sekarang butuh auth (crumb) untuk beberapa endpoint, jadi beberapa data fundamental mungkin terbatas tanpa FMP API key
- CORS proxy gratis (allorigins, codetabs) bisa intermittent - app handle dengan retry + fallback
- **Bukan saran investasi** - hanya alat bantu analisis

## Lisensi

MIT - bebas dipakai & dimodifikasi
