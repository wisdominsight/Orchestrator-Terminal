# DOKUMEN 4: EXECUTION PLAN — ORCHESTRATOR TERMINAL v0.1 (REVISI 2)

## TOOLS
- **Build:** Claude (1 file HTML, CSS+JS inline)
- **Test:** Chrome browser di laptop ADVAN (bukan Incognito)
- **Scan:** Kimi (konsistensi dokumen)

## KNOWLEDGE YANG DI-INJECT KE CLAUDE BUILDER
- SPEC FINAL REVISI 2 (Dok 1-3 gabungan)
- Execution Plan ini

---

## STEPS

### Step 0: Cek Ujung (Quick Cek)
Sebelum build, validasi:
- [ ] Bentuk akhir sudah terlihat jelas? → Layout 2 kolom, 4 textarea Darma per AI (dengan Focus Mode) di kiri, 4 tab panel AI di kanan
- [ ] Main action ≤ 3 tap/klik? → Copy for [AI] = 1 klik (input data bukan action)
- [ ] Semua data yang tampil sudah jelas sumbernya? → Semua dari textarea user input
- [ ] Ada minimal 1 error/empty scenario? → Panel kosong di-skip, Load JSON corrupt → alert
- [ ] Ada referensi UI/flow dari produk existing? → Tidak ada (workflow manual existing sebagai referensi)

### Step 1: Build HTML
- Claude generate 1 file HTML lengkap berdasarkan SPEC FINAL REVISI 2
- Output: `orchestrator-terminal.html`
- **Catatan implementasi:**
  - 4 AI: ChatGPT, Gemini, Z.ai, Claude. Rotasi fixed.
  - localStorage schema: 8 key data (4 Darma + 4 respon AI) + 4 key config (mode, custom instruction, active tab, last copy)
  - Load Session: validasi struktur JSON SEBELUM update state. Jika validasi gagal → alert error, state tidak disentuh
  - Clipboard: gunakan `document.execCommand('copy')` sebagai primary (file:// protocol)
  - Focus Mode: klik header → expand full, sembunyikan 3 lain + controls. Klik lagi → normal

### Step 2: Test di Device
- Buka file di Chrome laptop ADVAN
- Jalankan semua Acceptance Criteria (18 item):
  1. Layout 2 kolom
  2. 4 textarea Darma per AI → auto-save + 💾 indicator
  3. 4 panel AI tersimpan setelah refresh
  4. Copy for ChatGPT → rotasi: Darma utk GPT → Gemini → Z.ai → Claude → instruksi
  5. Copy for Gemini → rotasi: Darma utk Gemini → Z.ai → Claude → ChatGPT → instruksi
  6. Copy for Z.ai → rotasi: Darma utk Z.ai → Claude → ChatGPT → Gemini → instruksi
  7. Copy for Claude → rotasi: Darma utk Claude → ChatGPT → Gemini → Z.ai → instruksi
  8. Section kosong di-skip
  9. Semua kosong → clipboard: instruksi saja
  10. Dropdown mode + custom textarea
  11. Ctrl+1/2/3/4 shortcut
  12. Focus Mode: expand/minimize per blok
  13. Last Copy Indicator: badge LAST + persist setelah refresh
  14. Clear All + refresh → tetap kosong + last copy reset
  15. Save Session (termasuk last copy)
  16. Load Session (restore last copy)
  17. Load Session invalid → alert error, state tidak berubah
  18. Responsive < 768px
- **Setelah test Clear All:** refresh browser → semua field harus tetap kosong + badge LAST hilang
- Catat yang FAIL

### Step 3: Fix (jika ada FAIL)
- Laporkan bug ke Claude
- Claude fix → test ulang item yang FAIL
- Loop sampai semua PASS

### Step 4: Test Responsive
- Resize browser < 768px
- Cek layout stack vertikal
- Cek semua fungsi tetap jalan di mode narrow
- Cek Focus Mode di mode narrow

### Step 5: Test Session Save/Load
- Isi semua panel (4 AI + 4 Darma per AI + mode + custom instruction)
- Klik salah satu Copy (untuk set last copy indicator)
- Save Session → download JSON
- Clear All → Load Session → konfirmasi → **semua 12 field kembali** (4 panel AI + 4 Darma per AI + mode + custom instruction + active tab + last copy)
- Test Load dengan file JSON invalid → harus alert error, state tidak berubah

---

## ANTISIPASI KENDALA

| Kendala | Mitigasi |
|---|---|
| Clipboard API (`navigator.clipboard`) tidak jalan di `file://` protocol | Fallback: `document.execCommand('copy')` sebagai primary |
| localStorage tidak persist di Incognito | Sudah di-note di SPEC: "bukan Incognito" |
| Layout 2 kolom terlalu sempit di ADVAN | Test di Step 2. Jika tidak nyaman → adjust ratio atau min-width |
| File input untuk Load Session styling beda tiap browser | Sembunyikan native input, trigger via tombol custom |
| Logic rotasi copy 4 AI kompleks → bug urutan | Test keempat tombol Copy secara terpisah di Step 2 (item 4-7) |
| 4 textarea Darma di kolom kiri memakan ruang vertikal | Focus Mode: expand 1 blok, sembunyikan 3 lain |
| Ctrl+1/2/3 conflict dengan Chrome tab switching | Di file:// protocol dengan 1 tab biasanya tidak masalah. Jika conflict → fix saat testing |
