# SOT – SMART MODERATOR V1.4  
**Sumber:** SOT V1.3 (Final) + Rekap Kumulatif Patchset V1.4 (Sesi 1–5, Diskusi Panel AI)  
**Core Panel (Debater):** ChatGPT → Gemini → Claude (mode Chat, *pure reasoner*, sekuensial)  
**Shadow Scribe:** DeepSeek (`SHADOW_SCRIBE_MODE`) — pencatat & pendeteksi alert, **tidak ikut debat inti**  
**Backlog Panel:** Kimi, Qwen (belum aktif di V1.4 – ketidakhadiran tidak menggugurkan konsensus, tercatat di Friction Log)  
**Status:** **CLOSED – Siap Implementasi & Dry-Run**  
**Versi:** V1.4 (Final, Patchset Sesi 1–5)

---

## DAFTAR ISI
1. Landasan – Adaptive Product Planning Framework  
2. Masalah Utama & Tujuan Smart Moderator  
3. Konsep & Definisi Smart Moderator  
4. Arsitektur – Local Feeder + Cloud Brain (Hybrid)  
5. Penolakan Pendekatan Lain  
6. Webhook Data Contract – Local Feeder ke GAS  
7. Kontrak Data Ledger – Struktur Google Sheets (8 Tab)  
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

### 3.0 Definisi & Komposisi Panel (Revisi V1.4)

**Definisi Smart Moderator (Revisi):**
- **Lama (V1.3):** Router AI yang mendistribusikan pesan antar-AI.
- **Baru (V1.4):** *Guardian* dengan 4 tugas:
  1. Menjaga keputusan yang sudah terkunci.
  2. Mencatat observasi lapangan Darma.
  3. Menjaga konsensus & integritas diskusi.
  4. Menjaga kualitas diskusi (anti-noise, anti-yes-man, anti-pengulangan).
- **Alasan:** Darma butuh sistem yang menjaga memori, mendeteksi pelanggaran, dan menampung temuan lapangan — bukan sekadar pengirim pesan.

**Komposisi Panel (Revisi total dari rotasi 5 AI di V1.3):**

| Peran | Anggota | Mode | Catatan |
|-------|---------|------|---------|
| **Core Panel** (debater) | ChatGPT → Gemini → Claude | Chat (stateless, *pure reasoner*) | Rotasi sekuensial, ikut debat penuh |
| **Shadow Scribe** | DeepSeek | Thinking Enabled | **Tidak ikut debat inti.** Membaca semua output Core Panel, menulis `ROUND_SUMMARY` & `SCRIBE_ALERT` (§8.10–§8.11). Output tidak dikirim balik ke Core Panel. |
| **Backlog** | Kimi, Qwen | — | Belum aktif di V1.4 |

**GAS = Orchestration Agent:** Pusat kendali state, rotasi, ledger, filter, pause. **Haram** baca file lokal atau eksekusi kode lokal — mencegah ekspektasi berlebihan dan menjaga batas arsitektur hybrid.

> Catatan: Sebutan "5 AI" pada bagian-bagian lain dokumen ini (§4, §9, dst.) yang masih merujuk rotasi lama (Kimi→Qwen→Gemini→Claude→DeepSeek) **sudah digantikan** oleh komposisi di atas. Lihat Impact Log #5 (§13).

### 3.1 Bukan Multi-Agent Biasa  
Multi-agent biasa: respons mentah dilempar antar AI → makin panjang, makin boros, AI cenderung yes-man.  
Smart Moderator: moderator pintar menyaring, hanya kirim **diff** (perbedaan/sanggahan) ke AI berikutnya, mencatat keputusan, dan hanya memanggil Darma saat menyentuh keputusan vital.

### 3.2 6 Fungsi Inti Moderator  
1. **Context Filter** – menentukan bagian mana yang perlu dibaca AI berikutnya.  
2. **Decision Ledger** – mencatat keputusan yang sudah/belum terkunci dan yang berubah.  
3. **Duplication Guard** – mencegah AI mengulang argumen yang sudah dibahas.  
4. **Vital Question Gate** – berhenti dan tanya Darma jika menyentuh keputusan vital, dengan bahasa awam, dampak praktis, dan pilihan jawaban sederhana.  
5. **Technical Autopilot** – pembahasan teknis turunan berjalan otomatis.  
6. **Final Cumulative Summary + Validation Guard** – merangkum hasil akhir, mendeteksi friction/alert, dan divalidasi lewat 5-Layer Validation Engine.

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
   ├── Rotasi Core Panel (ChatGPT → Gemini → Claude, sekuensial)
   ├── Panggil API LLM + Meta-LLM Filter
   ├── Trigger async Shadow Scribe (DeepSeek) tiap akhir round → `WAITING_SCRIBE` → `SCRIBE_DONE`/`SCRIBE_ERROR` (§8.10)
   ├── Routing Logic (Auto-Promote, Vital Gate, Stuck Detection, Max Rounds/Topic, SCRIBE_ALERT Tier, 5-Layer Validation)
   ├── Error Handling API & JSON Filter
   ├── Concurrency Control (LockService)
   └── Alarm + Laporan Telegram tiap putaran
        │
        ▼
[Google Sheets — DATABASE VISUAL & AUDIT, 8 TAB]
   ├── RAW_ARCHIVE (teks mentah)
   ├── CLAIM_LEDGER (gelanggang debat)
   ├── DECISION_LEDGER (keputusan terkunci)
   ├── VITAL_TRIGGER (antarmuka Darma)
   ├── SESSION_STATE (konfigurasi + pointer resume)
   ├── OBSERVATION_LEDGER (observasi lapangan Darma) — baru V1.4
   ├── SYSTEM_ONTOLOGY (jangkar definisi sistem) — baru V1.4
   └── ROUND_SUMMARY (rekap tiap round oleh Shadow Scribe) — baru V1.4
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

## 7. KONTRAK DATA LEDGER – STRUKTUR GOOGLE SHEETS (8 TAB, REVISI V1.4)

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
| `Round_Diajukan` | **GAS** | Dari `SESSION_STATE.Round_In_Topic` (V1.4 — menggantikan `Current_Round` global, lihat §7 Tab 5) |
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
| `Trigger_Source` | **(Baru V1.4)** Asal pemicu fatal: `CORE_PANEL_BUG_HUNT`, `SHADOW_SCRIBE`, `ZERO_CONTEXT_REVIEW`, `SCRIBE_VALIDATION`. `SHADOW_SCRIBE` = alert sah dari Scribe tentang isi debat; `SCRIBE_VALIDATION` = alert sistem tentang Scribe-nya sendiri yang salah/kurang (dipicu Tier 1 GAS atau Meta-LLM Cross-Check). **Catatan terbuka:** rekap tidak menjelaskan nilai kolom ini untuk trigger dari Vital Gate manual (§8.2, bukan hasil deteksi otomatis) — perlu diklarifikasi saat implementasi GAS, bukan diasumsikan di sini. |

