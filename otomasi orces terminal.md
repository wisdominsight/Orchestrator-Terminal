# ** OTOMATISASI ORCHESTRATOR TERMINAL v1.0**
**Status:** **CLOSED** – Siap Implementasi  
**Panel AI yang Sepakat:** Kimi, Qwen, Claude, DeepSeek, Gemini  
**Tanggal:** 25 Mei 2026

---

## **I. INTI MASALAH & TUJUAN**

**Masalah Utama:**
Diskusi multi-AI di Orchestrator Terminal v0.1 sangat efektif menghasilkan keputusan berkualitas tinggi, tetapi prosesnya **100% manual** (Darma harus menyalin-tempel respon antar AI). Ini menghabiskan waktu dan sangat membosankan.

**Tujuan:**
Mengotomatisasi seluruh alur diskusi (mengirim prompt, menerima respon, merotasi konteks, dan memberikan kendali ke Darma) tanpa menghilangkan esensi cross-validation antar AI dan pengawasan manusia.

**Prinsip Darma (Tidak Dapat Ditawar):**
- Build paling kompleks selama itu opsi terbaik
- Value creation maksimal, tidak menunggu fase lain jika ada ide bagus
- Darma adalah AI Orchestrator, bukan programmer murni
- **Semua komunikasi sistem ke Darma wajib dalam bahasa Indonesia yang mudah dipahami orang awam**, bukan bahasa teknis pemrograman

---

## **II. KEPUTUSAN ARSITEKTUR (MUTLAK & DISEPAKATI)**

### **2.1 Framework Berat Ditolak**
CrewAI, LangGraph, Dify, Flowise → **ditolak**. Framework tersebut terlalu berat (*overkill*) untuk kebutuhan diskusi linear dengan logika rotasi spesifik. Mereka dirancang untuk eksekusi tugas, bukan debat terstruktur dengan validasi silang.

### **2.2 Stack Pemenang: GAS + Google Sheet + Telegram Bot**

| Komponen | Fungsi | Alasan Pemilihan |
|:---|:---|:---|
| **Google Apps Script (GAS)** | Mesin penggerak utama | Gratis, tanpa server, Darma sudah terbiasa |
| **Google Sheet** | Penyimpanan sesi diskusi | Transparan, bisa dibaca dan diedit langsung oleh Darma |
| **Telegram Bot** | Notifikasi dan kendali jarak jauh | Ringan, gratis, real-time, sudah dikenal Darma |

**Keunggulan Mutlak:**
- **Zero Cost & Zero Server:** Tidak ada biaya hosting atau pemeliharaan server
- **Familiaritas:** Darma sudah menggunakan GAS dan Sheets untuk proyek lain
- **Transparansi Penuh:** Semua data diskusi tersimpan rapi di Sheet, bisa dibaca kapan saja tanpa perlu membuka kode
- **Kendali Penuh:** Semua perintah penting bisa dilakukan dari Telegram

### **2.3 Metode Eksekusi: Sequential (Satu per Satu)**

**Keputusan:** AI dipanggil **satu per satu** secara berurutan, bukan bersamaan.

**Alasan Sederhana:** Esensi diskusi adalah saling menanggapi. AI kedua harus membaca pendapat AI pertama sebelum memberi pendapatnya. Jika semua dipanggil bersamaan, tidak akan terjadi debat dan validasi silang.

---

## **III. DAFTAR AI YANG DIGUNAKAN (FULL ACTIVE v1)**

| No | AI | Status di v1 | Keterangan |
|:---|:---|:---|:---|
| 1 | **Kimi** | ✅ Aktif | API tersedia, sering dipakai untuk coding |
| 2 | **Qwen** | ✅ Aktif | API tersedia, sering dipakai untuk coding |
| 3 | **Gemini** | ✅ Aktif | API tersedia, sering dipakai untuk project GAS |
| 4 | **Claude** | ✅ Aktif | API tersedia, sering dipakai untuk diskusi produk |
| 5 | **DeepSeek** | ✅ Aktif | API tersedia, andalan untuk menulis dokumen |

**AI yang Dikeluarkan:** Meta AI dan Z.ai tidak memiliki API publik yang memadai sehingga tidak bisa diotomatisasi.

---

## **IV. ALUR KERJA SISTEM (WORKFLOW)**

### **4.1 Cara Memulai Sesi Baru**
Darma menulis pertanyaan atau arahan baru di:
- **Sheet khusus** yang sudah disediakan, atau
- **Kirim pesan ke Telegram Bot** dengan format yang sudah ditentukan

