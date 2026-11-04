# RENCANA REVISI ZLCE 3.22 (P1-CP2)
**Proyek:** ZLCE3.22 Studio → ZLCE Mobile Workbench  
**Tahap:** Track 1 — Revisi ZLCE3.22 | Checkpoint: P1-CP2 (Rencana Revisi)  
**Dokumen Induk:** `docs/revisi/AUDIT_3.22.md`  
**Status:** DRAFT USULAN REVISI — MENUNGGU PERSETUJUAN PENGGUNA  

Dokumen ini memetakan seluruh usulan revisi hasil audit P1-CP1 menjadi paket pekerjaan terstruktur. Setiap usulan memuat rincian perubahan, berkas terdampak, risiko regresi, cara verifikasi, dan alokasi track.

---

## DAFTAR USULAN REVISI (REV-1 s/d REV-12)

### REV-1: Konsolidasi Skrip CLI dan Sinkronisasi Antar-Direktori
- **Rujukan Temuan:** `AUDIT_3.22.md` Bagian C.1 (Duplikasi skrip di 3 lokasi).
- **Masalah:** Terdapat 3 salinan file yang sama (`scripts/`, `scripts/zlce/`, `zlce-starter-kit/scripts/`). Perubahan pada salah satu file berisiko membuat salinan lain desinkron.
- **Rencana Perubahan Tepat:**
  1. Tetapkan `scripts/zlce/` sebagai **lokasi kanonik primer** untuk skrip CLI (`zlce-check.py`, `zlce-collect.py`, `zlce-snapshot.sh`), konsisten dengan dokumentasi `README.md:112` dan `LANDING_PROMPT.md:52`.
  2. Pada folder `scripts/`, jadikan skrip sebagai wrapper penerus (forwarder) yang memanggil `scripts/zlce/<script>`, atau pertahankan salinan yang dipastikan identik lewat script verifikasi integritas checksum.
  3. Perbarui `zlce-starter-kit/scripts/` agar tersinkronisasi otomatis dengan sumber kanonik.
- **Berkas Terdampak:** `scripts/*`, `scripts/zlce/*`, `zlce-starter-kit/scripts/*`.
- **Risiko Regresi:** Rendah. Pemanggilan lama melalui `python3 scripts/zlce-check.py` tetap berfungsi normal.
- **Cara Verifikasi:** Jalankan skrip dari root, dari dalam `scripts/`, dan dari `scripts/zlce/`; bandingkan output dan exit code.
- **Dependensi:** Tidak ada.
- **Prioritas:** Tinggi (P1).
- **Alokasi:** **Track 1 (Perbaikan Fondasi)**.

---

### REV-2: Sinkronisasi Dokumen Prompt Kanonik (`LANDING_PROMPT.md` & `HANDOVER_PROMPT.md`)
- **Rujukan Temuan:** `AUDIT_3.22.md` Bagian C.2 (Duplikasi prompt di root dan starter kit).
- **Masalah:** Duplikasi manual dokumen prompt di root dan `zlce-starter-kit/` tanpa mekanisme jaminan sinkronisasi.
- **Rencana Perubahan Tepat:**
  1. Tetapkan dokumen di root (`LANDING_PROMPT.md` dan `HANDOVER_PROMPT.md`) sebagai sumber kanonik tunggal.
  2. Tambahkan pemeriksaan integritas checksum pada skrip uji otomatis (`scripts/run-tests.py`) untuk memastikan file di `zlce-starter-kit/` selalu identik 100% dengan versi root.
- **Berkas Terdampak:** `LANDING_PROMPT.md`, `HANDOVER_PROMPT.md`, `zlce-starter-kit/*`.
- **Risiko Regresi:** Nihil (tidak ada perubahan semantik teks protokol).
- **Cara Verifikasi:** `diff -u` antara root dan starter kit menghasilkan zero diff.
- **Dependensi:** REV-5 (Test runner).
- **Prioritas:** Sedang (P2).
- **Alokasi:** **Track 1 (Perbaikan Fondasi)**.

