# SOT – SMART MODERATOR V1.4.2  
**Sumber:** SOT V1.4 Final + Patchset V1.4.1 + Patchset V1.4.2 (Pra-Sesi & Sesi 1–4, Diskusi Panel AI)  
**Core Panel (Debater):** ChatGPT → Gemini → Claude (mode Chat, *pure reasoner*, sekuensial)  
**Shadow Scribe:** DeepSeek (`SHADOW_SCRIBE_MODE`) — pencatat & pendeteksi alert, **tidak ikut debat inti**  
**Backlog Panel:** Kimi, Qwen (belum aktif di V1.4.2 – ketidakhadiran tidak menggugurkan konsensus, tercatat di Friction Log)  
**Status:** **CLOSED – Siap Implementasi & Dry-Run (34 Item)**  
**Versi:** V1.4.2 (Final, Patchset V1.4 Sesi 1–5 + Patchset V1.4.1 Sesi 1–4 & Gap Akhir + Patchset V1.4.2 Pra-Sesi & Sesi 1–4)

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
20. Resolved Micro-Decisions & Deferred Items

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

## 7. KONTRAK DATA LEDGER – STRUKTUR GOOGLE SHEETS (8 TAB, REVISI V1.4.1)

### TAB 1: `RAW_ARCHIVE` (Arsip Mentah)  
- **Fungsi:** Menyimpan teks respons asli dari setiap AI, utuh. Untuk audit manual Darma jika terjadi halusinasi. **Tidak** dikirim ulang ke AI lain.  
- **Kolom:** `Log_ID/Payload_ID`, `Timestamp`, `Session_ID`, `AI_Actor`, `Full_Response`, `Token_Cost`.  
- **Edge Case:** Jika `Full_Response` > 50.000 karakter, GAS memotong dan mencatat di `VITAL_TRIGGER`.

### TAB 2: `CLAIM_LEDGER` (Gelanggang Debat) – 9 Kolom Final (Revisi V1.4.1)  
| Kolom | Diisi Oleh | Keterangan |
|-------|------------|------------|
| `Claim_ID` | **GAS** | Format: `CLM-{session_id}-{counter}` (hindari kolisi) |
| `Topic_Session` | **GAS** | **(Baru V1.4.1)** ID topik saat klaim diajukan, dari `SESSION_STATE.Current_Topic_Session`. Mencegah klaim topik lama bercampur topik baru; query `OPEN_CLAIMS` kini di-filter `Session_ID + Topic_Session` (§10.1). |
| `Ref_ID` | Meta-LLM Filter | `Claim_ID` yang disanggah (jika `REBUTTAL`), `null` jika `NEW_IDEA`. **(V1.4.1)** Rujukan lintas topik (*cross-topic*) tetap diizinkan dalam satu `Session_ID`; klaim rebuttal-nya sendiri dicatat dengan `Topic_Session` topik aktif. **(V1.4.2) Imutabilitas cross-topic:** Jika `Ref_ID` berasal dari `Topic_Session` berbeda, GAS **dilarang** mengubah `Status` atau `Round_Terakhir_Sanggah` klaim lama secara otomatis. Cross-topic `Ref_ID` hanya bersifat referensi historis. Perubahan klaim lintas topik hanya boleh terjadi lewat `DECISION_LEDGER` / Vital Gate dengan keputusan eksplisit Darma. |
| `Kategori` | Meta-LLM Filter | `NEW_IDEA`, `REBUTTAL`, `TECHNICAL_DETAIL`, `RISK_WARNING` |
| `Konteks_Vital` | Meta-LLM Filter | `ANCHOR`, `CAPABILITY`, `FLOW`, `ARCH`, `OPERATIONAL`, `TECHNICAL` |
| `Isi_Singkat` | Meta-LLM Filter | Maksimal 15 kata, inti klaim tanpa basa-basi |
| `Status` | GAS | `DEBATABLE` (default), `REJECTED`, `PROMOTED`, `STALE` **(Baru V1.4.1)** — GAS mass-update semua `DEBATABLE` di topik tsb menjadi `STALE` saat topik ditutup (`Topic_Closure=TRUE`, §7 Tab 8), mencegah *ghost audit*. |
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
| `Trigger_Source` | **(Revisi V1.4.2)** Wajib diisi di **semua** baris `VITAL_TRIGGER` (audit lengkap, tanpa kecuali). Enum sah (**13 nilai**): `CORE_PANEL_BUG_HUNT`, `SHADOW_SCRIBE`, `ZERO_CONTEXT_REVIEW`, `SCRIBE_VALIDATION`, `VITAL_GATE`, `API_ERROR`, `MAX_ROUND_LIMIT`, `WATCHDOG_DEADLOCK`, `MANUAL_OVERRIDE`, `STUCK_DETECTION`, `META_LLM_FILTER_ERROR` **(Baru V1.4.2 — §8.8, Tipe=`SYSTEM_FRICTION`, bisa bypass)**, `OBSERVATION_CONFLICT` **(Baru V1.4.2 — Observation Ledger bentrok DECISION_LEDGER ACTIVE, Tipe=`WAITING_DARMA`, wajib berhenti total, resolusi: `A`/`B`/`REVISI`/`STOP`)**, `UNKNOWN`. `SHADOW_SCRIBE` = alert sah dari Scribe tentang isi debat; `SCRIBE_VALIDATION` = alert sistem tentang Scribe-nya sendiri yang salah/kurang; `VITAL_GATE` = Vital Gate manual (§8.2); `UNKNOWN` = fallback, counter ≥3 dalam satu sesi → otomatis `VITAL_TRIGGER` baru (§8.12). **Larangan:** `SYSTEM_FRICTION` hanya boleh dipakai di kolom `Tipe`, dilarang masuk `Trigger_Source`. |

### TAB 5: `SESSION_STATE` (Konfigurasi + Pointer Resume) – Revisi V1.4.1  
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
| `Runtime_State` | `WAITING_AI`, `WAITING_FILTER`, `WAITING_SCRIBE`, `SCRIBE_DONE`, `SCRIBE_ERROR`, `WAITING_DARMA`, `ERROR_RETRY`, `FINAL_CUMULATIVE_SUMMARY`, `PAUSED_MANUAL` **(Baru V1.4.1 — jeda manual Darma, beda dari `WAITING_DARMA`; lihat §9.10)**, `DONE` |
| `Estimated_Cost_Session` | **(Baru V1.4, Revisi V1.4.1)** Angka numerik murni — akumulasi taksiran biaya API per sesi, dihitung **deterministik oleh GAS** dari token usage × harga per token di Script Properties — **atau** `UNKNOWN` jika harga belum dikonfigurasi (membedakan "gratis"=0 dari "belum dikonfigurasi"). **Bukan** estimasi naratif LLM. |
| `Cost_Status` | **(Baru V1.4.1)** `OK` / `STALE` / `UNKNOWN` — dihitung otomatis GAS dengan membandingkan `Cost_Price_Updated` terhadap ambang `Cost_Staleness_Days` (default 30 hari, configurable di Script Properties, §12). |
| `Cost_Price_Updated` | **(Baru V1.4.1)** Tanggal verifikasi harga **tertua** di antara semua model yang dipakai dalam sesi tsb — prinsip *Oldest Date Wins* ("rantai terlemah"). |
| `Token_Usage_Cumulative` | **(Baru V1.4)** Total token terpakai (kumulatif sesi) — kolom independen, **tetap dipertahankan terpisah** dari 3 kolom biaya/status di atas. |
| `Version_Lock_Status` | **(Baru V1.4)** `PENDING` / `LOCKED` — dipakai Lapis 5 (§8.16) saat proses kunci versi dokumen. Berbeda dari `Discussion_State = CLOSED` yang dipakai untuk penutupan sesi topik biasa. |
| `Next_Action` | **(Baru V1.4, Revisi V1.4.1)** Instruksi runtime **transien** — di-reset ke `NONE` setelah eksekusi berhasil & penulisan ke Sheets terkonfirmasi (cegah *stale instruction loop*). **Guard utama sistem tetap `Runtime_State`, bukan kolom ini** (cegah *dual hold-state conflict*; pause dipegang `WAITING_DARMA`/`PAUSED_MANUAL`, stop oleh `DONE`) — lifecycle & urutan eksekusi lengkap di §8.17. Enum sah (6 nilai): `NONE`, `BYPASS_LANJUT`, `RETRY_SAME_AI`, `WAIT_DARMA`, `STOP`, `RESUME_CORE_PANEL`. |
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