Sistem akan otomatis membuat "sesi diskusi baru" dan mulai memanggil AI pertama.

### **4.2 Urutan Pemanggilan AI (Fixed Rotation v1)**
Urutan tetap: **Kimi → Qwen → Gemini → Claude → DeepSeek**

Untuk setiap AI, sistem akan:
1. Menyusun "paket pertanyaan" yang berisi:
   - Aturan main diskusi (dari mode yang dipilih Darma)
   - Pertanyaan atau arahan terbaru dari Darma
   - Ringkasan hasil diskusi sebelumnya (agar obrolan nyambung)
   - Tanggapan AI lain di putaran yang sama (agar bisa saling mengomentari)
   - Format jawaban wajib yang harus diikuti AI
2. Mengirim paket tersebut ke API AI yang bersangkutan
3. Menerima dan menyimpan jawabannya di Sheet
4. Istirahat sejenak (jeda beberapa detik) sebelum memanggil AI berikutnya

### **4.3 Fitur "Rem Darurat" (Approval Gate)**
Sistem akan **berhenti sejenak dan minta persetujuan Darma** jika:
- AI menyebut kata-kata sensitif: "budget", "deploy", "hapus", "biaya", "beli", "langganan"
- AI dengan sengaja menulis `[TANYA DARMA]` atau `[MINTA PERSETUJUAN]` di jawabannya

Saat berhenti, bot Telegram akan mengirim pesan ke Darma berisi cuplikan jawaban AI yang memicu alarm. Darma bisa membalas:
- `OK` atau `LANJUT` → sistem lanjut ke AI berikutnya
- `REVISI: [arahan baru]` → sistem mengulang pertanyaan ke AI yang sama dengan arahan baru
- `STOP` → sistem menghentikan seluruh sesi

### **4.4 Tombol Kendali Manual di Telegram**
Darma bisa mengirim perintah ini kapan saja:
- `/pause` → menjeda diskusi (seperti tombol rem darurat)
- `/lanjut` → melanjutkan diskusi yang dijeda
- `/stop` → menghentikan seluruh sesi
- `/status` → menanyakan kondisi diskusi saat ini

### **4.5 Cara Menjaga Obrolan Tetap Nyambung (Ringkasan)**
Setiap kali memulai putaran baru, sistem akan meminta **DeepSeek** untuk membuat ringkasan singkat dari hasil diskusi putaran sebelumnya. Ringkasan inilah yang diberikan ke semua AI, bukan obrolan panjang yang bikin bingung dan boros biaya.

### **4.6 Fitur Keamanan Tambahan**

**Batas Maksimal Biaya:**
- Darma bisa menentukan batas maksimal berapa putaran diskusi dalam satu sesi (misal: maksimal 10 putaran)
- Jika sudah mencapai batas, sistem akan berhenti dan kirim notifikasi ke Telegram

**Mati Otomatis Jika Ditinggal:**
- Jika sistem meminta persetujuan Darma tapi tidak dibalas lebih dari **24 jam**, sistem akan mematikan dirinya sendiri
- Bot akan mengirim pesan terakhir: "Sesi sudah 24 jam tidak ada kabar. Sesi dihentikan otomatis."

---

## **V. PENGAWAS DISKUSI & ALARM OTOMATIS**

*Untuk Darma:* Sistem punya "alarm" yang akan memberitahu jika diskusi mulai melenceng atau muter-muter. Darma tidak perlu memeriksa setiap saat — cukup lihat warna di Sheet atau baca pesan Telegram.

### **5.1 Alarm Warna di Google Sheet**
Di Sheet, kolom status akan memiliki warna otomatis:

| Warna | Arti | Kapan Muncul |
|:---|:---|:---|
| 🟢 **Hijau** | Aman, diskusi sehat | AI mulai sepakat atau sudah sepakat |
| 🟡 **Kuning** | Waspada, diskusi mulai muter | Beberapa AI masih berdebat tanpa kemajuan |
| 🔴 **Merah** | Butuh Darma | Ada error, macet, atau sistem berhenti |

Darma cukup buka Sheet dan lirik warnanya untuk tahu kondisi diskusi.

### **5.2 Laporan Singkat di Telegram Setiap Putaran**
Setelah satu putaran selesai (5 AI sudah semua menjawab), bot akan mengirim pesan seperti ini:

