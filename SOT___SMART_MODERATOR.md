# SOT – SMART MODERATOR V1.3  
**Sumber:** SOT Arahan Penting + Otomatisasi Orchestrator Terminal v1.0 + Diskusi Panel AI  
**Panel AI Aktif:** ChatGPT, Gemini, Claude, DeepSeek  
**AI Tidak Terlihat di Log:** Kimi, Qwen (ketidakhadiran tidak menggugurkan konsensus – tercatat di Friction Log)  
**Status:** **CLOSED – Siap Implementasi & Dry-Run**  
**Versi:** V1.3 (Final)

---

## DAFTAR ISI
1. Landasan – Adaptive Product Planning Framework  
2. Masalah Utama & Tujuan Smart Moderator  
3. Konsep & Definisi Smart Moderator  
4. Arsitektur – Local Feeder + Cloud Brain (Hybrid)  
5. Penolakan Pendekatan Lain  
6. Webhook Data Contract – Local Feeder ke GAS  
7. Kontrak Data Ledger – Struktur Google Sheets (5 Tab)  
8. Routing Logic GAS – Aturan Lengkap (termasuk semua patch)  
9. Mekanisme Orkestrasi – Detail Alur  
10. Prompt Master Meta-LLM Filter – Final  
11. AI Prompt Packet Contract  
12. Secret Storage & Keamanan Kredensial  
13. Change Governance & Impact Log  
14. Biaya, Keamanan & Reliabilitas  
15. Role & Permission  
16. Obsidian – Posisi Final  
17. Backlog Arsitektur (Architecture-Ready Later)  
18. Friction Log Komprehensif  
19. Langkah Selanjutnya (Dry-Run)

---

## 1. LANDASAN – ADAPTIVE PRODUCT PLANNING FRAMEWORK

Smart Moderator dibangun di atas kerangka kerja **5 Keputusan Vital + Change Governance** yang sudah disepakati Darma sebelumnya. Seluruh keputusan teknis harus tunduk pada struktur ini.

**5 Keputusan Vital:**
1. **Problem–Value Anchor** + Domain Assumption Check  
2. **Capability Strategy** (Core Now / Strategic Advance / Architecture-Ready / Reject, dengan 3 filter)  
3. **Flow–Rules–State Lock** + Role & Permission  
4. **Evolvable Architecture + Data/API Contract** (satu kesatuan)  
5. **Risk–Validation Gate** + Operational Constraint  

**Change Governance:** Setiap perubahan besar wajib dicatat dengan **Impact Log 3 Baris** (Kontrak Lama → Kontrak Baru → Blast Radius).

**Penerapan untuk proyek Smart Moderator sendiri:**
- **Domain Assumption Check:**  
  - *Diketahui:* Tool dipakai Darma sendiri sebagai AI Orchestrator, untuk diskusi multi-AI sebelum build.  
  - *Belum diketahui:* Biaya API riil per sesi, limit payload GAS, reliabilitas JSON filter, batas nyata eksekusi 6 menit GAS.  
  - *Perlu diverifikasi:* Dry-run 1–2 AI, payload besar, Telegram pause/resume, keamanan Local Feeder.  
- **Capability Strategy:** Semua fitur V1 = Core Now (default, tool internal tanpa user eksternal).

---

## 2. MASALAH UTAMA & TUJUAN SMART MODERATOR

**Masalah:** Diskusi multi-AI saat ini 100% manual (Darma menyalin-tempel respons antar AI). Menghabiskan waktu dan membosankan.  
**Tujuan:** Membangun **Smart Moderator** yang mampu:
1. Membaca file lokal (SoT, bahan diskusi) tanpa tempel manual.  
2. Mengurangi pengulangan – AI berikutnya tidak membaca ulang hal yang sudah diketahui.  
3. Hanya bertanya ke Darma untuk **keputusan vital/inti**, bukan teknis turunan — dan wajib memakai bahasa Indonesia sederhana yang mudah dipahami orang awam, bukan jargon teknis.  
4. Tetap menghasilkan validasi silang dan debat terarah.  

**Prinsip Darma (tidak dapat ditawar):** Darma tetap AI Orchestrator, value creation maksimal, build kompleks selama opsi terbaik, kendali penuh di keputusan vital, teknis boleh diotomatisasi.

---

## 3. KONSEP & DEFINISI SMART MODERATOR

### 3.1 Bukan Multi-Agent Biasa  
Multi-agent biasa: respons mentah dilempar antar AI → makin panjang, makin boros, AI cenderung yes-man.  
Smart Moderator: moderator pintar menyaring, hanya kirim **diff** (perbedaan/sanggahan) ke AI berikutnya, mencatat keputusan, dan hanya memanggil Darma saat menyentuh keputusan vital.

### 3.2 6 Fungsi Inti Moderator  
1. **Context Filter** – menentukan bagian mana yang perlu dibaca AI berikutnya.  
2. **Decision Ledger** – mencatat keputusan yang sudah/belum terkunci dan yang berubah.  
3. **Duplication Guard** – mencegah AI mengulang argumen yang sudah dibahas.  
4. **Vital Question Gate** – berhenti dan tanya Darma jika menyentuh keputusan vital, dengan bahasa awam, dampak praktis, dan pilihan jawaban sederhana.  
5. **Technical Autopilot** – pembahasan teknis turunan berjalan otomatis.  
6. **Final Adversarial Review** – 1–2 putaran terakhir untuk mencari celah, miss, dan inkonsistensi.