### TAB 8: `ROUND_SUMMARY` (Rekap Tiap Round oleh Shadow Scribe) – Baru V1.4, Revisi V1.4.1  
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
| `Topic_Closure` | **(Baru V1.4.1)** `TRUE` / `FALSE` — menandai ringkasan putaran ini sebagai penutup resmi topik. **Diisi GAS saja** (bukan DeepSeek, patuh Anti-Otonomi §8.13), saat: Darma eksplisit menutup topik, **atau** `Max_Rounds_Per_Topic` tercapai dan Darma konfirmasi ganti topik — **bukan** mengandalkan `Discussion_State=CLOSED` global (revisi dari draf awal, lihat §9.10). Guard tambahan: tidak ada `VITAL_TRIGGER`/`WAITING_DARMA` terbuka untuk `Topic_Session` tsb. |

**Penulis:** **DeepSeek (Shadow Scribe)** menulis `ROUND_SUMMARY` tiap akhir round, **kecuali `Topic_Closure`** yang diisi GAS. GAS hanya menyimpan dan merutekan sisanya — **bukan tab gabungan**, fungsi audit & konteks ringkasnya berdiri sendiri dari tab lain.  
**Parsing wajib:** `split(";")` → `trim()` → `startsWith("FATAL:")` / `startsWith("WARNING:")` — **bukan** `.includes()` (mencegah false-positive).

---

## 8. ROUTING LOGIC GAS – ATURAN LENGKAP (TERMASUK SEMUA PATCH)

### 8.1 Auto-Promote (Otomatis, Tanpa Darma)  
**Syarat (semua harus terpenuhi):**  
- `Konteks_Vital == TECHNICAL`  
- `(Round_In_Topic - Round_Terakhir_Sanggah) >= 2` — bertahan 2 putaran penuh tanpa `REBUTTAL` baru.  
- `Status == DEBATABLE`  
- **(V1.4.1)** Evaluasi di-scope per `Topic_Session` aktif (`Session_ID + Topic_Session`) — klaim dari topik lain tidak ikut dihitung.  
- **(V1.4.2) Hard-Guard — konflik DECISION_LEDGER ACTIVE:** Jika klaim baru memiliki `Ref_ID` yang menunjuk ke `Source_Claim_ID` yang sudah `Decision_Status=ACTIVE` di `DECISION_LEDGER`, **Auto-Promote dimatikan** → klaim wajib masuk **Vital Gate** (`WAITING_DARMA`, §8.2). Mencegah dua keputusan `ACTIVE` yang saling bertentangan tertulis bersamaan sebelum Lapis 3 sempat menangkap (*race condition*).  

**Aksi:** `PROMOTED` → masuk `DECISION_LEDGER` (`Approved_By = AUTO`).

### 8.2 Vital Gate (Wajib Tanya Darma)  
**Syarat:** `Konteks_Vital != TECHNICAL` dan klaim membutuhkan keputusan.  

**Aksi:** `VITAL_TRIGGER` (Tipe: `WAITING_DARMA`, `Trigger_Source: VITAL_GATE`), `Runtime_State = WAITING_DARMA`, notifikasi Telegram, pause sesi. GAS wajib menyimpan paket pertanyaan awam ke kolom `Pemicu` sebelum mengirim notifikasi Telegram, agar keputusan Darma bisa diaudit ulang.

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
**Syarat:** Klaim `DEBATABLE` dengan `Konteks_Vital != TECHNICAL` — konflik berulang tanpa kemajuan, **di-scope per `Topic_Session` aktif (V1.4.1)**.  

**Aksi:**  
- **N=2 putaran:** Warning via Telegram (sistem tetap jalan).  
- **N=3 putaran:** Pause/Stop, `VITAL_TRIGGER` (Tipe: `SYSTEM_FRICTION`, `Trigger_Source: STUCK_DETECTION`), Darma wajib turun tangan.

### 8.4 Auto-REJECTED (Final – Sintesis Claude)  
- **Jalur TECHNICAL:** Jika sebuah `REBUTTAL` ter-Auto-Promote, GAS otomatis set klaim yang disanggah (`Ref_ID`) → `REJECTED`. **(V1.4.2) Guard cross-topic:** Auto-REJECTED **hanya berlaku** jika `Ref_ID` berada dalam `Topic_Session` yang **sama**. Jika `Ref_ID` lintas topik, GAS **dilarang** mengubah status klaim lama secara otomatis — catat sebagai cross-topic rebuttal; jika perlu mengubah keputusan lama, wajib lewat `DECISION_LEDGER` / Vital Gate.  
- **Jalur VITAL:** TIDAK ADA auto-reject. Darma cukup memilih `A`, `B`, `REVISI [teks]`, atau `STOP`. GAS yang memetakan pilihan Darma ke `Affects_Claim_ID` secara internal; Darma tidak perlu menyebut `Claim_ID`.  
- Tidak ada `Rebuttal_Type` — pertahankan prinsip boring.

