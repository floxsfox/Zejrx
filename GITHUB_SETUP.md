# GITHUB_SETUP — dari nol sampai APK (semua lewat GitHub Web)

Kamu butuh: akun GitHub dan browser. Tanpa PC, tanpa Android Studio, tanpa terminal.

Berkas yang kamu pegang: `FileXStudio-Source.zip`, `extract-project.yml`, `build-apk.yml`.

## 1. Buat repository

1. GitHub → **New repository** → beri nama (mis. `FileXStudio`).
2. Centang **Add a README file** (supaya branch utama langsung ada). Private atau public bebas.
3. **Create repository**.

## 2. Pasang workflow ekstraksi

1. Di halaman repo: **Add file → Create new file**.
2. Kolom nama: ketik `.github/workflows/extract-project.yml` (tanda `/` otomatis membuat folder).
3. Tempel seluruh isi `extract-project.yml`.
4. **Commit changes** langsung ke branch utama.

Workflow ini harus ada di branch utama, kalau tidak tombol **Run workflow** tidak muncul.

## 3. Upload ZIP

1. **Add file → Create new file**, nama `input/.gitkeep`, isi kosong, commit (cuma untuk membuat folder `input`).
2. Buka folder `input` → **Add file → Upload files** → pilih `FileXStudio-Source.zip` → commit.
   (Batas upload via browser: 25 MB per berkas. ZIP ini jauh di bawahnya.)

Nama file default yang dicari workflow adalah `input/FileXStudio.zip`. Kamu bisa ganti nama berkas sebelum upload,
atau isi kolom `zip_path` saat menjalankan workflow (mis. `input/FileXStudio-Source.zip`).

## 4. Soal build-apk.yml (baca ini, ada batasan GitHub)

`GITHUB_TOKEN` bawaan GitHub **tidak boleh** mengubah berkas di `.github/workflows/`. Akibatnya ekstraksi
akan meng-commit semua berkas proyek **kecuali** `.github/workflows/build-apk.yml`, dan memberi peringatan.
Dua cara:

- **Cara A (paling mudah):** setelah ekstraksi, **Add file → Create new file**, nama `.github/workflows/build-apk.yml`,
  tempel isi `build-apk.yml`, commit.
- **Cara B (otomatis):** sebelum ekstraksi, buat Personal Access Token (scope `repo` + `workflow`; atau fine-grained:
  Contents *Read and write* + Workflows *Read and write* untuk repo ini). Simpan di repo:
  **Settings → Secrets and variables → Actions → New repository secret**, nama `EXTRACT_PUSH_TOKEN`.
  Dengan ini `build-apk.yml` ikut ter-commit. Catatan: push memakai token ini juga bisa memicu build APK otomatis.

## 5. Jalankan ekstraksi

1. Tab **Actions** → **Extract Project ZIP** → **Run workflow**.
2. Isi `zip_path` bila perlu. `remove_zip` = hapus ZIP dari repo setelah sukses (opsional).
3. Tunggu centang hijau. Log menampilkan jumlah berkas baru/diubah dan yang dilewati.

Workflow ini menolak ZIP rusak, kosong, berisi path berbahaya (`../`, path absolut, symlink), atau bukan proyek Gradle.
Kalau tidak ada perubahan, ia selesai tanpa membuat commit kosong.

## 6. Cek hasil ekstraksi

Di halaman repo harus terlihat langsung di root: `app/`, `gradle/`, `build.gradle.kts`, `settings.gradle.kts`,
`gradle.properties`, `README.md`, dst. Tidak boleh ada folder pembungkus `FileXStudio/FileXStudio/`.
Pastikan juga `.github/workflows/build-apk.yml` ada (lihat langkah 4).

## 7. Jalankan build APK

1. **Actions → Build APK → Run workflow**.
2. Tunggu selesai (build pertama bisa 10 menit lebih karena mengunduh dependensi).

Catatan: `gradlew` dan `gradle-wrapper.jar` tidak ikut di ZIP. Workflow membuatnya otomatis untuk build itu
(versi Gradle diambil dari `gradle-wrapper.properties`) dan memvalidasinya. Tidak ada yang perlu kamu lakukan.

## 8. Unduh APK

1. Buka run yang selesai → bagian **Artifacts** di bawah.
2. Unduh **FileXStudio-Debug-APK** (ZIP berisi `.apk`). Ekstrak, lalu pasang di HP (izinkan "install dari sumber tidak dikenal").
3. **FileXStudio-Release-APK** hanya ada bila secret penandatanganan diisi (lihat `BUILD_GUIDE.md`).

## 9. Kalau gagal

| Gejala | Penyebab / solusi |
|---|---|
| Tombol **Run workflow** tidak ada | Workflow belum ada di branch utama |
| `ZIP not found` | Path/nama ZIP salah; samakan dengan `zip_path` |
| `Path traversal entry rejected` | ZIP berisi path berbahaya; buat ulang ZIP |
| `No Gradle settings file` | ZIP bukan proyek yang benar |
| `Android project not found` (build) | Ekstraksi belum dijalankan |
| Build gagal di langkah Gradle | Buka log, cari baris `e:` (error Kotlin) atau `What went wrong`, lalu kirim ke saya |
| Push ekstraksi ditolak | Cek **Settings → Actions → General → Workflow permissions** = *Read and write*, atau aturan proteksi branch |

Detail lebih lengkap: `BUILD_GUIDE.md`.
