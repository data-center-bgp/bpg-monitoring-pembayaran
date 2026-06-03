PRODUCT REQUIREMENTS DOCUMENT
Sistem Pengajuan Pembayaran Digital

Produk SiPay — Sistem Pengajuan Pembayaran
Perusahaan Barokah Perkasa Group
Versi Dokumen 1.0
Tanggal 2 Juni 2026
Tech Stack React + Vite (Frontend), Supabase (Backend)
Status Draft

 

1. Ringkasan Eksekutif
   Dokumen ini merupakan Product Requirements Document (PRD) untuk sistem pengajuan pembayaran berbasis web yang akan digunakan secara internal oleh Barokah Perkasa Group dan entitas anak perusahaannya. Sistem ini menggantikan proses manual berbasis spreadsheet Excel yang saat ini digunakan, dengan tujuan meningkatkan akurasi data, mempercepat proses approval, dan menyediakan visibilitas real-time terhadap status pembayaran.

Nama Produk SiPay — Sistem Pengajuan Pembayaran

Tujuan Utama Digitalisasi alur pengajuan dan persetujuan pembayaran vendor untuk 3 entitas perusahaan: Barokah Gemilang Perkasa (BGP), Armada Samudra Global (ASG), dan Bahtera Nusantara Internasional (BNI)

Pengguna Utama Staff operasional (pengaju), Finance (approver pembayaran), Admin Sistem (manajemen akun & master data)

2. Latar Belakang & Permasalahan
   2.1 Kondisi Saat Ini
   Proses pengajuan pembayaran saat ini dilakukan menggunakan file Microsoft Excel dengan sheet Report yang memuat hingga ratusan baris data. Proses ini memiliki beberapa keterbatasan mendasar:
   • Kolom kode (Kode Perusahaan, Kode Nomor Form) dibuat secara manual menggunakan formula VLOOKUP untuk keperluan sinkronisasi antar-sheet, yang rentan terhadap kesalahan input
   • Tidak ada mekanisme approval yang terstruktur dan terekam secara sistem
   • Tidak ada notifikasi otomatis ketika pengajuan berpindah status
   • Akses bersamaan ke file Excel menyebabkan risiko konflik data dan overwrite
   • Tidak ada dashboard monitoring terpusat untuk memantau status pembayaran secara real-time
   • Pencarian dan filtering data historis yang terbatas dan lambat

2.2 Struktur Data Eksisting (dari File Excel)
Berdasarkan analisis file Excel Barokah*Perkasa_Group*-\_Pengajuan_Pembayaran.xlsx, sistem saat ini memiliki struktur sebagai berikut:

Sheet Konten Jumlah Data
Vessel Code Master data kapal: nama, kode 4 huruf, perusahaan (BGP/ASG/BNI), dan fleet (I–VII) 103 kapal
Department Master data departemen perusahaan 31 departemen
Report Data pengajuan pembayaran dengan 23 kolom termasuk header form, line item, dan timeline approval Ratusan record

2.3 Pola Kode Form (Logika yang Akan Diotomasi)
Kode Nomor Form saat ini dibuat manual dengan pola berikut:

FP--[DDMMYY]-[NomorForm]-[NomorItem]
Contoh: FP--280126-001-002 → Form pertama tgl 28-Jan-2026, item ke-2

Pada sistem baru, kode ini akan digenerate otomatis oleh backend sesuai pola yang sama, sehingga kolom merah di spreadsheet (yang sebelumnya diisi formula) tidak lagi diperlukan secara manual.

3. Tujuan & Sasaran Produk

# Tujuan Metrik Keberhasilan

1 Eliminasi proses manual berbasis Excel Zero penggunaan Excel untuk pencatatan baru setelah go-live
2 Mempercepat proses approval pembayaran Waktu rata-rata approval turun ≤30% dibanding baseline Excel
3 Meningkatkan akurasi & integritas data Nol kesalahan kode form / kode kapal akibat input manual
4 Visibilitas real-time status pembayaran Finance dapat melihat semua pengajuan pending dalam 1 dashboard
5 Audit trail yang lengkap 100% perubahan status tercatat dengan timestamp & user ID
6 Manajemen akun terpusat Admin dapat buat/nonaktifkan akun tanpa menu self-register

4. Pengguna & Peran (User Roles)
   Sistem menggunakan 4 peran utama. Semua akun dibuat oleh Admin — tidak tersedia menu registrasi mandiri.