---

### REV-3: Penyelarasan Penyimpanan Sesi: UI Ekspor/Impor Markdown Persisten & Status Cache
- **Rujukan Temuan:** `AUDIT_3.22.md` Bagian C.3 (Penyimpanan state hanya di localStorage).
- **Masalah:** Web app hanya mengandalkan `localStorage` browser. Pengguna berisiko mengira tombol Simpan menulis langsung ke filesystem, dan data bisa hilang jika cache browser dibersihkan.
- **Rencana Perubahan Tepat:**
  1. Pertahankan seluruh logika persistensi yang ada pada `src/hooks/useSessionPersistence.ts` (autosave debounce, riwayat snapshot undo/redo in-memory tetap berfungsi utuh).
  2. Tambahkan label informatif di antarmuka (`SessionStateTab.tsx` dan `Header.tsx`) bahwa sesi tersimpan di *Local Cache Browser*.
  3. Tambahkan tombol aksi eksplisit:
     - **"Ekspor Markdown"**: Mengunduh teks mentah sebagai berkas `SESSION_STATE.md` ke perangkat pengguna.
     - **"Impor Markdown"**: Membaca berkas `.md` dari komputer/tablet pengguna dan memuatnya ke dalam editor workbench setelah melewati validasi sintaks ZLCE.
- **Berkas Terdampak:** `src/components/Header.tsx`, `src/components/SessionStateTab.tsx`, `src/hooks/useSessionPersistence.ts`.
- **Risiko Regresi:** Rendah. Tidak merusak skema penyimpanan `localStorage` yang sudah berjalan.
- **Cara Verifikasi:** Uji impor berkas markdown valid dan berkas markdown rusak; uji ekspor unduhan berkas; pastikan undo/redo dan autosave tetap berjalan normal.
- **Dependensi:** Tidak ada.
- **Prioritas:** Tinggi (P1).
- **Alokasi:** **Track 1 (Perbaikan Fondasi)**.

---

### REV-4: Pembersihan dan Klarifikasi Dependensi Pasif & Lingkungan
- **Rujukan Temuan:** `AUDIT_3.22.md` Bagian C.4 (Dependensi backend/AI tidak terpakai).
- **Masalah:** `@google/genai`, `express`, `dotenv`, `@types/express` terpasang di `package.json` tetapi tidak diimpor dalam kode aplikasi SPA. `.env.example` mencantumkan `GEMINI_API_KEY`.
- **Rencana Perubahan Tepat:**
  1. Hapus paket `@google/genai`, `express`, `dotenv`, `@types/express` dari `package.json` untuk merampingkan footprint proyek client-side murni, ATAU dokumentasikan perannya jika dicadangkan untuk server proxy backend.
  2. Perbarui `.env.example` dan dokumentasi README untuk menegaskan prinsip ZLCE: *Aplikasi beroperasi secara mandiri (offline & AI-agnostic) tanpa ketergantungan wajib pada API key server*.
  3. Pastikan aturan `.gitignore` tetap melindungi berkas kredensial `.env*`.
- **Berkas Terdampak:** `package.json`, `.env.example`, `README.md`.
- **Risiko Regresi:** Sangat rendah. Build React client-side tidak menggunakan modul tersebut.
- **Cara Verifikasi:** Jalankan `npm run build` dan `npm run lint` untuk memastikan kompilasi tetap sukses 100%.
- **Dependensi:** Tidak ada.
- **Prioritas:** Sedang (P2).
- **Alokasi:** **Track 1 (Perbaikan Fondasi)**.

---