### TAB 5: `SESSION_STATE` (Konfigurasi + Pointer Resume) – Revisi V1.4  
| Kolom | Keterangan |
|-------|------------|
| `Session_ID` | ID sesi |
| `Mode` | `antiyesman`, `konvergensi`, `kristalisasi`, `custom` |
| `Custom_Instruction` | Instruksi khusus jika Mode = custom |
| `Current_Topic_Session` | **(Baru V1.4)** ID topik diskusi yang sedang berjalan |
| `Topic_Title` | **(Baru V1.4)** Judul topik saat ini |
| `Round_In_Topic` | **(Baru V1.4, menggantikan `Current_Round` global)** Putaran saat ini **di dalam topik aktif**; hanya naik setelah Claude (AI terakhir Core Panel) selesai; reset ke 0 setiap ganti `Current_Topic_Session` |
| `Max_Rounds_Per_Topic` | **(Baru V1.4, menggantikan `Max_Rounds` global)** Batas putaran penuh per topik (default: **3** — Round 1 Eksplorasi, Round 2 Sanggahan, Round 3 Konvergensi) |
| `Current_AI_Turn` | AI Core Panel yang sedang/selanjutnya dipanggil |
| `Discussion_State` | `EXPLORING`, `DISCUSSING`, `READY_TO_CLOSE`, `CLOSED` |
| `Runtime_State` | `WAITING_AI`, `WAITING_FILTER`, `WAITING_SCRIBE`, `SCRIBE_DONE`, `SCRIBE_ERROR`, `WAITING_DARMA`, `ERROR_RETRY`, `FINAL_CUMULATIVE_SUMMARY`, `DONE` |
| `Estimated_Cost_Session` | **(Baru V1.4)** Akumulasi taksiran biaya API per sesi — dihitung **deterministik oleh GAS** dari token usage × harga per token di Script Properties. **Bukan** estimasi naratif LLM. |
| `Token_Usage_Cumulative` | **(Baru V1.4)** Total token terpakai (kumulatif sesi) |
| `Version_Lock_Status` | **(Baru V1.4)** `PENDING` / `LOCKED` — dipakai Lapis 5 (§8.16) saat proses kunci versi dokumen. Berbeda dari `Discussion_State = CLOSED` yang dipakai untuk penutupan sesi topik biasa. |
| `Next_Action` | **(Baru V1.4)** Aksi runtime berikutnya setelah friction/error, diisi GAS saat `Runtime_State` masuk kondisi non-normal. Nilai: `BYPASS_LANJUT`, `RETRY_SAME_AI`, `WAIT_DARMA`, atau `STOP`. Dipakai a.l. oleh Watchdog Scribe (§8.10). |
| `Last_Updated` | Timestamp terakhir diupdate |

