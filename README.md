# Brief Tugas: Sistem Manajemen Rumah Sakit (Laravel & Eloquent ORM)

## Deskripsi Tugas
Dalam tugas ini, Anda diminta untuk merancang dan membangun **Sistem Informasi Manajemen Rumah Sakit (SIMRS)** berbasis web menggunakan framework Laravel. Penekanan utama pada penugasan ini adalah penerapan **Eloquent ORM** untuk mengelola skema database yang berskala besar. Anda dituntut untuk membangun *Admin Dashboard* yang mencakup modul-modul CRUD saling terhubung, sehingga seluruh tabel dapat terisi dan beroperasi sebagaimana sistem rumah sakit pada dunia nyata.

---

## Spesifikasi ERD (Entity Relationship Diagram)
Aplikasi ini memiliki skema yang kompleks dengan **12 tabel** yang saling berelasi. Anda wajib membuat *Migration* dan *Model (Eloquent)* beserta relasinya dengan spesifikasi kolom berikut:

1. **`users`** (Tabel Autentikasi Sistem)
   - **Kolom:** `id`, `name`, `email`, `password`, `role` (enum: admin, doctor, patient), `timestamps`
   - **Relasi:** *Has One* Patient, *Has One* Doctor.

2. **`specializations`** (Spesialisasi Medis)
   - **Kolom:** `id`, `name` (cth: Kardiologi, Umum), `description`, `timestamps`
   - **Relasi:** *Has Many* Doctors.

3. **`doctors`** (Data Dokter)
   - **Kolom:** `id`, `user_id` (FK), `specialization_id` (FK), `experience_years`, `consultation_fee`, `timestamps`
   - **Relasi:** *Belongs To* User, *Belongs To* Specialization, *Has Many* Appointments, *Has Many* Medical Records.

4. **`patients`** (Profil Pasien)
   - **Kolom:** `id`, `user_id` (FK), `dob` (Tanggal Lahir), `gender`, `address`, `phone`, `timestamps`
   - **Relasi:** *Belongs To* User, *Has Many* Appointments, *Has Many* Admissions, *Has Many* Bills.

5. **`rooms`** (Data Kamar Rawat Inap)
   - **Kolom:** `id`, `room_number`, `type` (VIP, Kelas 1, dll), `capacity`, `current_occupancy`, `timestamps`
   - **Relasi:** *Has Many* Admissions.

6. **`admissions`** (Data Pasien Rawat Inap)
   - **Kolom:** `id`, `patient_id` (FK), `room_id` (FK), `admission_date`, `discharge_date` (nullable), `status`, `timestamps`
   - **Relasi:** *Belongs To* Patient, *Belongs To* Room, *Has One* Bill.

7. **`appointments`** (Janji Temu Dokter)
   - **Kolom:** `id`, `patient_id` (FK), `doctor_id` (FK), `appointment_date`, `reason`, `status`, `timestamps`
   - **Relasi:** *Belongs To* Patient, *Belongs To* Doctor.

8. **`medicines`** (Data Inventaris Obat)
   - **Kolom:** `id`, `name`, `description`, `price`, `stock`, `timestamps`
   - **Relasi:** *Has Many* Prescriptions.

9. **`medical_records`** (Rekam Medis Pasien)
   - **Kolom:** `id`, `patient_id` (FK), `doctor_id` (FK), `diagnosis`, `record_date`, `timestamps`
   - **Relasi:** *Belongs To* Patient, *Belongs To* Doctor, *Has Many* Prescriptions.

10. **`prescriptions`** (Resep Obat)
    - **Kolom:** `id`, `medical_record_id` (FK), `medicine_id` (FK), `dosage`, `quantity`, `timestamps`
    - **Relasi:** *Belongs To* Medical Record, *Belongs To* Medicine.

11. **`bills`** (Tagihan Rumah Sakit)
    - **Kolom:** `id`, `patient_id` (FK), `admission_id` (FK, nullable), `total_amount`, `status` (unpaid, paid), `issued_date`, `timestamps`
    - **Relasi:** *Belongs To* Patient, *Belongs To* Admission, *Has Many* Payments.

