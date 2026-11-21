# Pembagian Tugas

Dokumen ini mengatur siapa mengerjakan apa, kapan batas waktunya, dan bagaimana
pekerjaan dinilai selesai. Isinya dipakai bersama [PRD](PRD.md) dan
[Project Overview](PROJECT-OVERVIEW.md). Kalau ada perbedaan nama objek atau
angka, PRD yang menang.

## Prinsip

Tiga kesepakatan ini berlaku sepanjang proyek.

**Satu berkas satu pemilik.** Setiap berkas SQL, bagian laporan, atau skrip punya
tepat satu orang yang boleh menulisnya. Anggota lain boleh membaca dan
mengusulkan perubahan lewat grup, tetapi tidak menyunting langsung. Aturan ini
mencegah konflik merge dan membuat tanggung jawab jelas saat presentasi.

**Skema dibekukan sebelum menulis objek.** `sql/01_struktur.sql` adalah janji
antar berkas. Setelah tag `skema-v1` dibuat, nama tabel, nama kolom, tipe data,
dan nama constraint tidak berubah tanpa kesepakatan kelompok. Berkas view,
trigger, prosedur, dan pengujian bergantung pada nama-nama itu.

**Bukti berupa perintah dan output.** "Sudah selesai" tanpa bukti tidak dihitung.
Setiap penyerahan menyertakan perintah yang dijalankan beserta potongan
keluarannya, minimal:

```text
bash tools/reset-db.sh     → tabel terdaftar: 11
bash tools/verify.sh       → Lulus: 34   Gagal: 0
```

Tangkapan layar boleh ditambahkan, tetapi tidak menggantikan keluaran teks.

## Peran

| Anggota | Peran | Berkas dan bagian milik | Panduan |
|---|---|---|---|
| Anggota 1 | Studi kasus, kontrak, prosedur, laporan | Studi kasus, delapan kebutuhan informasi, kamus data, ERD, `sql/00-kesepakatan.sql`, `sql/04_prosedur.sql` (prosedur `sp_daftar_kunjungan` dan `sp_bayar_kunjungan`), `bukti/`, Bab I dan III laporan, sampul, tabel peran | [Panduan 1](panduan-anggota-1-kontrak-erd.md) |
| Anggota 2 | Normalisasi dan data | `sql/02_data.sql`, dokumentasi 1NF sampai BCNF, tangkapan layar query dasar | [Panduan 2](panduan-anggota-2-normalisasi-data.md) |
| Anggota 3 | Struktur dan transformasi | `sql/01_struktur.sql`, tabel transformasi, skema relasional, aljabar relasional | [Panduan 3](panduan-anggota-3-struktur-transformasi.md) |
| Anggota 4 | View, trigger, dan indeks | `sql/03_view_trigger.sql`, `sql/06_indeks.sql` | [Panduan 4](panduan-anggota-4-view-trigger-prosedur.md) |
| Anggota 5 | Keamanan dan query | `sql/05_keamanan.sql`, sepuluh query Q1 sampai Q10, `bukti/10-kueri.txt` | [Panduan 5](panduan-anggota-5-keamanan-kueri.md) |
| Anggota 6 | Pengujian dan rilis | `sql/07_pengujian.sql`, `tools/`, `bukti/` pendamping uji, checklist, PDF | [Panduan 6](panduan-anggota-6-pengujian-rilis.md) |

Anggota 1 dan Anggota 3 paling siap menulis SQL, karena keduanya memegang
keputusan skema dan kontrak awal. Anggota 2 dan Anggota 6 boleh berasal dari
anggota yang baru pertama memakai DBMS. Anggota 4 dan Anggota 5 beban objeknya
sedang, tetapi berkasnya bergantung pada `skema-v1`, jadi jangan menumpuk
semuanya di minggu terakhir.

Pembagian ini membagi beban secara setara, bukan memisahkan kemampuan. Peran
bukan pagar mati: anggota yang selesai lebih cepat boleh membantu menulis
dokumentasi, asalkan bukan berkas milik orang lain. Yang tidak boleh adalah dua
orang menyunting satu berkas pada waktu bersamaan tanpa kesepakatan.

## Fase dan Tag

Pekerjaan dibagi menjadi lima fase. Setiap fase ditutup dengan tag Git, sehingga
kemajuan bisa dilihat dari riwayat repo, bukan dari cerita di grup chat.

