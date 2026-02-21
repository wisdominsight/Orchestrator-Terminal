# ORCHESTRATOR TERMINAL v0.1 — SPEC FINAL (REVISI 2)

## APA INI
1 file HTML lokal (CSS+JS inline) = command center diskusi multi-AI (4 AI). Gantikan workflow manual (tab browser + notepad) jadi 1 tampilan terpadu.

## CONSTRAINT
- Single file HTML. Zero library/framework/CDN/backend/API
- Offline (buka langsung dari file explorer di Chrome)
- Device: laptop ADVAN layar kecil

---

## LAYOUT

```
┌──────────────────────┬──────────────────────┐
│  KOLOM KIRI (40%)    │  KOLOM KANAN (60%)   │
│                      │                      │
│  [Respon utk GPT]  ⤢│  [Tab: GPT|Gem|Z|Cld]│
│  [Respon utk Gem]  ⤢│  [Textarea respon]   │
│  [Respon utk Z.ai] ⤢│                      │
│  [Respon utk Cld]  ⤢│                      │
│                      │                      │
│  [Dropdown Mode]     │                      │
│  [Copy for GPT]      │                      │
│  [Copy for Gemini]   │                      │
│  [Copy for Z.ai]     │                      │
│  [Copy for Claude]   │                      │
│  [Clear All]         │                      │
│  [💾 Save] [📂 Load] │                      │
└──────────────────────┴──────────────────────┘
```

⤢ = Focus Mode toggle (klik header untuk expand/minimize)

Responsive: width < 768px → stack vertikal (input atas, panel bawah).

---

## KOMPONEN

### 1. Textarea Respon Darma per AI (Kolom Kiri — Atas)
4 textarea terpisah:
- "Respon untuk ChatGPT" — placeholder: "Respon/arahan untuk ChatGPT..."
- "Respon untuk Gemini" — placeholder: "Respon/arahan untuk Gemini..."
- "Respon untuk Z.ai" — placeholder: "Respon/arahan untuk Z.ai..."
- "Respon untuk Claude" — placeholder: "Respon/arahan untuk Claude..."

Masing-masing:
- Resizable, auto-save localStorage, debounce 2 detik
- Indicator: 💾 muncul saat saving, hilang setelah saved
- Opsional — boleh kosong (di-skip saat copy)
- Warna accent: GPT=#74aa9c, Gemini=#8b9cf7, Z.ai=#b8c94e, Claude=#d4956a

**TIDAK ADA "Input Darma umum".** Semua respon Darma selalu kontekstual per AI.

### 2. Focus Mode (per blok Darma)
- Klik header/label AI → textarea expand full height, 3 blok AI lain + controls tersembunyi
- Klik lagi → kembali ke tampilan normal
- Auto-focus textarea saat expand
- Tujuan: ruang tulis luas untuk respon panjang + mudah baca ulang

### 3. Dropdown Mode Instruksi
| Mode | Teks yang di-append ke output |
|---|---|
| Anti-Yes Man | "jangan asal yes man. apapun pendapatmu yang terbaik meski beda, aku sangat terbuka. jadilah obyektif, kritis, bijak dan ahli-berpengalaman dalam domain ini. bagaimana menurutmu?" |
| Konvergensi | "jadilah obyektif, kritis, bijak dan ahli-berpengalaman dalam domain ini. bagaimana menurutmu?" |
| Custom | textarea kecil muncul DI BAWAH dropdown (hidden saat mode lain). Isi persist di localStorage — tidak pernah auto-hapus meski ganti mode |

### 4. Tombol "Copy for [AI]" — LOGIC ROTASI

4 tombol: Copy for ChatGPT | Copy for Gemini | Copy for Z.ai | Copy for Claude

**Rotasi fixed:** ChatGPT → Gemini → Z.ai → Claude → ChatGPT → dst.

**Logic Copy for [Target]:**