12. **`payments`** (Riwayat Transaksi Pembayaran)
    - **Kolom:** `id`, `bill_id` (FK), `payment_method`, `amount`, `payment_date`, `timestamps`
    - **Relasi:** *Belongs To* Bill.

---

## Tahapan Pengerjaan: Setup Sistem

### 1. Install Laravel Project
- Buat project baru: `composer create-project laravel/laravel hospital-app`

### 2. Konfigurasi Database
- Buat database `db_hospital` dan hubungkan pada file `.env`.

### 3. Pembuatan Migration
- Buat file migration untuk 12 tabel secara berurutan agar tidak melanggar aturan *Foreign Key*. Tabel tanpa FK (seperti `users`, `rooms`, `medicines`, `specializations`) harus dibuat pertama.

### 4. Pembuatan Model (Eloquent ORM)
- Buat Model untuk setiap tabel. Tentukan `$fillable` untuk *mass assignment*.
- Definisikan metode relasi untuk ke-12 model secara komprehensif.

### 5. Seeder Data Dummy
- Otomatiskan pengisian data menggunakan Seeder/Factory.
- Masukkan setidaknya 5 Kamar, 20 Obat, 5 Spesialisasi, 5 Dokter, dan 10 Pasien, beserta riwayat janji temu dan tagihannya agar database langsung siap digunakan.

---

## Perancangan Fitur CRUD (Mengelola Seluruh Tabel)

Untuk menuntaskan tahapan pembuatan Route (6), Controller (7), dan operasi antarmuka View Read/Create/Show/Edit/Delete (8-12), Anda wajib menerapkan *Route Resource* pada `routes/web.php` dan merancang fungsi CRUD pada 4 modul utama di bawah ini:

### Tahapan Persiapan UI (Layout Utama)
- **View Dashboard:** Buat satu file utama di `resources/views/layouts/app.blade.php`. Terapkan kerangka Bootstrap atau Tailwind CSS.
- **Navigasi:** Tambahkan sebuah Navbar yang memuat link (Route) menuju 4 fitur di bawah ini. Pastikan file View dari seluruh modul menge-`extend` layout utama ini.

### Modul 1: Manajemen Master Data (`specializations`, `rooms`, `medicines`)
Modul ini bertugas untuk menyiapkan data-data statis pendukung operasional rumah sakit.
- **Route & Controller:** Buat `Route::resource` terpisah untuk `specializations`, `rooms`, dan `medicines`. Generate 3 Controller terkait menggunakan Artisan.
- **View Read (Index):** Buat halaman tabel HTML untuk menampilkan daftar obat, kamar, dan keahlian spesialisasi secara terpisah.
- **View Create & Store:** Buat Halaman Tambah Data (Form HTML). Pada Controller method `store`, gunakan `$request->validate()` lalu jalankan perintah `Model::create()` (misalnya `Medicine::create($request->all())`).
- **View Edit, Update, & Delete:** Buat halaman Form Update yang otomatis terisi nilai lama (*pre-filled*). Buat logika `update()` dan fungsi hapus data `delete()`.

### Modul 2: Registrasi SDM & Pasien (`users`, `doctors`, `patients`)
Modul ini mencatat data profil manusia dengan menerapkan *Multi-Table Insert*.
- **Route & Controller:** Definisikan route resource untuk `doctors` dan `patients`. Generate `DoctorController` dan `PatientController`.
- **View Create & Store (Pasien & Dokter):** 
  - Rancang Form HTML yang menggabungkan input data akun (Nama, Email, Password) dan data profil spesifik (Alamat/Tanggal Lahir untuk Pasien, atau Spesialisasi/Pengalaman untuk Dokter).
  - Saat dikirim (Submit), di dalam Controller Anda **wajib** melakukan *insert* secara berurutan ke 2 tabel:
    1. Lakukan insert ke tabel `users` terlebih dahulu untuk kredensial login (`$user = User::create(...)`).
    2. Simpan identitas sisanya ke tabel `patients` atau `doctors` dengan memasukkan ID dari user yang baru terbuat (`user_id => $user->id`).
