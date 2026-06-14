# SOT ARAHAN PENTING SEBELUM DISKUSI

---

## 1. KONTEKS & PERAN DARMA

| Elemen | Isi Kesepakatan |
|---|---|
| Peran Darma | **AI Orchestrator**, bukan pure coder |
| Tanggung jawab teknis | Detail teknis (kode, stack spesifik) adalah **turunan** dari keputusan vital — AI boleh memutuskan teknis |
| Tanggung jawab inti | Darma mengendalikan **keputusan vital/inti** dan memastikan AI tidak saling menyesatkan |
| Prinsip build | Build paling kompleks selama itu opsi terbaik, value creation maksimal, **tidak menunda inovasi hanya karena "di luar scope MVP"** |
| Panel AI | Darma berdiskusi dengan banyak AI sekaligus untuk validasi silang, saling menantang, dan meminimalisir halusinasi |

---

## 2. MASALAH YANG DITEMUKAN & SOLUSI YANG DISEPAKATI

| # | Masalah | Solusi yang Disepakati |
|---|---|---|
| 1 | **"Scope Lock" tradisional terlalu menahan inovasi** — fitur advance tertunda meski sudah diketahui nilainya saat diskusi awal | Diganti menjadi **Capability Strategy** (klasifikasi: Core Now / Strategic Advance Now / Architecture-Ready Later / Reject) |
| 2 | **Darma sering belum punya "product knowledge" cukup** saat mulai proyek | **(a)** AI wajib menggunakan **bahasa sederhana yang orang awam mudah pahami** saat bertanya hal non-teknis; **(b)** Di Problem–Value Anchor ditambahkan **Domain Assumption Check** (apa yang diketahui, belum diketahui, perlu diverifikasi) |
| 3 | **AI bisa menghalusinasi fitur advance dan melabelinya "strategis"** agar terlihat pintar | Setiap usulan "Strategic Advance Now" wajib dibuktikan dengan **3 filter**: (a) meningkatkan value instan, (b) tech stack mampu handle tanpa over-engineering, (c) **memperkuat Problem–Value Anchor** (bukan sekadar canggih) |
| 4 | **Istilah "Evolvable Architecture" terlalu abstrak untuk AI** | AI wajib memberi **parameter konkret**: modularitas, decoupling, pemisahan logika bisnis dan data |
| 5 | **Data dan API dibahas terpisah bisa menyebabkan bongkar ulang** | Data Model + State + Business Rules + API Contract dibahas **dalam satu kesatuan** (Evolvable Architecture + Data/API Contract) |
| 6 | **Testing/deployment/SOT diposisikan setara dengan keputusan vital** | Disepakati sebagai **output turunan**, tapi **operational constraint-nya** (hosting, budget, keamanan dasar, backup SLA) wajib diketahui sejak awal karena bisa memengaruhi arsitektur |
| 7 | **Izin "rombak SoT" bisa menjadi lisensi regresi** | Diperkenalkan **Change Governance** dengan **Impact Log 3 Baris**: (1) Kontrak Lama → apa yang diubah, (2) Kontrak Baru → opsi lebih baik apa, (3) Blast Radius → komponen mana yang ikut terdampak/patah |

---

## 3. ADAPTIVE PRODUCT PLANNING FRAMEWORK (5 Keputusan Vital + Change Governance)

Ini adalah **struktur inti** yang disepakati untuk setiap diskusi rencana sebelum build:

### Poin 1: Problem–Value Anchor
- **Isi:**
  - Masalah apa yang diselesaikan
  - Untuk siapa
  - Nilai apa yang diciptakan
  - Kenapa layak dibuild
- **Domain Assumption Check (subbagian wajib):**
  - Apa yang kita **tahu** tentang domain/user/pasar
  - Apa yang **belum kita tahu**
  - Apa yang **perlu diverifikasi** sebelum build
- **Alasan disepakati:** Jangkar utama yang tidak berubah meski fitur bergeser. Tanpa ini, inovasi jadi liar tanpa arah. Bahasa sederhana + Domain Check memastikan Darma bisa validasi pemahaman AI tanpa harus jadi expert domain.

### Poin 2: Capability Strategy (pengganti Scope Lock)
- **Isi:** Bukan "apa fitur minimal?", tapi **"kemampuan apa yang harus dimiliki sistem agar value-nya kuat dan kompetitif?"**
- **Klasifikasi fitur:**

| Kategori | Makna |
|---|---|
| **Core Now** | Wajib ada agar produk berguna dan value terasa |
| **Strategic Advance Now** | Fitur advance yang layak langsung dibuild karena menaikkan value signifikan |
| **Architecture-Ready Later** | Belum dibuild sekarang, tapi fondasi arsitektur harus disiapkan agar tidak rewrite besar nanti |
| **Reject / Noise** | Terlihat keren tapi tidak menguatkan Problem–Value Anchor |

- **3 Filter wajib untuk "Strategic Advance Now":**
  1. Apakah fitur ini **meningkatkan value secara instan**?
  2. Apakah **tech stack mampu menghandle** tanpa over-engineering?
  3. Apakah fitur ini benar-benar **memperkuat Problem–Value Anchor**, atau hanya membuat sistem terlihat canggih?
