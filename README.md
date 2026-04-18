# ⚡ CipherStack

> **Node-based cascade encryption builder — runs 100% in the browser, zero dependencies, zero build step.**

[![Single File](https://img.shields.io/badge/delivery-single%20HTML%20file-orange)](cipherstack.html)

---

## What Is Cascade Encryption?

Cascade encryption passes plaintext through a **chain of independent algorithms in sequence**. Each node's output becomes the next node's input. The final ciphertext has been transformed by every layer — breaking it requires breaking all of them.

CipherStack lets you **visually build, configure, and run** these pipelines in real time.

---

## Quick Start

**No install. No build. Just open.**

```bash
# Option 1 — serve locally (recommended; required for AI assistant)
python3 -m http.server 8080
# then visit: http://localhost:8080/cipherstack.html

# Option 2 — open directly (all features except AI)
open cipherstack.html       # macOS
start cipherstack.html      # Windows
```

---

## Deploy in 30 Seconds

**Deployed Link Preview:**
1. Go to : https://darkgreen-fox-338032.hostingersite.com/


**Netlify drag-and-drop:**
1. Go to [app.netlify.com/drop](https://app.netlify.com/drop)
2. Drag `cipherstack.html` onto the page — done.

**GitHub Pages:**
```bash
git init && git add cipherstack.html README.md
git commit -m "feat: CipherStack"
git push origin main
# Settings → Pages → Deploy from main branch
```

**Vercel:**
```bash
npx vercel --prod cipherstack.html
```

---

## Cipher Library

### Configurable (mandatory — user sets keys/params)

| Cipher | Config | Notes |
|--------|--------|-------|
| **Caesar Shift** | shift: −25…25 | Rotates alpha chars by N mod 26. Non-alpha unchanged. |
| **XOR Cipher** | key: string | UTF-8 byte-level XOR with repeating key. Output: lowercase hex. Symmetric — XOR is its own inverse. |
| **Vigenère** | keyword: alpha string | Polyalphabetic: C=(P+K) mod 26. Keyword index advances on alpha only. |
| **Rail Fence** | rails: 2–10 | Zigzag transposition across N rails. Pure rearrangement — no char altered. |
| **Columnar Transposition** | key: alpha string | Rows of key.length columns; read in alphabetical column order. Null-byte padding avoids stripping legit trailing chars. |
| **AES-256** | passphrase, mode: CBC\|GCM | **Real AES-256** via `window.crypto.subtle` (Web Crypto API). PBKDF2 key derivation (100k iterations, SHA-256). GCM mode is authenticated encryption. Output: Base64. |

### Bonus (config-free)

| Cipher | Notes |
|--------|-------|
| **Base64** | `TextEncoder → btoa`. Full Unicode safe. |
| **ROT13** | Caesar shift-13. Self-inverse. |
| **Reverse** | Spread to code points (`[...input]`) — handles emoji correctly. |

---

## AES-256 — Real Cryptography

Previous versions faked AES with RC4. This version uses the browser's **native `crypto.subtle`**:

```js
// PBKDF2: passphrase → 256-bit AES key
const key = await crypto.subtle.deriveKey(
  { name:'PBKDF2', salt, iterations:100_000, hash:'SHA-256' },
  baseKey,
  { name:'AES-CBC', length:256 },
  false, ['encrypt','decrypt']
);

// Real AES-256-CBC encryption
const cipherBuf = await crypto.subtle.encrypt(
  { name:'AES-CBC', iv },
  key,
  new TextEncoder().encode(plaintext)
);
```

- **PBKDF2** with 100,000 iterations — brute-force resistant key derivation
- **Deterministic IV** from passphrase — reproducible decrypt without storing IV
- **CBC** for compatibility, **GCM** for authenticated encryption (tamper-detectable)
- The pipeline engine uses `await Promise.resolve(cipher.encrypt(...))` — sync and async ciphers work identically

---

## Architecture

Single HTML file. Internal structure mirrors a proper multi-file project:

```
cipherstack.html
├── CIPHERS {}          One object per cipher — uniform async interface
├── engineEncrypt()     Async forward traversal (node[0]→node[n-1])
├── engineDecrypt()     Async backward traversal (node[n-1]→node[0])
├── STATE {}            Plain JS object — no framework
├── render()            Imperative DOM; re-renders on state change
└── AI Panel            Gemini 2.5 Flash REST — pipeline-context-aware
```

**Cipher interface contract** — adding a new cipher never requires changing the engine:

```js
{
  id: string,
  name: string,
  configSchema: ConfigField[],   // auto-renders the config form
  defaultConfig: object,
  encrypt(input, config): string | Promise<string>,
  decrypt(input, config): string | Promise<string>
}
```

---

## Features

| Feature | Detail |
|---------|--------|
| Drag-to-reorder | Native HTML5 drag API + ↑/↓ buttons |
| Minimum 3 nodes | Run disabled with tooltip below threshold |
| Intermediate I/O | Every node shows received input and produced output |
| Live preview | Debounced 300ms re-run on keystroke |
| Error isolation | One failing node shows red border; pipeline continues |
| Export / Import | Full pipeline → `cipherstack-pipeline.json` |
| Round-Trip Test | Encrypt → decrypt → PASS/FAIL badge |
| Shareable URL | Pipeline config Base64-encoded in URL hash |
| Ctrl+Z Undo | 30-level history stack |
| Demo pipelines | Classic 3-cipher / Strong 5-cipher / Fun Mix |
| AI Assistant | Gemini 2.5 Flash — explain, suggest, test |

---

## AI Assistant

Uses **Gemini 2.5 Flash** (Google's free-tier model, 500 req/day).

- Explains any cipher in the library
- Generates interesting demo texts to test pipelines
- Recommends cipher combinations
- Aware of current pipeline state (injects live context)
- Full conversation memory (last 10 exchanges)

**CORS note:** Works on any `http://` or `https://` origin. Blocked on `file://` URLs — serve locally or deploy.

---

## Correctness Checklist

- [x] Caesar `'Hello World!'` +13 → decrypt → original (non-alpha unchanged)
- [x] XOR with emoji input (`🔐`) → round-trip
- [x] Vigenère `keyword='key'`, `input='ATTACKATDAWN'` → round-trip
- [x] Rail Fence 3 rails → round-trip
- [x] Columnar `key='ZEBRAS'` → round-trip (null-byte padding, not X)
- [x] AES-256-CBC `key='mykey'` → Base64 → decrypt → original
- [x] AES-256-GCM → round-trip
- [x] 3-node: Caesar + XOR + Vigenère → round-trip
- [x] 5-node: all configurable → round-trip
- [x] Empty string on all ciphers — no crash
- [x] Single character on all ciphers — round-trip
- [x] Reorder nodes → results update
- [x] Remove middle node → remaining nodes chain correctly
- [x] <3 nodes → Run button disabled

---

## Tech Stack

| | |
|--|--|
| UI | Vanilla HTML / CSS / JS |
| Fonts | Inter + JetBrains Mono (Google Fonts) |
| Cryptography | `window.crypto.subtle` (Web Crypto API — built into every browser) |
| AI | Gemini 2.5 Flash REST API |
| Animations | Pure CSS keyframes + custom properties |
| Build | None — single static file |

---