### 3.3 Definisi Vital vs Autopilot  

| Kategori | Cakupan | Aksi Sistem |
|----------|---------|-------------|
| **VITAL** | Problem–Value Anchor, kategori fitur, business rules, role/permission, operational constraint (hosting, budget, keamanan), perubahan arsitektur besar | **Stop, tanya Darma via Telegram** |
| **AUTOPILOT** | Detail teknis turunan: skema DB, sintaks, struktur folder, testing, dokumentasi, unit test yang tidak mengubah kontrak | **Jalan otomatis tanpa interupsi** |

---

## 4. ARSITEKTUR – LOCAL FEEDER + CLOUD BRAIN (HYBRID)

### 4.1 Diagram Alur  

```text
[Terminal Agent: Claude Code / Codex CLI]
        │ Baca folder proyek lokal, SoT, bahan diskusi
        │ Hasilkan ringkasan terstruktur (bukan kode mentah penuh)
        │ Tampilkan preview → minta persetujuan Darma (Y/N)
        ▼
[Webhook POST ke GAS]  ← Payload Contract (§6)
        │
        ▼
[Google Apps Script (GAS) — OTAK ORKESTRASI]
   ├── Validasi payload (CacheService anti-duplikasi)
   ├── Rotasi AI (Kimi → Qwen → Gemini → Claude → DeepSeek)
   ├── Panggil API LLM + Meta-LLM Filter
   ├── Routing Logic (Auto-Promote, Vital Gate, Stuck Detection, Max Rounds)
   ├── Error Handling API & JSON Filter
   ├── Concurrency Control (LockService)
   └── Alarm + Laporan Telegram tiap putaran
        │
        ▼
[Google Sheets — DATABASE VISUAL & AUDIT]
   ├── RAW_ARCHIVE (teks mentah)
   ├── CLAIM_LEDGER (gelanggang debat)
   ├── DECISION_LEDGER (keputusan terkunci)
   ├── VITAL_TRIGGER (antarmuka Darma)
   └── SESSION_STATE (konfigurasi + pointer resume)
        │
        ▼
[Telegram Bot — KENDALI JARAK JAUH]
   ├── Notifikasi keputusan vital & error
   ├── Perintah: /pause, /lanjut, /stop, /status
   └── Persetujuan: A/B untuk keputusan vital, LANJUT/REVISI/STOP untuk error atau friction
```

### 4.2 Pembagian Tanggung Jawab  

| Komponen | Peran | Alasan |
|----------|-------|--------|
| **Terminal Agent** (Claude Code/Codex) | **Local Context Feeder** – membaca folder proyek lokal di awal sesi, menghasilkan ringkasan terstruktur. **TIDAK** menjadi runtime 24/7. | Paling kuat baca lokal, hemat copas. Tapi mati saat laptop mati. |
| **GAS + Google Sheets** | **Cloud Brain** – otak orkestrasi, database visual, logging, audit. | Zero cost hosting, familiar buat Darma, transparan, serverless, jalan 24/7. |
| **Telegram Bot** | **Kendali Jarak Jauh** – notifikasi, pause/lanjut/stop, persetujuan vital. | Ringan, gratis, real-time, sudah dikenal Darma. |

---

## 5. PENOLAKAN PENDEKATAN LAIN  

| Pendekatan Ditolak | Alasan Penolakan |
|--------------------|------------------|
| **Node.js/Python CLI sebagai core V1 penuh** | Kompleks, butuh UI/logging sendiri, tidak bisa kontrol HP, tidak 24/7, audit buruk. |
| **Framework berat** (CrewAI, LangGraph, Dify, Flowise) | Overkill untuk diskusi linear dengan logika rotasi spesifik. |
| **Semua AI dipanggil paralel** | Tidak terjadi debat/validasi silang karena AI tidak saling membaca. |
| **Runtime di terminal agent saja** | Bentrok dengan kendali HP, auto-stop 24 jam, dan kebutuhan serverless. |
| **Obsidian sebagai pengganti Sheets** | Tidak punya API real-time, tidak bisa monitoring live (warna, formula, trigger Telegram). |

---

## 6. WEBHOOK DATA CONTRACT – LOCAL FEEDER KE GAS

**Kesepakatan:** Payload JSON dari Local Feeder ke GAS via `doPost()`.

```json
{
  "Payload_ID": "string (hash unik, untuk deduplikasi CacheService)",
  "Session_ID": "string",
  "Project_Name": "string",
  "Source_Type": "LOCAL_FEEDER",
  "Context_Summary": "string (ringkasan terstruktur, bukan kode mentah)",
  "Files_Scanned": ["string"],
  "Files_Excluded": ["string"],
  "Created_At": "ISO 8601 timestamp"
}
```

**Aturan Tambahan:**
- Local Feeder wajib menampilkan **preview** dan minta **persetujuan Darma (Y/N)** sebelum mengirim webhook.  
- Jika `Context_Summary` terlalu besar, **Local Feeder** yang melakukan pemotongan, bukan GAS.  
- **File yang dikecualikan mutlak:** `.env`, `*.key`, `credentials.json`, `node_modules`, backup besar.

