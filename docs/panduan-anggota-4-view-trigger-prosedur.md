# Panduan Kenzie: View, Trigger, dan Indeks

## Peran

Kamu adalah Kenzie. Kamu menulis dua berkas yang membuat database bukan
hanya menyimpan data, tetapi juga menjaga aturan dan memakai data dengan cepat:
`sql/03_view_trigger.sql` dan `sql/06_indeks.sql`.

Kamu juga pemeriksa pertama skema buatan Farras. Sebelum menulis view,
trigger, atau indeks, pastikan nama tabel dan kolom di `sql/01_struktur.sql`
sudah sesuai PRD.

## Berkas Milik

| Berkas | Isi |
|---|---|
| `sql/03_view_trigger.sql` | Dua view dan empat trigger |
| `sql/06_indeks.sql` | Dua indeks beserta EXPLAIN sebelum dan sesudah |
| `bukti/06_indeks.txt` | Keluaran EXPLAIN sebelum dan sesudah |
| Bahan Bab II | Penjelasan view, trigger, indeks, dan rencana eksekusi |

Prosedur `sql/04_prosedur.sql` milik Agit. Hak akses dan sepuluh query
milik Kafka. Skema bukan milikmu. Kalau butuh kolom baru untuk trigger atau
indeks, kirim usulan ke Farras. Jangan menambah kolom lewat `ALTER TABLE` di
berkas 03 dan 06.

## Konteks untuk AI

Proyek ini memakai MySQL 8.0, database `klinik_pratama`, dan skema sebelas tabel
pada `sql/01_struktur.sql`. Sebagai Kenzie kamu menulis view, trigger, dan
indeks. Semua nama objek sudah ditetapkan dan tidak boleh diganti.

Nama yang wajib dipakai:

- View: `v_kunjungan_lengkap`, `v_pendapatan_poli`.
- Trigger: `trg_cek_kuota_jadwal`, `trg_cek_bentrok_jadwal`,
  `trg_cek_stok_obat`, `trg_kurangi_stok`.
- Indeks: `idx_pasien_nama`, `idx_kunjungan_tanggal`.

Pesan galat yang diuji lewat `SIGNAL SQLSTATE '45000'` harus sama persis:
`Kuota jadwal sudah penuh`, `Jadwal dokter bentrok`, `Stok obat tidak cukup`.

Jangan memakai `FOREIGN_KEY_CHECKS=0`, jangan mengganti `DELIMITER` di luar
klien `mysql`, dan jangan menambah trigger atau indeks baru yang tidak diminta.

## Milestone 1: Dua View

### 1.1 `v_kunjungan_lengkap`

View ini menggabungkan kunjungan dengan pasien, jadwal, dokter, poli, dan status
pembayaran. Kolom yang dihasilkan:

```text
id_kunjungan, tanggal_kunjungan, no_antrean, status_kunjungan,
id_pasien, nama_pasien, id_dokter, nama_dokter, tipe_dokter,
nama_poli, ruangan, jam_mulai, jam_selesai, status_bayar, total_tagihan
```

Gunakan `LEFT JOIN pembayaran` dan `COALESCE(b.status_bayar, 'Belum Ada
Tagihan')` supaya kunjungan tanpa tagihan tetap muncul. Pada data contoh, view
harus menghasilkan 14 baris.

View ini contoh kemandirian data logis. Kalau struktur tabel berubah, view yang
disesuaikan; laporan yang membaca view tidak perlu diubah.

### 1.2 `v_pendapatan_poli`

View ini meringkas pendapatan per poli per bulan. Kolomnya: `nama_poli`, `bulan`
(format `%Y-%m`), `jumlah_kunjungan`, `total_pendapatan`. Hanya baris dengan
`status_bayar = 'Lunas'` yang dihitung, supaya angka pendapatan tidak tercampur
piutang.

### 1.3 Periksa

```sql
SELECT * FROM v_kunjungan_lengkap LIMIT 5;
SELECT * FROM v_pendapatan_poli;
```

`verify.sh` bagian C memeriksa jumlah baris view. Kalau jumlahnya bukan 14,
periksa join dan data contoh lebih dulu sebelum mengubah view.

## Milestone 2: Empat Trigger

Trigger harus dibuat setelah view. `DELIMITER //` hanya berlaku di klien
`mysql`, bukan bagian dari perintah SQL.

### 2.1 `trg_cek_kuota_jadwal`

Waktu: `BEFORE INSERT` pada `kunjungan`. Ambil kuota jadwal, hitung kunjungan
yang statusnya bukan `Batal`, lalu tolak jika jumlahnya sudah mencapai kuota.

```sql
IF v_terpakai >= v_kuota THEN
  SIGNAL SQLSTATE '45000' SET MESSAGE_TEXT = 'Kuota jadwal sudah penuh';
END IF;
```

Uji 4 memakai jadwal 7 yang berkuota 4 dan sudah terisi 4. Pendaftaran kelima
harus ditolak.

### 2.2 `trg_cek_bentrok_jadwal`

Waktu: `BEFORE INSERT` pada `jadwal_praktik`. Periksa jadwal lain milik dokter
yang sama pada tanggal yang sama dengan rentang jam beririsan:

```sql
WHERE id_dokter = NEW.id_dokter
  AND tanggal = NEW.tanggal
  AND jam_mulai < NEW.jam_selesai
  AND jam_selesai > NEW.jam_mulai
```