### REV-5: Penambahan Test Runner Otomatis & Pembersihan Badge CI / Evaluasi
- **Rujukan Temuan:** `AUDIT_3.22.md` Bagian C.5 (Ketiadaan skrip test, badge CI mati, dan klarifikasi klaim 14/14).
- **Masalah:** `package.json` tidak memiliki skrip `test`. Badge CI menunjuk ke workflow yang tidak ada.
- **Rencana Perubahan Tepat:**
  1. Buat skrip automated test suite `scripts/run-tests.py` yang mengeksekusi:
     - Uji integritas skrip CLI dan dokumen template (diff dan hashing).
     - Uji eksekusi `zlce-check.py` terhadap berkas valid di `examples/` dan kasus uji invalid buatan (verifikasi deteksi pelanggaran R4, R5, R12).
     - Uji sintaksis skrip shell dengan validasi POSIX.
  2. Tambahkan script `"test": "python3 scripts/run-tests.py"` pada `package.json`.
  3. Perbaiki `README.md`: Hapus badge workflow CI yang mati atau arahkan badge ke status test runner lokal yang valid; perjelas deskripsi badge "14/14 Lulus" sebagai hasil benchmark AI Studio kualitatif.
- **Berkas Terdampak:** `package.json`, `README.md`, berkas baru `scripts/run-tests.py`.
- **Risiko Regresi:** Tidak ada.
- **Cara Verifikasi:** Jalankan `npm test` di terminal; verifikasi semua pengujian lolos (exit code 0).
- **Dependensi:** REV-1, REV-2, REV-7, REV-10.
- **Prioritas:** Tinggi (P1).
- **Alokasi:** **Track 1 (Perbaikan Fondasi)**.

---

### REV-6: Perbaikan URL Repositori dan Tautan Dokumentasi
- **Rujukan Temuan:** `AUDIT_3.22.md` Bagian C.6 (Tautan lama `ZLCEv3.2.2`).
- **Masalah:** `README.md` baris 9, 15, 101, 102 masih menunjuk ke repositori lama `acimau99-netizen/ZLCEv3.2.2`.
- **Rencana Perubahan Tepat:**
  Perbarui semua URL clone, issues, dan dokumentasi di `README.md` dan `docs/` menjadi:
  `https://github.com/acimau99-netizen/ZLCE3.22-studio.git`
- **Berkas Terdampak:** `README.md`, `docs/USAGE.md`.
- **Risiko Regresi:** Nihil.
- **Cara Verifikasi:** Jalankan `grep -rn "ZLCEv3.2.2" README.md docs/` dan pastikan tidak ada tautan salah yang tersisa.
- **Dependensi:** Tidak ada.
- **Prioritas:** Sedang (P2).
- **Alokasi:** **Track 1 (Perbaikan Fondasi)**.

---

### REV-7: Penyelarasan Cakupan Validator Web (`zlceValidator.ts`) vs CLI (`zlce-check.py`)
- **Rujukan Temuan:** `AUDIT_3.22.md` Bagian C.7 (Kesenjangan aturan validasi).
- **Masalah:** Dokumen yang lolos di CLI bisa gagal di web (karena web memeriksa bagian 0, 5, 6, 7 dan Aturan Jam X1). Sebaliknya, web tidak bisa memeriksa cross-check `TASKS.md` dan Git.
- **Rencana Perubahan Tepat:**
  1. Perbarui `scripts/zlce/zlce-check.py`:
     - Tambahkan pemeriksaan keberadaan 8 bagian wajib (0 sampai 7).
     - Tambahkan validasi format timestamp Aturan X1 (format waktu valid atau fallback `jam: ?`).
  2. Pada antarmuka Web (`src/utils/zlceValidator.ts` dan `src/components/SessionStateTab.tsx`):
     - Tambahkan catatan visual yang transparan bahwa pemeriksaan cross-check terhadap `TASKS.md` dan status Git adalah verifikasi lapisan kedua yang dilakukan via CLI/Termux.
