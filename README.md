# 🔓 Cloudflare Turnstile Solver

Solver Cloudflare Turnstile otomatis menggunakan `puppeteer-real-browser`. Mendukung dua mode solve, screen recording, dan proxy.

## ✨ Fitur

- Solve Cloudflare Turnstile secara otomatis
- **Mode Sitekey** — inject fake page dengan sitekey eksplisit
- **Mode Page** — solve langsung dari halaman target (auto-detect turnstile)
- Screen recording tiap sesi (`.mp4`) via `puppeteer-screen-recorder`
- Support proxy dengan autentikasi
- Output token dalam format JSON

## 📦 Instalasi

```bash
npm install puppeteer-real-browser puppeteer-screen-recorder
```

## 🔧 Penggunaan sebagai Module

```js
const TurnstileSolver = require('./turnstile');

const solver = new TurnstileSolver({
  timeout: 60000,       // ms tunggu token
  record: true,         // aktifkan screen recording
  recordDir: './recordings',
  // proxy: { host: '...', port: 8080, username: 'user', password: 'pass' },
  width: 1280,
  height: 720,
});

await solver.initialize();

// Mode 1: pakai sitekey eksplisit
const result = await solver.solve('https://example.com', '0x4AAAAAAA...');

// Mode 2: langsung dari halaman target (tanpa sitekey)
const result = await solver.solve('https://example.com');

console.log(result);
await solver.cleanup();
```

### Options Konstruktor

| Option | Tipe | Default | Deskripsi |
|--------|------|---------|-----------|
| `timeout` | `number` | `60000` | Batas waktu tunggu token (ms) |
| `record` | `boolean` | `false` | Aktifkan screen recording |
| `recordDir` | `string` | `./recordings` | Folder output recording |
| `proxy` | `object` | `null` | `{ host, port, username, password }` |
| `width` | `number` | `1280` | Viewport & recording width |
| `height` | `number` | `720` | Viewport & recording height |

### Method

| Method | Deskripsi |
|--------|-----------|
| `initialize()` | Launch browser (opsional, auto-called saat solve) |
| `solve(url, siteKey?)` | Solve turnstile — pakai sitekey kalau diisi, auto-detect kalau tidak |
| `solveWithSitekey(url, siteKey)` | Inject fake page dengan sitekey eksplisit |
| `solveFromPage(url)` | Solve langsung dari halaman target |
| `cleanup()` | Tutup browser |

## 📤 Output

Berhasil:

```json
{
  "success": true,
  "creator": "XAi Community",
  "token": "0.eyJhbGci...",
  "time": 4.231
}
```

Gagal:

```json
{
  "success": false,
  "error": "Token invalid or empty",
  "time": 60.012
}
```

## 📁 Recording

Kalau `record: true`, setiap sesi disimpan ke `recordDir` dengan nama:

```
recordings/
├── sitekey_0x4AAAAAAA_1747123456789.mp4   ← mode sitekey
└── page_solve_1747123456789.mp4            ← mode page
```

## ⚙️ Cara Kerja

**Mode Sitekey (`solveWithSitekey`)**
1. Request interception aktif — halaman target dibalas dengan fake HTML berisi widget Turnstile
2. `puppeteer-real-browser` + `turnstile: true` handle solve otomatis
3. Token diambil dari input hidden `[name="cf-response"]`

**Mode Page (`solveFromPage`)**
1. Script inject via `evaluateOnNewDocument` untuk polling `window.turnstile.getResponse()`
2. Token yang ditemukan ditulis ke input hidden `[name="cf-response"]`
3. `waitForSelector` menunggu sampai token siap

## 📋 Requirements

- Node.js >= 18
- `ffmpeg` (dibutuhkan `puppeteer-screen-recorder` untuk encode MP4)
- Xvfb (untuk headless di Linux — dihandle otomatis oleh `puppeteer-real-browser`)

## ⚠️ Disclaimer

Tool ini dibuat untuk keperluan edukasi dan testing. Penggunaan untuk bypass proteksi tanpa izin pemilik situs adalah tanggung jawab pengguna sepenuhnya.

## 📜 License

MIT License

Copyright (c) 2025 XAi Community

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
