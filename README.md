# 🤖 Enterprise AI Attendance Agent & Analytics System (n8n + Gemini AI)

Sistem ekosistem absensi karyawan harian berbasis AI Agent interaktif dan generator laporan analitik mingguan otomatis untuk HRD yang berjalan secara simultan dalam satu kanvas *workflow*. Sistem ini dibangun menggunakan arsitektur *low-code* tangguh, hemat biaya token API, serta dilengkapi penanganan eror tingkat produksi (*production-ready error handling*).

---

## 1. Problem Statement (Masalah Bisnis)
Banyak perusahaan menghadapi kendala dalam mengelola absensi karyawan lapangan atau yang memiliki mobilitas tinggi. Aplikasi absensi tradisional sering kali dirasa berat, kaku, dan membutuhkan proses adaptasi yang lama bagi karyawan. 

Di sisi lain, Tim HRD kehilangan banyak waktu produktif setiap akhir pekan hanya untuk merekap data mentah dari Google Sheets secara manual guna mencari tren kehadiran, karyawan yang sering izin, atau kendala operasional perusahaan.

### Solusi yang Dibangun:
Sistem otomatisasi cerdas berbasis *Low-Code AI* yang menyelesaikan dua masalah sekaligus:
1. **Sisi Karyawan (Harian):** Absensi instan via Telegram menggunakan bahasa sehari-hari yang diekstraksi otomatis secara valid oleh AI.
2. **Sisi HRD (Mingguan):** Sistem analitik berbasis waktu (*cron-job*) yang otomatis menarik, mengelompokkan, merangkum, dan mengirimkan laporan *Auto-Summary* langsung ke Telegram HRD setiap Jumat sore.

---

## 2. System Architecture (Arsitektur Sistem)

Seluruh sistem ini berjalan terintegrasi di dalam satu *workflow* n8n tunggal (*Monolith Canvas*) yang terbagi menjadi dua fungsi utama:

### Alur 1: Registrasi & Absensi Harian (Interactive AI Agent)
Alur ini menangani interaksi langsung dengan karyawan secara *real-time*, lengkap dengan validasi alasan izin otomatis:

```text
[ Trigger: Telegram Chat ] ──> [ GSheets: Cek Akses Karyawan ]
                                              │
                     ┌────────────────────────┴────────────────────────┐
                     ▼ (Belum Terdaftar)                               ▼ (Terdaftar)
       [ Telegram: Notif Akses Ditolak ]                         [ Switch: Cek Menu ]
                                                                       │
                                                       ┌───────────────┴───────────────┐
                                                       ▼ (Menu Absen)                  ▼ (Menu Tanya HR)
                                         [ Gemini: Ekstraktor Absen ]         [ Gemini: Agent Konsultasi HR ]
                                                       │                               │
                                         [ Switch: Cek Validitas Data ]       [ Telegram: Balasan Agent Chat ]
                                                       │
                     ┌─────────────────────────────────┴─────────────────────────────────┐
                     ▼ (Data Tidak Valid / Alasan Izin Tidak Masuk Akal)                 ▼ (Data Valid & Masuk Akal)
       [ Telegram: Gagal Ekstraksi AI ]                                     [ GSheets: Catat Kehadiran ]
       (Menyuruh isi alasan yang valid)                                                  │
                                                                            [ Switch: Cek Status Masuk ]
                                                                             ┌───────────┴───────────┐
                                                                             ▼ (Hadir)               ▼ (Izin)
                                                                 [ Telegram: Sukses Catat Masuk ]  [ Telegram: Sukses Catat Izin ]
