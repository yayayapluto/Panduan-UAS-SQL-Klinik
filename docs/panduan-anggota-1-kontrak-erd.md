# Panduan Agit: Kontrak, ERD, dan Prosedur

## Peran

Kamu adalah Agit. Tiga pekerjaan besarmu: menetapkan bahan konseptual
sebelum tabel dibuat, menggambar diagram relasi entitas, dan menulis bagian
laporan yang menjelaskan proyek dari luar. Kamu juga pemeriksa terakhir daftar
periksa buatan Rassya.

Pekerjaanmu menentukan pekerjaan lima orang lain. Kalau nama entitas atau
kardinalitas berubah setelah mereka mulai menulis SQL, seluruh berkas harus
diperiksa ulang. Karena itu selesaikan Milestone 1 lebih dulu dan umumkan hasil
kesepakatan di grup.

## Berkas Milik

| Berkas atau bagian | Isi |
|---|---|
| `sql/00-kesepakatan.sql` | Kesepakatan nama objek, aturan bisnis, kardinalitas, dan kebutuhan informasi |
| `sql/04_prosedur.sql` | Dua stored procedure: `sp_daftar_kunjungan` dan `sp_bayar_kunjungan` |
| `bukti/` | Keluaran nyata setiap skrip, dirapikan bersama Rassya |
| `panduan/gambar/erd.tex` atau gambar draw.io | Diagram relasi entitas |
| `laporan/template-laporan.tex` bagian Bab I | Deskripsi basis data, studi kasus, ruang lingkup, pembagian peran |
| `laporan/template-laporan.tex` bagian Bab III | Kesimpulan dan saran |
| Sampul dan tabel peran laporan | Nama dan NIM keenam anggota beserta uraian tugas |
| Bahan subbab perbandingan NoSQL | Pemetaan dokumen JSON dan alasan memilih relasional |

Berkas SQL milikmu ada dua: `sql/00-kesepakatan.sql` dan `sql/04_prosedur.sql`;
struktur dan data tetap milik anggota lain. Nama tabel dan kolom yang kamu
tetapkan dipakai oleh Dapa sampai Kafka, jadi perubahan nama harus
lewat kesepakatan dengan mereka.

## Konteks untuk AI

Proyek ini adalah basis data klinik pratama dengan MySQL 8.0 dan tepat sebelas
tabel. Sebagai Agit kamu menulis `sql/00-kesepakatan.sql`,
`sql/04_prosedur.sql`, studi kasus, kamus data, ERD, dan Bab I serta Bab III
laporan. Kamu tidak menyentuh `sql/01_struktur.sql` sampai
`sql/07_pengujian.sql` selain berkas 04, dan tidak mengubah nama yang sudah
ditetapkan di sana.

Fakta yang wajib dipakai apa adanya:

- Lima poli: Poli Umum, Poli Anak, Poli Gigi, Poli Penyakit Dalam, Poli Mata.
- Sebelas entitas: poli, dokter, jadwal_praktik, pasien, kunjungan, rekam_medis,
  tindakan, detail_tindakan, obat, detail_resep, pembayaran.
- Delapan aturan bisnis dan delapan kebutuhan informasi pada [PRD](PRD.md).
- Relasi 1:0..1 pada rekam_medis dan pembayaran, M:N pada detail_tindakan dan
  detail_resep.

Jangan menambah entitas, kolom, atau aturan baru tanpa kesepakatan kelompok.
Kalau AI mengusulkan tabel keempat belas, mintalah ia menunjukkan kebutuhan
informasi mana yang tidak bisa dijawab oleh sebelas tabel yang ada.

## Milestone 1: Kesepakatan dan Kamus Data

### 1.1 Tetapkan studi kasus

Tulis alur klinik dalam satu halaman:

1. Pasien datang ke meja depan dan mendaftar pada satu sesi jadwal.
2. Petugas memberi nomor antrean.
3. Dokter memeriksa, mencatat keluhan, diagnosis, tekanan darah, suhu, berat, dan
   tinggi badan.
4. Dokter memberi tindakan dan menulis resep.
5. Petugas menghitung tagihan dari tindakan ditambah obat, lalu mencatat
   pembayaran.

Setiap langkah harus menyebut siapa pelakunya dan data apa yang dihasilkan.
Bagian ini menjadi bahan Bab I laporan.

### 1.2 Tetapkan delapan aturan bisnis

