# Panduan Kafka: Hak Akses dan Sepuluh Query

## Ringkasan

Kamu menulis `sql/05_keamanan.sql`, lalu menyusun sepuluh query Q1 sampai Q10
beserta berkas hasilnya `bukti/10-kueri.txt`. Pekerjaanmu membuktikan dua hal
yang tidak terlihat dari isi tabel: siapa yang boleh menulis apa, dan apakah
pertanyaan pada PRD benar-benar bisa dijawab SQL.

Kamu juga pemeriksa kedua skema buatan Farras. Nama tabel, kolom, dan
constraint pada `sql/01_struktur.sql` harus sudah beku lewat tag `skema-v1`
sebelum hak akses ditulis.

## Berkas Milik

| Berkas atau bagian | Isi |
|---|---|
| `sql/05_keamanan.sql` | Tiga pengguna, hak akses, `SHOW GRANTS` |
| `bukti/kueri-panduan.sql` | Sepuluh query Q1 sampai Q10 |
| `bukti/10-kueri.txt` | Keluaran nyata sepuluh query |
| `bukti/05_keamanan.txt` | Keluaran skrip keamanan dan penolakan 1142 |
| Bahan Bab II | Tabel hak akses dan tabel hasil query |

Berkas `sql/06_indeks.sql` milik Kenzie, sedangkan `sql/07_pengujian.sql` dan
`tools/` milik Rassya. Kalau pengujian menemukan hak akses atau query yang
salah, perbaiki berkas 05 dan kueri milikmu, bukan skrip pengujiannya.

## Prasyarat

1. `bash tools/reset-db.sh` selesai dengan `tabel terdaftar: 11`.
2. `bash tools/verify.sh` lulus bagian A dan B sebelum kamu mulai.
3. `sql/03_view_trigger.sql` dan `sql/04_prosedur.sql` sudah ada, karena Q3, Q4,
   dan Q9 membaca tabel yang dipakai view dan prosedur.
4. Kamu menjalankan skrip sebagai `root`; pemberian hak akses tidak bisa
   dilakukan oleh akun biasa.

## Langkah 1: Tiga Pengguna dan Hak Akses

### 1.1 Buat akun

Tiga pengguna dengan kata sandi `Klinik#2026`:

| Pengguna | Hak | Alasan |
|---|---|---|
| `petugas_klinik` | SELECT pada poli, dokter, jadwal_praktik; SELECT dan INSERT pada pasien, kunjungan | Meja depan melihat jadwal dan mendaftarkan pasien, tetapi tidak membuka rekam medis atau pembayaran |
| `dokter_klinik` | SELECT pada semua tabel; INSERT dan UPDATE pada rekam_medis, detail_resep | Dokter membaca riwayat, lalu menulis hasil pemeriksaan dan resep |
| `admin_klinik` | ALL PRIVILEGES pada `klinik_pratama.*` | Pengelola sistem memerlukan hak penuh |

Skrip memakai `DROP USER IF EXISTS` agar bisa dijalankan ulang, lalu
`CREATE USER ... IDENTIFIED BY 'Klinik#2026'`, lalu `GRANT`, lalu
`FLUSH PRIVILEGES`, lalu `SHOW GRANTS` sebagai bukti.

### 1.2 Ambil bukti `SHOW GRANTS`

```bash
docker exec -i kbd-mysql mysql -uroot -pkbd klinik_pratama \
  < sql/05_keamanan.sql > bukti/05_keamanan.txt
```

Keluaran harus memuat tiga blok `Grants for ...` yang cocok dengan tabel di
atas. `admin_klinik` harus menampilkan `ALL PRIVILEGES ON *.*` atau
`klinik_pratama.*` sesuai grant yang ditulis.

### 1.3 Buktikan penolakan 1142

Bukti penolakan diambil dengan koneksi memakai akun petugas:

```bash
docker exec -i kbd-mysql mysql -upetugas_klinik -p'Klinik#2026' klinik_pratama \
  -e "INSERT INTO obat (nama_obat,satuan,harga,stok) VALUES ('Uji Ilegal','Tablet',1000.00,1);"
```

Keluaran yang benar:

```text
ERROR 1142 (42000): INSERT command denied to user 'petugas_klinik'@'%' for table 'obat'
```

Simpan keluaran itu di `bukti/05_keamanan.txt`. `verify.sh` bagian F memeriksa
`SHOW GRANTS` dan penolakan ini.

## Langkah 2: Sepuluh Query Q1 sampai Q10

### 2.1 Tulis `bukti/kueri-panduan.sql`

