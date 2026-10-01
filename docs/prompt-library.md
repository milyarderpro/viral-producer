# Pustaka Prompt Viral Producer

Gunakan prompt berikut di chat yang memakai plugin **Viral Producer**.

Belum memahami alurnya? Baca [Panduan Pengguna](user-guide.md).

- `READ-ONLY`: hanya membaca repository.
- `WRITE`: dapat mengubah repository.
- Ganti contoh `P-000009`, nomor fakta, topik, atau negara sesuai kebutuhan.
- Jalankan satu prompt `WRITE` pada satu waktu.

## 1. Pemeriksaan Awal

### Periksa versi dan koneksi — READ-ONLY

```text
Periksa versi sistem, repository, branch, production state, dan koneksi GitHub. Jangan mengubah file apa pun.
```

### Ringkasan status produksi — READ-ONLY

```text
Tampilkan revision, next Post ID, next Fact ID, jumlah post aktif, dan jumlah post ready. Jangan mengubah repository.
```

### Daftar seluruh post aktif — READ-ONLY

```text
Tampilkan seluruh post aktif beserta Post ID, topic, country focus, dan statusnya. Jangan mengubah repository.
```

## 2. Membuat Post

### Satu post dengan rotasi otomatis — WRITE

```text
Buatkan satu post baru menggunakan rotasi topik dan country focus default. Simpan sebagai draft hanya setelah seluruh validasi lulus.
```

### Menentukan topik — WRITE

```text
Buatkan satu post baru tentang animals-nature. Gunakan country focus default dan simpan sebagai draft setelah seluruh validasi lulus.
```

### Menentukan negara — WRITE

```text
Buatkan satu post baru untuk country focus AU. Pilih topik menggunakan rotasi default. Naskah akhir tetap menggunakan natural American English.
```

### Menentukan topik dan negara — WRITE

```text
Buatkan satu post geography-history dengan country focus US. Simpan sebagai draft setelah source, duplicate, scope, editorial, dan quality gate lulus.
```

### Membuat beberapa post — WRITE

```text
Buatkan tiga post menggunakan rotasi default. Kerjakan dan simpan secara berurutan, satu post pada satu waktu. Jangan melakukan parallel repository writes.
```

Topik yang didukung:

- `geography-history`
- `animals-nature`
- `body-science`
- `food-home`
- `inventions-records`
- `practical`

Country focus yang didukung: `US`, `CA`, `UK`, `AU`, dan `GLOBAL`.

## 3. Menampilkan dan Memeriksa Post

### Tampilkan post — READ-ONLY

```text
Tampilkan P-000009 beserta status, topic, country focus, dan clean copy. Jangan mengubah repository.
```

### Tampilkan sumber — READ-ONLY

```text
Tampilkan seluruh sumber P-000009. Jelaskan klaim yang didukung setiap sumber dan qualifier penting yang harus dipertahankan. Jangan mengubah repository.
```

### Audit satu post — READ-ONLY

```text
Audit P-000009 untuk source support, scope, duplicate, operator diversity, viral strength, quality rationale, dan generation audit. Jangan mengubah repository.
```

## 4. Revisi Post

### Memperbaiki wording — WRITE

Gunakan jika fakta dan maknanya tetap sama.

```text
Buat Fact 3 pada P-000009 lebih punchy tanpa mengubah canonical claim, angka, lokasi, waktu, scope, qualifier, atau Fact ID. Simpan hanya jika seluruh gate tetap lulus.
```

### Mengganti fakta — WRITE

Gunakan jika menginginkan klaim yang benar-benar berbeda.

```text
Ganti Fact 4 pada P-000009 dengan fakta yang benar-benar berbeda. Riset dan verifikasi penggantinya, periksa seluruh duplicate ledger, lalu hitung ulang audit dan quality post.
```

### Memperbaiki beberapa wording — WRITE

```text
Periksa wording keenam fakta P-000009. Perbaiki hanya kalimat yang kurang natural atau kurang punchy tanpa mengubah makna faktual. Tampilkan rencana perubahan sebelum menyimpan.
```

Perubahan angka, lokasi, waktu, qualifier, atau kesimpulan termasuk **ganti fakta**, bukan revisi wording.

## 5. Approve atau Reject

### Approve post — WRITE

