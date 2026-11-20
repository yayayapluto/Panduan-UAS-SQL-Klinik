# Panduan GitHub: Dua Repo

Ada dua repo dengan tugas berbeda.

| Repo | Visibilitas | Isi | Dipakai oleh |
|---|---|---|---|
| [Panduan-UAS-SQL-Klinik](https://github.com/yayayapluto/Panduan-UAS-SQL-Klinik) | Privat | panduan lengkap: README, docs, dan scaffold SQL | pemilik, untuk diunduh lalu dikirim ke tim |
| [Projek-UAS-SQL-Klinik](https://github.com/yayayapluto/Projek-UAS-SQL-Klinik) | Publik | repo kerja: README, scaffold SQL, dan runner SQL di `tools/` | tim, tempat push hasil kerja |

Repo panduan tetap privat. Tim tidak perlu akses ke sana. Mereka menerima
salinan ZIP dari pemilik dan memakai repo publik untuk mengirim pekerjaan.

## Mengirim Panduan ke Tim

Unduh repo privat sebagai ZIP:

```text
Buka repo Panduan-UAS-SQL-Klinik > Code > Download ZIP
```

Lewat CLI:

```bash
gh api repos/yayayapluto/Panduan-UAS-SQL-Klinik/zipball/main > panduan-uas.zip
```

Kirim ZIP itu ke grup. Jangan menambahkan ZIP ke repo publik karena isinya akan
menjadi folder bersarang. Tim mengekstrak ZIP di luar repo kerja.

ZIP berisi dokumen dan scaffold. Repo publik berisi scaffold yang sama. Tim
memakai ZIP untuk membaca panduan, lalu menulis solusi di repo publik.

## Undang Anggota ke Repo Publik

Hanya repo publik yang perlu collaborator. Pemilik membuka:

```text
Repo Projek-UAS-SQL-Klinik > Settings > Collaborators > Add people
```

Pilih akses `Write`, kirim undangan, lalu anggota menerimanya.

Lewat CLI:

```bash
gh api --method PUT \
  repos/yayayapluto/Projek-UAS-SQL-Klinik/collaborators/<akun-anggota> \
  -f permission=push
```

Ganti `<akun-anggota>` dengan username GitHub.

## Clone Repo Kerja

Setiap anggota mengerjakan repo publik.

```bash
git clone https://github.com/yayayapluto/Projek-UAS-SQL-Klinik.git
cd Projek-UAS-SQL-Klinik
```

Periksa identitas Git:

```bash
git config user.name
git config user.email
```

Kalau kosong, isi sekali:

```bash
git config --global user.name "Nama Kamu"
git config --global user.email "email@contoh.com"
```

## Menjalankan Satu Berkas SQL

Setelah database siap, uji berkas yang sedang dikerjakan tanpa memuat ulang
semua skrip:

```bash
bash tools/run-sql.sh 02
```

Di CMD atau PowerShell:

```text
tools\run-sql.cmd 02
```

Runner memilih container `kbd-mysql`, lalu container `mysql`, lalu MySQL host
seperti Laragon atau XAMPP. Pakai `--name <container>` untuk container lain dan
`--host` untuk memaksa MySQL host. Berkas `07` otomatis memakai `--force`.

## Alur Kerja Anggota

Jangan bekerja langsung di `main`. Satu tugas satu branch.

```bash
git switch main
git pull --rebase origin main
git switch -c anggota-2/data-contoh

# sunting hanya berkas milik sendiri, misalnya sql/02_data.sql
git add sql/02_data.sql
git commit -m "feat(sql): tambah data contoh poli dan dokter"
git push -u origin anggota-2/data-contoh
```

Setelah push, buka pull request ke `main`, isi ringkasan dan hasil pemeriksaan,
lalu minta review silang sesuai pembagian tugas.

Nama branch:

```text
anggota-1/<topik>
anggota-2/<topik>
anggota-3/<topik>
anggota-4/<topik>
anggota-5/<topik>
```

Format commit:

```text
feat: fitur baru
fix: perbaikan
docs: dokumentasi
test: pengujian
chore: alat atau berkas pendukung
```

## Menarik Perubahan Sebelum Push

```bash
git switch main
git pull --rebase origin main
git switch anggota-2/data-contoh
git rebase main
git push --force-with-lease
```

`--force-with-lease` hanya untuk branch sendiri. Jangan memaksa push ke `main`.

Konflik paling sering muncul bila dua orang menyunting berkas yang sama. Setiap
berkas punya pemilik tunggal, jadi konflik seharusnya jarang. Selesaikan konflik
di branch sendiri, jalankan pemeriksaan, lalu push.

## Tag Fase

Tag dipasang setelah fase selesai dan PR digabung:

```bash
git switch main
git pull --rebase origin main
git tag -a kesepakatan-v1 -m "Kesepakatan nama objek dan aturan bisnis"
git push origin kesepakatan-v1
```

Urutan tag: `kesepakatan-v1`, `skema-v1`, `objek-sql`, `uji-hijau`, `rilis`.

## Aturan Repo Publik

1. Jangan commit token, kunci, kata sandi pribadi, atau berkas `.env`.
2. Kata sandi `Klinik#2026` di scaffold hanya kredensial latihan database lokal.
3. Jangan mengunggah data pasien nyata. Data contoh harus fiktif.
4. Jangan mengubah berkas anggota lain tanpa persetujuan pemiliknya.
5. Jangan push ke `main` langsung. Selalu lewat branch dan pull request.
6. Tarik perubahan dengan `git pull --rebase` sebelum push.
7. Periksa `git status` sebelum commit agar berkas sementara tidak ikut.

## Pemeriksaan Sebelum Push

Di repo kerja:

```bash
bash tools/reset-db.sh
bash tools/verify.sh
```

Keluaran akhir harus:

```text
Lulus: 34   Gagal: 0
SEMUA PEMERIKSAAN LULUS
```

Untuk scaffold, pastikan berkas SQL masih hanya komentar:

```bash
grep -v '^--' sql/01_struktur.sql | grep -v '^$'
```

Keluarannya harus kosong.

## Kalau Panduan Diperbarui

Pemilik mengubah repo privat, lalu mengunduh ZIP baru dan mengirimnya lagi ke
grup. Tim tidak perlu clone repo privat. Perubahan scaffold yang harus masuk ke
repo publik dikerjakan lewat branch dan PR seperti biasa.