Sepuluh query dari [PRD §7](PRD.md#7-kebutuhan-informasi-dan-query) disimpan
dalam satu berkas, berurutan Q1 sampai Q10, masing-masing didahului komentar
`-- Qn` supaya hasilnya mudah dicocokkan.

| Query | Tujuan | Teknik | Acuan hasil |
|---|---|---|---|
| Q1 | Pasien lahir sebelum 2005 | `WHERE`, `ORDER BY` | 10 baris, tertua Gilang Ramadhan 1980-02-18 |
| Q2 | Dokter, poli, lantai | `JOIN` dua tabel | 6 dokter |
| Q3 | Riwayat kunjungan lengkap | `JOIN` lima tabel | 14 baris |
| Q4 | Sisa kuota per jadwal | `LEFT JOIN`, `COUNT`, `GROUP BY` | jadwal 7 sisa 0 |
| Q5 | Tagihan belum lunas | `JOIN`, `WHERE`, `ORDER BY` | 3 baris |
| Q6 | Pasien Poli Anak dan pasien tertua | subquery `IN`, subquery skalar | 2 pasien; Gilang Ramadhan |
| Q7 | Poli di atas rata-rata pendapatan | `GROUP BY`, `HAVING`, subquery | Poli Umum, Poli Gigi, Poli Penyakit Dalam |
| Q8 | Obat paling banyak keluar | `SUM`, `COUNT`, `LIMIT` | Amlodipine 5mg 60 tablet |
| Q9 | Kategori biaya kunjungan | `CASE` | 5 Ringan, 3 Sedang, 2 Berat |
| Q10 | Peringkat dokter | fungsi jendela `RANK()` | dr. Andi Pratama peringkat 1 |

Q9 tidak berasal langsung dari satu pertanyaan kebutuhan, tetapi dipakai untuk
melatih `CASE` dan membaca sebaran biaya.

### 2.2 Jalankan dan simpan

```bash
docker exec -i kbd-mysql mysql -uroot -pkbd klinik_pratama \
  < bukti/kueri-panduan.sql > bukti/10-kueri.txt
```

Jalankan selalu setelah `reset-db.sh`, bukan setelah percobaan pengujian, supaya
jumlah baris sesuai tabel acuan di atas.

### 2.3 Cocokkan dengan laporan

Satu angka yang berubah berarti ada dua berkas yang harus diperbarui sekaligus:
`bukti/10-kueri.txt` dan tabel hasil di laporan. Jangan mengubah data contoh agar
satu query terlihat lebih rapi; data contoh milik Dapa.

## Pemeriksaan Lulus

1. `sql/05_keamanan.sql` dijalankan ulang dua kali tanpa galat.
2. `SHOW GRANTS` menampilkan tiga akun sesuai tabel hak akses.
3. Penulisan oleh `petugas_klinik` ditolak dengan `ERROR 1142`.
4. Sepuluh query Q1 sampai Q10 berjalan tanpa galat dan hasilnya cocok dengan
   tabel acuan.
5. `bukti/05_keamanan.txt` dan `bukti/10-kueri.txt` berasal dari database hasil
   `reset-db.sh` terakhir.
6. `verify.sh` bagian F lulus.

## Kesalahan Umum

1. Memberi `GRANT ALL` kepada petugas agar tidak muncul galat, lalu pemeriksaan
   hak akses gagal.
2. Membuat pengguna tanpa `DROP USER IF EXISTS`, sehingga skrip gagal saat
   dijalankan kedua kali.
3. Menguji penolakan 1142 sebagai `root`; galat tidak muncul karena `root` punya
   hak penuh.
4. Menjalankan Q1 sampai Q10 sebelum `reset-db.sh`, lalu hasilnya meleset satu
   atau dua baris.
5. Menyalin angka query ke laporan dari percobaan lama, bukan dari
   `bukti/10-kueri.txt` terakhir.

## Serah Terima

- [ ] Tiga pengguna ada dengan hak sesuai tabel.
- [ ] Penolakan 1142 terbukti dan tersimpan.
- [ ] Skrip 05 bisa dijalankan ulang.
- [ ] `bukti/kueri-panduan.sql` memuat Q1 sampai Q10 berurutan.
- [ ] `bukti/10-kueri.txt` cocok dengan tabel acuan PRD §7.
- [ ] `verify.sh` bagian F lulus.
- [ ] Bahan Bab II memuat tabel hak akses dan tabel hasil query.
- [ ] Rassya menerima berkas ini dan menjalankan ulang `verify.sh` dari salinan
      bersih.