1. **Pertama:** Respon Darma untuk [Target] (jika ada — ini respon terbaru yang belum Target baca)
2. **Lalu:** Rotasi AI lain, dimulai dari AI **setelah Target** dalam urutan rotasi. Untuk tiap AI:
   - Respon AI tersebut
   - Diikuti respon Darma untuk AI tersebut (jika ada)
3. **Terakhir:** Mode Instruksi (selalu di paling bawah)

**Urutan rotasi per target:**
- Copy for ChatGPT → Darma utk GPT, lalu: Gemini + Darma utk Gemini, Z.ai + Darma utk Z.ai, Claude + Darma utk Claude, Instruksi
- Copy for Gemini → Darma utk Gemini, lalu: Z.ai + Darma utk Z.ai, Claude + Darma utk Claude, ChatGPT + Darma utk GPT, Instruksi
- Copy for Z.ai → Darma utk Z.ai, lalu: Claude + Darma utk Claude, ChatGPT + Darma utk GPT, Gemini + Darma utk Gemini, Instruksi
- Copy for Claude → Darma utk Claude, lalu: ChatGPT + Darma utk GPT, Gemini + Darma utk Gemini, Z.ai + Darma utk Z.ai, Instruksi

**Contoh Copy for Gemini (semua terisi):**
```
aku (darma):
{respon Darma untuk Gemini}
---
respon z.ai:
{isi panel Z.ai}
---
aku (darma):
{respon Darma untuk Z.ai}
---
respon claude:
{isi panel Claude}
---
aku (darma):
{respon Darma untuk Claude}
---
respon chatgpt:
{isi panel ChatGPT}
---
aku (darma):
{respon Darma untuk ChatGPT}
---
{Instruksi Akhir dari dropdown}
```

**Rules:**
- Separator `---` antar setiap section
- SKIP section kosong (panel AI kosong atau respon Darma kosong → tidak masuk output, separator juga skip)
- Semua panel AI kosong + semua respon Darma kosong → output: instruksi akhir saja
- Label format: `respon [nama ai]:` dan `aku (darma):`
- Setelah copy → tombol berubah "✓ Copied!" selama 1.5 detik

### 5. Last Copy Indicator
- Badge "LAST" muncul di tombol Copy yang terakhir diklik
- Persist di localStorage (tetap tampil setelah refresh)
- Di-reset saat Clear All
- Tersimpan dan ter-restore saat Save/Load Session
- Tujuan: saat kembali dari jeda, langsung tahu AI mana yang terakhir dikirimi

### 6. Clear All
- Konfirmasi: "Yakin hapus semua?" (satu-satunya aksi yang butuh konfirmasi selain Load Session)
- Hapus semua textarea + localStorage + last copy indicator + reset focus mode

### 7. 💾 Save Session
- Download semua state sebagai 1 file JSON
- Nama file otomatis: `OT-session-YYYY-MM-DD-HHmm.json`
- TANPA konfirmasi (non-destructive)
- Isi JSON: semua panel respon AI, semua respon Darma per AI, mode, custom instruction, active tab, last copy target

### 8. 📂 Load Session
- Upload file JSON → isi semua panel otomatis
- Validasi struktur JSON SEBELUM update state. Jika validasi gagal → alert error, state TIDAK berubah
- DENGAN konfirmasi: "Ini akan menimpa semua data saat ini. Lanjutkan?"
- Setelah load → update localStorage + restore last copy indicator

### 9. Panel Respon AI (Kolom Kanan)
- 4 tab: ChatGPT | Gemini | Z.ai | Claude
- Klik tab → tampilkan textarea AI tersebut
- Setiap textarea editable + auto-save localStorage
- Keyboard shortcut: Ctrl+1=ChatGPT, Ctrl+2=Gemini, Ctrl+3=Z.ai, Ctrl+4=Claude
- Shortcut HANYA aktif saat focus TIDAK di textarea

---

## LOCALSTORAGE SCHEMA