**F0: Kesepakatan (`kesepakatan-v1`), pemilik Anggota 1.** Menyepakati delapan
aturan bisnis, ruang lingkup, kamus data sebelas entitas, dan pembagian berkas.
Tag ini menandai bahwa nama entitas, atribut, dan cakupan sudah tidak berubah.
Berkas `sql/00-kesepakatan.sql` ditulis pada fase ini.

**F1: Skema (`skema-v1`), pemilik Anggota 3 dengan Anggota 2.** Anggota 3
menulis `sql/01_struktur.sql` sampai seluruh tabel, kunci, dan constraint ada.
Anggota 2 menulis `sql/02_data.sql` yang memenuhi jumlah baris dan keadaan
khusus. Setelah tag ini, perubahan nama tabel atau kolom harus lewat
kesepakatan.

**F2: Objek dan query (`objek-sql`), pemilik Anggota 4, Anggota 1, dan
Anggota 5 dengan Anggota 2.** Anggota 4 menulis dua view, empat trigger, dua
indeks, serta bukti `EXPLAIN` sebelum dan sesudah indeks. Anggota 1 menulis dua
prosedur, `sp_daftar_kunjungan` dan `sp_bayar_kunjungan`. Anggota 5 menulis tiga
pengguna, hak akses, bukti galat 1142, dan sepuluh query yang keluarannya
disimpan di `bukti/10-kueri.txt`. Anggota 2 menyelesaikan dokumentasi normalisasi
dan tangkapan layar query dasar. Pekerjaan boleh paralel karena berkasnya
berbeda, tetapi semuanya membaca `skema-v1`.

**F3: Integrasi dan pengujian (`uji-hijau`), pemilik Anggota 6.** Anggota 6
menulis `sql/07_pengujian.sql`, menjalankan `tools/verify.sh`, memperbarui
`bukti/`, dan menjalankan `demo-isolasi.sh`. Tag dipasang setelah 34 pemeriksaan
lulus dan seluruh bukti diperbarui.

**F4: Rilis (`rilis`), pemilik Anggota 1 untuk laporan dengan seluruh tim.**
Laporan Bab I sampai III selesai, PDF panduan dan PDF laporan terbangun, daftar
periksa Bab 14 panduan diperiksa. Setiap anggota menyerahkan bagian laporan yang
menjadi pegangannya sebelum tag dipasang. Tag `rilis` adalah versi yang
dikumpulkan.

Ringkasan penutup tiap fase: PDF laporan selesai, PDF panduan terbangun, daftar
periksa Bab 14 diperiksa.

Urutan tag: `kesepakatan-v1` → `skema-v1` → `objek-sql` → `uji-hijau` → `rilis`.
Tag tidak boleh melompati fase, karena tag berikutnya mengasumsikan tag
sebelumnya sudah ada.

## Aturan Berkas

**Skema hanya lewat Anggota 3 dengan Anggota 2.** Nama tabel, kolom, tipe data,
dan constraint diubah di `sql/01_struktur.sql` oleh Anggota 3. Jika Anggota 4
atau Anggota 5 membutuhkan kolom baru untuk trigger atau indeks, usulan dikirim
ke Anggota 3, bukan ditambahkan lewat `ALTER TABLE` di berkas lain. Anggota 3
yang memutuskan dan menyunting skema, Anggota 2 yang menyesuaikan data contoh,
lalu keduanya memberi tahu Anggota 1, Anggota 4, Anggota 5, dan Anggota 6 karena
semuanya memakai nama yang sama.

**Data contoh hanya lewat Anggota 2.** Angka pada `sql/02_data.sql` memengaruhi
hasil query, tabel di panduan, `bukti/`, dan 34 pemeriksaan. Karena itu hanya
Anggota 2 yang menyunting berkas itu, dan setiap perubahan harus disertai
`reset-db.sh` serta `verify.sh` yang lulus.

