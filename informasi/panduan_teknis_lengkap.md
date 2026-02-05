
# Spesifikasi Teknis Pengembangan Kampus Digital Enterprise

Dokumen ini berisi daftar kebutuhan teknis (Technical Requirements) untuk meningkatkan aplikasi Kampus Digital menjadi sistem terintegrasi penuh (ERP).

---

## 1. Integrasi PDDikti Neo Feeder
**Tujuan:** Sinkronisasi data otomatis antara Kampus Digital dan Server PDDikti Kemdikbud.

### Kebutuhan Infrastruktur
1.  **Server Middleware (Backend API)**
    *   **Bahasa:** Node.js (Express/NestJS) atau PHP (Laravel).
    *   **Fungsi:** Menjembatani komunikasi Frontend React dengan Neo Feeder.
    *   **Alasan:** Browser (React) tidak bisa mengakses Feeder langsung karena protokol keamanan browser (CORS) dan untuk melindungi kredensial Feeder.
2.  **Koneksi Jaringan (VPN/IP Static)**
    *   Jika Feeder ada di jaringan lokal kampus, server Middleware harus berada di jaringan yang sama atau terhubung via VPN (Mikrotik/OpenVPN).

### Kebutuhan Data (Mapping)
Tabel di database lokal harus memiliki kolom `id_feeder` (UUID) untuk menyimpan ID balasan dari PDDikti.

| Tabel Lokal | Kolom Tambahan | Keterangan |
| :--- | :--- | :--- |
| `mahasiswa` | `id_pd` (UUID) | ID unik mahasiswa dari Feeder |
| `dosen` | `id_ptk` (UUID) | ID unik dosen dari Feeder |
| `matakuliah` | `id_mk` (UUID) | ID unik matakuliah dari Feeder |
| `kelas` | `id_kls` (UUID) | ID unik kelas perkuliahan |
| `nilai` | `id_nilai` (UUID) | ID unik riwayat nilai |

### Alur Kerja (Workflow)
1.  **Get Token:** Middleware login ke Feeder menggunakan Username & Password WS.
2.  **Push Data:** Middleware mengirim data mahasiswa baru (format JSON) ke Feeder.
3.  **Store UUID:** Feeder membalas dengan UUID. Middleware menyimpan UUID ke database lokal.
4.  **Sync Nilai:** Middleware mengirim rekap nilai per semester menggunakan UUID mahasiswa dan UUID kelas.

---

## 2. Integrasi Pembayaran (Payment Gateway)
**Tujuan:** Pembayaran UKT otomatis, Virtual Account (VA), dan QRIS.

### Kebutuhan Administratif
*   Akta Pendirian Yayasan/PT.
*   NPWP Badan & NIB (Nomor Induk Berusaha).
*   KTP Pimpinan/Direktur.
*   Rekening Koran Perusahaan.

### Kebutuhan Teknis
1.  **Provider:** Midtrans, Xendit, atau Duitku.
2.  **Backend Webhook Handler:**
    *   Endpoint URL (misal: `https://api.kampus.ac.id/webhook/payment`).
    *   Logic untuk update status di tabel `transaksi` dan `tagihan` menjadi 'LUNAS' saat menerima sinyal dari Bank.
3.  **Security:** Signature Key validation untuk memastikan request benar-benar dari Bank.

---

## 3. Learning Management System (LMS) & CBT
**Tujuan:** E-Learning, Upload Tugas, dan Ujian Online.

### Kebutuhan Storage
File materi (PDF/PPT) dan tugas mahasiswa tidak boleh disimpan di Database.
1.  **Object Storage:** Cloudflare R2, AWS S3, atau MinIO (Self-hosted).
2.  **CDN (Content Delivery Network):** Agar file materi bisa diakses cepat oleh ribuan mahasiswa sekaligus.

### Kebutuhan CBT (Ujian)
1.  **Bank Soal Database:** Struktur tabel khusus untuk menyimpan Soal, Opsi Jawaban, dan Kunci Jawaban.
2.  **Session Manager:** Mengatur durasi ujian (Timer server-side) agar mahasiswa tidak bisa memanipulasi waktu lokal laptop.
3.  **Anti-Cheat (Basic):** Fitur full-screen browser detector atau disable right-click (Frontend).

---

## 4. Roadmap Migrasi Database
Saat ini aplikasi menggunakan JSON (`public/data/database`). Untuk fitur di atas, wajib migrasi ke SQL.

**Rekomendasi Stack:**
*   **Database:** PostgreSQL (lebih baik untuk data kompleks) atau MySQL.
*   **ORM:** Prisma (Node.js) atau Eloquent (Laravel).

**Tahapan:**
1.  Buat skema Database SQL (Tabel Mahasiswa, KRS, Nilai, dll).
2.  Import data dari file JSON yang ada ke Database SQL.
3.  Ubah kode React `useEffect` (fetch JSON) menjadi pemanggilan API ke Backend Server.