---

## 7. KONTRAK DATA LEDGER – STRUKTUR GOOGLE SHEETS (5 TAB)

### TAB 1: `RAW_ARCHIVE` (Arsip Mentah)  
- **Fungsi:** Menyimpan teks respons asli dari setiap AI, utuh. Untuk audit manual Darma jika terjadi halusinasi. **Tidak** dikirim ulang ke AI lain.  
- **Kolom:** `Log_ID/Payload_ID`, `Timestamp`, `Session_ID`, `AI_Actor`, `Full_Response`, `Token_Cost`.  
- **Edge Case:** Jika `Full_Response` > 50.000 karakter, GAS memotong dan mencatat di `VITAL_TRIGGER`.

### TAB 2: `CLAIM_LEDGER` (Gelanggang Debat) – 8 Kolom Final  
| Kolom | Diisi Oleh | Keterangan |
|-------|------------|------------|
| `Claim_ID` | **GAS** | Format: `CLM-{session_id}-{counter}` (hindari kolisi) |
| `Ref_ID` | Meta-LLM Filter | `Claim_ID` yang disanggah (jika `REBUTTAL`), `null` jika `NEW_IDEA` |
| `Kategori` | Meta-LLM Filter | `NEW_IDEA`, `REBUTTAL`, `TECHNICAL_DETAIL`, `RISK_WARNING` |
| `Konteks_Vital` | Meta-LLM Filter | `ANCHOR`, `CAPABILITY`, `FLOW`, `ARCH`, `OPERATIONAL`, `TECHNICAL` |
| `Isi_Singkat` | Meta-LLM Filter | Maksimal 15 kata, inti klaim tanpa basa-basi |
| `Status` | GAS | `DEBATABLE` (default), `REJECTED`, `PROMOTED` |
| `Round_Diajukan` | **GAS** | Dari `SESSION_STATE.Current_Round` |
| `Round_Terakhir_Sanggah` | **GAS** | Diupdate setiap ada `REBUTTAL` baru |

### TAB 3: `DECISION_LEDGER` (Jangkar Source of Truth) – 9 Kolom Final  
| Kolom | Keterangan |
|-------|------------|
| `Decision_ID` | ID unik keputusan |
| `Kategori_Vital` | Domain keputusan |
| `Kontrak_Terkunci` | Deskripsi keputusan |
| `Versi` | v1, v2, … |
| `Blast_Radius` | Komponen yang terdampak jika diubah |
| `Source_Claim_ID` | `Claim_ID` asal keputusan ini |
| `Approved_By` | `AUTO` atau `DARMA` |
| `Approved_At` | Timestamp persetujuan |
| `Decision_Status` | `ACTIVE` atau `SUPERSEDED` |

### TAB 4: `VITAL_TRIGGER` (Antarmuka Darma)  
| Kolom | Keterangan |
|-------|------------|
| `Trigger_ID` | ID unik |
| `Session_ID` | ID sesi |
| `Tipe` | `WAITING_DARMA` (keputusan vital) / `SYSTEM_FRICTION` (error/stuck/limit) |
| `Pemicu` | Deskripsi masalah |
| `Affects_Claim_ID` | (Opsional) `Claim_ID` yang terdampak, diisi GAS secara internal. Darma tidak perlu melihat atau mengisi ID teknis. |
| `Resolusi_Darma` | Untuk `WAITING_DARMA`: `A`, `B`, `REVISI [teks]`, `STOP`. Untuk `SYSTEM_FRICTION`: `LANJUT`, `REVISI [teks]`, `STOP`. |

### TAB 5: `SESSION_STATE` (Konfigurasi + Pointer Resume) – Final  
| Kolom | Keterangan |
|-------|------------|
| `Session_ID` | ID sesi |
| `Mode` | `antiyesman`, `konvergensi`, `kristalisasi`, `custom` |
| `Max_Rounds` | Batas maksimal putaran penuh (default: 10) |
| `Custom_Instruction` | Instruksi khusus jika Mode = custom |
| `Current_Round` | Putaran saat ini (hanya naik setelah AI terakhir selesai) |
| `Current_AI_Turn` | AI yang sedang/selanjutnya dipanggil |
| `Discussion_State` | `EXPLORING`, `DISCUSSING`, `READY_TO_CLOSE`, `CLOSED` |
| `Runtime_State` | `WAITING_AI`, `WAITING_FILTER`, `WAITING_DARMA`, `ERROR_RETRY`, `FINAL_REVIEW`, `DONE` |
| `Last_Updated` | Timestamp terakhir diupdate |

**Catatan:**  
- `Paused_Until_Darma` dihapus (redundan dengan `Runtime_State = WAITING_DARMA`).  
- `pause_keywords` digantikan oleh klasifikasi semantik `Konteks_Vital` (lebih akurat).  
- Pemisahan `Discussion_State` & `Runtime_State` karena dua dimensi berbeda: progres konten diskusi vs status eksekusi teknis.

---

## 8. ROUTING LOGIC GAS – ATURAN LENGKAP (TERMASUK SEMUA PATCH)

### 8.1 Auto-Promote (Otomatis, Tanpa Darma)  
**Syarat (semua harus terpenuhi):**  
- `Konteks_Vital == TECHNICAL`  
- `(Current_Round - Round_Terakhir_Sanggah) >= 2` — bertahan 2 putaran penuh tanpa `REBUTTAL` baru.  
- `Status == DEBATABLE`  