Peran Deskripsi Akses Utama
Admin Sistem Mengelola akun pengguna dan master data (kapal, departemen, perusahaan, kode budget) Manajemen user, CRUD master data, akses semua laporan
Staff / Pengaju Membuat dan mengajukan form pengajuan pembayaran Buat form baru, lihat status form milik sendiri/departemennya, upload attachment
Finance Mereview pengajuan yang masuk, konfirmasi penerimaan dokumen, dan tandai pembayaran selesai Lihat semua pengajuan, update status (terima / bayar), filter & export data
Viewer / Monitor Hanya dapat melihat dashboard dan laporan, tidak dapat melakukan aksi apapun Read-only dashboard & laporan

5. Fitur & Persyaratan Fungsional
   5.1 Autentikasi & Manajemen Akun
   • Login menggunakan email + password melalui Supabase Auth
   • Tidak ada halaman Register — akun dibuat oleh Admin Sistem
   • Password reset via email link (fitur Supabase bawaan)
   • Session token JWT dikelola secara otomatis oleh Supabase
   • Admin dapat mengaktifkan / menonaktifkan akun tanpa menghapus data historis
   • Admin dapat mengubah role pengguna kapan saja

5.2 Manajemen Master Data (khusus Admin)
Admin dapat melakukan CRUD untuk semua data referensi:

Master Data Field Utama Catatan
Perusahaan (Companies) Nama, Kode (BGP/ASG/BNI) Digunakan sebagai entitas pemilik form
Kapal (Vessels) Nama Kapal, Kode 4 Huruf, Perusahaan, Fleet (I–VII) 103 data awal diimport dari Excel; status aktif/nonaktif
Departemen Nama Departemen 31 data awal dari Excel; dapat ditambah/diedit
Vendor Nama Vendor Dapat ditambah langsung saat membuat form pengajuan (ad-hoc)
Kode Budget Kode, Deskripsi Contoh: 01.5-3830.002; dapat dikelola oleh Admin
Pengguna Nama, Email, Role, Departemen, Status Admin tidak dapat hapus akun — hanya nonaktifkan

5.3 Form Pengajuan Pembayaran
5.3.1 Header Form (1 form = 1 vendor per hari)
Field Tipe Data Wajib? Keterangan
Tanggal Invoice Diterima dari Vendor Date Tidak Boleh kosong jika tidak ada invoice fisik
Tanggal Pembuatan Pengajuan Date Ya Default: hari ini, dapat diubah oleh pengaju
Nama Vendor Dropdown + Input bebas Ya Pilih dari master atau ketik vendor baru
Nama Perusahaan Dropdown Ya BGP / ASG / BNI — dari master companies
Departemen Dropdown Ya Dari master departemen sesuai perusahaan
PIC Yang Mengajukan Text Ya Nama lengkap penanggung jawab form
Nomor Form Ke- Integer (auto) Auto Urutan form pada hari yang sama, auto-increment per hari
Kode Nomor Form Text (auto-generate) Auto Format: FP--[DDMMYY]-[NomorForm]; dihasilkan sistem

5.3.2 Line Item Pengajuan (1 form dapat memiliki banyak item)
Field Tipe Data Wajib? Keterangan
Nomor Item Integer (auto) Auto Urutan item dalam form, mulai dari 1
Kode Item Text (auto-generate) Auto Format: FP--[DDMMYY]-[NomorForm]-[NomorItem]
Uraian Text area Ya Deskripsi pekerjaan / barang yang dibayarkan
Qty Decimal Ya Jumlah satuan
Harga Satuan Currency (IDR) Ya Harga per satuan dalam Rupiah
Total Currency (auto) Auto Qty × Harga Satuan, dihitung otomatis
Nama Kapal Dropdown Tidak Dari master vessels, difilter sesuai perusahaan yang dipilih
Fleet Text (auto-fill) Auto Terisi otomatis dari data kapal; dapat dikosongkan
Kode Budget Dropdown Tidak Dari master kode budget
Keterangan Text Tidak Catatan tambahan
Nomor Invoice Text Tidak Nomor invoice dari vendor (jika ada)

5.4 Alur Status Pengajuan (Approval Flow)
Setiap form pengajuan memiliki 5 status yang berjalan secara berurutan:

Status Aktor Trigger Aksi Timestamp yang Dicatat
DRAFT Staff Form dibuat, belum disubmit created_at
SUBMITTED Staff Staff klik tombol "Ajukan ke Finance" submitted_to_finance_at
RECEIVED Finance Finance klik "Konfirmasi Dokumen Diterima" received_by_finance_at
PAID Finance Finance klik "Tandai Sudah Dibayar" paid_at + paid_by (user ID)
REJECTED Finance Finance klik "Kembalikan ke Pengaju" dengan alasan rejected_at + rejection_reason

Catatan penting: Status REJECTED memungkinkan Staff untuk mengedit dan re-submit form. Setiap perubahan status dicatat di tabel audit log beserta user ID dan timestamp.

5.5 Dashboard Monitoring
Kartu KPI Utama (tampil di bagian atas dashboard)
• Total Pengajuan Hari Ini
• Menunggu Konfirmasi Finance (status SUBMITTED)
• Sudah Diterima Finance, Belum Dibayar (status RECEIVED)
• Dibayar Bulan Ini (status PAID)
• Total Nilai Pengajuan Pending (dalam Rupiah)
• Total Nilai Dibayar Bulan Ini

Tabel Daftar Pengajuan
• Tampilkan semua form dengan kolom: Kode Form, Tanggal, Vendor, Perusahaan, Departemen, PIC, Total Nilai, Status, Aksi
• Filter: Perusahaan, Departemen, Status, Rentang Tanggal, Kapal, Fleet
• Search: Pencarian bebas berdasarkan nama vendor, kode form, atau uraian
• Sorting: Semua kolom dapat diurutkan ascending/descending
• Pagination: 25/50/100 baris per halaman

Chart & Visualisasi
• Bar chart: Jumlah pengajuan per departemen (bulan ini)
• Line chart: Tren nilai pembayaran 6 bulan terakhir
• Pie chart: Distribusi pengajuan per perusahaan (BGP/ASG/BNI)
• Tabel: Top 10 vendor berdasarkan total nilai pengajuan

5.6 Fitur Export & Laporan
• Export tabel ke format Excel (.xlsx) dengan filter yang aktif
• Export tabel ke format CSV
• Laporan rekap bulanan: total nilai per departemen, per perusahaan, per status
• Filter export: rentang tanggal, perusahaan, departemen, status

5.7 Lampiran Invoice (File Upload)
Finance membutuhkan kemampuan verifikasi dokumen fisik sebelum memproses pembayaran. Lampiran dikelola di level form — bukan per item — karena dalam praktiknya 1 invoice dari vendor seringkali mencakup lebih dari 1 item dalam form yang sama. Mengelola lampiran per item akan memaksa Staff mengupload file yang sama berulang kali, dan tidak sejalan dengan cara Finance mereview pengajuan secara keseluruhan.

Karena 1 form bisa memiliki item-item dengan nomor invoice berbeda, sistem mendukung multiple lampiran per form. Setiap lampiran memiliki field label opsional agar Finance dapat mencocokkan dokumen fisik dengan baris item yang relevan (contoh: label "Invoice 5989" merujuk ke item yang mencantumkan Inv 5989 di kolom Nomor Invoice).

5.7.1 Ketentuan Upload (sisi Staff)
• Lampiran diupload di level form, bukan per item
• Satu form dapat memiliki beberapa lampiran (maks 10 file per form) untuk mengakomodasi form yang itemnya memiliki nomor invoice berbeda-beda
• Setiap lampiran memiliki field Label (opsional, maks 100 karakter) — contoh: "Invoice 5989", "Invoice 5990 dan 5991", "Kuitansi Pengiriman"
• Format file yang diizinkan: PDF, JPG, JPEG, PNG
• Batas ukuran: maksimal 10 MB per file
• Upload dapat dilakukan saat membuat form baru (status DRAFT) maupun saat mengedit (status REJECTED)
• Staff dapat menghapus lampiran selama form masih berstatus DRAFT atau REJECTED
• Setelah form disubmit (status SUBMITTED ke atas), lampiran terkunci — tidak dapat dihapus oleh Staff, hanya oleh Admin