- **Berkas Terdampak:** `scripts/zlce/zlce-check.py`, `src/utils/zlceValidator.ts`, `docs/USAGE.md`.
- **Risiko Regresi:** Dokumen lama yang tidak lengkap bagiannya akan menghasilkan WARN/ERROR di CLI. Solusi: berikan penanda peringatan (WARN) yang terukur agar tidak memutus dokumen yang sedang dalam transisi, kecuali dipanggil dengan `--strict`.
- **Cara Verifikasi:** Jalankan `zlce-check.py` dan validator web terhadap `examples/SESSION_STATE.example.md`; pastikan kedua validator memberikan vonis konsisten.
- **Dependensi:** REV-1.
- **Prioritas:** Tinggi (P1).
- **Alokasi:** **Track 1 (Perbaikan Fondasi)**.

---

### REV-8: Dokumentasi dan Penyempitan Konfigurasi Dev-Server
- **Rujukan Temuan:** `AUDIT_3.22.md` Bagian C.8 (Dev server mengikat ke `0.0.0.0`).
- **Masalah:** Pengikatan `--host=0.0.0.0` membuka akses port 3000 ke seluruh jaringan lokal.
- **Rencana Perubahan Tepat:**
  1. Dokumentasikan di `README.md` bahwa binding `--host=0.0.0.0` di container AI Studio diperlukan untuk reverse-proxy Cloud Run, namun pada instalasi lokal mandiri disarankan memakai `npm run dev -- --host=localhost` untuk membatasi paparan jaringan.
  2. Pastikan konfigurasi dev script di `package.json` tetap mematuhi port 3000 yang disyaratkan AI Studio.
- **Berkas Terdampak:** `README.md`, `package.json`.
- **Risiko Regresi:** Sangat rendah jika tidak mengubah default container.
- **Cara Verifikasi:** Dev server tetap dapat diakses di container AI Studio.
- **Dependensi:** Tidak ada.
- **Prioritas:** Rendah (P3).
- **Alokasi:** **Track 1 (Dokumentasi/Perbaikan)**.

---

### REV-9: Peningkatan Portabilitas POSIX Skrip Bash `zlce-snapshot.sh` (Termux & Android)
- **Rujukan Temuan:** `AUDIT_3.22.md` Bagian C.9 (Sintaks GNU date dan head pada snapshot shell).
- **Masalah:** `date -Iseconds` dan `head -400` gagal pada Busybox/Toybox Android.
- **Rencana Perubahan Tepat:**
  1. Ubah `date -Iseconds` menjadi format portabel POSIX:
     `date +'%Y-%m-%dT%H:%M:%S%z' 2>/dev/null || date`
  2. Ubah `head -400` menjadi `head -n 400`.
  3. Ubah `head -80 "$f"` menjadi `head -n 80 "$f"`.
  4. Ganti `grep -Iq` dengan deteksi file teks portabel atau penanganan fallback yang aman.
- **Berkas Terdampak:** `scripts/zlce/zlce-snapshot.sh` (dan salinannya).
- **Risiko Regresi:** Nihil. Sintaks standar POSIX didukung di Linux, macOS, dan Android/Termux.
- **Cara Verifikasi:** Jalankan skrip di terminal bash; periksa format tanggal dan kelengkapan snapshot.
- **Dependensi:** REV-1.
- **Prioritas:** Tinggi (P1).
- **Alokasi:** **Track 1 (Perbaikan Fondasi)**.

---

### REV-10: Peringatan Kegagalan Git Eksplisit & Fleksibilitas Argumen di `zlce-check.py`
- **Rujukan Temuan:** `AUDIT_3.22.md` Bagian C.11 (Silent failure pada git dan hardcoded TASKS.md).
- **Masalah:** Jika git gagal atau direktori bukan repo git, pemeriksaan dilewati tanpa peringatan. Pengujian terhadap folder `examples/` gagal karena nama file berakhiran `.example.md`.
- **Rencana Perubahan Tepat:**
  1. Pada `scripts/zlce/zlce-check.py`:
     - Jika `--no-git` tidak disertakan dan pemanggilan Git mengembalikan error/None saat mencoba memeriksa repo, cetak pesan peringatan eksplisit:
       `[WARN] Pemeriksaan Git dilewati (git tidak merespons, safe.directory, atau bukan git work-tree)`.
     - Tambahkan argumen opsional `--tasks <path>` agar pengguna dapat menentukan berkas tugas alternatif (misalnya untuk menguji `examples/TASKS.example.md`).
     - Berikan fallback otomatis: jika `TASKS.md` tidak ditemukan di direktori target tetapi `TASKS.example.md` ada, gunakan berkas tersebut.