**Aksi:** `PROMOTED` → masuk `DECISION_LEDGER` (`Approved_By = AUTO`).

### 8.2 Vital Gate (Wajib Tanya Darma)  
**Syarat:** `Konteks_Vital != TECHNICAL` dan klaim membutuhkan keputusan.  

**Aksi:** `VITAL_TRIGGER` (Tipe: `WAITING_DARMA`), `Runtime_State = WAITING_DARMA`, notifikasi Telegram, pause sesi. GAS wajib menyimpan paket pertanyaan awam ke kolom `Pemicu` sebelum mengirim notifikasi Telegram, agar keputusan Darma bisa diaudit ulang.

**Aturan Bahasa Wajib untuk Darma:**  
Saat sistem bertanya ke Darma, dilarang memakai jargon teknis sebagai inti pertanyaan. Sistem wajib menerjemahkan keputusan teknis menjadi bahasa awam yang menjelaskan:
1. Keputusan apa yang perlu dipilih.
2. Dampak praktis dari setiap pilihan.
3. Risiko jika salah memilih.
4. Saran default AI.
5. Format balasan sederhana: `A`, `B`, `REVISI [teks]`, atau `STOP`.

**Contoh prinsip:**  
Bukan bertanya: "Apakah multi-session concurrency masuk Core Now?"  
Tetapi bertanya: "Apakah versi pertama cukup menangani 1 diskusi dulu, atau harus bisa banyak diskusi sekaligus? Jika banyak diskusi dari awal, sistem lebih rumit dan lebih rawan error. Saran aman: mulai dari 1 diskusi dulu."

### 8.3 Stuck Detection (Deteksi Macet)  
**Syarat:** Klaim `DEBATABLE` dengan `Konteks_Vital != TECHNICAL` — konflik berulang tanpa kemajuan.  

**Aksi:**  
- **N=2 putaran:** Warning via Telegram (sistem tetap jalan).  
- **N=3 putaran:** Pause/Stop, `VITAL_TRIGGER` (Tipe: `SYSTEM_FRICTION`), Darma wajib turun tangan.

### 8.4 Auto-REJECTED (Final – Sintesis Claude)  
- **Jalur TECHNICAL:** Jika sebuah `REBUTTAL` ter-Auto-Promote, GAS otomatis set klaim yang disanggah (`Ref_ID`) → `REJECTED`.  
- **Jalur VITAL:** TIDAK ADA auto-reject. Darma cukup memilih `A`, `B`, `REVISI [teks]`, atau `STOP`. GAS yang memetakan pilihan Darma ke `Affects_Claim_ID` secara internal; Darma tidak perlu menyebut `Claim_ID`.  
- Tidak ada `Rebuttal_Type` — pertahankan prinsip boring.

### 8.5 Cek Batas Putaran (`Max_Rounds`)  
- `Max_Rounds` = jumlah putaran penuh (satu putaran = semua 5 AI selesai).  
- `Current_Round` hanya naik setelah DeepSeek (AI terakhir) selesai.  
- Sebelum memulai round baru, GAS cek: `Current_Round >= Max_Rounds`?  
  → **Ya:** `Runtime_State = WAITING_DARMA`, `VITAL_TRIGGER` (Tipe: `SYSTEM_FRICTION`, Pemicu: "Batas putaran tercapai").  
- **Final Review** = 1 putaran tambahan khusus **setelah** `Current_Round == Max_Rounds`, tidak dihitung ke `Max_Rounds`.

### 8.6 Deduplikasi & Concurrency  
- **LockService** – mencegah dua eksekusi tulis Sheet bersamaan.  
- **CacheService + `Payload_ID`** – menolak payload yang sudah diproses sebelumnya (idempotensi).

### 8.7 Error Handling API AI Utama (Patch Final)  
```
Saat AI utama gagal setelah retry 1x:
→ Runtime_State = ERROR_RETRY
→ RAW_ARCHIVE: insert row {AI_Actor: <nama AI>, Full_Response: "[ERROR - API GAGAL]", Token_Cost: 0}
→ VITAL_TRIGGER: Tipe=SYSTEM_FRICTION, Pemicu="<AI> gagal merespon 2x"
→ Telegram notif, pause

Resolusi_Darma untuk kasus ini:
- LANJUT  → skip giliran AI tersebut untuk round ini, Current_AI_Turn = AI berikutnya dalam rotasi
- REVISI[teks] → coba panggil AI yang sama lagi (misal Darma sudah perbaiki API key)
- STOP    → hentikan sesi
```

### 8.8 Error Handling Meta-LLM Filter  
```
try { JSON.parse(output) }
catch {
    retry 1x dengan prompt repair
    jika tetap invalid → VITAL_TRIGGER (Tipe: SYSTEM_FRICTION), pause
}
Ref_ID tidak valid → drop Ref_ID (jadi null) atau trigger friction, jangan diproses diam-diam
Output kosong → boleh [] (bukan error)
```

---

## 9. MEKANISME ORKESTRASI – DETAIL ALUR

### 9.1 Urutan Rotasi AI  
**Kimi → Qwen → Gemini → Claude → DeepSeek** (sequential, satu per satu).