5.7.2 Akses Lampiran (sisi Finance)
• Finance melihat semua lampiran di bagian bawah halaman Detail Form, dalam satu panel "Lampiran Invoice"
• Setiap lampiran menampilkan: label (jika ada), nama file asli, ukuran file, tanggal upload, nama pengupload
• Finance dapat mengunduh atau membuka file di tab baru langsung dari browser via signed URL
• Finance dapat mencocokkan lampiran dengan item melalui label di lampiran dan kolom Nomor Invoice di baris item yang bersangkutan
• Indikator visual di daftar pengajuan: form yang belum memiliki lampiran apapun ditandai ikon peringatan agar Finance tahu perlu konfirmasi dokumen tambahan
• Finance tidak dapat menghapus atau mengedit lampiran — hanya melihat dan mengunduh

5.7.3 Penyimpanan File
Aspek Detail
Storage Provider Supabase Storage (bucket: invoice-attachments)
Struktur Path {company_code}/{YYYY}/{MM}/{form_id}/{uuid}.{ext}
Akses URL Signed URL dengan expiry 1 jam — tidak dapat dibuka tanpa autentikasi aktif
Visibilitas Bucket Private — tidak dapat diakses publik tanpa signed URL
Nama File Simpan UUID + ekstensi asli (misal: a1b2c3d4.pdf) untuk menghindari konflik nama antar upload
Nama File Tampil Nama file asli saat diupload, disimpan di kolom original_name di tabel attachments
Label Field teks opsional per lampiran untuk memudahkan Finance mencocokkan dengan nomor invoice di baris item

6. Arsitektur Sistem & Tech Stack
   6.1 Tech Stack

Layer Teknologi Justifikasi
Frontend React 18 + Vite Build tool modern, HMR cepat, ekosistem besar
UI Library shadcn/ui + Tailwind CSS Komponen siap pakai, desain konsisten, highly customizable
State Management Zustand atau React Query (TanStack Query) React Query untuk server state & caching, Zustand untuk UI state
Backend Supabase PostgreSQL managed, Auth built-in, Row Level Security, Realtime
Database PostgreSQL (via Supabase) Relational DB yang cocok untuk data terstruktur pengajuan pembayaran
Authentication Supabase Auth JWT, session management, password reset email built-in
File Storage Supabase Storage Penyimpanan lampiran invoice (PDF/JPG/PNG); bucket private dengan Signed URL
File Upload (UI) react-dropzone Komponen drag-and-drop untuk upload lampiran, validasi tipe & ukuran file di sisi client
Hosting Frontend Vercel atau Cloudflare Pages Deploy otomatis dari Git, CDN global

6.2 Skema Database (SQL Tables)
Berikut adalah rancangan tabel database yang menggantikan struktur sheet Excel. Kolom-kolom merah di Excel (yang dulunya formula sync) kini menjadi relasi antar tabel dan field auto-generate:

Tabel 1: companies
Kolom Tipe Constraint Keterangan
id UUID PK, DEFAULT gen_random_uuid() Primary key
name TEXT NOT NULL Nama lengkap: Barokah Gemilang Perkasa, dst.
code VARCHAR(10) NOT NULL, UNIQUE Kode singkat: BGP, ASG, BNI
is_active BOOLEAN DEFAULT true Status aktif perusahaan
created_at TIMESTAMPTZ DEFAULT NOW() Waktu dibuat

Tabel 2: vessels (menggantikan sheet "Vessel Code")
Kolom Tipe Constraint Keterangan
id UUID PK
name TEXT NOT NULL Nama kapal, misal: TB. BB 99
code VARCHAR(10) UNIQUE Kode 4 huruf: BB99, ELNU, dst.
company_id UUID FK → companies.id Relasi ke perusahaan pemilik kapal
fleet VARCHAR(10) Nomor fleet: I, II, III, … VII
is_active BOOLEAN DEFAULT true

Tabel 3: departments
Kolom Tipe Constraint Keterangan
id UUID PK
name TEXT NOT NULL, UNIQUE Nama departemen

Tabel 4: vendors
Kolom Tipe Constraint Keterangan
id UUID PK
name TEXT NOT NULL Nama vendor/supplier
created_at TIMESTAMPTZ DEFAULT NOW()

Tabel 5: budget_codes
Kolom Tipe Constraint Keterangan
id UUID PK
code VARCHAR(50) NOT NULL, UNIQUE Contoh: 01.5-3830.002
description TEXT Deskripsi kode budget