> 🔄 **Putaran 3 selesai**
> Status: 🟢 Aman
> Yang sudah sepakat: 3 dari 5 AI
> ⚠️ Masih dibahas: masalah biaya operasional
>
> [Ketik OK untuk lanjut] [Ketik STOP untuk hentikan]

### **5.3 Deteksi Masalah Otomatis**
Sistem akan otomatis mendeteksi:
- **Diskusi muter-muter:** Jika 3 AI berturut-turut statusnya "masih menjelajah" padahal sudah di atas putaran ke-5
- **Diskusi macet:** Jika AI menulis masalah yang sama persis 2 kali berturut-turut
- **Darurat macet:** Jika masalah yang sama muncul 3 kali berturut-turut, sistem langsung berhenti dan minta Darma turun tangan

### **5.4 AI Pengawas (Dipanggil Hanya Jika Perlu)**
Jika sistem mendeteksi anomali (diskusi muter atau macet), sistem akan memanggil satu AI khusus (Claude atau Gemini) untuk membaca rangkuman diskusi dan memberikan penilaian singkat. Hasil penilaian ini akan dikirim ke Telegram Darma.

**Penting:** AI Pengawas ini **tidak bisa mematikan sistem**. Dia hanya memberi saran. Kendali tetap penuh di tangan Darma.

### **5.5 Aturan Wajib: Bahasa Komunikasi Sistem**
Semua pesan dari sistem ke Darma (lewat Telegram atau laporan) **wajib menggunakan bahasa Indonesia sehari-hari yang mudah dipahami**. Tidak boleh ada istilah teknis pemrograman seperti "CIRCULAR_RISK detected", "state drift", "exponential backoff", atau istilah asing lainnya.

**Contoh yang benar:** "Diskusi kayaknya mulai muter-muter nih, coba dicek."
**Contoh yang salah:** "CIRCULAR_RISK terdeteksi di round 5. Memerlukan intervensi manual."

---

## **VI. CARA DARMA MENGAUDIT & MENGEVALUASI**

### **6.1 Audit Manual (Lihat Sendiri di Sheet)**
Darma bisa membuka Google Sheet kapan saja untuk melihat:
- Semua jawaban AI di setiap putaran (tab `session_log`)
- Status terkini setiap AI (hijau/kuning/merah)
- Catatan kejadian penting: kapan diskusi macet, kapan Darma turun tangan (tab `audit_events`)

### **6.2 Laporan Otomatis Setelah Sesi Selesai**
Setelah sesi ditutup, sistem akan otomatis membuat laporan ringkasan di tab baru berisi:
- Berapa total putaran diskusi
- Apakah tercapai kesepakatan atau tidak
- Daftar semua masalah yang muncul selama diskusi
- Estimasi biaya per AI
- AI mana yang paling lambat atau sering error
- Ringkasan keputusan yang bisa langsung dipakai

### **6.3 Fitur Putar Ulang Sesi**
Darma bisa mengekspor seluruh isi diskusi menjadi file yang bisa dibuka ulang di Orchestrator Terminal v0.1 untuk diperiksa kembali atau disimulasikan.

### **6.4 Jejak Intervensi Manual**
Jika suatu saat Darma mengedit langsung isi Sheet (misal mengubah jawaban AI atau status), sistem akan mencatatnya: "Darma melakukan perubahan manual pada [tanggal dan jam]".

---

## **VII. STRUKTUR GOOGLE SHEET**

### **Tab 1: `session_log` (Catatan Utama Diskusi)**

| Kolom | Isinya |
|:---|:---|
| `round_id` | Kode unik untuk setiap jawaban |
| `session_id` | Kode sesi diskusi |
| `round` | Putaran ke berapa |
| `ai_target` | AI mana yang menjawab |
| `darma_snapshot` | Pertanyaan Darma yang berlaku saat itu |
| `response` | Jawaban lengkap dari AI |
| `state` | Status: EXPLORING (menjelajah) / DISCUSSING (berdiskusi) / READY_TO_CLOSE (hampir sepakat) / CLOSED (selesai) / ERROR / SKIPPED / PAUSED (dijeda) |
| `is_paused` | Apakah sedang menunggu Darma? |
| `timestamp` | Waktu jawaban disimpan |
| `response_time_ms` | Berapa detik AI merespon |
| `drift_flag` | Apakah jawaban ini melenceng? |
| `stuck_flag` | Apakah jawaban ini macet? |

