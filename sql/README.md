# Scaffold SQL Klinik Pratama

Paket ini adalah kerangka kerja SQL untuk tim baru yang mengerjakan proyek Klinik Pratama. Semua berkas di dalamnya hanya berisi komentar, penanda TODO, dan kontrak nama objek. Tidak ada satu pun pernyataan SQL yang aktif, jadi paket ini belum menjadi solusi dan tidak dapat dijalankan apa adanya.

## Isi Paket

| Berkas | Pemilik | Isi |
| --- | --- | --- |
| `sql/00-kesepakatan.sql` | Anggota 1 | Kesepakatan nama objek, aturan bisnis, kardinalitas, dan kebutuhan informasi. |
| `sql/01_struktur.sql` | Anggota 3 | Sebelas tabel dasar beserta kunci primer, kunci asing, dan constraint. |
| `sql/02_data.sql` | Anggota 2 | Jumlah baris, urutan pengisian, dan keadaan khusus data contoh untuk sebelas tabel. |
| `sql/03_view_trigger.sql` | Anggota 4 | Dua view dan empat trigger kuota, bentrok jadwal, serta stok obat. |
| `sql/04_prosedur.sql` | Anggota 1 | Dua prosedur pendaftaran kunjungan dan pembayaran. |
| `sql/05_keamanan.sql` | Anggota 5 | Tiga user klinik beserta hak aksesnya. |
| `sql/06_indeks.sql` | Anggota 4 | Dua indeks untuk pencarian pasien dan rentang tanggal kunjungan. |
| `sql/07_pengujian.sql` | Anggota 6 | Sepuluh uji perilaku untuk constraint, trigger, prosedur, view, dan hak akses. |

## Cara Pakai

Folder `sql/` di ZIP ini sama dengan folder `sql/` di repo publik. Kerjakan
berkas di clone repo publik, bukan di dalam hasil ekstrak ZIP. ZIP dipakai untuk
membaca panduan dan kontrak.

Jangan menyalin folder ZIP ke dalam clone repo. Folder `sql/` yang sudah ada di
repo itu yang dipakai; menyalin ulang dapat menimpa pekerjaan anggota lain.

## Urutan Kerja

1. `01_struktur.sql`
2. `02_data.sql`
3. `03_view_trigger.sql`
4. `04_prosedur.sql`
5. `05_keamanan.sql`
6. `06_indeks.sql`
7. `07_pengujian.sql`, dikerjakan terpisah setelah enam berkas sebelumnya selesai.

Berkas 03 sampai 06 bergantung pada 01 dan 02. View dan trigger membaca tabel dari 01, prosedur memakai data dari 02, dan pengujian di 07 memeriksa seluruh objek yang dibuat sebelumnya. Kerjakan 01 dan 02 lebih dulu sampai bersih.

## Aturan Pengisian

1. Kerjakan daftar TODO di kepala berkas milikmu: tulis pernyataan SQL pada blok objek terkait, lalu ganti `[ ]` menjadi `[x]`.
2. Biarkan daftar TODO sampai fase ditutup. Tanda `[x]` menjadi bukti kemajuan.
3. Jangan mengubah nama tabel, kolom, constraint, view, trigger, prosedur, atau user yang sudah ditetapkan.
4. Jangan menyentuh berkas anggota lain. Setiap berkas punya pemilik tunggal.

Tipe data dan detail tipe tidak disalin di scaffold. Rujuk [`../docs/PRD.md`](../docs/PRD.md) untuk kontrak lengkap.

## Menjalankan Satu Berkas SQL

Runner `tools/run-sql.sh` menjalankan satu berkas tanpa reset. Cocok untuk
menguji berkas yang sedang dikerjakan.

```bash
bash tools/run-sql.sh 02
bash tools/run-sql.sh 7
```

Di CMD atau PowerShell:

```text
tools\run-sql.cmd 02
```

Runner mencari container `kbd-mysql`, lalu container `mysql`, lalu MySQL host
seperti Laragon atau XAMPP. Opsi yang tersedia:

```text
--name <container>   paksa container tertentu
--host               lewati container, langsung ke MySQL host
```

Kata sandi container default `kbd`; MySQL host default kosong. Set `KBD_PW` bila
berbeda. Port host bisa diganti lewat `KBD_PORT`. Berkas `07` otomatis memakai
`--force`.

## Tool Pendukung

Paket ini sudah menyertakan `tools/run-sql.sh` dan pembungkus
`tools/run-sql.cmd` untuk CMD/PowerShell. Tool `tools/reset-db.sh`,
`tools/verify.sh`, dan `tools/demo-isolasi.sh` belum ada; ketiganya ditulis Anggota 6
pada Fase F3 mengikuti
[`../docs/panduan-anggota-6-pengujian-rilis.md`](../docs/panduan-anggota-6-pengujian-rilis.md).

## Kriteria Selesai

Pekerjaan dinyatakan selesai jika seluruh kondisi berikut terpenuhi.

1. `tabel terdaftar: 11`
2. `Lulus: 34   Gagal: 0`
3. Sepuluh uji di `07_pengujian.sql` memberi hasil sesuai kontrak.
4. Berkas bukti diperbarui dari keluaran terakhir, bukan hasil percobaan sebelumnya.

## Rujukan

- [`../docs/PRD.md`](../docs/PRD.md)
- [`../docs/PEMBAGIAN-TUGAS.md`](../docs/PEMBAGIAN-TUGAS.md)
- [`../docs/panduan-anggota-6-pengujian-rilis.md`](../docs/panduan-anggota-6-pengujian-rilis.md)