Tabel 6: user_profiles (extends Supabase Auth users)
Kolom Tipe Constraint Keterangan
id UUID PK, FK → auth.users.id Sama dengan Supabase Auth user ID
full_name TEXT NOT NULL
role TEXT CHECK IN ('admin','staff','finance','viewer')
department_id UUID FK → departments.id Departemen pengguna (opsional untuk admin)
company_id UUID FK → companies.id Perusahaan terkait (opsional)
is_active BOOLEAN DEFAULT true Nonaktif jika akun diblokir admin

Tabel 7: payment_forms (header pengajuan)
Kolom Tipe Constraint Keterangan
id UUID PK
form_code VARCHAR(30) UNIQUE, NOT NULL Auto-generate: FP--DDMMYY-XXX
form_number INTEGER NOT NULL Urutan form per hari (reset setiap hari)
invoice_date DATE Tanggal Invoice Diterima dari Vendor
submission_date DATE NOT NULL Tanggal Pembuatan Pengajuan
vendor_id UUID FK → vendors.id Relasi ke master vendor
vendor_name_raw TEXT Nama vendor jika diketik manual (ad-hoc)
company_id UUID FK → companies.id, NOT NULL Perusahaan pengaju
department_id UUID FK → departments.id, NOT NULL
pic_name TEXT NOT NULL Nama PIC yang mengajukan
created_by UUID FK → auth.users.id User yang membuat form
status TEXT DEFAULT 'draft' draft | submitted | received | paid | rejected
submitted_to_finance_at TIMESTAMPTZ Tanggal Diajukan ke Finance
received_by_finance_at TIMESTAMPTZ Tanggal Pengajuan Diterima Finance
paid_at TIMESTAMPTZ Tanggal Pembayaran oleh Finance
paid_by UUID FK → auth.users.id User Finance yang menandai bayar
rejected_at TIMESTAMPTZ
rejection_reason TEXT Alasan penolakan dari Finance
created_at TIMESTAMPTZ DEFAULT NOW()
updated_at TIMESTAMPTZ DEFAULT NOW()

Tabel 8: payment_items (line item pengajuan)
Kolom Tipe Constraint Keterangan
id UUID PK
form_id UUID FK → payment_forms.id, NOT NULL Relasi ke header form
item_number INTEGER NOT NULL Urutan item dalam form (1, 2, 3, …)
item_code VARCHAR(40) UNIQUE Auto-generate: FP--DDMMYY-XXX-YYY
description TEXT NOT NULL Uraian pekerjaan/barang
qty DECIMAL(12,2) NOT NULL
unit_price BIGINT NOT NULL Harga satuan dalam Rupiah (tanpa desimal)
total BIGINT GENERATED AS (qty \* unit_price) Dihitung otomatis oleh database
vessel_id UUID FK → vessels.id Kapal terkait (opsional)
fleet VARCHAR(10) Auto-fill dari vessel, dapat di-override
budget_code_id UUID FK → budget_codes.id
notes TEXT Keterangan tambahan
invoice_number TEXT Nomor Invoice vendor
created_at TIMESTAMPTZ DEFAULT NOW()

Tabel 9: audit_logs (riwayat perubahan status)
Kolom Tipe Keterangan
id UUID PK
form_id UUID FK Form yang berubah
actor_id UUID FK User yang melakukan aksi
action TEXT Contoh: 'status_changed', 'form_edited', 'attachment_uploaded'
from_status TEXT Status sebelumnya
to_status TEXT Status setelahnya
notes TEXT Catatan tambahan (misal: alasan reject)
created_at TIMESTAMPTZ Timestamp aksi

Tabel 10: attachments (lampiran invoice per form)
Lampiran dikelola di level form. Field label memungkinkan Finance mencocokkan dokumen dengan item tertentu melalui kolom Nomor Invoice di payment_items.

Kolom Tipe Constraint Keterangan
id UUID PK, DEFAULT gen_random_uuid()
form_id UUID FK → payment_forms.id, NOT NULL Form induk lampiran; satu form dapat memiliki banyak lampiran
label VARCHAR(100) NULL Label opsional untuk identifikasi, misal: "Invoice 5989", "Kuitansi Pengiriman"
storage_path TEXT NOT NULL Path di Supabase Storage: {company_code}/{YYYY}/{MM}/{form_id}/{uuid}.{ext}
original_name TEXT NOT NULL Nama file asli saat diupload (untuk ditampilkan ke user)
mime_type VARCHAR(100) NOT NULL Contoh: 'application/pdf', 'image/jpeg', 'image/png'
file_size_bytes INTEGER NOT NULL Ukuran file dalam bytes; validasi maks 10MB (10.485.760 bytes) di sisi backend
uploaded_by UUID FK → auth.users.id, NOT NULL User yang mengupload
created_at TIMESTAMPTZ DEFAULT NOW() Waktu upload