### **Tab 2: `audit_events` (Catatan Kejadian Penting)**

| Kolom | Isinya |
|:---|:---|
| `event_id` | Kode unik kejadian |
| `session_id` | Kode sesi |
| `event_type` | Jenis kejadian: DRIFT (melenceng), STUCK (macet), ERROR, MANUAL_INTERVENTION (Darma turun tangan), CIRCUIT_BREAK (rem darurat aktif) |
| `detail` | Penjelasan singkat kejadian |
| `timestamp` | Waktu kejadian |

### **Tab 3: `session_config` (Pengaturan Sesi)**

| Kolom | Isinya |
|:---|:---|
| `config_key` | Nama pengaturan |
| `config_value` | Nilai pengaturan |

Pengaturan yang bisa diubah Darma:
- `max_rounds`: Batas maksimal putaran (default: 10)
- `mode`: Mode diskusi (antiyesman / konvergensi / kristalisasi / custom)
- `custom_instruction`: Instruksi khusus dari Darma (jika mode custom)
- `pause_keywords`: Kata-kata pemicu rem darurat (default: budget, deploy, hapus, biaya, beli, langganan)

---

## **VIII. RINGKASAN FITUR UTAMA**

| Fitur | Fungsi | Untuk Siapa |
|:---|:---|:---|
| **Otomatisasi Rotasi** | AI dipanggil satu per satu sesuai urutan | Menggantikan copas manual Darma |
| **Rem Darurat** | Sistem berhenti jika AI bicara hal sensitif | Memberi kendali ke Darma |
| **Tombol Telegram** | Darma bisa pause/lanjut/stop dari HP | Kendali jarak jauh |
| **Alarm Warna** | Sheet menampilkan warna hijau/kuning/merah | Audit sekilas tanpa baca detail |
| **Laporan Telegram** | Ringkasan singkat tiap putaran | Info cepat tanpa buka Sheet |
| **AI Pengawas** | AI khusus mengecek jika diskusi bermasalah | Membantu Darma mendeteksi masalah |
| **Batas Biaya** | Maksimal putaran per sesi | Mencegah boros API |
| **Mati Otomatis** | Sistem berhenti jika Darma tidak respon 24 jam | Mencegah sistem menggantung |
| **Laporan Akhir** | Ringkasan otomatis setelah sesi selesai | Dokumentasi keputusan |
| **Putar Ulang** | Ekspor sesi untuk dibuka di terminal lama | Evaluasi mendalam |
| **Bahasa Awam** | Semua pesan sistem pakai bahasa Indonesia sederhana | Darma tidak perlu paham istilah teknis |

---

## **IX. APPENDIX: PANDUAN UJI COBA AMAN (SAFETY CHECKLIST)**

*Untuk Darma:* Sebelum sistem dijalankan penuh dengan 5 AI sekaligus, ikuti panduan ini agar aman dan tidak boros biaya.

### **Langkah 1: Uji Coba dengan 1 AI Dulu**
- Jalankan sistem dengan **hanya 1 AI aktif** (misal: DeepSeek)
- Beri 1 pertanyaan sederhana
- Cek apakah jawabannya masuk ke Sheet dengan benar
- Cek apakah notifikasi Telegram berfungsi

### **Langkah 2: Uji Coba dengan 2 AI**
- Aktifkan 2 AI (misal: DeepSeek + Claude)
- Pastikan AI kedua membaca jawaban AI pertama
- Pastikan ringkasan berfungsi

### **Langkah 3: Uji Coba Rem Darurat**
- Tulis pertanyaan yang sengaja memancing AI menyebut kata sensitif
- Pastikan sistem berhenti dan kirim notifikasi ke Telegram
- Balas `OK` dari Telegram dan pastikan sistem lanjut

### **Langkah 4: Uji Coba Tombol Kendali**
- Kirim `/pause` dari Telegram saat diskusi berjalan
- Kirim `/status` untuk cek kondisi
- Kirim `/lanjut` untuk melanjutkan

### **Langkah 5: Full Run**
- Setelah semua langkah di atas berhasil, sistem siap dijalankan dengan 5 AI penuh

### **Tips Keamanan:**
- Selalu mulai dengan `max_rounds = 3` untuk sesi pertama
- Pantau Sheet di 2 putaran pertama untuk memastikan semua AI merespon dengan benar
- Jika ada AI yang terus error, bisa di-skip sementara dengan mengosongkan API key-nya di pengaturan