- **Berkas Terdampak:** `scripts/zlce/zlce-check.py`.
- **Risiko Regresi:** Rendah. Default tanpa argumen tetap mencari `TASKS.md` di root proyek.
- **Cara Verifikasi:** Jalankan `python3 scripts/zlce/zlce-check.py examples/ --state examples/SESSION_STATE.example.md` dan pastikan pemeriksaan silang task berhasil berjalan tanpa error.
- **Dependensi:** REV-1, REV-7.
- **Prioritas:** Tinggi (P1).
- **Alokasi:** **Track 1 (Perbaikan Fondasi)**.

---

### REV-11: Penambahan `.gitattributes`, Normalisasi Line Endings (LF), dan File Permissions
- **Rujukan Temuan:** `AUDIT_3.22.md` Bagian C.12 (Ketiadaan `.gitattributes`, CRLF pada shell script, mode 100644).
- **Masalah:** Skrip `zlce-snapshot.sh` mengandung CRLF (`\r\n`) yang memicu error fatal `bad interpreter` di Linux/Termux, dan file mode tidak executable.
- **Rencana Perubahan Tepat:**
  1. Buat berkas `.gitattributes` di root dengan konfigurasi:
     ```gitattributes
     * text=auto eol=lf
     *.sh text eol=lf
     *.py text eol=lf
     *.md text eol=lf
     *.json text eol=lf
     *.ts text eol=lf
     *.tsx text eol=lf
     *.zip binary
     ```
  2. Lakukan normalisasi CRLF menjadi LF pada seluruh skrip shell dan python di repositori.
  3. Setel bit eksekusi di Git index untuk skrip shell dan python (`git update-index --chmod=+x scripts/zlce/*.sh scripts/zlce/*.py`).
- **Berkas Terdampak:** `.gitattributes`, `scripts/zlce/zlce-snapshot.sh`, `scripts/zlce/zlce-check.py`, `scripts/zlce/zlce-collect.py`.
- **Risiko Regresi:** Nihil. Mencegah error fatal runtime di Linux dan Android.
- **Cara Verifikasi:** Periksa dengan `file scripts/zlce/zlce-snapshot.sh` (harus ASCII text executable, tanpa carriage returns) dan `git ls-files -s scripts/zlce/` (harus `100755`).
- **Dependensi:** REV-1.
- **Prioritas:** Kritis (P0).
- **Alokasi:** **Track 1 (Perbaikan Fondasi)**.

---

### REV-12 (DITUNDA KE TRACK 2): Model Multi-AI, Multi-Akun, Tim, dan Provenance
- **Rujukan Temuan:** `AUDIT_3.22.md` Bagian C.13 (Asumsi single-actor pada matriks, WAL, dan handover).
- **Masalah:** ZLCE 3.22 Core saat ini hanya mendukung 1 task aktif, 1 IN_PROGRESS (Rule R4 menolak >1), dan 1 blok WAL tanpa metadata identitas aktor, akun AI, atau lease task.
- **Alasan Penundaan ke Track 2:** Sesuai Master Prompt Bagian 0 dan 3, Track 1 dibatasi secara ketat hanya untuk perbaikan dan konsolidasi fondasi yang ada tanpa mengubah mekanisme inti (*Preserve ZLCE Core*). Perubahan skema matriks task, aturan konkurensi tim, dan integrasi multi-akun AI adalah tujuan utama perancangan Track 2 (Fase 3 & 4 pada P2-CP2 dan P2-CP3).
- **Alokasi:** **Ditunda ke Track 2 (Fitur Baru Mobile Workbench)**.