7. Keamanan & Row Level Security (RLS)
   Supabase mendukung Row Level Security (RLS) di level PostgreSQL. Kebijakan RLS yang akan diterapkan:

Tabel / Bucket Role Hak Akses
payment_forms staff SELECT (milik sendiri / dept sendiri), INSERT, UPDATE (hanya status draft/rejected)
payment_forms finance SELECT (semua), UPDATE (hanya kolom status & timestamp approval)
payment_forms admin SELECT ALL, INSERT, UPDATE, DELETE
payment_forms viewer SELECT (semua, read-only)
payment_items staff CRUD hanya untuk item milik form sendiri
payment_items finance/viewer SELECT only
attachments (tabel) staff INSERT (form milik sendiri, status draft/rejected), SELECT (form milik sendiri), DELETE (hanya jika form masih draft/rejected)
attachments (tabel) finance SELECT ALL (baca & unduh lampiran semua form untuk verifikasi), tidak bisa INSERT/DELETE
attachments (tabel) admin CRUD penuh termasuk DELETE lampiran dari form yang sudah paid
attachments (tabel) viewer SELECT only
invoice-attachments (Storage bucket) staff INSERT ke path form milik sendiri; SELECT via signed URL untuk form milik sendiri
invoice-attachments (Storage bucket) finance SELECT via signed URL untuk semua file di semua form (untuk verifikasi pembayaran)
invoice-attachments (Storage bucket) admin INSERT, SELECT, DELETE (full access)
user_profiles admin CRUD penuh
user_profiles semua role SELECT (hanya profil sendiri)
audit_logs semua role SELECT (bisa lihat, tidak bisa edit/hapus)

8. Persyaratan Non-Fungsional
   Kategori Persyaratan
   Performa Halaman dashboard load < 3 detik; response API < 500ms untuk query umum
   Skalabilitas Mampu menangani 200+ pengguna bersamaan dengan Supabase free/pro tier
   Ketersediaan Uptime target 99.5% (tidak termasuk maintenance terencana)
   Keamanan HTTPS wajib; token JWT expire 1 jam; refresh token disimpan httpOnly cookie
   Responsivitas UI responsif untuk desktop (1280px+) dan tablet (768px+); mobile read-only
   Browser Support: Chrome 90+, Firefox 90+, Edge 90+, Safari 14+
   Data Retention Data historis disimpan permanen; tidak ada hard-delete untuk form yang sudah paid
   Bahasa Antarmuka Bahasa Indonesia; format tanggal DD/MM/YYYY; format mata uang Rp X.XXX

9. Peta Halaman & Navigasi
   Halaman URL Akses Role Deskripsi Singkat
   Login /login Semua (publik) Form email + password, tombol lupa password
   Dashboard / Semua (terautentikasi) KPI cards, chart, daftar pengajuan terbaru
   Daftar Pengajuan /pengajuan Semua Tabel semua form + filter + search + export
   Detail Pengajuan /pengajuan/:id Semua Detail form, timeline status, aksi approval (Finance)
   Buat Pengajuan Baru /pengajuan/baru Staff, Admin Form multi-step: header + tambah item
   Edit Pengajuan /pengajuan/:id/edit Staff (hanya draft/rejected), Admin Edit form yang belum disubmit atau dikembalikan
   Laporan /laporan Finance, Admin, Viewer Rekap bulanan, export, filter lanjutan
   Master Data — Kapal /master/kapal Admin CRUD master vessels
   Master Data — Dept /master/departemen Admin CRUD master departemen
   Master Data — Vendor /master/vendor Admin CRUD master vendor
   Master Data — Budget /master/budget Admin CRUD kode budget
   Manajemen User /admin/users Admin Buat akun, ubah role, aktif/nonaktifkan
   Profil Saya /profil Semua Lihat profil, ubah password

10. Rencana Migrasi Data dari Excel
    10.1 Data yang Akan Dimigrasikan
    • Master Kapal: 103 record dari sheet "Vessel Code" → tabel vessels + companies
    • Master Departemen: 31 record dari sheet "department" → tabel departments
    • Data Pengajuan Historis: Record aktif dari sheet "Report" → payment_forms + payment_items