```text
Audit ulang P-000009. Jika seluruh source, scope, duplicate, operator, strength, quality, dan generation audit lulus, approve post dan masukkan ke ready queue.
```

### Reject post — WRITE

```text
Tolak P-000009 dan ikuti lifecycle rejection sesuai data contract. Jangan menggunakan kembali Post ID atau Fact ID yang sudah dikonsumsi.
```

## 6. Ready Queue

### Tampilkan post berikutnya — READ-ONLY

```text
Tampilkan post berikutnya yang siap saya copy. Jangan mengubah repository.
```

### Tampilkan seluruh ready queue — READ-ONLY

```text
Tampilkan seluruh post dalam ready queue, diurutkan sesuai repository. Jangan mengubah status atau menghapus post dari antrean.
```

### Periksa satu ready post — READ-ONLY

```text
Periksa apakah P-000009 berstatus ready, muncul tepat satu kali di ready queue, dan clean copy-nya sama dengan active record. Jangan mengubah repository.
```

## 7. Setelah Posting ke Facebook

### Tandai sebagai posted — WRITE

Jalankan hanya setelah konten benar-benar diposting secara manual.

```text
Saya sudah memposting P-000009 ke Facebook. Tandai sebagai posted menggunakan waktu sekarang sesuai operational timezone, lalu verifikasi archive, published facts, active drafts, ready queue, dan production state.
```

### Periksa status setelah posting — READ-ONLY

```text
Periksa apakah P-000009 sudah berstatus posted, tidak ada di active drafts atau ready queue, dan memiliki enam published facts yang cocok dengan archive. Jangan mengubah repository.
```

## 8. Audit Database

### Audit lengkap tanpa perbaikan — READ-ONLY

```text
Audit seluruh database untuk duplicate Post ID, duplicate Fact ID, duplicate claim signature, semantic duplicate, malformed JSON atau JSONL, orphan fact, archive mismatch, counter mismatch, lifecycle mismatch, dan ready-queue mismatch. Jangan mengubah apa pun.
```

### Audit duplikasi saja — READ-ONLY

```text
Audit seluruh active reservations, published facts, blocked facts, dan archives untuk exact duplicate serta kemungkinan semantic duplicate. Jangan mengubah repository.
```

### Periksa partial operation — READ-ONLY

```text
Periksa apakah ada partial lifecycle operation, recovery marker, atau derived-data inconsistency. Laporkan temuan dan tindakan yang disarankan tanpa melakukan repair.
```

### Deterministic recovery — WRITE

Gunakan hanya setelah audit menemukan masalah yang jelas.

```text
Lakukan deterministic recovery untuk inconsistency yang sudah ditemukan. Jangan mengubah authoritative post atau fact content, jangan mengalokasikan ID baru, dan verifikasi ulang seluruh postcondition.
```

## 9. Konflik atau Hasil Tidak Jelas

### Periksa hasil write yang tidak pasti — READ-ONLY

```text
Operasi sebelumnya berhenti atau hasilnya tidak jelas. Periksa repository terbaru untuk menentukan write mana yang sudah berhasil. Jangan mengulang write dan jangan mengubah file.
```

### Muat ulang setelah SHA conflict — WRITE

```text
Muat ulang seluruh state dan SHA terbaru setelah conflict. Hitung ulang operasi dari awal, pertahankan ID yang sudah dikonsumsi, lalu lanjutkan hanya jika data contract mengizinkan.
```

## 10. Acceptance Test untuk Administrator

Jangan menjalankan acceptance test untuk produksi rutin.

### Menjalankan satu test tertentu

```text
Baca specification terbaru dan jalankan hanya acceptance test AT-01. Ikuti scope test tersebut dan jangan menjalankan test lain.
```

Beberapa acceptance test dapat mengubah repository. Jalankan hanya jika memahami expected mutation dan recovery-nya.

## 11. Prompt yang Sebaiknya Dihindari

Hindari prompt yang tidak jelas seperti:

```text
Perbaiki semuanya.
```

```text
Posting sekarang.
```

```text
Ganti dengan fakta yang lebih bagus.
```

Sebutkan Post ID, posisi fakta, dan tindakan yang diinginkan. Untuk audit atau pemeriksaan, tambahkan kalimat **“Jangan mengubah repository.”**