### 9.2 Cara Memulai Sesi Baru  
- **Opsi utama (V1):** Darma menjalankan Local Feeder di terminal → ringkasan dikirim via webhook ke GAS.  
- **Opsi Architecture-Ready Later:** Mulai dari Telegram langsung (tanpa Local Feeder).

### 9.3 Dual-Layer Filter  

**Layer 1: Meta-LLM Filter** (LLM cepat/murah, misal Gemini Flash)  
- Input: `[RAW_INPUT]` (respons AI) + `[OPEN_CLAIMS]` (daftar klaim `DEBATABLE`).  
- Output: Array JSON 5 field per klaim (lihat §10).  
- Tidak menghasilkan narasi bebas.  

**Layer 2: GAS Query + Template (tanpa LLM)**  
- GAS query `WHERE Status=DEBATABLE` dari `CLAIM_LEDGER`.  
- Susun jadi **Diff Context** (teks bullet-point pendek) untuk AI berikutnya.  
- Tidak butuh LLM kedua — hemat biaya, deterministik.

### 9.4 Vital Question Gate  
Sistem **berhenti dan bertanya ke Darma** (via Telegram) jika mendeteksi perubahan pada:  
- Problem–Value Anchor  
- Kategori fitur (Core/Strategic/Reject)  
- Business rules (Flow–Rules–State)  
- Role & Permission  
- Operational constraint (budget, hosting, keamanan)  
- Perubahan besar pada arsitektur atau kontrak data  
- Atau ketika AI sengaja menulis `[TANYA DARMA]` / `[MINTA PERSETUJUAN]`

**Format Pertanyaan ke Darma:**  
Setiap pertanyaan vital ke Darma wajib memakai struktur berikut:

```text
KEPUTUSAN YANG PERLU DARMA PILIH:
[Jelaskan dengan bahasa awam, bukan jargon teknis]

PILIHAN A:
[Dampak praktis pilihan A]

PILIHAN B:
[Dampak praktis pilihan B]

RISIKO UTAMA:
[Apa yang bisa bermasalah jika salah memilih]

SARAN AI:
[Satu saran default yang paling aman]

BALAS DENGAN:
A / B / REVISI [teks] / STOP
```

**Larangan:**
Jangan bertanya ke Darma dengan istilah teknis mentah seperti `concurrency`, `architecture-ready`, `Core Now`, `schema`, `runtime`, `payload`, atau `state machine` tanpa penjelasan awam.

### 9.5 Technical Autopilot  
Jika diskusi hanya menyangkut `Konteks_Vital == TECHNICAL`, sistem **berjalan otomatis** tanpa interupsi Darma.

### 9.6 Adversarial Final Review (1–2 Putaran Terakhir)  
- Setelah konvergensi tercapai (atau `Current_Round == Max_Rounds`), 1 AI (DeepSeek) berperan sebagai **Devil's Advocate**.  
- Tugas: mencari celah keamanan, edge case fatal, inkonsistensi dengan SoT awal.  
- Hasil sanggahan masuk `CLAIM_LEDGER` sebagai `REBUTTAL`.  
- **Final Review adalah FASE, bukan tab terpisah.**  
- Final Review = 1 putaran tambahan, **tidak dihitung ke `Max_Rounds`**.

### 9.7 Flow REVISI (Darma mengirim revisi)  
```
Resolusi_Darma = REVISI[teks]
→ GAS panggil ulang Current_AI_Turn yang SAMA
→ Current_Round TIDAK increment
→ Instruksi Darma di-append ke packet prompt
→ Runtime_State = WAITING_AI
```

### 9.8 Alarm Warna Sheet & Laporan Telegram  
- Setiap selesai 1 putaran penuh → GAS kirim ringkasan via Telegram (wajib **bahasa Indonesia sederhana**).  
- Warna Sheet otomatis:  
  - 🟢 Hijau: `Discussion_State = CLOSED / READY_TO_CLOSE` DAN tidak ada `VITAL_TRIGGER` pending.  
  - 🟡 Kuning: `Discussion_State = DISCUSSING`.  
  - 🔴 Merah: Ada `VITAL_TRIGGER` pending (`WAITING_DARMA`) ATAU `Runtime_State = ERROR_RETRY`.

### 9.9 Auto-Stop 24 Jam  
Jika Darma tidak merespons permintaan persetujuan dalam 24 jam → sesi dihentikan otomatis, `Discussion_State = CLOSED`, Telegram notifikasi terakhir.

---

## 10. PROMPT MASTER META-LLM FILTER – FINAL

### 10.1 INPUT  
```
[RAW_INPUT]: Respons AI saat ini (teks mentah)
[OPEN_CLAIMS]: Array {Claim_ID, Isi_Singkat} yang statusnya DEBATABLE
```

### 10.2 OUTPUT (JSON Array Murni)  
```json
[
  {
    "Ref_ID": "<Claim_ID dari OPEN_CLAIMS>" atau null,
    "Kategori": "NEW_IDEA | REBUTTAL | TECHNICAL_DETAIL | RISK_WARNING",
    "Konteks_Vital": "ANCHOR | CAPABILITY | FLOW | ARCH | OPERATIONAL | TECHNICAL",
    "Isi_Singkat": "<maks 15 kata, inti klaim tanpa basa-basi>",
    "Status": "DEBATABLE"
  }
]
```