Aturan harus bisa diperiksa mesin. Kalimat seperti "pelayanan harus ramah" tidak
dipakai karena tidak punya penjaga teknis. Delapan aturan yang sudah disepakati
ada di [PRD §6](PRD.md#6-aturan-bisnis-dan-penjaganya). Pastikan setiap aturan
menyebut penjaga dan pengujiannya.

### 1.3 Susun kamus data

Kamus data adalah daftar resmi nama entitas dan atributnya. Semua berkas SQL
memakai nama ini; tidak ada singkatan baru yang muncul di tengah pengerjaan.
Isi tabel berikut di dokumen kesepakatan dan pertahankan urutannya.

| Entitas | Atribut | Keterangan |
|---|---|---|
| poli | id_poli, nama_poli, lantai | Tempat layanan; nama poli unik |
| dokter | id_dokter, id_poli, nama_dokter, no_sip, email, no_telp, tipe_dokter | Tenaga medis; no_sip dan email unik |
| jadwal_praktik | id_jadwal, id_dokter, tanggal, jam_mulai, jam_selesai, ruangan, kuota | Satu baris untuk satu sesi pada satu tanggal |
| pasien | id_pasien, nama_pasien, nik, email, no_telp, tanggal_lahir, alamat | NIK 16 digit dan unik |
| kunjungan | id_kunjungan, id_pasien, id_jadwal, tanggal_kunjungan, no_antrean, status_kunjungan | Pendaftaran pasien pada satu sesi |
| rekam_medis | id_rekam, id_kunjungan, keluhan, diagnosis, tekanan_darah, suhu, berat_badan, tinggi_badan, imt, tanggal_periksa | Hasil pemeriksaan; satu kunjungan paling banyak satu rekam |
| tindakan | id_tindakan, nama_tindakan, tarif | Master tindakan dan tarif |
| detail_tindakan | id_detail_tindakan, id_kunjungan, id_tindakan, jumlah | Tindakan pada satu kunjungan |
| obat | id_obat, nama_obat, satuan, harga, stok | Master obat dan sisa stok |
| detail_resep | id_detail_resep, id_rekam, id_obat, jumlah, aturan_pakai | Obat yang diresepkan |
| pembayaran | id_pembayaran, id_kunjungan, tanggal_bayar, metode, total_tagihan, status_bayar | Tagihan satu kunjungan |

### 1.4 Tetapkan delapan kebutuhan informasi

Tulis sebagai pertanyaan, bukan daftar fitur, karena pertanyaan langsung bisa
diuji dengan query. Daftarnya ada di [PRD §7](PRD.md#7-kebutuhan-informasi-dan-query).
Pastikan setiap pertanyaan bisa dijawab oleh tabel yang ada. Kalau ada yang
tidak bisa, periksa kamus data sebelum menambah entitas.

## Milestone 2: Diagram Relasi Entitas

### 2.1 Gambar entitas dan relasi

Gambar sebelas entitas tanpa atribut supaya terbaca. Daftar atribut sudah ada di
kamus data. Setiap relasi diberi nama kerja, misalnya "poli memiliki dokter" dan
"kunjungan menerima tindakan".

### 2.2 Tulis kardinalitas

| Relasi | Kardinalitas | Bentuk tabel hasil |
|---|---|---|
| poli memiliki dokter | 1 : N | kunci tamu id_poli di dokter |
| dokter punya jadwal praktik | 1 : N | kunci tamu id_dokter di jadwal_praktik |
| pasien mendaftar kunjungan | 1 : N | kunci tamu id_pasien di kunjungan |
| jadwal diisi kunjungan | 1 : N | kunci tamu id_jadwal di kunjungan |
| kunjungan menghasilkan rekam medis | 1 : 0..1 | id_kunjungan unik di rekam_medis |
| kunjungan menerima tindakan | M : N | tabel penghubung detail_tindakan |
| kunjungan ditagih pembayaran | 1 : 0..1 | id_kunjungan unik di pembayaran |
| rekam medis diresepkan obat | M : N | tabel penghubung detail_resep |

Tabel ini menjadi bahan Bab II dan dipakai Farras saat menulis skema. Jika Farras
menemukan kardinalitas yang tidak cocok dengan SQL, kesepakatan diubah lebih
dulu, bukan SQL-nya diam-diam.

### 2.3 Catat spesialisasi dokter

Model extended ER menggambarkan dokter umum dan dokter spesialis sebagai
subentitas. Implementasinya memakai satu tabel dengan kolom
`tipe_dokter ENUM('Umum','Spesialis')`. Tulis alasan keputusan ini di laporan:
atribut tambahan masih sedikit, sehingga memecah tabel menjadi tiga hanya
menambah penggabungan.

### 2.4 Hindari empat kesalahan gambar

1. Menulis kunci tamu di dalam belah ketupat relasi. Kunci tamu adalah hasil
   transformasi, bukan bagian diagram konseptual.
2. Menggambar relasi M:N lalu tetap menaruh kunci tamu di salah satu entitas.
   Relasi M:N harus menjadi tabel penghubung sendiri.
3. Menyimpan total tagihan di entitas kunjungan. Nilainya turunan dari tindakan
   dan obat.
4. Membuat entitas jadwal yang memuat kolom hari sekaligus tanggal. Hari bisa
   diturunkan dari tanggal.

## Milestone 3: Dua Prosedur

Berkas `sql/04_prosedur.sql` berisi dua prosedur simpan. Keduanya menjadi bukti
sifat atomicity pada bab transaksi: seluruh langkah berjalan dalam satu
transaksi, dan kegagalan di tengah jalan tidak meninggalkan baris setengah jadi.
Tulis `DROP PROCEDURE IF EXISTS` sebelum tiap `CREATE PROCEDURE` supaya skrip
aman dijalankan ulang.

### 3.1 `sp_daftar_kunjungan`

Parameter masuk `p_id_pasien` dan `p_id_jadwal`; keluaran `id_kunjungan_baru`
dan `no_antrean`. Langkah:

1. `START TRANSACTION`.
2. Ambil tanggal jadwal. Jika `NULL`, kirim `Jadwal tidak ditemukan`.
3. Jika tanggal sudah lewat dari `CURDATE()`, kirim
   `Jadwal sudah lewat, tidak bisa didaftari`.
4. Hitung antrean: `COALESCE(MAX(no_antrean), 0) + 1` untuk jadwal itu.
5. `INSERT` kunjungan berstatus `Menunggu`.
6. `COMMIT`.
7. Kembalikan `id_kunjungan_baru` dan `no_antrean`.

Kuota tidak diperiksa di sini karena sudah dijaga `trg_cek_kuota_jadwal`.
Menulis pemeriksaan yang sama di dua tempat membuat pesan galat berbeda dan
sulit diuji.

### 3.2 `sp_bayar_kunjungan`

Parameter masuk `p_id_kunjungan` dan `p_metode`; keluaran `total_dibayar`.
Langkah:

1. `START TRANSACTION`.
2. Hitung `COUNT(*)` pembayaran untuk kunjungan. Jika lebih dari nol, kirim
   `Kunjungan sudah dibayar`.
3. Hitung total dari tindakan ditambah obat:

   ```text
   SUM(detail_tindakan.jumlah * tindakan.tarif)
   +
   SUM(detail_resep.jumlah * obat.harga)
   ```

4. `INSERT` pembayaran dengan `CURDATE()`, metode dari parameter, status
   `Lunas`.
5. `COMMIT`.
6. Kembalikan `total_dibayar`.

Kedua prosedur memakai `EXIT HANDLER FOR SQLEXCEPTION` yang menjalankan
`ROLLBACK` dan `RESIGNAL`. Ini yang membuat atomicity bisa dibuktikan: jika
salah satu langkah gagal, tidak ada baris setengah jadi.

### 3.3 Periksa

Uji 7 mendaftarkan pasien pada jadwal Desember 2026, menambah satu tindakan dan
satu resep, lalu membayar. Total yang diharapkan 155.000, dari 150.000 biaya
pembersihan karang gigi ditambah 5.000 harga sepuluh tablet Paracetamol.
Pemanggilan kedua harus ditolak `Kunjungan sudah dibayar`. `verify.sh` bagian G
memeriksa tiga hal: total cocok hitungan manual, pembayaran kedua ditolak, dan
tidak ada baris bertotal nol.

### 3.4 Transaksi yang dibuktikan

- **Atomicity.** Total dihitung dan pembayaran disimpan dalam satu transaksi.
- **Consistency.** Pemeriksaan pembayaran ganda berjalan sebelum insert.
- **Isolation.** Sesi lain tidak melihat baris sebelum `COMMIT`; demo tingkat
  isolasi dijalankan Rassya lewat `tools/demo-isolasi.sh`.
- **Durability.** Setelah `COMMIT`, baris tetap ada walaupun koneksi putus.

## Milestone 4: Bab I dan Bab III Laporan

### 4.1 Bab I

Isi minimal:

- deskripsi basis data dan tujuan;
- studi kasus klinik;
- ruang lingkup dan empat hal di luar cakupan beserta alasannya;
- tabel pembagian peran beserta uraian tugas tiap anggota;
- kamus data dan delapan kebutuhan informasi.

### 4.2 Bab III

Isi kesimpulan yang menjawab tujuan, bukan mengulang daftar tabel. Contoh pola:
"Basis data ini menjawab delapan kebutuhan informasi; empat di antaranya menuntut
penggabungan tiga tabel atau lebih." Saran pengembangan harus konkret, misalnya
penambahan pengadaan obat atau jadwal berulang, dengan menyebut konsekuensi
skemanya.

### 4.3 Subbab perbandingan NoSQL

Materi minggu 15 dinilai lewat pemetaan, bukan pembangunan server. Tulis:

- satu contoh dokumen JSON yang melekatkan kunjungan, rekam medis, dan resep;
- alasan data yang selalu dibaca bersama sebaiknya dilekatkan, sedangkan data
  obat dan tarif sebaiknya dirujuk;
- tiga alasan skema relasional tetap lebih tepat untuk klinik ini: aturan yang
  ditegakkan mesin, laporan agregasi lintas poli, dan perubahan harga obat yang
  harus konsisten.

## Milestone 5: Pemeriksaan Akhir

1. Bandingkan kamus data dengan `sql/01_struktur.sql`. Nama dan tipe harus sama.
2. Bandingkan ERD dengan tabel kardinalitas. Tidak boleh ada relasi yang hilang.
3. Periksa daftar periksa buatan Rassya. Setiap butir harus punya berkas
   bukti, bukan hanya tanda centang.
4. Cocokkan sampul dan tabel peran laporan dengan pembagian tugas enam anggota.
5. Perbarui Bab I dan Bab III setelah angka dari `bukti/` final.

## Checklist

- [ ] Delapan aturan bisnis tertulis, masing-masing punya penjaga dan pengujian.
- [ ] Kamus data memuat sebelas entitas dan seluruh atributnya.
- [ ] Delapan kebutuhan informasi ditulis sebagai pertanyaan yang bisa diuji.
- [ ] ERD memuat 1:N, M:N, dan 1:0..1.
- [ ] Tabel kardinalitas cocok dengan `sql/01_struktur.sql`.
- [ ] Bab I memuat studi kasus, ruang lingkup, peran, kamus data.
- [ ] Bab III memuat kesimpulan dan saran yang konkret.
- [ ] Subbab NoSQL memuat satu dokumen JSON dan alasan memilih relasional.
- [ ] `sql/00-kesepakatan.sql` memuat kamus data dan kardinalitas yang sama.
- [ ] Dua prosedur di `sql/04_prosedur.sql` memakai transaksi dan `EXIT HANDLER`;
      pembayaran kedua ditolak.
- [ ] Sampul dan tabel peran laporan memuat keenam anggota.
- [ ] Daftar periksa Rassya diperiksa dengan bukti.

## Kesalahan yang Sering Terjadi

1. Menulis nama entitas dengan singkatan berbeda dari kamus data, misalnya
   `jadwal` padahal SQL memakai `jadwal_praktik`.
2. Menempatkan total tagihan di kunjungan sehingga muncul angka ganda.
3. Menggambar M:N tanpa tabel penghubung.
4. Menyalin template laporan tanpa mengganti nama anggota dan NIM.
5. Menulis kesimpulan yang mengulang Bab I, bukan menjawab tujuan.
6. Menaruh pemeriksaan kuota di `sp_daftar_kunjungan`, sehingga pesan galatnya
   berbeda dari trigger.
7. Menyimpan status `Lunas` tanpa menghitung ulang total dari tindakan dan obat.

## Prompt Siap Tempel

```text
Saya Agit pada proyek basis data klinik pratama dengan MySQL 8.0 dan
sebelas tabel. Saya sedang menyusun kamus data dan ERD. Berikut aturan umum
proyek dan berkas sql/01_struktur.sql.

Periksa apakah kamus data saya cocok dengan skema: nama tabel, nama kolom, dan
kardinalitas. Sebutkan ketidakcocokan beserta baris yang bermasalah. Jangan
menambah tabel atau kolom baru, dan jangan mengubah nama yang sudah ada.
```

## Definition of Done

Fase Agit selesai jika kamus data, ERD, dan tabel kardinalitas disepakati
sebelum `skema-v1`; `sql/00-kesepakatan.sql` dan `sql/04_prosedur.sql` sudah
dijalankan; Bab I dan Bab III terisi dengan angka yang cocok dengan `bukti/`;
subbab NoSQL ada; dan PDF laporan terbangun dengan sampul serta tabel peran enam
anggota.