```javascript
const STORAGE_KEYS = {
  DARMA_GPT: 'ot_darma_gpt',
  DARMA_GEMINI: 'ot_darma_gemini',
  DARMA_ZAI: 'ot_darma_zai',
  DARMA_CLAUDE: 'ot_darma_claude',
  RESP_GPT: 'ot_resp_gpt',
  RESP_GEMINI: 'ot_resp_gemini',
  RESP_ZAI: 'ot_resp_zai',
  RESP_CLAUDE: 'ot_resp_claude',
  MODE: 'ot_mode',
  CUSTOM: 'ot_custom_instruction',
  TAB: 'ot_active_tab',
  LAST_COPY: 'ot_last_copy'
};
```

Auto-save: setiap textarea `input` event, debounced 2 detik.
Auto-load: saat halaman dibuka, isi semua dari localStorage.

---

## ACCEPTANCE CRITERIA

- [ ] Buka HTML di Chrome → layout 2 kolom benar
- [ ] Ketik di textarea Darma per AI → auto-save (refresh → teks masih ada). 💾 indicator muncul saat saving
- [ ] Paste respon di panel GPT/Gemini/Z.ai/Claude → masing-masing tersimpan
- [ ] Copy for ChatGPT → clipboard: Darma utk GPT → Gemini + Darma utk Gemini → Z.ai + Darma utk Z.ai → Claude + Darma utk Claude → instruksi
- [ ] Copy for Gemini → clipboard: Darma utk Gemini → Z.ai + Darma utk Z.ai → Claude + Darma utk Claude → ChatGPT + Darma utk GPT → instruksi
- [ ] Copy for Z.ai → clipboard: Darma utk Z.ai → Claude + Darma utk Claude → ChatGPT + Darma utk GPT → Gemini + Darma utk Gemini → instruksi
- [ ] Copy for Claude → clipboard: Darma utk Claude → ChatGPT + Darma utk GPT → Gemini + Darma utk Gemini → Z.ai + Darma utk Z.ai → instruksi
- [ ] Section kosong (panel AI atau respon Darma) tidak masuk clipboard
- [ ] Semua panel + semua respon Darma kosong → clipboard: instruksi saja
- [ ] Dropdown: Anti-Yes Man → instruksi berubah. Custom → textarea muncul. Mode lain → textarea hilang, isi custom persist
- [ ] Ctrl+1/2/3/4 → tab berpindah (hanya di luar textarea)
- [ ] Focus Mode: klik header AI → textarea expand full, 3 lain tersembunyi. Klik lagi → kembali normal
- [ ] Last Copy Indicator: badge "LAST" muncul di tombol terakhir diklik, persist setelah refresh
- [ ] Clear All → konfirmasi → semua bersih + localStorage cleared + last copy reset. Refresh → tetap kosong
- [ ] 💾 Save Session → download JSON (nama auto, tanpa konfirmasi, termasuk last copy)
- [ ] 📂 Load Session → upload JSON → konfirmasi → semua panel terisi + last copy restored
- [ ] Load Session dengan file invalid/corrupt → alert error, state TIDAK berubah
- [ ] Responsive: < 768px → stack vertikal

---

## SCOPE NEGATIF (TIDAK masuk v0.1)

- ❌ API call ke AI manapun
- ❌ History/versioning
- ❌ Export selain JSON session
- ❌ Dark mode / tema
- ❌ Drag-and-drop
- ❌ Multi-session/project
- ❌ Round counter
- ❌ Context tracking otomatis
- ❌ Input Darma umum (semua respon Darma kontekstual per AI)

---

## INSTRUKSI UNTUK AI BUILDER

1. Baca seluruh dokumen ini
2. Output = 1 file HTML lengkap (CSS+JS inline), copas-ready
3. Zero library/framework
4. Semua Acceptance Criteria harus terpenuhi
5. Jangan tambah fitur di luar scope
6. Cara pakai: simpan sebagai `.html`, buka di Chrome (bukan Incognito)
7. Clipboard: gunakan `document.execCommand('copy')` sebagai primary (file:// protocol). Fallback ke `navigator.clipboard` jika tersedia
8. Catatan implementasi Load Session: validasi struktur JSON SEBELUM update state. Jika gagal → alert error, jangan sentuh state