- **Alasan disepakati:** Memberi ruang gerilya inovasi tanpa kehilangan struktur. Cocok untuk Darma yang ingin value creation maksimal sejak awal, namun tetap punya pagar anti-noise.

### Poin 3: Flow–Rules–State Lock
- **Isi:**
  - **Flow:** Siapa melakukan apa, user journey, alur normal dan alternatif
  - **Rules:** Aturan bisnis yang tidak boleh dilanggar (invariant rules)
  - **State:** Status/state sistem yang mungkin terjadi
  - **Role & Permission:** Hak akses kasar setiap aktor
- **Alasan disepakati:** Banyak produk gagal bukan karena kurang fitur, tapi karena alur kerja, aturan, dan statusnya kacau. Ini pondasi yang tidak boleh cair.

### Poin 4: Evolvable Architecture + Data/API Contract
- **Isi (dibahas dalam satu kesatuan, tidak boleh dipisah):**
  - Struktur sistem dan modul
  - Model data dan state management
  - Business rules di level data
  - API Contract (endpoint, request-response, integration map)
  - Extensibility: bagaimana sistem bisa ditambah fitur tanpa rewrite besar
- **Guardrail wajib:** "Evolvable" harus diberi parameter konkret — modularitas, decoupling logic dan data, pemisahan layer.
- **Alasan disepakati:** Memisahkan Data dan API menyebabkan circular dependency dan bongkar ulang. Fokusnya adalah fondasi adaptif yang memungkinkan inovasi fitur tanpa arsitektur runtuh.

### Poin 5: Risk–Validation Gate
- **Isi:**
  - Apa yang bisa **membunuh project** (risiko tertinggi)?
  - **Asumsi paling berbahaya** yang belum terbukti
  - **Edge case & abuse case** (kondisi aneh, penyalahgunaan)
  - Risiko teknis, user, biaya, keamanan, maintenance
  - **Operational constraint wajib**: hosting, budget, keamanan dasar, backup SLA — karena bisa memengaruhi arsitektur sejak awal
- **Output turunan dari 5 keputusan di atas** (bukan diskusi setara):
  - Testing plan
  - Deployment plan
  - Backup/recovery plan
  - Log/monitoring plan
  - Dokumentasi teknis detail
  - Source of Truth (SOT) — dikompilasi dari output keputusan 1–5

### Change Governance: Impact Log 3 Baris
- **Kapan dipakai:** Setiap kali ada usulan perubahan besar terhadap keputusan yang sudah dikunci (rombak SoT).
- **Format wajib:**

| Baris | Isi |
|---|---|
| **KONTRAK LAMA** | Apa yang diubah? Keputusan sebelumnya apa? |
| **KONTRAK BARU** | Opsi lebih baik apa yang ditemukan? |
| **BLAST RADIUS** | Komponen mana saja yang ikut terdampak/patah akibat perubahan ini? |

- **Alasan disepakati:** Tanpa mekanisme ini, kebebasan "merombak SoT jika ada opsi lebih baik" akan jadi lisensi untuk regresi dan inkonsistensi sistem. Impact Log memaksa AI mencatat dampak sebelum mengubah kode.

---

## 4. ATURAN KOMUNIKASI & OPERASIONAL YANG DISEPAKATI

| # | Aturan | Kategori |
|---|---|---|
| 1 | **Bahasa sederhana** saat AI bertanya hal non-teknis ke Darma | Komunikasi |
| 2 | **Domain Assumption Check** di Problem–Value Anchor | Validasi domain |
| 3 | **3 Filter** untuk setiap usulan "Strategic Advance Now" | Anti-halusinasi fitur |
| 4 | **Parameter konkret** untuk "Evolvable Architecture" | Anti-abstraksi |
| 5 | **Data + API tidak boleh dipisah** dalam sesi diskusi | Anti-circular dependency |
| 6 | **Operational constraint wajib diketahui awal** (hosting, budget, keamanan) | Anti-arsitektur salah fondasi |
| 7 | **Change Governance via Impact Log** (Kontrak Lama → Baru → Blast Radius) | Anti-regresi |
| 8 | **Aturan prioritas tertinggi tetap berlaku** (ABSTAIN > SALAH, LABEL klaim, PISAHKAN fakta/reasoning, EXPOSE asumsi, DILARANG referensi palsu, Akurasi > Kelengkapan, Gate Challenge adaptif) | Fondasi integritas diskusi |

---

## 5. URUTAN EKSEKUSI YANG DISEPAKATI

```
1. Problem–Value Anchor      → kunci dulu + Domain Assumption Check
2. Capability Strategy        → klasifikasi fitur + 3 filter
3. Flow–Rules–State Lock      → alur, aturan, status, role
4. Evolvable Architecture     → struktur + data + API (satu kesatuan)
5. Risk–Validation Gate       → uji sebelum build + operational constraint
   ↓
   TURUNAN: Testing, Deploy, SOT, Backup, Log, Dokumen teknis (AI generate)
   ↓
   CHANGE GOVERNANCE: setiap perubahan besar → Impact Log 3 Baris
```