### 10.3 Aturan Klasifikasi Mutlak  
1. **Kategori** hanya boleh: `NEW_IDEA`, `REBUTTAL`, `TECHNICAL_DETAIL`, `RISK_WARNING`.  
2. **Konteks_Vital** hanya boleh: `ANCHOR`, `CAPABILITY`, `FLOW`, `ARCH`, `OPERATIONAL`, `TECHNICAL`.  
3. **FAIL-SAFE (Konteks_Vital):** Jika klaim menyentuh biaya, keamanan, alur utama, atau **Anda RAGU**, WAJIB pilih `OPERATIONAL` atau `ARCH`. Pilih `TECHNICAL` **HANYA** untuk detail turunan murni.  
4. **FAIL-SAFE (Semantic Duplicate):** Bandingkan `RAW_INPUT` dengan `OPEN_CLAIMS`. Jika ide/klaim sudah ada, JANGAN ekstrak sebagai `NEW_IDEA`. **Tapi jika RAGU apakah duplikat → TETAP ekstrak** (kehilangan klaim baru lebih mahal dari sedikit bengkak).  
5. **Isi_Singkat:** Maksimal 15 kata, langsung ke inti.  
6. **Status:** Wajib `DEBATABLE`.  
7. **Ref_ID:** Wajib diisi `Claim_ID` dari `OPEN_CLAIMS` jika `REBUTTAL`. Jika `NEW_IDEA`, isi `null`.  
8. **DILARANG:** Salam, narasi, basa-basi, kesimpulan, teks di luar JSON.

### 10.4 Setelah GAS Terima Output  
1. Generate `Claim_ID` = `CLM-{session_id}-{counter}` (increment GAS).  
2. Isi `Round_Diajukan` = `Current_Round`.  
3. Isi `Round_Terakhir_Sanggah` = `Current_Round` (untuk klaim baru) ATAU update baris `Ref_ID` jika `REBUTTAL`.  
4. Insert row ke `CLAIM_LEDGER`.

---

## 11. AI PROMPT PACKET CONTRACT  

Setiap AI debat menerima paket konteks terstruktur (bukan hanya `OPEN_CLAIMS`):

```text
[Instruksi Sistem / Mode Diskusi]
[Context_Summary dari Local Feeder]
[DECISION_LEDGER – hanya yang ACTIVE]
[OPEN_CLAIMS – klaim dengan Status DEBATABLE]
[Klaim Baru dari AI Sebelumnya di Putaran Ini]
[Aturan Output Wajib – format respons, label, navigasi, dll.]
```

**Edge Case:** Jika paket terlalu besar dan berisiko melebihi *context window* AI, GAS wajib membatasi panjang total teks (misal potong `OPEN_CLAIMS` ke N klaim terbaru) dan trigger `SYSTEM_FRICTION` sebagai peringatan ke Darma.

---

## 12. SECRET STORAGE & KEAMANAN KREDENSIAL  

**Aturan Mutlak:**  
- **API Key semua LLM dan Token Telegram dilarang disimpan di Google Sheet.**  
- Semua kredensial disimpan di **`PropertiesService.getScriptProperties()`** GAS (terenkripsi oleh Google).  
- Kode GAS membaca kredensial dari Script Properties saat runtime.

---

## 13. CHANGE GOVERNANCE & IMPACT LOG  

Setiap perubahan terhadap struktur yang sudah dikunci wajib dicatat dengan **Impact Log 3 Baris**.

### Impact Log 1: Perubahan dari SOT awal (3 tab) ke 5 tab  
| Baris | Isi |
|-------|-----|
| **KONTRAK LAMA** | `session_log` + `audit_events` + `session_config` (3 tab, per-respons log) |
| **KONTRAK BARU** | `RAW_ARCHIVE` + `CLAIM_LEDGER` + `DECISION_LEDGER` + `VITAL_TRIGGER` + `SESSION_STATE` (5 tab, claim-based) |
| **BLAST RADIUS** | Alarm warna & laporan Telegram disesuaikan baca dari `CLAIM_LEDGER`+`SESSION_STATE`. Logika drift/stuck diperbaiki. `Paused_Until_Darma` dihapus (redundan). `pause_keywords` diganti `Konteks_Vital`. |

### Impact Log 2: Penambahan session_config ke SESSION_STATE  
| Baris | Isi |
|-------|-----|
| **KONTRAK LAMA** | `session_config` terpisah (mode, max_rounds, pause_keywords, custom_instruction) — 3 tab lama |
| **KONTRAK BARU** | `mode`, `max_rounds`, `custom_instruction` → 3 kolom baru di `SESSION_STATE`; `pause_keywords` → diganti `Konteks_Vital` (sudah ada) |
| **BLAST RADIUS** | Routing Logic +1 rule cek `Max_Rounds`. Tidak pengaruh ke `CLAIM_LEDGER`/`DECISION_LEDGER`. |

### Impact Log 3: Status SKIPPED yang hilang  
| Baris | Isi |
|-------|-----|
| **KONTRAK LAMA** | `state=SKIPPED` di Dok1 (3-tab lama), hilang saat redesign 5-tab. |
| **KONTRAK BARU** | Skip AI dikembalikan via `Resolusi_Darma=LANJUT` pada error API (tanpa status enum baru). |
| **BLAST RADIUS** | Tidak ada perubahan skema tab. Routing Logic §8 bertambah 1 aturan error handling. |