---

## MATRIKS RANGKUMAN USULAN REVISI TRACK 1

| ID | Judul Revisi | Berkas Terdampak | Prioritas | Track |
|:---|:---|:---|:---:|:---:|
| **REV-1** | Konsolidasi skrip CLI kanonik di `scripts/zlce/` | `scripts/*`, `scripts/zlce/*` | P1 | Track 1 |
| **REV-2** | Sinkronisasi dokumen prompt root & starter-kit | `LANDING_PROMPT.md`, `HANDOVER_PROMPT.md` | P2 | Track 1 |
| **REV-3** | UI Ekspor/Impor Markdown & status Local Cache | `src/components/SessionStateTab.tsx`, `Header.tsx` | P1 | Track 1 |
| **REV-4** | Pembersihan dependensi pasif & update `.env.example` | `package.json`, `.env.example`, `README.md` | P2 | Track 1 |
| **REV-5** | Test runner otomatis (`npm test`) & perbaikan badge | `package.json`, `README.md`, `scripts/run-tests.py` | P1 | Track 1 |
| **REV-6** | Perbaikan tautan repositori ke `ZLCE3.22-studio` | `README.md`, `docs/USAGE.md` | P2 | Track 1 |
| **REV-7** | Penyelarasan aturan validator Web vs CLI | `scripts/zlce/zlce-check.py`, `zlceValidator.ts` | P1 | Track 1 |
| **REV-8** | Dokumentasi keamanan dev-server | `README.md`, `package.json` | P3 | Track 1 |
| **REV-9** | Portabilitas POSIX `zlce-snapshot.sh` (Termux) | `scripts/zlce/zlce-snapshot.sh` | P1 | Track 1 |
| **REV-10**| Warning eksplisit Git failure & opsi `--tasks` | `scripts/zlce/zlce-check.py` | P1 | Track 1 |
| **REV-11**| `.gitattributes`, normalisasi LF, dan `chmod +x` | `.gitattributes`, `scripts/zlce/*` | P0 | Track 1 |
| **REV-12**| Multi-AI, Multi-Akun, Tim, dan Provenance | Ditunda ke P2-CP2/P2-CP3 | - | **Track 2** |

---

## STRATEGI EKSEKUSI P1-CP3 (BILA DISETUJUI)

1. **Pengaman Git:** Pekerjaan dilakukan di branch `main` dengan titik aman tag `baseline-3.22-asli`.
2. **Urutan Eksekusi Atomik:**
   - **Batch 1 (Fondasi Berkas & Portabilitas):** REV-11 (.gitattributes, LF, chmod +x) → REV-1 (Konsolidasi skrip) → REV-9 (Portabilitas bash snapshot).
   - **Batch 2 (Validator & Sinkronisasi):** REV-7 (Penyelarasan validator) → REV-10 (Git warning & path flexibility) → REV-2 (Sinkronisasi prompt kanonik).
   - **Batch 3 (Test Runner & CI):** REV-5 (Test suite otomatis `scripts/run-tests.py` & `npm test`).
   - **Batch 4 (Web App UI & Dependensi):** REV-3 (Ekspor/Impor Markdown & status cache) → REV-4 (Pembersihan dependensi pasif).
   - **Batch 5 (Dokumentasi & Tautan):** REV-6 (Perbaikan tautan repo) → REV-8 (Dokumentasi dev-server).
3. **Setiap commit** memuat ID REV terkait dan hanya di-commit setelah `compile_applet` dan `npm test` lulus.

---

## STATUS CHECKPOINT P1-CP2

Rencana revisi telah selesai disusun di `docs/revisi/REVISION_PLAN.md`.

**BERHENTI. Menunggu persetujuan pengguna:**  
Silakan tentukan paket revisi yang disetujui, misalnya:  
- *"Setujui P1-CP2 (REV-1 s/d REV-11)"*, atau  
- Sebutkan daftar REV tertentu yang ingin dimasukkan/dikecualikan.