**Nama objek tidak diganti.** Daftar nama ada di [PRD §5.4](PRD.md#54-objek-bernama).
Anggota 1 memegang `sql/00-kesepakatan.sql` dan menjadi rujukan nama yang sudah
disepakati; berkas yang harus diperbarui saat nama berubah adalah
`sql/00-kesepakatan.sql`, `sql/01_struktur.sql`, `sql/02_data.sql`,
`sql/03_view_trigger.sql`, `sql/04_prosedur.sql`, `sql/05_keamanan.sql`,
`sql/06_indeks.sql`, `sql/07_pengujian.sql`, panduan, laporan, dan `bukti/`.
Biayanya jauh lebih besar daripada manfaatnya, jadi nama dibekukan di F0.

**Pengujian tidak disesuaikan dengan hasil yang gagal.** Jika `verify.sh` gagal,
yang diperbaiki adalah skema, data, atau objeknya, bukan pemeriksaannya. Satu
pengecualian: pemeriksaan yang memang salah tulis dan sudah disepakati Anggota 6
serta pemilik berkas terkait.

**Berkas bersama disepakati.** `README.md`, `docs/ai/`, `tools/`, dan
`.gitignore` tidak dimiliki satu orang. Perubahan yang memengaruhi perintah kerja
semua orang diumumkan di grup sebelum dikirim.

## Pendampingan Anggota Baru

Anggota 2 dan Anggota 6 bukan latar belakang basis data, jadi pendampingan masuk
ke rencana kerja, bukan tindakan darurat menjelang deadline.

Review dilakukan maksimal satu hari setelah penyerahan. Review menyebut hal
konkret: nama berkas, nomor baris, pesan galat, dan perbaikan yang disarankan.
Komentar "rapikan lagi" tidak dihitung sebagai review; yang dihitung adalah
"trigger `trg_cek_stok_obat` membaca `stok` sebelum `trg_kurangi_stok`
berjalan, jadi urutannya sudah benar, tetapi pesan galatnya belum diuji di Uji
6".

Satu sesi kerja bersama per fase. Anggota 3 dan Anggota 2 duduk selama 30 sampai
60 menit untuk membahas skema dan data yang macet, misalnya urutan pembuatan
tabel atau baris yang melanggar kunci tamu. Setelah `skema-v1`, Anggota 1 dan
Anggota 5 mengadakan sesi serupa untuk prosedur dan kueri: urutan pemanggilan
prosedur, parameter, serta angka pada `bukti/10-kueri.txt`. Sesi dicatat
tanggalnya di pesan grup.

Checklist "bisa menjelaskan" berlaku per objek yang dipegang. Sebelum penyerahan
boleh digabung, pemiliknya harus bisa menjawab tanpa membuka berkas:

- apa yang dilakukan objek itu;
- kapan objek itu berjalan atau dipanggil;
- pesan galat apa yang dihasilkan jika aturannya dilanggar;
- pengujian mana yang membuktikannya.

Jika tidak bisa menjawab, objeknya belum dianggap selesai meskipun
`verify.sh` lulus.

## Jadwal

Empat minggu, dengan asumsi sekitar 8 sampai 10 jam kerja per orang per minggu.

| Minggu | Fokus | Keluaran | Penanggung jawab |
|---|---|---|---|
| M1 | Kesepakatan dan konsep | delapan aturan bisnis, kamus data, ERD, ruang lingkup, `sql/00-kesepakatan.sql`, tag `kesepakatan-v1` | Anggota 1, dibantu seluruh tim |
| M2 | Skema dan data | `sql/01_struktur.sql`, `sql/02_data.sql`, dokumentasi normalisasi, tag `skema-v1` | Anggota 2 dan Anggota 3 |
| M3 | Objek, query, dan pengujian | dua view, empat trigger, dua prosedur, tiga pengguna, dua indeks, sepuluh query, `07_pengujian.sql`, `verify.sh` lulus | Anggota 1, Anggota 4, Anggota 5, dan Anggota 6 |
| M4 | Rilis | ulangan dari database kosong, `bukti/` diperbarui, PDF panduan, PDF laporan, daftar periksa | seluruh tim, Anggota 1 memimpin laporan |

M2 dan M3 paling padat karena skema harus selesai sebelum objek ditulis. Jika
jadwal mepet, yang dipotong adalah tambahan penjelasan di laporan, bukan
pengujian atau perbaikan data.

Minggu 4 sengaja dipakai untuk menjalankan ulang seluruh skrip dari keadaan
kosong. Kesalahan yang paling sering muncul justru di sini: urutan pembuatan
tabel salah, atau ada data yang hanya bekerja karena sisa percobaan sebelumnya.

## Definition of Done

Kriteria berikut berlaku per fase. Fase belum boleh ditutup jika ada satu butir
yang belum terpenuhi.

**F0: `kesepakatan-v1`.** Delapan aturan bisnis tertulis dan masing-masing punya
penjaga teknis. Kamus data memuat sebelas entitas beserta atributnya. ERD
memuat kardinalitas 1:N, M:N, dan 1:0..1. Ruang lingkup dan empat hal di luar
cakupan disepakati. Pembagian berkas tercatat. Anggota 1 wajib menyelesaikan
`sql/00-kesepakatan.sql` pada fase ini.

**F1: `skema-v1`.** `reset-db.sh` berhasil membuat 11 tabel. Setiap tabel punya
kunci primer. Sepuluh kunci tamu memakai `ON DELETE RESTRICT ON UPDATE
CASCADE`. Setiap tabel berisi minimal lima baris. Tidak ada kunci tamu yatim.
`verify.sh` bagian A dan B lulus. Fase ini milik Anggota 3 dengan Anggota 2.

**F2: `objek-sql`.** Dua view, empat trigger, dan dua indeks dari Anggota 4 ada
dengan nama yang ditetapkan PRD, dan `EXPLAIN` sebelum dan sesudah indeks
menunjukkan perubahan kolom `key`. Dua prosedur dari Anggota 1 ada dengan nama
yang sama. Tiga pengguna dari Anggota 5 ada dengan nama yang sama. Percobaan
menulis oleh `petugas_klinik` ditolak 1142. Anggota 5 menyelesaikan sepuluh query
dan angkanya sama dengan `bukti/10-kueri.txt`, sedangkan Anggota 2 menyelesaikan
dokumentasi normalisasi. Bagian C, D, dan F pada `verify.sh` lulus.

**F3: `uji-hijau`, milik Anggota 6.** Sepuluh skenario pada `07_pengujian.sql`
memberi hasil sesuai tabel pengujian. `verify.sh` berakhir
`Lulus: 34   Gagal: 0`. Seluruh berkas `bukti/` sudah diperbarui dari database
hasil `reset-db.sh`. Demo isolasi menampilkan perbedaan `READ COMMITTED` dan
`REPEATABLE READ` serta galat 1213.

**F4: `rilis`, dipimpin Anggota 1.** PDF panduan terbangun tanpa rujukan
menggantung. PDF laporan terbangun. Isi laporan cocok dengan angka pada `bukti/`
dan sampul memuat nama serta NIM enam anggota. Daftar periksa Bab 14 panduan
sudah diperiksa satu per satu.

## Pemetaan Materi ke Pemilik

Tabel ini memastikan setiap materi kuliah punya wujud nyata dan ada orang yang
bertanggung jawab menjelaskannya saat demo.

| Minggu | Materi | Wujud wajib | Pemilik |
|---|---|---|---|
| W1 | Konsep dasar, arsitektur DBMS, kemandirian data | penjelasan Bab 1, contoh kemandirian lewat view dan indeks | Anggota 1 |
| W2 | Model relasional, istilah relasi, kunci | skema sebelas tabel beserta daftar kunci | Anggota 3 |
| W3 | Perancangan basis data, identifikasi kebutuhan | delapan aturan bisnis, kamus data | Anggota 1 |
| W4 | Model relasi entitas, kardinalitas | ERD dan tabel kardinalitas | Anggota 1 |
| W5 | Transformasi ke tabel, aljabar relasional | tabel transformasi, contoh sigma, pi, join, agregasi | Anggota 3 |
| W6 | Normalisasi sampai 3NF | tabel perubahan tiap tahap | Anggota 2 |
| W7 | BCNF dan dependensi fungsional | daftar dependensi dan pembuktian BCNF | Anggota 2 |
| W8 | Ujian tengah semester | tidak ada luaran baru | tidak ada |
| W9 | DDL, tipe data, constraint | `sql/01_struktur.sql` | Anggota 3 |
| W10 | DML, INSERT, UPDATE, DELETE | `sql/02_data.sql` | Anggota 2 |
| W11 | Join, subquery, agregasi | Q1 sampai Q3 dan Q6 | Anggota 5, diuji Anggota 6 |
| W12 | Pengelompokan, fungsi jendela | Q4, Q5, Q7 sampai Q10 | Anggota 5, diuji Anggota 6 |
| W13 | Transaksi, ACID, isolasi | `sql/04_prosedur.sql`, `bukti/isolasi.txt` | Anggota 1, diuji Anggota 6 |
| W14 | Keamanan, hak akses, indeks | `sql/06_indeks.sql` (indeks dan `EXPLAIN`) | Anggota 4 |
| W14 | Keamanan, hak akses | `sql/05_keamanan.sql` (tiga pengguna, hak akses, galat 1142) | Anggota 5 |
| W15 | NoSQL dan tren teknologi | pemetaan dokumen JSON, tabel perbandingan | Anggota 1 |

W8 tidak punya luaran baru karena pekan ujian. Bahan normalisasi tetap dipakai
untuk belajar.

W13 sampai W15 sering terlewat karena pengerjaan berhenti setelah query selesai.
Di proyek ini ketiganya wajib dan dibuktikan dengan skrip yang benar-benar
dijalankan, bukan hanya dijelaskan.