### Impact Log 4: Bahasa awam untuk Vital Gate Darma  
| Baris | Isi |
|-------|-----|
| **KONTRAK LAMA** | Pertanyaan vital ke Darma masih berisiko memakai istilah teknis mentah dan format jawaban `LANJUT`. |
| **KONTRAK BARU** | Pertanyaan vital wajib memakai bahasa awam, menjelaskan dampak praktis, risiko, saran default, dan pilihan `A/B/REVISI/STOP`. |
| **BLAST RADIUS** | Telegram prompt, `VITAL_TRIGGER.Resolusi_Darma`, `Affects_Claim_ID`, dan audit keputusan vital disesuaikan. |

---

## 14. BIAYA, KEAMANAN & RELIABILITAS  

| Aspek | Keputusan Final |
|-------|-----------------|
| **Model Biaya** | API token per pemakaian (variabel), bukan paket bulanan terminal agent. |
| **Terminal Agent** | Hanya context loading awal – biaya minimal, hanya saat dipakai. |
| **Secret Storage** | `PropertiesService.getScriptProperties()` – API key & token Telegram tidak boleh di Sheet. |
| **Keamanan Local Feeder** | `.claudeignore` / exclude list wajib: `.env`, `*.key`, `node_modules`, backup besar. Preview + persetujuan Darma (Y/N) sebelum webhook. |
| **Concurrency** | LockService GAS. |
| **Deduplikasi Event** | CacheService + `Payload_ID`. |
| **Batas Putaran** | `Max_Rounds` (default 10), dicek sebelum panggil AI. Final Review di luar hitungan `Max_Rounds`. |
| **Auto-Stop 24 Jam** | Jika Darma tidak respons → sesi dihentikan otomatis. |
| **Kendali Manual** | Telegram: `/pause`, `/lanjut`, `/stop`, `/status`. Untuk keputusan vital: `A`, `B`, `REVISI [teks]`, `STOP`. Untuk error/friction: `LANJUT`, `REVISI [teks]`, `STOP`. |
| **Komunikasi** | Semua pesan sistem ke Darma wajib **bahasa Indonesia sederhana**, bukan jargon teknis. Jika menyangkut keputusan vital, sistem wajib menjelaskan dampak praktis, risiko, saran default, dan pilihan balasan sederhana. |

---

## 15. ROLE & PERMISSION  

| Aktor | Hak Akses | Batasan |
|-------|-----------|---------|
| **Darma** | Full control. Satu-satunya yang bisa approve `VITAL_TRIGGER`, edit Sheet manual, stop sesi. | - |
| **GAS** | System executor. Satu-satunya penulis `SESSION_STATE`, `DECISION_LEDGER`, `VITAL_TRIGGER`. | Hanya menulis via logika yang sudah disepakati. |
| **5 AI Debat** | Read `OPEN_CLAIMS` only (dari `CLAIM_LEDGER` yang diberikan sebagai Diff Context). | Tidak punya akses langsung ke Sheet. Tidak bisa menulis/mengubah data. |
| **Meta-LLM Filter** | Tulis `CLAIM_LEDGER` saja (via output JSON). | Tidak boleh mengunci keputusan. Tidak boleh menulis `DECISION_LEDGER`. |
| **Terminal Agent** | Baca folder proyek yang diizinkan. Membuat ringkasan. | Tidak boleh mengirim file sensitif. Tidak boleh eksekusi tanpa preview + persetujuan Darma. |

---

## 16. OBSIDIAN – POSISI FINAL  

**Keputusan:** Backlog opsional / Architecture-Ready Later.  

| Fungsi | Tool |
|--------|------|
| Runtime Moderator 24/7 | GAS + Sheets + Telegram |
| Baca file lokal di awal sesi | Terminal Agent (Claude Code/Codex) |
| Arsip pengetahuan jangka panjang | Obsidian vault (opsional pasca-sesi) |

---

## 17. BACKLOG ARSITEKTUR (ARCHITECTURE-READY LATER)  

| Fitur | Alasan Ditunda |
|-------|----------------|
| **Memulai sesi langsung dari Telegram** (tanpa Local Feeder) | V1 fokus pada webhook dari terminal. Entry Telegram jadi opsional pasca-V1. |
| **Integrasi Obsidian** | Arsip pasca-sesi. Tidak mempengaruhi core. |
| **Multi-sesi paralel** | V1 single-session dulu. |

---

## 18. FRICTION LOG KOMPREHENSIF  