10.2 Pendekatan Migrasi
• Buat script Python/Node.js untuk parse Excel dan insert ke Supabase via API atau SQL
• Mapping kolom Excel ke kolom SQL sesuai tabel di Bab 6.2
• Validasi: cek duplikasi kode form, normalisasi nama vendor yang tidak konsisten
• Kolom merah (Kode Perusahaan, Kode Nomor Form) di Excel: tidak diimport langsung, melainkan di-resolve menjadi relasi FK dan kode auto-generate
• Data historis diimport dengan status PAID (jika kolom Tanggal Pembayaran terisi) atau SUBMITTED/RECEIVED sesuai kondisi kolomnya

10.3 Validasi Pasca-Migrasi
• Verifikasi jumlah record: total baris Excel = total record di database
• Cek konsistensi nilai total per perusahaan antara Excel dan database
• Review sampling 10% record secara manual

11. Asumsi & Risiko
    11.1 Asumsi
    • Semua pengguna memiliki akses internet dan perangkat desktop/laptop
    • Admin sistem memiliki akses ke Supabase dashboard untuk pembuatan akun awal
    • Data master (kapal, departemen) relatif stabil dan tidak berubah drastis
    • Tidak diperlukan integrasi dengan sistem ERP atau akuntansi eksternal pada fase pertama

11.2 Risiko & Mitigasi
Risiko Dampak Probabilitas Mitigasi
Data historis Excel tidak konsisten (nama vendor tidak standar) Tinggi Tinggi Cleaning data manual sebelum migrasi; buat master vendor dulu
User resistance terhadap sistem baru Sedang Sedang Training singkat + panduan penggunaan (user guide)
Koneksi internet tidak stabil di lokasi kapal Sedang Rendah Tampilkan pesan offline yang jelas; data draft dapat disimpan lokal sementara (localStorage)
Supabase free tier limit (500MB DB, 1GB storage) Rendah Rendah Monitor penggunaan; upgrade ke Pro tier jika diperlukan (estimasi ~$25/bulan)

12. Rencana Pengembangan (Milestones)
    Fase Durasi (Estimasi) Deliverable
    Fase 0: Setup 1 minggu Setup Supabase project, Vite+React boilerplate, konfigurasi RLS, import master data
    Fase 1: Auth & Master Data 1–2 minggu Halaman login, manajemen user oleh admin, CRUD semua master data
    Fase 2: Form Pengajuan 2–3 minggu Buat/edit/hapus form pengajuan, auto-generate kode form, multi-item per form
    Fase 3: Approval Flow 1–2 minggu Workflow status (draft→submitted→received→paid/rejected), audit log, notifikasi in-app
    Fase 4: Dashboard & Laporan 1–2 minggu KPI cards, chart, filter, export Excel/CSV
    Fase 5: Migrasi & UAT 1–2 minggu Migrasi data historis, user acceptance testing, bug fixing, go-live

Total estimasi: 7–12 minggu (tergantung ketersediaan developer dan kompleksitas migrasi data)

13. Pertanyaan Terbuka (Open Issues)
    Poin-poin berikut perlu dikonfirmasi sebelum development dimulai:

# Pertanyaan Pemilik Jawaban

1 Apakah ada kebutuhan notifikasi email otomatis saat status form berubah (misal: email ke Finance ketika ada pengajuan baru masuk)? Finance / IT
2 Apakah format kode form FP--DDMMYY-XXX sudah final, atau perlu diubah? Finance
3 Apakah Finance perlu melihat pengajuan dari semua perusahaan (BGP+ASG+BNI) dalam satu tampilan, atau terpisah per perusahaan? Finance Manager
4 Apakah data pengajuan historis di Excel perlu diimport semua, atau hanya data 1–2 tahun terakhir saja? Admin Sistem
5 Siapa yang bertanggung jawab sebagai Admin Sistem pertama dan melakukan onboarding pengguna? IT / HR
6 Apakah ada batas maksimal total ukuran lampiran per form yang perlu disesuaikan (saat ini dirancang 5 file × 10MB per item)? Finance / Admin

 
— Akhir Dokumen —
PRD ini bersifat living document dan akan diperbarui seiring perkembangan proyek.