**Catatan:**  
- `Paused_Until_Darma` dihapus (redundan dengan `Runtime_State = WAITING_DARMA`).  
- `pause_keywords` digantikan oleh klasifikasi semantik `Konteks_Vital` (lebih akurat).  
- Pemisahan `Discussion_State` & `Runtime_State` karena dua dimensi berbeda: progres konten diskusi vs status eksekusi teknis.  
- **(V1.4)** `Current_Round` & `Max_Rounds` global **dihapus**, digantikan total oleh `Round_In_Topic` & `Max_Rounds_Per_Topic` mengikuti hierarki `Session_ID → Topic_Session → Round_In_Topic` (lihat Impact Log #6, §13).

---

### TAB 6: `OBSERVATION_LEDGER` (Observasi Lapangan Darma) – Baru V1.4  
| Kolom | Diisi Oleh | Keterangan |
|-------|------------|------------|
| `Obs_ID` | GAS | ID unik observasi |
| `Session_ID` | GAS | ID sesi |
| `Round_Dicatat` | GAS | `Round_In_Topic` saat observasi dicatat |
| `Darma_Input_Raw` | Darma (manual) | Teks **verbatim** observasi Darma — **tidak boleh diparafrase** oleh sistem |
| `Obs_Key` | Meta-LLM Filter | Ringkasan maksimal 15 kata, diekstrak dari `Darma_Input_Raw` tanpa mengubah teks asli |
| `Status` | GAS | `OPEN → LINKED → SOLVED / CONFLICT` |
| `Linked_Claim_ID` | GAS | `Claim_ID` terkait (jika ada) |
| `Resolution_Note` | GAS | Catatan penyelesaian |

**Aturan:** Observasi mentah Darma adalah fakta lapangan dan **tidak didebat**; interpretasi & solusi teknisnya **boleh didebat**. Jika `Darma_Input_Raw` bentrok dengan `DECISION_LEDGER` yang `ACTIVE` → otomatis `VITAL_TRIGGER`.

---

### TAB 7: `SYSTEM_ONTOLOGY` (Jangkar Definisi Sistem, Tabular) – Baru V1.4  
| Kolom | Keterangan |
|-------|------------|
| `Anchor_Tag` | `COMP_CORE`, `FLOW_INTI`, `GOVERNANCE`, `DEFENSE` |
| `Content` | Isi ringkas dalam Markdown |
| `Version` | Versi isi anchor |
| `Last_Updated` | Timestamp terakhir diupdate |

GAS hanya melakukan retrieval/injection ke prompt AI — **tidak** melakukan parsing semantik atas isinya.

**Isi final ringkas per Anchor Tag:**

| Anchor_Tag | Content |
|---|---|
| `COMP_CORE` | **GAS:** Otak orkestrasi, pengelola state Sheets, *stateless loop control*. Batas: haram eksekusi/baca file lokal. **Core Panel:** ChatGPT → Gemini → Claude (pure reasoners via chat API). **Shadow Scribe:** DeepSeek — aktif merekap & deteksi alert, tidak ikut debat inti. |
| `FLOW_INTI` | **Hierarki:** `Session_ID` → `Topic_Session` → `Round_In_Topic`. **Mekanisme:** Pindah `Topic_Session` reset `Round_In_Topic` ke 0. `Max_Rounds_Per_Topic` dihitung per topik. DeepSeek merekap diam-diam tiap akhir putaran. |
| `GOVERNANCE` | **Vital Gate:** Interupsi Darma wajib jika menyentuh Arsitektur, Alur Inti, Budget, Keamanan, Target Solusi. **Eskalasi:** `WARNING` diabaikan 2 putaran → aksi setara FATAL (label asal tetap WARNING demi audit). Perubahan besar wajib `Impact Log`. **6 Kriteria FATAL (Jangkar Eksak):** `[FATAL-1]` Kontradiksi `DECISION_LEDGER ACTIVE`. `[FATAL-2]` Risiko salah build besar. `[FATAL-3]` Pelanggaran keamanan/kredensial/data. `[FATAL-4]` Klaim "semua sepakat" padahal tidak (false-consensus). `[FATAL-5]` Observasi penting Darma diabaikan. `[FATAL-6]` Konflik arsitektur/flow inti. |
| `DEFENSE` | **Isolasi Kredensial:** API Key & token wajib Script Properties. **Integritas Fakta:** `Darma_Input_Raw` tidak boleh diparafrase. **Reliability Guard:** `LockService`, `CacheService`+`Payload_ID`, Auto-Stop 24 jam. **Anti-Echo Chamber:** Uji injeksi false-consensus di Dry-Run. |

---

### TAB 8: `ROUND_SUMMARY` (Rekap Tiap Round oleh Shadow Scribe) – Baru V1.4  
| Kolom | Keterangan |
|-------|------------|
| `Session_ID` | ID sesi |
| `Topic_Session` | ID topik aktif |
| `Round_In_Topic` | Putaran ke berapa dalam topik |
| `Core_Delta` | Ringkasan perubahan klaim dari Core Panel di round ini |
| `Decision_Delta` | Ringkasan keputusan baru/berubah |
| `Open_Friction` | Friksi yang masih terbuka |
| `Scribe_Alert` | Alert dari Shadow Scribe, format `TIER: pesan [JANGKAR]`, multi-alert dipisah `;` |
| `Next_Focus` | Fokus yang disarankan untuk round berikutnya |

**Penulis:** **DeepSeek (Shadow Scribe)** menulis `ROUND_SUMMARY` tiap akhir round. GAS hanya menyimpan dan merutekan — **bukan tab gabungan**, fungsi audit & konteks ringkasnya berdiri sendiri dari tab lain.  
**Parsing wajib:** `split(";")` → `trim()` → `startsWith("FATAL:")` / `startsWith("WARNING:")` — **bukan** `.includes()` (mencegah false-positive).

---

## 8. ROUTING LOGIC GAS – ATURAN LENGKAP (TERMASUK SEMUA PATCH)

### 8.1 Auto-Promote (Otomatis, Tanpa Darma)  
**Syarat (semua harus terpenuhi):**  
- `Konteks_Vital == TECHNICAL`  
- `(Round_In_Topic - Round_Terakhir_Sanggah) >= 2` — bertahan 2 putaran penuh tanpa `REBUTTAL` baru.  
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

### 8.5 Cek Batas Putaran (`Max_Rounds_Per_Topic`) — Revisi V1.4  
- `Max_Rounds_Per_Topic` = jumlah putaran penuh **dalam 1 topik** (satu putaran = ChatGPT → Gemini → Claude selesai semua). Default **3**.  
- `Round_In_Topic` hanya naik setelah Claude (AI terakhir Core Panel) selesai. Pemanggilan Shadow Scribe (DeepSeek) berjalan **async terpisah** (§8.10) dan **tidak** mempengaruhi increment `Round_In_Topic`.  
- Ganti `Current_Topic_Session` → `Round_In_Topic` reset ke 0.  
- Sebelum memulai round baru, GAS cek: `Round_In_Topic >= Max_Rounds_Per_Topic`?  
  → **Ya:** `Runtime_State = WAITING_DARMA`, `VITAL_TRIGGER` (Tipe: `SYSTEM_FRICTION`, Pemicu: "Batas putaran topik tercapai").  
- **Final Review** (lihat §9.6, direvisi total jadi `FINAL_CUMULATIVE_SUMMARY`) = sekali di **akhir sesi**, **tidak** dihitung ke `Max_Rounds_Per_Topic`.

### 8.6 Deduplikasi & Concurrency  
- **LockService** – mencegah dua eksekusi tulis Sheet bersamaan.  
- **CacheService + `Payload_ID`** – menolak payload yang sudah diproses sebelumnya (idempotensi).

### 8.7 Error Handling API AI Utama (Core Panel — ChatGPT/Gemini/Claude)  
> Berlaku untuk Core Panel. Error Shadow Scribe (DeepSeek) punya jalur terpisah — lihat §8.10 (`SCRIBE_ERROR`).
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

### 8.9 Mode Operasional & Parameter Model (Baru V1.4)  
- **Core Panel:** wajib Mode Chat (stateless, deterministik).  
- **Shadow Scribe:** wajib Mode Thinking, default `deepseek-v4-flash` dengan `thinking.enabled`.  
- **Semua Model ID disimpan di Script Properties** (`ChatGPT_Model_ID`, `Gemini_Model_ID`, `Claude_Model_ID`, `DeepSeek_Model_ID`) — **dilarang di-hardcode** di SOT atau kode GAS.  
- Saat dry-run, cek dulu lineup model aktif tiap provider, baru isi configurable ID.

| Komponen | Mode | Temperature | Max Output | Catatan |
|---|---|---|---|---|
| Meta-LLM Filter | Chat (Stateless) | 0.0 | 500 tokens | Harus JSON/klasifikasi rapi |
| Core Panel | Chat (Stateless) | 0.5 | 4.000 tokens | Stabil, kritis, tidak terlalu liar |
| Shadow Scribe | Thinking Enabled | 0.3 | 8.000 tokens | Rekap teliti, bukan kreatif |
| Zero-Context Review | Chat (Stateless) | 0.5 | 4.000 tokens | Diturunkan dari parameter Core Panel, bukan konfigurasi independen |

### 8.10 Arsitektur 2-Eksekusi Shadow Scribe (Async, Baru V1.4)  
- **Flow:** Core round selesai → `Runtime_State = WAITING_SCRIBE` → trigger terpisah panggil DeepSeek → `SCRIBE_DONE` / `SCRIBE_ERROR`.  
- **Watchdog 10 menit:** `WAITING_SCRIBE` >10 menit → force `SCRIBE_ERROR` + `Next_Action = BYPASS_LANJUT`. Jika berulang ≥2 kali dalam satu sesi topik → `VITAL_TRIGGER`.  
- State baru di `SESSION_STATE.Runtime_State`: `WAITING_SCRIBE`, `SCRIBE_DONE`, `SCRIBE_ERROR`.  
- **Bypass Guard Logging:** kejadian `BYPASS_LANJUT` dicatat di `ROUND_SUMMARY.Open_Friction`.

### 8.11 `SCRIBE_ALERT` Tier (FATAL/WARNING) & Jangkar Kode (Baru V1.4)  
- **FATAL:** Langsung `VITAL_TRIGGER` + pause. Wajib salah satu dari 6 kriteria (jangkar eksak, lihat `SYSTEM_ONTOLOGY.GOVERNANCE` §7 Tab 7):  
  `[FATAL-1]` Kontradiksi `DECISION_LEDGER ACTIVE`.  
  `[FATAL-2]` Risiko salah build besar.  
  `[FATAL-3]` Pelanggaran keamanan/kredensial/data.  
  `[FATAL-4]` Klaim "semua sepakat" padahal tidak (false-consensus).  
  `[FATAL-5]` Observasi penting Darma diabaikan.  
  `[FATAL-6]` Konflik arsitektur/flow inti.  
- **Aturan jangkar (dua scope — wajib dipahami benar agar tidak overreach):**  
  - **Individual Alert:** Setiap `SCRIBE_ALERT:FATAL` cukup menyertakan **minimal satu** jangkar valid dari daftar di atas.  
  - **Completeness Check:** Hanya berlaku ketika Scribe atau sistem mengklaim sedang mencantumkan **daftar lengkap** kriteria FATAL (mis. dalam dokumen definisi atau rekap final). Saat itu saja, seluruh 6 jangkar wajib hadir.  
- **WARNING:** Masuk `ROUND_SUMMARY`, tidak pause. Diabaikan 2 round → naik jadi aksi setara FATAL (pola Stuck Detection §8.3); label histori tetap `WARNING` demi audit.  
- **Parsing wajib:** `split(";")` → `trim()` → `startsWith("FATAL:")` / `startsWith("WARNING:")` — **bukan** `.includes()`.

### 8.12 Pemisahan Jalur FATAL & `Trigger_Source` (Baru V1.4)  
`VITAL_TRIGGER` (§7 Tab 4) selalu diisi `Trigger_Source` saat fatal: `CORE_PANEL_BUG_HUNT`, `SHADOW_SCRIBE`, `ZERO_CONTEXT_REVIEW`, atau `SCRIBE_VALIDATION`. Tujuannya agar fatal dari bug-hunt, scribe, dan 0-context review bisa diaudit terpisah — jalur ini berbeda dari `Severity: FATAL` hasil `BUG_HUNT` (§8.14) yang masuk Vital Gate lewat rute sendiri.

### 8.13 Anti-Otonomi (Runtime Guard, Baru V1.4)  
Seluruh model (Core Panel, Shadow Scribe, Meta-LLM Filter) hanya boleh mengembalikan teks/JSON statis. **GAS adalah satu-satunya eksekutor mutasi data** — tidak ada LLM yang menulis langsung ke Sheets.

### 8.14 Mode `BUG_HUNT` (Evidence-Based, Baru V1.4)  
- Output 7 kolom: `Issue_Type`, `Evidence_Ref`, `Expected`, `Failure`, `Violated_Rule`, `Severity`, `Fix_Direction`.  
- `Evidence_Ref` **wajib**; klaim tanpa ini → `INVALID`. Omission bug (celah karena sesuatu tidak ada) tetap valid lewat format `MISSING @ [Komponen]`.  
- `Severity: FATAL` dari Core Panel masuk Vital Gate — jalur **berbeda** dari `SCRIBE_ALERT:FATAL` (§8.11), diaudit lewat `Trigger_Source = CORE_PANEL_BUG_HUNT`.

### 8.15 Indikator Selesai & Uji 0-Konteks (Baru V1.4)  
- **Indikator selesai:** saturasi klaim kritis — uji 0-konteks tidak menghasilkan klaim baru di ranah `ANCHOR`, `CAPABILITY`, `FLOW`, `ARCH`, `OPERATIONAL`, `DEFENSE`.  
- **Uji 0-Konteks (Opsi B):** memakai 3 model Core Panel secara stateless, dengan input: `DECISION_LEDGER ACTIVE` + `SYSTEM_ONTOLOGY` + `OBSERVATION_LEDGER` (status `OPEN`/`LINKED`) + `ROUND_SUMMARY` terakhir + bahan topik.  
- Temuan `TECHNICAL_DETAIL` kecil → masuk backlog, tidak menggagalkan penutupan topik.

### 8.16 5-Layer Validation Engine (Baru V1.4)  
Validasi hasil diskusi tidak bergantung satu aktor, melainkan sistem berlapis desentralisasi. Urutan nomor lapis berdasarkan **otoritas**, bukan urutan eksekusi.

| Lapis (Otoritas) | Aktor | Fungsi | Kapan Berjalan |
|---|---|---|---|
| **Lapis 1** | Darma (Veto Mutlak) | Juri tertinggi; bisa abort/interupsi kapan saja | Setiap saat |
| **Lapis 2** | Core Panel N+1 | Membaca `ROUND_SUMMARY` putaran N sebagai konteks, menyanggah jika ada deviasi fakta | Setiap putaran berikutnya |
| **Lapis 3** | GAS Tier-1 (String-match) | **Forbidden Check:** mendeteksi klaim/aturan yang **tidak ada** di `DECISION_LEDGER ACTIVE` / `SYSTEM_ONTOLOGY` / 6 kriteria FATAL. **Completeness Check:** mendeteksi aturan wajib yang hilang/dikurangi — khusus saat output mengklaim sebagai daftar lengkap. Jika jangkar `[FATAL-1]` s/d `[FATAL-6]` <6 pada klaim daftar lengkap → `VITAL_TRIGGER`. **Individual Alert:** cukup 1 jangkar valid (§8.11). | Tiap akhir round, tanpa panggil API |
| **Lapis 4** | Meta-LLM Tier-2 (Khusus Putaran Akhir) | Audit semantik ringkas saat `Round_In_Topic == Max_Rounds_Per_Topic`, menutup celah "tidak ada N+1" | Hanya di putaran terakhir topik |
| **Lapis 5** | Diff Checklist (Automated Pre-Hook via `DECISION_LEDGER`) | Query `DECISION_LEDGER` lintas `Topic_Session`, membandingkan semua `Kontrak_Terkunci` yang `ACTIVE` dengan draf final. Berjalan otomatis saat fungsi `lockVersion()` dipanggil; jika ada patch hilang, transisi ke `Version_Lock_Status = LOCKED` diblokir. | Khusus sebelum lock versi major (V1.4→V1.5, dst.) |

**Catatan:**  
- Lapis 5 **bukan** tab permanen baru, melainkan fungsi GAS *on-demand* yang membaca `DECISION_LEDGER` — `PATCH_MANIFEST` tidak diperlukan sebagai tab terpisah.  
- `Version_Lock_Status` disimpan di `SESSION_STATE` (§7 Tab 5), khusus kunci versi dokumen, dibedakan dari `Discussion_State = CLOSED` (penutupan sesi topik biasa).

---

## 9. MEKANISME ORKESTRASI – DETAIL ALUR

### 9.1 Urutan Rotasi AI (Revisi Total V1.4)  
**Core Panel (debater):** **ChatGPT → Gemini → Claude** (sekuensial, satu per satu, stateless).  
**Shadow Scribe (DeepSeek):** **tidak** ikut rotasi debat — dipanggil terpisah secara async setiap akhir round (§8.10), membaca output Core Panel dan menulis `ROUND_SUMMARY` + `SCRIBE_ALERT`.  
**Backlog:** Kimi, Qwen — belum aktif di V1.4.

> ~~Kimi → Qwen → Gemini → Claude → DeepSeek~~ (rotasi V1.3, digantikan total — lihat Impact Log #5, §13)

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
- **(Baru V1.4)** Atau ketika `SCRIBE_ALERT:FATAL` terdeteksi (§8.11) — masuk `VITAL_TRIGGER` dengan `Trigger_Source` terisi (§8.12)

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

### 9.6 Final Cumulative Summary (Revisi Total V1.4)  

**Mekanisme V1.3 — DIHAPUS total:** ~~Setelah konvergensi tercapai, 1 AI (DeepSeek) berperan sebagai Devil's Advocate, ikut putaran debat terakhir, mencari celah keamanan/edge case/inkonsistensi, hasil sanggahan masuk `CLAIM_LEDGER`.~~ Mekanisme ini bentrok dengan peran DeepSeek sebagai `SHADOW_SCRIBE_MODE` (berisiko memberi kesan ikut debat inti, bukan sekadar mencatat).

**Mekanisme V1.4 — `FINAL_CUMULATIVE_SUMMARY + SCRIBE_ALERT`:**  
- Default **ON**, dijalankan **sekali** di akhir sesi (bukan tiap topik, bukan "1-2 putaran terakhir").  
- Input: `ROUND_SUMMARY` (seluruh round) + `DECISION_LEDGER` (ACTIVE) + `CLAIM_LEDGER` (klaim penting) + Friction Log.  
- Penulis: DeepSeek (Shadow Scribe) — **tetap tidak ikut debat inti**; output tidak masuk `CLAIM_LEDGER` sebagai `REBUTTAL` biasa.  
- Jika ditemukan masalah → jalur `SCRIBE_ALERT` (§8.11). Jika fatal → `VITAL_TRIGGER` dengan `Trigger_Source = SHADOW_SCRIBE`.  
- **Validasi akhir tetap lewat 5-Layer Validation Engine (§8.16) — DeepSeek bukan hakim final tunggal.**

### 9.7 Flow REVISI (Darma mengirim revisi)  
```
Resolusi_Darma = REVISI[teks]
→ GAS panggil ulang Current_AI_Turn yang SAMA
→ Round_In_Topic TIDAK increment
→ Instruksi Darma di-append ke packet prompt
→ Runtime_State = WAITING_AI
```

### 9.8 Alarm Warna Sheet & Laporan Telegram  
- Setiap selesai 1 putaran penuh → GAS kirim ringkasan via Telegram (wajib **bahasa Indonesia sederhana**).  
- Warna Sheet otomatis:  
  - 🟢 Hijau: `Discussion_State = CLOSED / READY_TO_CLOSE` DAN tidak ada `VITAL_TRIGGER` pending.  
  - 🟡 Kuning: `Discussion_State = DISCUSSING`.  
  - 🔴 Merah: Ada `VITAL_TRIGGER` pending (`WAITING_DARMA`) ATAU `Runtime_State = ERROR_RETRY` / `SCRIBE_ERROR`.

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
2. Isi `Round_Diajukan` = `Round_In_Topic`.  
3. Isi `Round_Terakhir_Sanggah` = `Round_In_Topic` (untuk klaim baru) ATAU update baris `Ref_ID` jika `REBUTTAL`.  
4. Insert row ke `CLAIM_LEDGER`.

---

## 11. AI PROMPT PACKET CONTRACT  

Setiap AI Core Panel menerima paket konteks terstruktur (bukan hanya `OPEN_CLAIMS`):

```text
[Instruksi Sistem / Mode Diskusi]
[Context_Summary dari Local Feeder]
[DECISION_LEDGER – hanya yang ACTIVE]
[ROUND_SUMMARY – putaran (Round_In_Topic) sebelumnya dalam topik aktif] ← Baru V1.4, wajib untuk Lapis 2 (§8.16)
[OBSERVATION_LEDGER – status OPEN/LINKED yang relevan] ← Baru V1.4
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
- **(Baru V1.4)** Semua Model ID (`ChatGPT_Model_ID`, `Gemini_Model_ID`, `Claude_Model_ID`, `DeepSeek_Model_ID`) juga wajib disimpan di Script Properties — **dilarang di-hardcode** di SOT atau kode GAS, karena lineup model tiap provider bisa berubah (lihat §8.9).

---

## 13. CHANGE GOVERNANCE & IMPACT LOG  

> **Status Sesi Patchset V1.4:** Sesi 1, 2, 3, 4, 5 — seluruhnya **CLOSED**.

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

### Impact Log 5 (V1.4): Komposisi Panel & Observation Ledger  
| Baris | Isi |
|-------|-----|
| **KONTRAK LAMA** | V1.3 — 5 AI ikut rotasi debat (Kimi, Qwen, Gemini, Claude, DeepSeek); DeepSeek berperan ganda sebagai debater sekaligus final reviewer (Devil's Advocate). |
| **KONTRAK BARU** | Core Panel = ChatGPT → Gemini → Claude (debater, mode Chat); DeepSeek = Shadow Scribe (pencatat, **tidak** ikut debat); Kimi/Qwen = Backlog. Tab baru `OBSERVATION_LEDGER` (§7 Tab 6) untuk observasi lapangan Darma — fakta tidak didebat, interpretasinya boleh. |
| **BLAST RADIUS** | §3.0 (definisi & komposisi panel), §4.1 (diagram rotasi), §9.1 (urutan rotasi), §9.6 (final review diganti total), §11 (Prompt Packet +Observation Ledger), Routing Logic +rule konflik Observation vs Decision Ledger, Dry-Run Item 18. |

### Impact Log 6 (V1.4): Topic Hierarchy & `Round_In_Topic`  
| Baris | Isi |
|-------|-----|
| **KONTRAK LAMA** | `SESSION_STATE` hanya punya `Current_Round` global tunggal; `Max_Rounds` global, default 10. |
| **KONTRAK BARU** | Hierarki `Session_ID → Topic_Session → Round_In_Topic`. `SESSION_STATE` (§7 Tab 5) +kolom `Current_Topic_Session`, `Topic_Title`, `Round_In_Topic`, `Max_Rounds_Per_Topic` (default **3**). Pindah topik reset `Round_In_Topic` ke 0. Tab baru `SYSTEM_ONTOLOGY` (Tab 7) & `ROUND_SUMMARY` (Tab 8, 8 kolom, ditulis Shadow Scribe tiap akhir round). |
| **BLAST RADIUS** | §8.1/§8.5 (baca `Round_In_Topic` bukan `Current_Round`), §7 Tab 2 (`Round_Diajukan` ikut berubah acuan), §10.4, §11 (Prompt Packet +`ROUND_SUMMARY` untuk validasi Lapis 2). |

### Impact Log 7 (V1.4): Indikator Selesai, Zero-Context Review, Bug Hunt  
| Baris | Isi |
|-------|-----|
| **KONTRAK LAMA** | Tidak ada indikator selesai yang teruji secara empiris; tidak ada mode investigasi bug terstruktur. |
| **KONTRAK BARU** | Indikator saturasi klaim kritis via uji 0-konteks (3 model Core Panel stateless, §8.15) + mode `BUG_HUNT` evidence-based (§8.14, 7 kolom, `Evidence_Ref` wajib, omission bug via `MISSING @ [Komponen]`). |
| **BLAST RADIUS** | `Trigger_Source` kolom baru di `VITAL_TRIGGER` (§7 Tab 4), §8.12 (pemisahan jalur fatal), Dry-Run Item 12–14, 17. |

### Impact Log 8 (V1.4): Mode Model, Async Shadow Scribe, Anti-Otonomi  
| Baris | Isi |
|-------|-----|
| **KONTRAK LAMA** | Belum ada batas mode (Chat vs Thinking) per peran model; Shadow Scribe dipanggil sinkron dalam alur yang sama dengan Core Panel; belum ada larangan eksplisit model mengeksekusi mutasi data. |
| **KONTRAK BARU** | Core Panel wajib Mode Chat (stateless); Shadow Scribe wajib Mode Thinking (default `deepseek-v4-flash`, `thinking.enabled`); arsitektur 2-eksekusi async (`WAITING_SCRIBE` → trigger terpisah → `SCRIBE_DONE`/`SCRIBE_ERROR`, watchdog 10 menit, §8.10); semua Model ID via Script Properties (dilarang hardcode, §12); Anti-Otonomi — semua LLM hanya kembalikan teks/JSON, GAS satu-satunya eksekutor mutasi data (§8.13). |
| **BLAST RADIUS** | §8.9–§8.10, §8.13, §12, Dry-Run Item 3, 4, 8, 10. |

### Impact Log 9 (V1.4): Skema Jangkar FATAL — Individual vs Completeness  
| Baris | Isi |
|-------|-----|
| **KONTRAK LAMA** | Draf awal patchset sempat mewajibkan kemunculan **6 jangkar** `[FATAL-1]` s/d `[FATAL-6]` di **setiap** `SCRIBE_ALERT:FATAL` individual — berisiko overreach/false-positive tinggi. |
| **KONTRAK BARU** | Dua scope dipisah tegas: **Individual Alert** cukup **1 jangkar valid**; **Completeness Check** (wajib 6 jangkar) **hanya** berlaku saat output mengklaim sebagai daftar lengkap kriteria FATAL (dokumen definisi/rekap final). |
| **BLAST RADIUS** | §8.11 (SCRIBE_ALERT Tier), §8.16 Lapis 3 (GAS Tier-1), Dry-Run Item 21(a)(b). |

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
| **Batas Putaran** | `Max_Rounds_Per_Topic` (default 3, per topik — bukan global). Dicek sebelum panggil round baru (§8.5). Final Cumulative Summary (§9.6) di luar hitungan ini, jalan sekali di akhir sesi. |
| **Monitoring Biaya** | **(Baru V1.4)** `Estimated_Cost_Session` & `Token_Usage_Cumulative` di `SESSION_STATE` — dihitung deterministik oleh GAS dari token usage × harga/token di Script Properties, **bukan** estimasi naratif LLM. |
| **Auto-Stop 24 Jam** | Jika Darma tidak respons → sesi dihentikan otomatis. |
| **Kendali Manual** | Telegram: `/pause`, `/lanjut`, `/stop`, `/status`. Untuk keputusan vital: `A`, `B`, `REVISI [teks]`, `STOP`. Untuk error/friction: `LANJUT`, `REVISI [teks]`, `STOP`. |
| **Komunikasi** | Semua pesan sistem ke Darma wajib **bahasa Indonesia sederhana**, bukan jargon teknis. Jika menyangkut keputusan vital, sistem wajib menjelaskan dampak praktis, risiko, saran default, dan pilihan balasan sederhana. |

---

## 15. ROLE & PERMISSION  

| Aktor | Hak Akses | Batasan |
|-------|-----------|---------|
| **Darma** | Full control. Satu-satunya yang bisa approve `VITAL_TRIGGER`, edit Sheet manual, stop sesi. Juga **Lapis 1** dalam 5-Layer Validation Engine (§8.16) — veto mutlak, bisa abort/interupsi kapan saja. | - |
| **GAS** | System executor. Satu-satunya penulis `SESSION_STATE`, `DECISION_LEDGER`, `VITAL_TRIGGER`. Juga **Lapis 3** (Tier-1 string-match check, §8.16). | Hanya menulis via logika yang sudah disepakati. |
| **Core Panel** (ChatGPT, Gemini, Claude) | Read `OPEN_CLAIMS`, `DECISION_LEDGER` (ACTIVE), `OBSERVATION_LEDGER` (OPEN/LINKED) — dari Prompt Packet (§11). Juga **Lapis 2** (membaca `ROUND_SUMMARY` putaran N sebagai N+1). | Tidak punya akses langsung ke Sheet. Tidak bisa menulis/mengubah data (Anti-Otonomi, §8.13). |
| **Shadow Scribe** (DeepSeek) | **(Baru V1.4)** Read seluruh output Core Panel per round. Tulis `ROUND_SUMMARY` (§7 Tab 8) & `SCRIBE_ALERT` saja. | **Tidak ikut debat inti.** Tidak bisa mengunci keputusan. Tidak boleh menulis `DECISION_LEDGER` atau `CLAIM_LEDGER` sebagai rebuttal biasa. |
| **Meta-LLM Filter** | Tulis `CLAIM_LEDGER` saja (via output JSON). Juga ekstrak `Obs_Key` di `OBSERVATION_LEDGER` tanpa mengubah `Darma_Input_Raw`. | Tidak boleh mengunci keputusan. Tidak boleh menulis `DECISION_LEDGER`. |
| **Terminal Agent** | Baca folder proyek yang diizinkan. Membuat ringkasan. | Tidak boleh mengirim file sensitif. Tidak boleh eksekusi tanpa preview + persetujuan Darma. |

> Kimi, Qwen: Backlog, belum punya hak akses di V1.4 (§3.0).

### 15.1 Peta Tanggung Jawab Darma (Baru V1.4)  

| Kategori | Contoh Konkret | Cara Tahu | Aksi Darma |
|---|---|---|---|
| **Otomatis penuh** | Rotasi Core Panel, Meta-LLM Filter, `ROUND_SUMMARY`, Tier 1 GAS check, Auto-Promote technical, Stuck Detection, LockService/CacheService | — | Tidak perlu apa-apa |
| **Push wajib respons** | `VITAL_TRIGGER` (`WAITING_DARMA`/`SYSTEM_FRICTION`), `SCRIBE_ALERT:FATAL`, watchdog deadlock berulang ≥2x | Notifikasi Telegram masuk | Balas `A/B/REVISI/STOP` atau `LANJUT` |
| **Cek berkala (santai)** | `OBSERVATION_LEDGER` status `OPEN`, `ROUND_SUMMARY.Open_Friction`, warna Sheet (merah/kuning/hijau), `Estimated_Cost_Session` | Buka Sheet sendiri kapan mau | Tidak urgent, pantau sesempatnya |
| **Lock versi (jarang)** | Lapis 5 Diff Checklist sebelum naik versi major | Trigger manual Darma, atau otomatis via `lockVersion()` | Review sebelum kunci dokumen |

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
| **Kimi & Qwen aktif di Core Panel/rotasi** | **(Baru V1.4)** Backlog penuh — fokus V1.4 pada Core Panel 3-AI + Shadow Scribe dulu, bukan kompleksitas 5-AI rotation. |

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

### Friction Log Sesi 1–5 (Patchset V1.4)

| FRICTION | CATATAN | STATUS |
|---|---|---|
| Peran DeepSeek ambigu di awal | Sempat dianggap peserta debat/final reviewer → dikoreksi jadi `SHADOW_SCRIBE_MODE`. | ✅ Dikoreksi |
| `SCRIBE_ALERT` auto-pause terlalu agresif | Usul auto-pause semua alert → dikoreksi jadi tier `FATAL`/`WARNING`. | ✅ Diperbaiki |
| Pertanyaan teknis dilempar ke Darma | Sempat tanya format `SYSTEM_ONTOLOGY` ke Darma, melanggar batas intervensi mikro. | ✅ Dikoreksi |
| `DEFENSE` belum lengkap di draf awal | Reliability guard (`LockService`, `CacheService`, Auto-Stop) sempat hilang dari draf awal `SYSTEM_ONTOLOGY`. | ✅ Diperbaiki |
| Wording DeepSeek & eskalasi WARNING ambigu | Diperbaiki: DeepSeek aktif (bukan pasif), label histori WARNING tidak diubah jadi FATAL. | ✅ Diperbaiki |
| Roadmap sempat bergeser jadi "arsitektur umum" | Dikembalikan ke topik semula. | ✅ Dikoreksi |
| Asumsi model ke-4 untuk uji 0-konteks | Sempat diasumsikan LLM independen, dikoreksi jadi reuse 3 model Core Panel stateless (Opsi B). | ✅ Diperbaiki |
| Round habis disangka selesai | Dikoreksi: round habis hanya batas operasional, bukan indikator kebenaran jika klaim kritis masih terbuka. | ✅ Diperbaiki |
| Bug-hunt rawan tak terbatas | Dikunci dengan mode khusus, `Evidence_Ref`, dan status `INVALID` jika tanpa bukti. | ✅ Diperbaiki |
| Jalur FATAL rawan bercampur | Ditambah `Trigger_Source` agar fatal dari bug-hunt, scribe, dan 0-context review bisa diaudit terpisah. | ✅ Ditambahkan |
| Amnesia Runtime Defense | Komponen pertahanan krusial V1.3 sempat terlewat dalam draf awal `DEFENSE`. | ✅ Diperbaiki |
| Reduksi Konteks Uji | Desain awal uji 0-konteks terlalu sempit, diperluas. | ✅ Diperbaiki |
| Blindspot Omission Bug | Aturan wajib `Evidence_Ref` berisiko membuang temuan celah kelalaian, diadaptasi dengan `MISSING @`. | ✅ Diperbaiki |
| Pricing model (DeepSeek V4-Pro) sempat tidak sinkron | Data temporal stale — sempat memakai harga sebelum cutoff diskon permanen 31 Mei 2026. Dikoreksi ke $0.435/$0.87 (harga terverifikasi saat itu; **harga aktual tetap wajib dicek live saat dry-run**, bukan dihardcode di SOT). | ✅ Dikoreksi |
| Klaim kuota trigger GAS belum terverifikasi solid | Sumber saling bertentangan; diputuskan tidak hardcode angka, pindah jadi item cek dashboard langsung (Dry-Run Item 2). | ✅ Diputuskan |
| Fragmentasi Struktur Patchset | Komponen parameter model, indikator saturasi, dan loop trigger sempat tercecer, disatukan kembali. | ✅ Disatukan |
| Nama model ChatGPT/Gemini/Claude sudah usang | Tidak diverifikasi seketat DeepSeek; dikoreksi jadi pola configurable Script Properties (§8.9, §12). | ✅ Diperbaiki |
| Regresi Item Dry-Run | Item "Uji Bias Agreement DeepSeek" sempat luput dari kompilasi awal, ditambahkan sebagai Item 18 (§19). | ✅ Ditambahkan |
| Spekulasi Metrik Latensi | Angka latensi tanpa dasar data, diganti instruksi pengukuran empiris saat dry-run (Item 1). | ✅ Dikoreksi |
| Self-validation blindspot | Shadow Scribe tidak bisa memvalidasi output sendiri. Ditutup dengan 5-Layer Validation Engine (§8.16). | ✅ Ditutup |
| Amnesia kompilasi AI | Dua fix yang sudah disepakati (configurable model, item dry-run) sempat hilang dari rekap. Ditutup dengan Lapis 5: Diff Checklist via `DECISION_LEDGER`. | ✅ Ditutup |
| Redundansi `PATCH_MANIFEST` | Usul tab baru tumpang tindih dengan `DECISION_LEDGER`; diubah jadi GAS report on-demand khusus lock versi major (§8.16). | ✅ Dibatalkan |
| Race Condition State | Celah intervensi manual saat Scribe berjalan belum diatur. Ditutup dengan prioritas input Darma (Dry-Run Item 19). | ✅ Ditutup |
| Omission Loop (Cost Monitor & Peta Tanggung Jawab) | Cost monitor dan Peta Tanggung Jawab Darma sempat hilang dari kompilasi final, ditambal di putaran terakhir. | ✅ Ditambal |
| Human-Error Dependency | Lapis 5 awalnya manual, diubah jadi *Automated Pre-Hook* agar tidak bergantung ingatan Darma. | ✅ Diperbaiki |
| Ambiguitas istilah "CLOSED" | Dipisah: `Version_Lock_Status` vs `Discussion_State` untuk hindari salah implementasi. | ✅ Dipisah |
| Mekanisme cost kalkulasi kosong | Dikunci: GAS hitung dari token usage × harga di Script Properties, bukan estimasi LLM. | ✅ Dikunci |
| Degradasi akurasi kompilasi panjang | Ketelitian sempat menurun pada kompilasi teks panjang (beberapa kali *omission*). Risiko sudah tertutup struktural oleh Lapis 5; solusi spesifik dievaluasi lagi jika pola berulang di dry-run nyata. | ⚠️ Dipantau |
| Celah referensi eksternal | Pointer ke diskusi lama diganti tabel Impact Log self-contained yang merujuk ke bagian dalam dokumen ini sendiri. | ✅ Diperbaiki |
| Kerentanan operasional Tier 1 | Deteksi kelengkapan teks bebas berisiko *false-positive* akibat variasi bahasa LLM; diselesaikan dengan Jangkar Kode Eksak `[FATAL-X]` & penyimpanan kanonik di `SYSTEM_ONTOLOGY`. | ✅ Diperbaiki |
| **Aturan jangkar FATAL salah scope** | Tier 1 sempat mewajibkan 6 jangkar di SETIAP alert individual, padahal cukup 1. Completeness check hanya untuk daftar lengkap. Lihat Impact Log 9 (§13), Dry-Run Item 21. | ✅ Diperbaiki |
| **Istilah `Token_Cost_Cumulative` rancu** | Diganti `Token_Usage_Cumulative` karena isinya jumlah token, bukan nominal biaya. | ✅ Diperbaiki |

---

## 19. LANGKAH SELANJUTNYA (DRY-RUN) — REVISED CHECKLIST V1.4 (21 ITEM)

> §19 V1.3 (9 langkah, berbasis rotasi 5-AI dan struktur 5 tab) **diganti total** — sebagian besar item lama sudah tidak relevan dengan arsitektur Core Panel + Shadow Scribe + 8 tab.

**Item 1–10 (Validasi Dasbor & Integrasi):**
1. **Uji Latensi & Endpoint:** Ukur durasi respons aktual endpoint `DeepSeek_Model_ID`, tentukan threshold setelah data dry-run pertama.
2. **Audit Kuota Kumulatif GAS:** Jalankan 1 sesi penuh, hitung total konsumsi runtime 3 trigger simultan (round-progress + scribe-call + watchdog) vs limit harian di Apps Script Dashboard.
3. **Uji State Transition:** `WAITING_SCRIBE` → trigger eksternal → `SCRIBE_DONE`.
4. **Uji Scribe Timeout Watchdog:** `WAITING_SCRIBE` >10 menit → force `SCRIBE_ERROR` + `BYPASS_LANJUT`.
5. **Uji Validasi Skema Tab Baru:** Tulis `SYSTEM_ONTOLOGY` & `OBSERVATION_LEDGER` tanpa overlap row.
6. **Uji Parameter Batasan Topik:** `Round_In_Topic` sentuh `Max_Rounds_Per_Topic` (default 3) → interupsi otomatis.
7. **Uji Sinkronisasi Token Cache:** Verifikasi biaya cache-miss vs cache-hit di log API DeepSeek.
8. **Uji Isolasi Stateless Core Panel:** Panggilan ChatGPT/Gemini/Claude bersih dari sisa histori sesi sebelumnya.
9. **Uji Integrasi Makro Penanda:** Teks balasan tidak memotong format Markdown tabel SOT.
10. **Uji Penanganan Scribe Error:** Putus koneksi API DeepSeek sengaja → sistem catat `SCRIBE_ERROR` dengan aman.

**Item 11–21 (Validasi Edge Case):**
11. **Uji Observation Conflict:** Observasi bentrok `DECISION_LEDGER ACTIVE` → `VITAL_TRIGGER`.
12. **Uji Zero-Context Review:** Panggil Core Panel stateless (Ledger + SOT + Summary terakhir) → tidak ada klaim kritis baru di luar 6 indikator saturasi.
13. **Uji BUG_HUNT Evidence:** Input tanpa `Evidence_Ref` → `INVALID`.
14. **Uji Omission Bug:** `MISSING @ [Component]` lolos validasi sebagai bukti valid.
15. **Uji Parsing False-Positive:** String `"WARNING: ini bukan FATAL"` tidak memicu pause.
16. **Uji Warning Escalation:** `WARNING` diabaikan 2 round → naik ke `VITAL_TRIGGER` di round ketiga.
17. **Uji Audit Trigger Source:** Semua rute fatal (`BUG_HUNT`, `SHADOW_SCRIBE`, `ZERO_CONTEXT_REVIEW`, `SCRIBE_VALIDATION`) mengisi `Trigger_Source` dengan benar.
18. **Uji Bias Agreement DeepSeek:** Injeksi skenario *false-consensus* (panel pura-pura sepakat padahal bertentangan) → DeepSeek wajib mendeteksi `SCRIBE_ALERT: FATAL`.
19. **Uji Manual Override saat `WAITING_SCRIBE`:** Darma kirim resolusi `A/B/REVISI/STOP` ketika Shadow Scribe masih berjalan → Input Darma menang; hasil scribe terlambat dicatat sebagai *overtaken* dan tidak mengubah state.
20. **Uji Automated Pre-Hook Diff Checklist:** Panggil fungsi `lockVersion()` dengan draf final yang sengaja menghilangkan 1 patch aktif dari `DECISION_LEDGER`. Sistem wajib mendeteksi bahwa patch hilang, menolak transisi ke `Version_Lock_Status = LOCKED`, dan melempar *error report*.
21. **Uji Canonical Completeness vs Individual Alert:**
    - **(a)** Minta Shadow Scribe restate ulang daftar lengkap 6 kriteria FATAL (skenario kompilasi definisi/rekap) dengan sengaja dipotong. Tier 1 GAS wajib hitung jangkar unik di output yang **mengklaim sebagai daftar lengkap**; jika <6 → `VITAL_TRIGGER` (`Trigger_Source: SCRIBE_VALIDATION`).
    - **(b)** Kirim 1 `SCRIBE_ALERT:FATAL` individual yang hanya menyebut `[FATAL-3]`. Pastikan **TIDAK** memicu `VITAL_TRIGGER` palsu — membuktikan aturan tidak overreach ke kasus individual.

**Setelah seluruh 21 item di atas sukses** (di luar penomoran checklist): kunci sebagai V1.4 production.

---

**=== STATUS AKHIR: CLOSED ===**

Panel AI telah mencapai **kesepakatan penuh** terhadap seluruh poin di atas, termasuk Patchset V1.4 (Sesi 1–5).  
Semua 5 Keputusan Vital dari SOT awal Darma telah tercover lengkap, sekarang diperkuat dengan 5-Layer Validation Engine (§8.16).  
**Dokumen ini adalah Source of Truth (SoT) untuk Smart Moderator V1.4 Final.**  
Setiap perubahan di masa depan wajib melalui Change Governance (Impact Log 3 Baris, §13), dan setiap kunci versi major wajib lolos Lapis 5 Diff Checklist (`lockVersion()`).