- **View Read & Update:** Saat menampilkan tabel data list (Index) dan memodifikasi data (Edit), lakukan teknik *Eager Loading* (Contoh: `Patient::with('user')->get()`). Saat fitur Update dijalankan, jangan lupa menyimpan perubahan pada tabel `users` dan tabel profilnya masing-masing.
- **Delete:** Ketika fungsi hapus ditekan, Anda cukup menghapus objek model `User`. Aturan *Cascade* pada database otomatis akan ikut melenyapkan profilnya.

### Modul 3: Layanan Rawat Jalan (`appointments`, `medical_records`, `prescriptions`)
Modul ini merupakan inti operasional pencatatan rekam medis dan penyerahan obat.
- **Manajemen Janji Temu (`appointments`):**
  - Buat route resource dan `AppointmentController`. Halaman View Create harus memuat menu Dropdown untuk memilih nama Pasien dan Dokter. Simpan data pertemuan ke tabel `appointments`.
- **Rekam Medis & Resep (Multi-Table CRUD Lanjutan):**
  - Buat route dan `MedicalRecordController`.
  - **View Create:** Halaman ini digunakan oleh dokter. Sediakan form untuk teks "Diagnosis". Di halaman yang sama, buat seksi "Resep Obat" (Dropdown memuat nama `medicines` dan field jumlah *Dosage/Quantity*).
  - **Store (Proses Transaksi Berantai):** Saat tombol simpan ditekan:
    1. Controller menyisipkan diagnosis ke tabel `medical_records`.
    2. Controller memasukkan data obat yang dipilih ke tabel `prescriptions` berpatokan pada `medical_record_id` yang baru tercipta di langkah 1.
    3. Jalankan sintaks pengurangan otomatis (`decrement`) pada atribut `stock` di tabel `medicines` sesuai jumlah obat yang keluar.
- **View Show (Detail Rekam Medis):** Buat halaman cetak Rangkuman Pemeriksaan yang menarik riwayat keseluruhan menggunakan relasi antar tabel (Menampilkan detail penyakit, instruksi dokter, dan rincian obat yang harus diminum pasien).

### Modul 4: Rawat Inap & Keuangan (`admissions`, `bills`, `payments`)
Modul akhir untuk siklus pemakaian kamar dan penerbitan faktur pembayaran rumah sakit.
- **Registrasi Rawat Inap (`admissions`):**
  - Buat `AdmissionController`. Di halaman View Create, sediakan Dropdown Pasien dan Dropdown Kamar (Hanya memunculkan list kamar kosong yang nilai `current_occupancy < capacity`).
  - **Store:** Saat `store` dieksekusi, nilai `current_occupancy` kamar tersebut di tabel `rooms` harus diperbarui bertambah 1 (`increment`).
  - **Update:** Saat pasien diizinkan pulang (Update status admission menjadi *Discharged*), kurangi kembali `current_occupancy` di kamar tersebut, dan sistem **harus otomatis** membuat nota tagihan baru di tabel `bills`.
- **Proses Tagihan & Bayar (`bills`, `payments`):**
  - Buat `BillController`. Sediakan tabel tagihan rawat inap pasien pada View Index.
  - Sediakan Halaman "Bayar Tagihan" (Create `payments`). Saat disimpan, Controller memasukkan catatan ke tabel `payments`. Jika nominal cukup, ubah nilai atribut `status` di tabel `bills` menjadi *paid* (lunas).
- **View Show (Kwitansi Lunas):** Buat satu halaman View detail terakhir yang menyerupai Kwitansi Rumah Sakit (memanggil kompilasi *Eager Loading* data Tagihan, rincian Rawat Inap, identitas Pasien, dan daftar cicilan/riwayat Pembayaran).

---
**Catatan Akhir:** Pastikan Anda menyematkan fungsi validasi (`$request->validate()`) di seluruh proses simpan/ubah Controller, memastikan keamanan *Mass Assignment* (pengaturan array `$fillable`), dan memaksimalkan *Eager Loading* guna mencegah perlambatan *N+1 Query Problem*.