Jika ada, kirim `Jadwal dokter bentrok`. Uji 5 memakai dokter 1 yang sudah
praktik 08.00 sampai 12.00, lalu mencoba jadwal 10.00 sampai 11.00.

### 2.3 `trg_cek_stok_obat`

Waktu: `BEFORE INSERT` pada `detail_resep`. Bandingkan stok obat dengan
`NEW.jumlah`. Jika stok lebih kecil, kirim `Stok obat tidak cukup`. Uji 6 memakai
obat 8 yang berstok 5 dan resep 99 botol.

### 2.4 `trg_kurangi_stok`

Waktu: `AFTER INSERT` pada `detail_resep`. Kurangi stok obat:

```sql
UPDATE obat SET stok = stok - NEW.jumlah WHERE id_obat = NEW.id_obat;
```

Urutan penting. Pemeriksaan stok berjalan `BEFORE`, pengurangan berjalan
`AFTER`. Kalau dibalik, stok sudah berkurang sebelum diperiksa dan pesan galat
tidak lagi bisa diandalkan.

### 2.5 Periksa

`verify.sh` bagian E menjalankan enam masukan salah dan memastikan semuanya
ditolak. Trigger kuota dan bentrok memakai `SIGNAL`, sehingga pesan yang dicari
adalah teks pesannya, bukan nomor galat.

## Milestone 3: Dua Indeks dan EXPLAIN

### 3.1 `idx_pasien_nama`

```sql
EXPLAIN SELECT * FROM pasien WHERE nama_pasien LIKE 'A%';
CREATE INDEX idx_pasien_nama ON pasien (nama_pasien);
EXPLAIN SELECT * FROM pasien WHERE nama_pasien LIKE 'A%';
```

| Keadaan | type | key | rows |
|---|---|---|---|
| Sebelum | ALL | NULL | 10 |
| Sesudah | range | idx_pasien_nama | 1 |

### 3.2 `idx_kunjungan_tanggal`

Indeks gabungan `(tanggal_kunjungan, id_jadwal)`. Kolom tanggal diletakkan lebih
dulu karena dipakai sendirian pada penyaringan; kolom kedua berguna saat laporan
menyaring tanggal sekaligus jadwal.

| Keadaan | type | key | rows |
|---|---|---|---|
| Sebelum | ALL | NULL | 14 |
| Sesudah | ref | idx_kunjungan_tanggal | 3 |

### 3.3 Skrip harus bisa diulang

`06_indeks.sql` membuang indeks lebih dulu jika sudah ada, memakai
`information_schema.statistics` dan `PREPARE`. Tanpa ini, menjalankan
`reset-db.sh` dua kali akan gagal dengan galat 1061 karena nama indeks sudah
dipakai.

### 3.4 Simpan bukti

```bash
docker exec -i kbd-mysql mysql -uroot -pkbd klinik_pratama \
  < sql/06_indeks.sql > bukti/06_indeks.txt
```

`verify.sh` bagian D membaca kolom `key` dari `EXPLAIN`. Jika isinya masih
`NULL`, indeks tidak dipakai optimizer dan pemeriksaan gagal.

## Checklist

- [ ] Dua view ada dengan nama yang ditetapkan dan menghasilkan data benar.
- [ ] `v_kunjungan_lengkap` menghasilkan 14 baris.
- [ ] Empat trigger ada, urutan BEFORE dan AFTER benar.
- [ ] Pesan galat trigger sama persis dengan yang diuji.
- [ ] Dua indeks ada dan dipakai optimizer pada EXPLAIN.
- [ ] Skrip 03 dan 06 bisa dijalankan ulang.
- [ ] `verify.sh` bagian C, D, dan E lulus.
- [ ] Bahan Bab II memuat potongan view, trigger, indeks, dan rencana eksekusi.

## Kesalahan yang Sering Terjadi

1. Membuat trigger sebelum view selesai, lalu nama kolom view berubah.
2. Memakai `AFTER INSERT` untuk pemeriksaan stok, sehingga data sudah masuk
   sebelum ditolak.
3. Menulis `DELIMITER` di dalam berkas yang dijalankan lewat `mysql -e`; opsi
   itu hanya dikenal klien interaktif.
4. Membuat indeks tanpa `EXPLAIN` sebelum, sehingga tidak ada bukti pengaruhnya.
5. Menjalankan `06_indeks.sql` dua kali tanpa membuang indeks lebih dulu.
6. Mengubah urutan kolom indeks gabungan tanpa memahami query yang dilayani.

## Prompt Siap Tempel

```text
Saya Kenzie. Saya sedang menulis sql/03_view_trigger.sql dan
sql/06_indeks.sql. Berikut aturan umum proyek, skema, dan potongan berkas saya.

Periksa: (1) setiap nama tabel dan kolom yang dipakai ada di skema, (2) trigger
stok memakai BEFORE untuk memeriksa dan AFTER untuk mengurangi, (3) indeks
dibuat setelah EXPLAIN sebelum dan kolom key pada EXPLAIN sesudah berisi nama
indeks. Sebutkan baris yang salah. Jangan mengganti nama objek dan jangan
menambah trigger atau indeks baru.
```

## Definition of Done

Fase Kenzie selesai jika dua view, empat trigger, dan dua indeks berjalan
dengan nama yang ditetapkan; bagian 03 dan 06 bisa dijalankan ulang lewat
`reset-db.sh`; `verify.sh` bagian C, D, dan E lulus; serta potongan view,
trigger, indeks, dan pesan galatnya sudah disimpan untuk laporan.