| FRICTION | CATATAN | STATUS |
|----------|---------|--------|
| Campur layer diskusi | Arsitektur, runtime, tool baca lokal dibahas bersamaan. | ✅ Dipisahkan |
| Saran ChatGPT (Local-first CLI penuh) | Ditolak panel karena bentrok kendali HP & serverless. ChatGPT koreksi diri. | ✅ Dikoreksi |
| Obsidian masuk di tengah | Valid tapi bukan runtime. Disepakati backlog opsional. | ✅ Diputuskan |
| Overclaim konsensus | DeepSeek nyatakan semua AI sepakat, Kimi/Qwen tidak terlihat. 4 AI mayoritas cukup. | ✅ Dicatat |
| Redundansi boolean (`Requires_Darma_Approval`, `Promotable_Auto`) | Ditolak Gemini & Claude. Routing dipindah ke GAS. | ✅ Dibatalkan |
| Bug enum `Konteks_Vital` | `TECHNICAL_DETAIL` (Kategori) dipakai sebagai Konteks_Vital. Diperbaiki. | ✅ Diperbaiki |
| STUCK vs PROMOTED tumpang tindih | Dipisah: PROMOTED = technical tanpa rebuttal; STUCK = konflik berulang. | ✅ Diperbaiki |
| Auto-REJECTED terlalu kasar | ChatGPT usul `Rebuttal_Type` (ditolak). Claude sintesis: TECHNICAL-only + `Affects_Claim_ID` opsional. | ✅ Disepakati |
| Draft Prompt Master tidak lengkap | Field `Round_*` hilang, `Ref_ID` tanpa `OPEN_CLAIMS`. Claude perbaiki. | ✅ Diperbaiki |
| Blindspot API Contract | Panel fokus ke Sheets, lupa standarisasi Webhook Payload. ChatGPT tambahkan. | ✅ Ditambahkan |
| session_config hilang | `mode`, `max_rounds`, `custom_instruction` tidak dipetakan. Claude tambahkan ke `SESSION_STATE`. | ✅ Ditambahkan |
| Alarm & Laporan Telegram terlupa | SOT awal §5.1-5.2 tidak muncul di rekap. DeepSeek tambahkan ke Routing Logic. | ✅ Ditambahkan |
| Domain Check terlalu disederhanakan | ChatGPT perbaiki: tool internal tetap punya domain (workflow Darma). | ✅ Diperbaiki |
| Flow REVISI belum dikunci | Claude tambahkan: panggil ulang AI sama, round tidak increment. | ✅ Ditambahkan |
| Role & Permission belum tercatat | ChatGPT tambahkan blok minimal. | ✅ Ditambahkan |
| Secret Storage nyaris terlewat | API key & token Telegram belum punya aturan penyimpanan aman. Ditambahkan ChatGPT. | ✅ Ditambahkan |
| API Error Policy terlalu disederhanakan Gemini | Gemini selalu pause, hilang opsi skip AI. Claude kembalikan via `Resolusi_Darma=LANJUT`. | ✅ Diperbaiki |
| AI Prompt Packet Contract belum ada | AI debat butuh konteks lengkap, bukan hanya `OPEN_CLAIMS`. Ditambahkan ChatGPT. | ✅ Ditambahkan |
| Max_Rounds rawan off-by-one | Definisi putaran penuh dan kenaikan `Current_Round` dipertegas. | ✅ Diperjelas |
| Final Review vs Max_Rounds | Tidak diklarifikasi sebelumnya. Diputuskan: Final Review di luar hitungan `Max_Rounds`. | ✅ Diputuskan |
| Status SKIPPED hilang | Ada di SOT awal, hilang di redesign. Dikembalikan via `Resolusi_Darma=LANJUT` pada error API. | ✅ Dikembalikan |
| Premature CLOSED beberapa kali | V1.2 dan V1.3 awal terlalu cepat dikunci sebelum semua gap ditemukan. | ✅ Dikoreksi |

---

## 19. LANGKAH SELANJUTNYA (DRY-RUN)  

1. **Persiapan:** Siapkan Google Sheet dengan 5 tab sesuai struktur final. Siapkan GAS dengan semua routing logic dan secret storage.  
2. **Uji 1 AI dulu:** Jalankan dengan DeepSeek saja – validasi webhook → filter → insert `CLAIM_LEDGER` → notifikasi Telegram.  
3. **Uji 2 AI:** Tambah Claude – pastikan AI kedua baca Diff Context dari `CLAIM_LEDGER`.  
4. **Uji Rem Darurat:** Pancing AI sebut keputusan vital → pastikan sistem pause + notifikasi Telegram + pertanyaan ke Darma memakai bahasa awam, menjelaskan dampak praktis, dan Darma bisa balas `A` / `B` / `REVISI [teks]` / `STOP`.  
5. **Uji Flow REVISI:** Darma kirim `REVISI[teks]` → pastikan AI yang sama dipanggil ulang.  
6. **Uji Error API:** Simulasikan AI gagal → pastikan retry 1x, lalu `SYSTEM_FRICTION`, Darma `LANJUT` untuk skip.  
7. **Uji Batas Putaran:** Set `Max_Rounds=2` → pastikan sistem berhenti setelah putaran terlampaui, Final Review tetap jalan.  
8. **Full Run:** 5 AI dengan `Max_Rounds=3` → evaluasi hasil akhir.  
9. **Setelah semua sukses:** Kunci sebagai V1.3 production.

---

**=== STATUS AKHIR: CLOSED ===**

Panel AI telah mencapai **kesepakatan penuh** terhadap seluruh poin di atas.  
Semua 5 Keputusan Vital dari SOT awal Darma telah tercover lengkap.  
**Dokumen ini adalah Source of Truth (SoT) untuk Smart Moderator V1.3 Final.**  
Setiap perubahan di masa depan wajib melalui Change Governance (Impact Log 3 Baris).