### 8.5 Cek Batas Putaran (`Max_Rounds_Per_Topic`) — Revisi V1.4  
- `Max_Rounds_Per_Topic` = jumlah putaran penuh **dalam 1 topik** (satu putaran = ChatGPT → Gemini → Claude selesai semua). Default **3**.  
- `Round_In_Topic` hanya naik setelah Claude (AI terakhir Core Panel) selesai. Pemanggilan Shadow Scribe (DeepSeek) berjalan **async terpisah** (§8.10) dan **tidak** mempengaruhi increment `Round_In_Topic`.  
- Ganti `Current_Topic_Session` → `Round_In_Topic` reset ke 0.  
- Sebelum memulai round baru, GAS cek: `Round_In_Topic >= Max_Rounds_Per_Topic`?  
  → **Ya:** `Runtime_State = WAITING_DARMA`, `VITAL_TRIGGER` (Tipe: `SYSTEM_FRICTION`, `Trigger_Source: MAX_ROUND_LIMIT`, Pemicu: "Batas putaran topik tercapai").  
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
→ VITAL_TRIGGER: Tipe=SYSTEM_FRICTION, Trigger_Source=API_ERROR, Pemicu="<AI> gagal merespon 2x"
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
    jika tetap invalid → VITAL_TRIGGER (Tipe: SYSTEM_FRICTION, Trigger_Source: META_LLM_FILTER_ERROR), pause
}
Ref_ID tidak valid → drop Ref_ID (jadi null) atau trigger friction, jangan diproses diam-diam
Output kosong → boleh [] (bukan error)
```

### 8.9 Mode Operasional & Parameter Model (Baru V1.4)  
- **Core Panel:** wajib Mode Chat (stateless, deterministik).  
- **Shadow Scribe:** wajib Mode Thinking, dengan `thinking.enabled`. **(Revisi V1.4.1)** `deepseek-v4-flash` hanya kandidat awal/rekomendasi — **bukan kontrak permanen**; model aktif wajib diverifikasi ulang dari provider saat dry-run, baru diisi ke `DeepSeek_Model_ID` (Script Properties).  
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
- **Strict Gate (Baru V1.4.1):** Putaran N+1 **wajib menunggu** `SCRIBE_DONE` sebelum mulai — menjaga integritas Lapis 2 Validasi (§8.16), karena Core Panel N+1 harus membaca `ROUND_SUMMARY` putaran N. Bypass hanya lewat `SCRIBE_ERROR` + `Next_Action = BYPASS_LANJUT`. Lifecycle lengkap `Next_Action` di §8.17.  
- **Watchdog 10 menit:** `WAITING_SCRIBE` >10 menit → force `SCRIBE_ERROR` + `Next_Action = BYPASS_LANJUT`. Jika berulang ≥2 kali dalam satu sesi topik → `VITAL_TRIGGER` (`Tipe: SYSTEM_FRICTION`, `Trigger_Source: WATCHDOG_DEADLOCK`, `Runtime_State = WAITING_DARMA`) **(dikunci V1.4.2 — konsisten dengan pola `API_ERROR`)**.  
- **(V1.4.2) Pemisahan layer SCRIBE_ERROR vs WATCHDOG_DEADLOCK:** `SCRIBE_ERROR` adalah **`Runtime_State`** — dipakai GAS untuk auto-bypass tanpa memerlukan input Darma. `WATCHDOG_DEADLOCK` adalah **`Trigger_Source`** — baru muncul saat `SCRIBE_ERROR` berulang ≥2x dan sistem butuh `/lanjut` dari Darma (via whitelist §9.10). Keduanya berbeda layer data; jangan dicampur.  
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
- **Parsing wajib:** `split(";")` → `trim()` → `startsWith("FATAL:")` / `startsWith("WARNING:")` — **bukan** `.includes()`. **(V1.4.2) Larangan deteksi substring teks bebas:** deteksi string `[FATAL-` di dalam teks bebas/prosa AI **tidak digunakan** — rawan false-positive. Sinyal fatal hanya sah dari sumber terstruktur (`SCRIBE_ALERT` via Shadow Scribe) yang sudah masuk §11 Prompt Packet.

### 8.12 Pemisahan Jalur FATAL & `Trigger_Source` (Baru V1.4, Revisi V1.4.2)  
`VITAL_TRIGGER` (§7 Tab 4) **wajib** diisi `Trigger_Source` di **semua** baris, tidak hanya fatal — enum lengkap **13 nilai** ada di §7 Tab 4. Tujuannya agar setiap pemicu (bug-hunt, scribe, 0-context review, vital gate manual, error, limit, dll.) bisa diaudit terpisah. Dua nilai baru di V1.4.2: `META_LLM_FILTER_ERROR` (parser gagal, §8.8) dan `OBSERVATION_CONFLICT` (observasi Darma bentrok `DECISION_LEDGER ACTIVE`).  
**(Baru V1.4.1) Fallback `UNKNOWN`:** dipakai saat GAS gagal mengklasifikasi sumber pemicu. Jika counter `UNKNOWN` ≥3 kali dalam satu sesi → GAS otomatis membuat `VITAL_TRIGGER` baru dengan `Trigger_Source = UNKNOWN` (mendeteksi degradasi fungsi parser; ambang 3 mencegah false-positive).

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

### 8.17 `Next_Action` Lifecycle & Strict Gate (Baru V1.4.1)  
Mengatur secara eksplisit interaksi antara `Runtime_State` (guard utama/penjaga status) dan `Next_Action` (instruksi transien) agar tidak terjadi *dual hold-state conflict*.  

- **Guard utama sistem adalah `Runtime_State`, bukan `Next_Action`.** Pause dipegang oleh `WAITING_DARMA` (menunggu keputusan) atau `PAUSED_MANUAL` (jeda sadar Darma, §9.10); stop dipegang oleh `DONE`.  
- **Urutan eksekusi GAS yang wajib (setiap siklus):** (1) Periksa `Runtime_State` → (2) Eksekusi `Next_Action` jika sistem tidak sedang pause/stop → (3) Reset `Next_Action = NONE`. Urutan ini mencegah eksekusi aksi saat sistem semestinya diam.  
- **Sifat transien:** `Next_Action` di-reset ke `NONE` setelah eksekusi berhasil **dan** penulisan ke Sheets terkonfirmasi — mencegah instruksi lama terbaca ulang (*stale instruction loop*).  
- **Untuk `WAIT_DARMA`:** GAS mengatur `Runtime_State = WAITING_DARMA` dan menyimpan baris `VITAL_TRIGGER`. Setelah penulisan ke Sheets terkonfirmasi, `Next_Action` di-reset ke `NONE` — status pause selanjutnya dijaga sepenuhnya oleh `Runtime_State`, bukan oleh `Next_Action` yang sudah kosong.  
- **Idempotensi:** Jika `Runtime_State = WAITING_DARMA` dan `Next_Action = WAIT_DARMA` muncul lagi akibat trigger berulang, GAS **mengabaikannya** dan langsung me-reset `Next_Action = NONE` — mencegah duplikasi `VITAL_TRIGGER`.  
- **Strict Gate Shadow Scribe:** lihat §8.10 — putaran N+1 tidak boleh mulai sebelum `SCRIBE_DONE`, kecuali via `SCRIBE_ERROR` + `Next_Action = BYPASS_LANJUT`.  
- **Manual override saat `WAITING_SCRIBE`:** secara standar dicatat di `ROUND_SUMMARY.Open_Friction` sebagai `OVERTAKEN_BY_DARMA` (tidak mengubah state). Eskalasi menjadi `VITAL_TRIGGER` (`Trigger_Source = MANUAL_OVERRIDE`) **hanya jika** terjadi konflik state yang nyata — mencegah antrian vital dipenuhi kejadian non-kritis. Hasil Scribe yang datang terlambat **tidak boleh** mengubah keputusan Darma yang sudah berjalan; maksimal dicatat sebagai *late/ignored result*, karena Darma adalah otoritas tertinggi.

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
- **(V1.4.2) Cross-topic ke `CLAIM_LEDGER` tidak aktif:** Karena `OPEN_CLAIMS` selalu difilter per `Topic_Session` aktif (§10.1), Meta-LLM Filter tidak pernah melihat `Claim_ID` dari topik lain — jalur ini secara teknis tidak ada. Tidak perlu diadakan sebelum terbukti dibutuhkan saat dry-run (Architecture-Ready Later).  

**Layer 2: GAS Query + Template (tanpa LLM)**  
- GAS query `WHERE Session_ID = current Session_ID AND Topic_Session = Current_Topic_Session AND Status = DEBATABLE` dari `CLAIM_LEDGER` **(Revisi V1.4.1 — selaras dengan §7 Tab 2, §10.1, §11)**.  
- Susun jadi **Diff Context** (teks bullet-point pendek) untuk AI berikutnya.  
- Tidak butuh LLM kedua — hemat biaya, deterministik.  
- **(Baru V1.4.2) Context Priority Rule** — klaim di-tier sebelum dikirim ke AI berikutnya:

| Tier | Kriteria | Nasib di Prompt |
|------|----------|-----------------|
| **Tier 1 – WAJIB DIBACA** | `Kategori IN (REBUTTAL, RISK_WARNING)` ATAU `Konteks_Vital != TECHNICAL` | Dikirim penuh |
| **Tier 2 – CUKUP TAHU** | `Kategori IN (NEW_IDEA, TECHNICAL_DETAIL)` DAN `Konteks_Vital = TECHNICAL` | Diringkas saja |
| **Tier 3 – TIDAK DIKIRIM** | `Status IN (REJECTED, STALE)` | Sudah terfilter otomatis oleh query `OPEN_CLAIMS` |

  `ROUND_SUMMARY` & `SCRIBE_ALERT` **tidak di-tier** — tetap dikirim penuh via §11 Prompt Packet (sinyal vital Shadow Scribe tidak boleh dipotong). `DECISION_LEDGER ACTIVE` juga dikirim penuh. Deteksi konflik klaim dengan `DECISION_LEDGER ACTIVE` tidak dilakukan di layer ini (tidak ada mekanisme real-time); ditutup oleh §10.3 Fail-Safe dan Lapis 3 GAS (§8.16). **Catatan waktu:** `ROUND_SUMMARY` putaran N baru tersedia *setelah* Claude (AI terakhir) selesai di putaran N — tidak bisa dipakai saat penyusunan Diff Context dalam putaran yang **sama**. Core Panel N+1 membacanya via §11 Prompt Packet, bukan via mekanisme Tier mid-round.

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
- **(Revisi V1.4.1)** Input `ROUND_SUMMARY` dibatasi **hanya** entri dengan `Topic_Closure = TRUE` (§7 Tab 8) — mencakup banyak topik dalam satu sesi tanpa membebani konteks, tanpa perlu tab baru.  
- Input: `ROUND_SUMMARY` (yang `Topic_Closure=TRUE`) + `DECISION_LEDGER` (ACTIVE) + `CLAIM_LEDGER` (klaim penting) + Friction Log.  
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
**(Revisi V1.4.1) Pengecualian:** Auto-Stop ini hanya berlaku untuk `Runtime_State = WAITING_DARMA` (menunggu keputusan vital). **`PAUSED_MANUAL` tidak pernah dihentikan otomatis** — jeda manual adalah keputusan sadar Darma, bukan situasi menunggu yang bisa hangus.

### 9.10 Disambiguasi Resume, Pause Manual, & `/lanjut` Whitelist (Baru V1.4.1, Revisi V1.4.2)  
Mengatur perbedaan jeda manual Darma vs menunggu keputusan sistem, serta perilaku perintah `/lanjut`.

- **Command Telegram operasional** (`/pause`, `/lanjut`, `/stop`, `/status`) **tidak pernah membuat baris `VITAL_TRIGGER`** — sifatnya operasional, bukan pemicu keputusan vital.  
- **`PAUSED_MANUAL` vs `WAITING_DARMA`:** `PAUSED_MANUAL` = Darma sengaja menjeda sistem (via `/pause`). `WAITING_DARMA` = sistem menunggu keputusan vital/error. Dua kondisi berbeda, tidak boleh disatukan.

**(V1.4.2) Prinsip Default-Deny Whitelist:**  
`/lanjut` **DITOLAK** untuk semua `Trigger_Source`, kecuali yang eksplisit ada di whitelist di bawah. Untuk enum di luar whitelist, sistem wajib tampilkan pilihan sah: `A` / `B` / `REVISI [teks]` / `STOP`.

**10 enum yang DITOLAK `/lanjut` (eksplisit):** `MAX_ROUND_LIMIT`, `STUCK_DETECTION`, `VITAL_GATE`, `OBSERVATION_CONFLICT`, `META_LLM_FILTER_ERROR`, `CORE_PANEL_BUG_HUNT`, `SHADOW_SCRIBE`, `ZERO_CONTEXT_REVIEW`, `SCRIBE_VALIDATION`, `UNKNOWN` — substansinya vital, tidak bisa di-skip begitu saja.

**Whitelist `/lanjut` yang sah:**

| Jalur | Kondisi | Aksi GAS |
|---|---|---|
| **Jalur 1 — Runtime Resume** | `Runtime_State = PAUSED_MANUAL` + tidak ada `VITAL_TRIGGER` terbuka | `Next_Action = RESUME_CORE_PANEL` |
| **Jalur 2a — API Error** | `Runtime_State = WAITING_DARMA` + `Trigger_Source = API_ERROR` | `Resolusi_Darma = LANJUT` → skip AI yang error |
| **Jalur 2b — Watchdog Deadlock** | `Runtime_State = WAITING_DARMA` + `Trigger_Source = WATCHDOG_DEADLOCK` | `Resolusi_Darma = LANJUT` → bypass/re-arm Scribe |

- **Unifikasi field target:** Semua `/lanjut` dari `WAITING_DARMA` (Jalur 2a/2b) selalu ditulis ke **`Resolusi_Darma = LANJUT`** (satu field tunggal). GAS menerjemahkan ke aksi teknis berbeda via *switch-case* berdasarkan `Trigger_Source`. Mencegah dualitas penulisan yang bisa memicu bug runtime.  
- **`RESUME_CORE_PANEL` terisolasi:** Nilai ini **hanya** untuk `PAUSED_MANUAL` — tidak pernah ditulis untuk `WAITING_DARMA`.  
- **`MANUAL_OVERRIDE` = N/A:** Bukan trigger yang menunggu resolusi baru, hanya catatan audit pasca-kejadian. `/lanjut` tidak relevan.  
- **State setelah `RESUME_CORE_PANEL`:** GAS kembalikan `Runtime_State = WAITING_AI`, `Next_Action = NONE`.  
- **Template Bahasa Awam (wajib):** Setiap `WAITING_DARMA` wajib menampilkan: (1) masalah dalam bahasa sehari-hari tanpa jargon, (2) pilihan `A`/`B`/`REVISI`/`STOP`, (3) catatan eksplisit bahwa `/lanjut` tidak berlaku jika di luar whitelist.  
- **Konvensi prosa intervensi:** Hindari kata "lanjut" di teks notifikasi/pesan sistem — gunakan "Ganti Topik", "Eksekusi", dll. Mencegah tabrakan visual dengan command `/lanjut`.

---

## 10. PROMPT MASTER META-LLM FILTER – FINAL

### 10.1 INPUT  
```
[RAW_INPUT]: Respons AI saat ini (teks mentah)
[OPEN_CLAIMS]: Array {Claim_ID, Isi_Singkat} yang statusnya DEBATABLE,
               di-filter Session_ID + Topic_Session aktif (Revisi V1.4.1, §7 Tab 2)
[DECISION_LEDGER_ACTIVE]: List keputusan dari DECISION_LEDGER dengan Decision_Status=ACTIVE
                          (Baru V1.4.2 — memberi Meta-LLM Filter bahan deteksi konflik di hulu)
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
3. **FAIL-SAFE (Konteks_Vital):** Jika klaim menyentuh biaya, keamanan, alur utama, atau **Anda RAGU**, WAJIB pilih `OPERATIONAL` atau `ARCH`. Pilih `TECHNICAL` **HANYA** untuk detail turunan murni. **(Baru V1.4.2) Fail-Safe konflik DECISION_LEDGER:** Jika klaim — walau tampak teknis — berpotensi bertentangan dengan salah satu entri `DECISION_LEDGER_ACTIVE` yang diterima di input, **wajib reklasifikasi `Konteks_Vital`** menjadi `ARCH` atau `OPERATIONAL` (mengikuti domain keputusan yang berpotensi terdampak), **bukan** `TECHNICAL`. Efek otomatis: klaim tidak lolos syarat Auto-Promote (`Konteks_Vital == TECHNICAL`) → otomatis masuk jalur `VITAL_GATE` (§8.2) tanpa komponen baru.  
4. **FAIL-SAFE (Semantic Duplicate):** Bandingkan `RAW_INPUT` dengan `OPEN_CLAIMS`. Jika ide/klaim sudah ada, JANGAN ekstrak sebagai `NEW_IDEA`. **Tapi jika RAGU apakah duplikat → TETAP ekstrak** (kehilangan klaim baru lebih mahal dari sedikit bengkak).  
5. **Isi_Singkat:** Maksimal 15 kata, langsung ke inti.  
6. **Status:** Wajib `DEBATABLE`.  
7. **Ref_ID:** Wajib diisi `Claim_ID` dari `OPEN_CLAIMS` jika `REBUTTAL`. Jika `NEW_IDEA`, isi `null`.  
8. **DILARANG:** Salam, narasi, basa-basi, kesimpulan, teks di luar JSON.

### 10.4 Setelah GAS Terima Output  
1. Generate `Claim_ID` = `CLM-{session_id}-{counter}` (increment GAS).  
2. Isi `Topic_Session` = `SESSION_STATE.Current_Topic_Session` **(Baru V1.4.1)**.  
3. Isi `Round_Diajukan` = `Round_In_Topic`.  
4. Isi `Round_Terakhir_Sanggah` = `Round_In_Topic` (untuk klaim baru) ATAU, jika `REBUTTAL`, periksa dulu: **jika `Ref_ID` berada dalam `Topic_Session` yang sama** → update `Round_Terakhir_Sanggah` pada klaim `Ref_ID`; **jika `Ref_ID` lintas `Topic_Session`** → **jangan** update klaim lama, hanya catat referensi historis (§7 Tab 2, V1.4.2).  
5. Insert row ke `CLAIM_LEDGER`.

---

## 11. AI PROMPT PACKET CONTRACT  

Setiap AI Core Panel menerima paket konteks terstruktur (bukan hanya `OPEN_CLAIMS`):

```text
[Instruksi Sistem / Mode Diskusi]
[Context_Summary dari Local Feeder]
[DECISION_LEDGER – hanya yang ACTIVE] ← satu-satunya komponen data lintas topik yang dikirim utuh (V1.4.2, §9.3); batasan natural arsitektur yang sudah ada
[ROUND_SUMMARY – putaran (Round_In_Topic) sebelumnya dalam topik aktif] ← Baru V1.4, wajib untuk Lapis 2 (§8.16)
[OBSERVATION_LEDGER – status OPEN/LINKED yang relevan] ← Baru V1.4
[OPEN_CLAIMS – klaim dengan Status DEBATABLE, di-scope per Topic_Session aktif ← Revisi V1.4.1, §10.1]
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
- **(Baru V1.4.1)** Harga per token tiap model disimpan di Script Properties per model (dipakai GAS untuk hitung `Estimated_Cost_Session`, §14). Ambang kedaluwarsa harga `Cost_Staleness_Days` (default **30 hari**) juga disimpan di Script Properties — configurable, bukan hardcode.

---

## 13. CHANGE GOVERNANCE & IMPACT LOG  

> **Status Sesi Patchset V1.4:** Sesi 1, 2, 3, 4, 5 — seluruhnya **CLOSED**.  
> **Status Sesi Patchset V1.4.1:** Sesi 1 (Trigger_Source), Sesi 2 (Next_Action Lifecycle), Sesi 3 (Topic_Session Lifecycle), Sesi 4 (Config Model & Biaya), Resolusi Gap Akhir (9 gap) — seluruhnya **CLOSED**.

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

### Impact Log 10 (V1.4.1): `Trigger_Source` Wajib & Enum Lengkap  
| Baris | Isi |
|-------|-----|
| **KONTRAK LAMA** | `VITAL_TRIGGER` hanya wajib `Trigger_Source` untuk jalur fatal tertentu (4 nilai); Vital Gate manual punya catatan terbuka tanpa nilai pasti; tidak ada fallback untuk sumber tak dikenal. |
| **KONTRAK BARU** | Semua baris `VITAL_TRIGGER` wajib `Trigger_Source` (audit lengkap, Opsi B), enum **11 nilai** saat V1.4.1 termasuk `VITAL_GATE`, `API_ERROR`, `MAX_ROUND_LIMIT`, `WATCHDOG_DEADLOCK`, `MANUAL_OVERRIDE`, `STUCK_DETECTION`, `UNKNOWN` (fallback, counter ≥3 → auto `VITAL_TRIGGER`). `SYSTEM_FRICTION` dikunci hanya untuk kolom `Tipe`. *(Kemudian direvisi ke **13 nilai** di V1.4.2 — lihat Impact Log 18.)* |
| **BLAST RADIUS** | §7 Tab 4, §8.2, §8.3, §8.12, implementasi GAS classifier. |

### Impact Log 11 (V1.4.1): `Next_Action` Lifecycle & Strict Gate  
| Baris | Isi |
|-------|-----|
| **KONTRAK LAMA** | `Next_Action` dipakai (§8.10) tapi belum punya kolom resmi, belum ada enum lengkap, belum ada aturan reset/guard, dan Strict Gate `SCRIBE_DONE` belum ditegaskan eksplisit. |
| **KONTRAK BARU** | Kolom `Next_Action` resmi di `SESSION_STATE` (6 enum: `NONE`, `BYPASS_LANJUT`, `RETRY_SAME_AI`, `WAIT_DARMA`, `STOP`, `RESUME_CORE_PANEL`), sifat transien (reset ke `NONE` setelah write confirmed), guard utama tetap `Runtime_State`, urutan eksekusi GAS dikunci, idempotensi `WAIT_DARMA` diatur, Strict Gate putaran N+1 wajib tunggu `SCRIBE_DONE`. |
| **BLAST RADIUS** | §7 Tab 5, §8.10, §8.17 (baru), urutan eksekusi GAS. |

### Impact Log 12 (V1.4.1): `Topic_Session` & `STALE` di `CLAIM_LEDGER`  
| Baris | Isi |
|-------|-----|
| **KONTRAK LAMA** | `CLAIM_LEDGER` tidak punya kolom `Topic_Session` — klaim topik lama berisiko campur dengan topik baru; `Status` tidak punya nilai untuk klaim "mati" saat topik ditutup (*ghost audit*). |
| **KONTRAK BARU** | Tambah kolom `Topic_Session`; query `OPEN_CLAIMS` di-filter `Session_ID + Topic_Session`; cross-topic `Ref_ID` tetap diizinkan; `Status` +enum `STALE` (GAS mass-update `DEBATABLE`→`STALE` saat topik tutup). |
| **BLAST RADIUS** | §7 Tab 2, §8.1 (Auto-Promote), §8.3 (Stuck Detection), §10.1 (Meta-LLM Filter), §11 (Prompt Packet), query `OPEN_CLAIMS`, cross-topic `Ref_ID`. |

### Impact Log 13 (V1.4.1): `Topic_Closure` di `ROUND_SUMMARY`  
| Baris | Isi |
|-------|-----|
| **KONTRAK LAMA** | `ROUND_SUMMARY` tidak punya penanda putaran mana yang jadi penutup resmi suatu topik; `FINAL_CUMULATIVE_SUMMARY` berisiko menelan seluruh `ROUND_SUMMARY` tanpa pembatasan per topik. |
| **KONTRAK BARU** | Tambah kolom `Topic_Closure` (`TRUE`/`FALSE`), **diisi GAS saja** (patuh Anti-Otonomi §8.13) saat Darma eksplisit tutup topik atau `Max_Rounds_Per_Topic` tercapai + konfirmasi ganti topik (**bukan** `Discussion_State=CLOSED` global). `FINAL_CUMULATIVE_SUMMARY` (§9.6) kini hanya pakai entri `Topic_Closure=TRUE`. |
| **BLAST RADIUS** | §7 Tab 8, §8.10, §9.6, §9.10, Anti-Otonomi (§8.13). |

### Impact Log 14 (V1.4.1): Lifecycle `Topic_Session` — Hybrid  
| Baris | Isi |
|-------|-----|
| **KONTRAK LAMA** | Siklus hidup `Topic_Session` (kapan dibuat, kapan diganti) tidak didefinisikan eksplisit. |
| **KONTRAK BARU** | Hybrid: GAS membuat `Topic_Session` pertama otomatis dari `Context_Summary`; Darma bisa ganti/buat topik baru kapan saja; tiap ganti topik, `Round_In_Topic` reset ke 0 (aturan lama, kini eksplisit terhubung ke lifecycle ini). |
| **BLAST RADIUS** | §7 Tab 5, §7 Tab 8, §8.5, §9.6, §9.10, §19. |

### Impact Log 15 (V1.4.1): Penegasan Model ID & `deepseek-v4-flash`  
| Baris | Isi |
|-------|-----|
| **KONTRAK LAMA** | §8.9 V1.4 menyebut `deepseek-v4-flash` sebagai "default" tanpa penegasan eksplisit bahwa ini bukan kontrak permanen — berisiko dibaca sebagai hardcode meski Model ID sudah diarahkan ke Script Properties. |
| **KONTRAK BARU** | `deepseek-v4-flash` ditegaskan hanya kandidat awal/rekomendasi, wajib diverifikasi ulang dari provider saat dry-run sebelum diisi ke `DeepSeek_Model_ID`. |
| **BLAST RADIUS** | §8.9, §12. |

### Impact Log 16 (V1.4.1): Struktur Biaya 3 Kolom & *Oldest Date Wins*  
| Baris | Isi |
|-------|-----|
| **KONTRAK LAMA** | Hanya ada `Estimated_Cost_Session` (angka) + `Token_Usage_Cumulative`, tanpa mekanisme deteksi harga kedaluwarsa atau status validitas biaya. |
| **KONTRAK BARU** | Tambah `Cost_Status` (`OK/STALE/UNKNOWN`) & `Cost_Price_Updated` (tanggal verifikasi harga tertua antar-model, *Oldest Date Wins*); ambang `Cost_Staleness_Days` (default 30 hari) di Script Properties; `Estimated_Cost_Session` bisa bernilai `UNKNOWN` (beda dari 0/gratis). `Token_Usage_Cumulative` **tidak berubah**, tetap kolom terpisah. |
| **BLAST RADIUS** | §7 Tab 5, §12, §14, kode GAS. |

### Impact Log 17 (V1.4.1): `PAUSED_MANUAL` & Disambiguasi `/lanjut`  
| Baris | Isi |
|-------|-----|
| **KONTRAK LAMA** | `Runtime_State` tidak punya state untuk jeda manual Darma — `/pause` berisiko ambigu dengan `WAITING_DARMA`; perintah `/lanjut` punya 2 makna berbeda (resume mesin vs skip AI error) tanpa aturan pembeda; Auto-Stop 24 jam (§9.9) berisiko menghanguskan jeda manual. |
| **KONTRAK BARU** | `Runtime_State` +`PAUSED_MANUAL`; `/lanjut` di-disambiguasi berdasarkan `Runtime_State` aktif (`PAUSED_MANUAL`→`RESUME_CORE_PANEL`, `WAITING_DARMA`→`Resolusi_Darma=LANJUT`, state lain→log `UNKNOWN`); state setelah resume dikunci (`WAITING_AI`+`Next_Action=NONE`); Auto-Stop §9.9 dikecualikan untuk `PAUSED_MANUAL`. |
| **BLAST RADIUS** | §7 Tab 5, §9.9, §9.10 (baru), implementasi GAS `/lanjut` handler. |

### Impact Log 18 (V1.4.2): Enum `Trigger_Source` 11→13 Nilai  
| Baris | Isi |
|-------|-----|
| **KONTRAK LAMA** | Enum `Trigger_Source` 11 nilai — §8.8 Meta-LLM Filter error dan `OBSERVATION_LEDGER` conflict belum punya nilai yang tepat; keduanya rawan dipaksa ke `UNKNOWN` meski sumber sudah diketahui, melemahkan audit. |
| **KONTRAK BARU** | Tambah `META_LLM_FILTER_ERROR` (`Tipe=SYSTEM_FRICTION`, bisa bypass via `LANJUT`/`REVISI`/`STOP`) dan `OBSERVATION_CONFLICT` (`Tipe=WAITING_DARMA`, wajib berhenti total, resolusi `A`/`B`/`REVISI`/`STOP`). Total enum 13 nilai. |
| **BLAST RADIUS** | §7 Tab 4, §8.8, §8.12. Semua penyebutan "11 nilai" di SOT sudah diupdate ke "13 nilai". |

### Impact Log 19 (V1.4.2): Cross-Topic Guard & Fail-Safe Meta-LLM Filter  
| Baris | Isi |
|-------|-----|
| **KONTRAK LAMA** | Tidak ada aturan eksplisit tentang imutabilitas klaim lama saat ada rebuttal lintas topik; Auto-Promote bisa bertabrakan dengan `DECISION_LEDGER ACTIVE` (*race condition*); Meta-LLM Filter tidak punya input `DECISION_LEDGER` untuk deteksi konflik di hulu. |
| **KONTRAK BARU** | (1) Cross-topic `Ref_ID` = referensi historis: dilarang mengubah `Status`/`Round_Terakhir_Sanggah` klaim lama otomatis (§7 Tab 2, §10.4). (2) Hard-Guard Auto-Promote: jika `Ref_ID` → `DECISION_LEDGER ACTIVE`, Auto-Promote dimatikan → wajib Vital Gate (§8.1). (3) Auto-REJECTED cross-topic diblokir (§8.4). (4) Input Meta-LLM Filter +`DECISION_LEDGER_ACTIVE` (§10.1). (5) Fail-Safe reklasifikasi `Konteks_Vital`: jika klaim berpotensi konflik dengan `DECISION_LEDGER ACTIVE` → reklasifikasi ke `ARCH`/`OPERATIONAL` → otomatis masuk `VITAL_GATE` (§10.3). |
| **BLAST RADIUS** | §7 Tab 2, §8.1, §8.4, §10.1, §10.3, §10.4. |

### Impact Log 20 (V1.4.2): `/lanjut` Default-Deny Whitelist  
| Baris | Isi |
|-------|-----|
| **KONTRAK LAMA** | `/lanjut` dari `WAITING_DARMA` hanya terkunci untuk `API_ERROR`; sumber lain belum terdefinisi (risiko salah aksi GAS). |
| **KONTRAK BARU** | Default-Deny Whitelist: `/lanjut` **DITOLAK** untuk semua enum kecuali 2 jalur sah (Jalur 2a: `API_ERROR`→skip AI; Jalur 2b: `WATCHDOG_DEADLOCK`→bypass Scribe). `WATCHDOG_DEADLOCK` dikunci eksplisit (`Tipe=SYSTEM_FRICTION`, `Runtime_State=WAITING_DARMA`). Semua `/lanjut` dari `WAITING_DARMA` ditulis ke `Resolusi_Darma=LANJUT` (unifikasi field); GAS terjemahkan via *switch-case*. Template Bahasa Awam wajib di setiap `WAITING_DARMA`. Hindari kata "lanjut" di prosa intervensi. |
| **BLAST RADIUS** | §9.10, §7 Tab 4, §8.10. |

> **Status Sesi Patchset V1.4:** Sesi 1–5 — seluruhnya **CLOSED**.  
> **Status Sesi Patchset V1.4.1:** Sesi 1–4 & Resolusi Gap Akhir — seluruhnya **CLOSED**.  
> **Status Sesi Patchset V1.4.2:** Pra-Sesi, Sesi 1 (Context Priority), Sesi 2 (PMD-1 Trigger_Source), Sesi 3 (PMD-2 Cross-Topic), Sesi 4 (PMD-3 /lanjut Whitelist) — seluruhnya **CLOSED**.

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
| **Monitoring Biaya** | **(Baru V1.4, Revisi V1.4.1)** 4 kolom di `SESSION_STATE`: `Estimated_Cost_Session` (numerik atau `UNKNOWN`), `Cost_Status` (`OK/STALE/UNKNOWN`), `Cost_Price_Updated` (tanggal harga **tertua** antar-model, *Oldest Date Wins*), `Token_Usage_Cumulative` (tetap terpisah). Dihitung deterministik oleh GAS dari token usage × harga/token di Script Properties; `Cost_Status` dibandingkan terhadap `Cost_Staleness_Days` (default 30 hari, §12). **Bukan** estimasi naratif LLM. |
| **Auto-Stop 24 Jam** | Jika Darma tidak respons saat `Runtime_State = WAITING_DARMA` → sesi dihentikan otomatis. **(Revisi V1.4.1)** Tidak berlaku untuk `PAUSED_MANUAL` (§9.9, §9.10). |
| **Kendali Manual** | Telegram: `/pause`, `/lanjut`, `/stop`, `/status` — **(V1.4.1) tidak pernah membuat baris `VITAL_TRIGGER`** (operasional, bukan vital). Untuk keputusan vital: `A`, `B`, `REVISI [teks]`, `STOP`. Untuk error/friction: `LANJUT`, `REVISI [teks]`, `STOP`. `/lanjut` di-disambiguasi berdasarkan `Runtime_State` aktif — lihat §9.10. |
| **Komunikasi** | Semua pesan sistem ke Darma wajib **bahasa Indonesia sederhana**, bukan jargon teknis. Jika menyangkut keputusan vital, sistem wajib menjelaskan dampak praktis, risiko, saran default, dan pilihan balasan sederhana. |

---

## 15. ROLE & PERMISSION  

| Aktor | Hak Akses | Batasan |
|-------|-----------|---------|
| **Darma** | Full control. Satu-satunya yang bisa approve `VITAL_TRIGGER`, edit Sheet manual, stop sesi. Juga **Lapis 1** dalam 5-Layer Validation Engine (§8.16) — veto mutlak, bisa abort/interupsi kapan saja. | - |
| **GAS** | System executor. Satu-satunya penulis `SESSION_STATE`, `DECISION_LEDGER`, `VITAL_TRIGGER`. Juga **Lapis 3** (Tier-1 string-match check, §8.16). | Hanya menulis via logika yang sudah disepakati. |
| **Core Panel** (ChatGPT, Gemini, Claude) | Read `OPEN_CLAIMS`, `DECISION_LEDGER` (ACTIVE), `OBSERVATION_LEDGER` (OPEN/LINKED) — dari Prompt Packet (§11). Juga **Lapis 2** (membaca `ROUND_SUMMARY` putaran N sebagai N+1). | Tidak punya akses langsung ke Sheet. Tidak bisa menulis/mengubah data (Anti-Otonomi, §8.13). |
| **Shadow Scribe** (DeepSeek) | **(Baru V1.4)** Read seluruh output Core Panel per round. Tulis `ROUND_SUMMARY` (§7 Tab 8) & `SCRIBE_ALERT` saja — **kecuali kolom `Topic_Closure`, yang hanya diisi GAS** (Revisi V1.4.1, §7 Tab 8, §8.13). | **Tidak ikut debat inti.** Tidak bisa mengunci keputusan. Tidak boleh menulis `DECISION_LEDGER` atau `CLAIM_LEDGER` sebagai rebuttal biasa. |
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

### Friction Log Patchset V1.4.1 (Sesi 1–4 & Resolusi Gap Akhir)

| FRICTION | CATATAN | STATUS |
|---|---|---|
| Naming collision `SYSTEM_FRICTION` | Diusulkan sebagai `Trigger_Source`, padahal sudah dipakai di `Tipe`. | ✅ Dikunci hanya untuk `Tipe` |
| Kemunculan mendadak `UNKNOWN ≥ 3` | Diresmikan sebagai patch tambahan setelah diskusi panel. | ✅ Ditambahkan |
| Konflik `Next_Action` vs `Runtime_State` | Diselesaikan dengan menjadikan `Runtime_State` sebagai guard tunggal. | ✅ Diperbaiki |
| `Topic_Session` hilang dari `CLAIM_LEDGER` | Celah arsitektur yang baru ditemukan di Sesi 3. | ✅ Ditambahkan |
| Ghost audit tanpa `STALE` | Diputuskan menambah `STALE` untuk kebersihan audit. | ✅ Ditambahkan |
| Ambiguitas eksekutor `Topic_Closure` | Dikoreksi dari DeepSeek menjadi GAS (patuh Anti-Otonomi §8.13). | ✅ Dikoreksi |
| Format sel biaya | Diselesaikan dengan normalisasi 3 kolom terpisah (`Cost_Status`, `Cost_Price_Updated`, plus `Estimated_Cost_Session`). | ✅ Diperbaiki |
| Logika harga multi-model | Diselesaikan dengan *Oldest Date Wins*. | ✅ Dikunci |
| Premature closure | AI eksternal sempat menyatakan diskusi selesai sebelum dry-run dan `RESUME_CORE_PANEL` dikunci — terjadi di luar runtime Smart Moderator, saat *off-system SOT discussion*, bukan bagian Core Panel/Shadow Scribe produk. | ✅ Diluruskan |
| `RESUME_CORE_PANEL` vs `LANJUT` tertukar | Dipertegas dengan disambiguasi berdasarkan `Runtime_State` aktif (§9.10). | ✅ Diperbaiki |
| `Topic_Closure` ambigu level sesi vs topik | Dikunci: mengandalkan konfirmasi eksplisit Darma (atau `Max_Rounds_Per_Topic`+konfirmasi), bukan `Discussion_State=CLOSED` global. | ✅ Direvisi |
| Blast radius Impact Log #12 kurang lengkap | Ditambah dampak ke Auto-Promote, Stuck Detection, Meta-LLM Filter, dan Prompt Packet. | ✅ Dilengkapi |
| Open note §7 Tab 4 belum ditutup | Diselesaikan: Vital Gate manual pakai `Trigger_Source = VITAL_GATE`. | ✅ Ditutup |
| `PAUSED_MANUAL` tidak masuk enum | Ditambahkan untuk membedakan jeda manual dari `WAITING_DARMA`. | ✅ Ditambahkan |
| Disambiguasi `/lanjut` belum ada | Dibuat aturan percabangan berdasarkan `Runtime_State` aktif (§9.10). | ✅ Dibuat |
| State setelah resume tidak jelas | Dikunci: `Runtime_State` kembali ke `WAITING_AI`, `Next_Action=NONE`. | ✅ Dikunci |
| Auto-stop 24 jam untuk jeda manual | Dipastikan tidak berlaku untuk `PAUSED_MANUAL` (§9.9). | ✅ Dipastikan |

---

### Friction Log Patchset V1.4.2 (Pra-Sesi & Sesi 1–4)

| FRICTION | CATATAN | STATUS |
|---|---|---|
| Anomali rotasi pra-sesi | Claude mendahului Gemini di pra-sesi diskusi V1.4.2 — tidak sesuai urutan Core Panel (ChatGPT→Gemini→Claude). Dicatat sebagai anomali prosedural; tidak mempengaruhi konsensus karena semua keputusan akhirnya ditandatangani panel lengkap. | ✅ Dicatat |
| `Divergence_Flag` diusulkan sebagai kolom baru | Ditolak sebelum dry-run — belum ada bukti empiris kebutuhan; menambah kolom berarti mengubah kontrak data, Meta-LLM Filter, dan Prompt Packet tanpa urgensi. Masuk Architecture-Ready Later. | ✅ Ditolak |
| Deteksi substring `[FATAL-` di teks bebas diusulkan | Ditolak (Claude) karena rawan false-positive. Sinyal fatal hanya sah dari sumber terstruktur (§8.11). | ✅ Ditolak |
| "Konflik DECISION_LEDGER ACTIVE" masuk Tier 1 | Diusulkan, lalu dihapus — tidak ada mekanisme deteksi real-time di Layer 2 Diff Context. Ditutup oleh §10.3 Fail-Safe dan Lapis 3 GAS (§8.16). | ✅ Dihapus dari Tier |
| Ambiguitas `SCRIBE_ERROR` vs `WATCHDOG_DEADLOCK` | Sempat rawan dicampur karena namanya mirip — dikunci: `SCRIBE_ERROR` = `Runtime_State` (auto-bypass GAS), `WATCHDOG_DEADLOCK` = `Trigger_Source` (butuh `/lanjut` Darma). | ✅ Dipisahkan |
| Cross-topic `Ref_ID` tanpa batas — risiko auto-mutasi klaim lama | Dikunci dengan imutabilitas cross-topic (§7 Tab 2, §8.4, §10.4) dan Hard-Guard Auto-Promote (§8.1). | ✅ Ditutup |
| Meta-LLM Filter tidak punya akses DECISION_LEDGER | Celah deteksi konflik di hulu — ditutup dengan menambah input ke-3 `DECISION_LEDGER_ACTIVE` (§10.1) dan Fail-Safe reklasifikasi `Konteks_Vital` (§10.3). | ✅ Ditutup |
| `META_LLM_FILTER_ERROR` dan `OBSERVATION_CONFLICT` tanpa `Trigger_Source` | Keduanya bisa memicu `VITAL_TRIGGER` tapi enum belum ada — ditutup dengan menambah 2 enum baru (total 13 nilai, §7 Tab 4). | ✅ Ditutup |
| `/lanjut` terlalu broad — bisa salah aksi untuk sumber non-API_ERROR | Ditutup dengan Default-Deny Whitelist (hanya `API_ERROR` + `WATCHDOG_DEADLOCK` yang sah, §9.10). | ✅ Ditutup |
| Verbatim Gemini tidak tersedia saat diskusi panel V1.4.2 | Claude hanya menerima parafrase ChatGPT — Darma memiliki catatan asli. Bukan blocker, tercatat sebagai catatan audit (§20). | ✅ Dicatat |

---

## 19. LANGKAH SELANJUTNYA (DRY-RUN) — REVISED CHECKLIST V1.4.2 (34 ITEM)

> §19 V1.3 (9 langkah, berbasis rotasi 5-AI dan struktur 5 tab) **diganti total** — sebagian besar item lama sudah tidak relevan dengan arsitektur Core Panel + Shadow Scribe + 8 tab. Item 22–34 ditambahkan di Patchset V1.4.1 untuk menguji `Trigger_Source`, `Next_Action`, `Topic_Session` lifecycle, struktur biaya, dan disambiguasi pause/resume. Patchset V1.4.2 tidak menambah item dry-run baru — perubahannya tercakup oleh item yang sudah ada (item 13 untuk §10.3, item 17 untuk `Trigger_Source`, item 18 untuk bias agreement, item 22–23 untuk enum baru).

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

**Item 22–32 (Validasi Patch V1.4.1):**
22. **Uji `Trigger_Source` Wajib:** Buat baris `VITAL_TRIGGER` baru tanpa `Trigger_Source` → sistem wajib menolak/menandai tidak valid.
23. **Uji Fallback `UNKNOWN`:** Picu klasifikasi sumber gagal 3 kali dalam satu sesi → `Trigger_Source = UNKNOWN` otomatis memicu `VITAL_TRIGGER` baru.
24. **Uji Manual Override saat `WAITING_SCRIBE`:** Darma kirim resolusi saat Scribe masih jalan → standar tercatat `OVERTAKEN_BY_DARMA` di `ROUND_SUMMARY.Open_Friction`, **tidak** otomatis jadi `VITAL_TRIGGER` kecuali ada konflik state nyata.
25. **Uji Late Scribe Tidak Override:** Hasil Scribe datang setelah Darma sudah putuskan → hasil tercatat *late/ignored result*, keputusan Darma tidak berubah.
26. **Uji Strict Gate:** Putaran N+1 **tidak boleh** mulai sebelum `Runtime_State = SCRIBE_DONE`.
27. **Uji Bypass Resmi:** `SCRIBE_ERROR` + `Next_Action = BYPASS_LANJUT` → putaran N+1 boleh lanjut tanpa `SCRIBE_DONE`.
28. **Uji Reset `Next_Action`:** Setelah eksekusi sukses & penulisan Sheets terkonfirmasi → `Next_Action` kembali ke `NONE`.
29. **Uji Filter `Topic_Session` pada `OPEN_CLAIMS`:** Klaim dari topik lama tidak ikut muncul di `OPEN_CLAIMS` topik aktif.
30. **Uji Batch Update `STALE`:** Saat topik ditutup (`Topic_Closure=TRUE`), semua klaim `DEBATABLE` di topik tsb otomatis jadi `STALE`.
31. **Uji `Topic_Closure` Hanya Diisi GAS:** Pastikan DeepSeek tidak pernah menulis langsung ke kolom ini (Anti-Otonomi §8.13).
32. **Uji Status Biaya:** Simulasikan harga model lewat dari `Cost_Staleness_Days` → `Cost_Status` otomatis berubah jadi `STALE`; harga belum dikonfigurasi → `Estimated_Cost_Session = UNKNOWN`.

**Item 33–34 (Validasi Pause Manual & Disambiguasi `/lanjut`):**
33. **Uji Siklus Pause Manual:** `/pause` → `Runtime_State = PAUSED_MANUAL` → `/lanjut` (tanpa `VITAL_TRIGGER` terbuka) → `Next_Action = RESUME_CORE_PANEL` → `Runtime_State = WAITING_AI`.
34. **Uji Disambiguasi `/lanjut` dari `WAITING_DARMA` (scope: `API_ERROR`):** Kirim `/lanjut` saat `Runtime_State = WAITING_DARMA` akibat `Trigger_Source = API_ERROR` → sistem **tidak** memicu `RESUME_CORE_PANEL`, melainkan `Resolusi_Darma = LANJUT` (skip AI). *Catatan: perilaku LANJUT dari `Trigger_Source` lain belum terkunci di V1.4.1 — lihat §9.10.*

**Setelah seluruh 34 item di atas sukses** (di luar penomoran checklist): kunci sebagai V1.4.1 production.

---

## 20. RESOLVED MICRO-DECISIONS & DEFERRED ITEMS

> Section ini mencatat keputusan yang sebelumnya terbuka (§20 V1.4.1) dan kini telah dikunci, serta item yang sengaja ditunda ke dry-run. Bukan bagian arsitektur aktif.

### PMD-1: `Trigger_Source` untuk Meta-LLM Filter Error & Observation Conflict — ✅ RESOLVED (V1.4.2)
**Keputusan:** Opsi A — tambah 2 enum baru: `META_LLM_FILTER_ERROR` + `OBSERVATION_CONFLICT`. Total `Trigger_Source` menjadi 13 nilai. Lihat Impact Log 18 & §7 Tab 4.

### PMD-2: Guard Cross-Topic `Ref_ID` — ✅ RESOLVED (V1.4.2)
**Keputusan:** Cross-topic `Ref_ID` = referensi historis murni. Dilarang mengubah `Status`/`Round_Terakhir_Sanggah` klaim lama otomatis. Hard-Guard Auto-Promote terhadap `DECISION_LEDGER ACTIVE`. Fail-Safe reklasifikasi `Konteks_Vital`. Lihat Impact Log 19 & §8.1, §8.4, §10.1, §10.3, §10.4.

### PMD-3: Makna `/lanjut` dari `WAITING_DARMA` selain `API_ERROR` — ✅ RESOLVED (V1.4.2)
**Keputusan:** Default-Deny Whitelist — hanya `API_ERROR` dan `WATCHDOG_DEADLOCK` yang menerima `/lanjut`. Semua enum lain ditolak. Lihat Impact Log 20 & §9.10.

---

### Deferred Items (Sengaja Ditunda ke Dry-Run)

| ID | Item | Status |
|----|------|--------|
| D1 | Wording `A`/`B` untuk `OBSERVATION_CONFLICT` (bahasa awam) | Deferred — dikerjakan saat dry-run |
| D2 | Wording `A`/`B` untuk Vital Gate baru (klaim teknis yang direklasifikasi karena konflik decision) | Deferred — dikerjakan saat dry-run |

### Catatan Audit

> **Verbatim Gemini:** Selama diskusi panel V1.4.2, Claude tidak pernah melihat teks asli Gemini — hanya via parafrase ChatGPT. Darma dikonfirmasi memiliki catatan asli Gemini. Ini bukan blocker; dicatat sebagai catatan audit saja.

---

**=== STATUS AKHIR: CLOSED ===**

Panel AI telah mencapai **kesepakatan penuh** terhadap seluruh poin di atas, termasuk Patchset V1.4 (Sesi 1–5), Patchset V1.4.1 (Sesi 1–4 & Resolusi Gap Akhir), dan Patchset V1.4.2 (Pra-Sesi & Sesi 1–4).  
Semua 5 Keputusan Vital dari SOT awal Darma telah tercover lengkap, sekarang diperkuat dengan 5-Layer Validation Engine (§8.16).  
**Dokumen ini adalah Source of Truth (SoT) untuk Smart Moderator V1.4.2 Final — closed secara desain, siap implementasi dan Dry-Run §19 (34 item).**  
Setiap perubahan di masa depan wajib melalui Change Governance (Impact Log 3 Baris, §13), dan setiap kunci versi major wajib lolos Lapis 5 Diff Checklist (`lockVersion()`).