# Panduan Pengguna Viral Producer

Viral Producer membantu membuat dan mengelola naskah trivia untuk Facebook Reels. Penjelasan diberikan dalam bahasa Indonesia, sedangkan naskah konten menggunakan natural American English.

Butuh perintah siap salin? Buka [Pustaka Prompt](prompt-library.md).

## 1. Yang Dilakukan AI

Setiap kali membuat post, AI akan:

1. membaca aturan dan database terbaru dari GitHub;
2. meriset minimal 18 kandidat fakta;
3. membuka dan memeriksa sumber;
4. menolak fakta lemah, berulang, tidak aman, atau tidak dapat diverifikasi;
5. memeriksa agar fakta belum pernah digunakan;
6. memilih enam fakta terbaik;
7. menulis dan mengaudit naskah;
8. menyimpan hasil yang lolos sebagai post baru.

AI tidak memposting langsung ke Facebook. Anda tetap menyalin dan memposting naskah secara manual.

## 2. Alur Produksi

```text
Buat draft
→ Periksa naskah dan sumber
→ Revisi wording atau ganti fakta
→ Approve
→ Copy dari ready queue
→ Posting manual ke Facebook
→ Tandai sebagai posted
```

## 3. Arti Status

| Status | Artinya |
|---|---|
| `draft` | Post sudah tersimpan dan masih dapat direvisi. |
| `ready` | Post sudah lolos pemeriksaan dan siap diposting. |
| `posted` | Anda sudah mempostingnya dan AI sudah mengarsipkannya. |

`approved` adalah tahap internal sebelum post masuk ke `ready queue`.

## 4. Memulai Percakapan

Gunakan plugin **Viral Producer** dalam chat baru. Jalankan pemeriksaan berikut sebelum mulai bekerja:

```text
Periksa versi sistem dan status repository pada branch main. Jangan mengubah file apa pun.
```

Pemeriksaan ini bersifat `read-only` dan tidak mengubah database.

## 5. Membuat Post

### Menggunakan rotasi topik otomatis

```text
Buatkan satu post baru menggunakan rotasi topik default. Simpan sebagai draft setelah seluruh validasi lulus.
```

### Menentukan topik

```text
Buatkan satu post tentang animals-nature dengan country focus GLOBAL. Simpan sebagai draft setelah seluruh validasi lulus.
```

Topik yang tersedia:

- `geography-history`
- `animals-nature`
- `body-science`
- `food-home`
- `inventions-records`
- `practical`

Country focus yang didukung:

- `US`
- `CA`
- `UK`
- `AU`
- `GLOBAL`

Jika tidak ditentukan, AI akan menggunakan rotasi terbaru dari database.

## 6. Memeriksa Post

Ganti ID pada contoh berikut dengan Post ID yang ingin diperiksa.

```text
Tampilkan P-000009 beserta status dan clean copy. Jangan mengubah repository.
```

Untuk melihat sumber:

```text
Tampilkan seluruh sumber P-000009 dan jelaskan klaim yang didukung setiap sumber. Jangan mengubah repository.
```

Kedua perintah tersebut bersifat `read-only`.

## 7. Revisi Wording atau Ganti Fakta

### Revisi wording

Gunakan jika fakta tetap sama dan Anda hanya ingin memperbaiki kalimat.

```text
Buat Fact 3 pada P-000009 lebih punchy tanpa mengubah canonical claim, angka, scope, qualifier, atau Fact ID.
```

### Ganti fakta

Gunakan jika Anda menginginkan fakta yang benar-benar berbeda.

```text
Ganti Fact 4 pada P-000009 dengan fakta yang benar-benar berbeda. Riset dan verifikasi penggantinya, lalu hitung ulang audit post.
```

Mengubah arti fakta, angka, lokasi, waktu, atau kesimpulan termasuk **ganti fakta**, bukan revisi wording.

## 8. Approve Post

Approve hanya setelah Anda puas dengan keenam fakta.

```text
Audit ulang P-000009. Jika seluruh gate lulus, approve dan masukkan post tersebut ke ready queue.
```

AI akan memeriksa ulang sumber, duplikasi, scope, kualitas, serta konsistensi data sebelum mengubah status menjadi `ready`.

## 9. Mengambil Naskah Siap Posting

```text
Tampilkan post berikutnya yang siap saya copy. Jangan mengubah repository.
```

AI akan menampilkan Post ID, status, dan clean copy dalam satu blok teks. Perintah ini tidak menghapus post dari antrean.

## 10. Setelah Posting ke Facebook

Setelah Anda benar-benar memposting naskah ke Facebook, jalankan:

```text
Saya sudah memposting P-000009 ke Facebook. Tandai sebagai posted dan verifikasi archive, published facts, active drafts, ready queue, serta production state.
```

Jangan menjalankan perintah ini sebelum konten benar-benar diposting.

## 11. Menolak Post

```text
Tolak P-000009 dan ikuti lifecycle rejection sesuai data contract.
```

ID yang sudah digunakan tidak akan dipakai kembali.

## 12. Audit Database

Audit tanpa perbaikan:

```text
Audit database untuk duplicate Fact ID, duplicate claim, archive mismatch, counter mismatch, orphan fact, dan ready-queue mismatch. Jangan mengubah apa pun.
```

Jika ditemukan masalah, minta AI menjelaskan temuan terlebih dahulu. Jalankan repair hanya setelah target perbaikannya jelas.

## 13. Aturan Penggunaan

- Gunakan hanya plugin **Viral Producer**, bukan plugin tes lama.
- Jalankan satu perintah yang mengubah repository pada satu waktu.
- Jangan membuat post dari dua chat secara bersamaan.
- Perintah `Tampilkan`, `Periksa`, dan `Audit ... jangan mengubah` bersifat `read-only`.
- Jangan mengedit file JSON atau JSONL secara manual.
- Jangan meminta AI menandai post sebagai `posted` sebelum dipublikasikan.
- Selalu gunakan Post ID lengkap, misalnya `P-000009`.

## 14. Jika Terjadi Masalah

### GitHub tidak terhubung

Hubungkan official GitHub app dan beri akses read/write hanya ke repository `milyarderpro/viral-producer`.

### AI dapat membaca tetapi tidak dapat menyimpan

Periksa permission GitHub untuk repository contents. Jangan ulangi perintah write berkali-kali sebelum status operasi sebelumnya diperiksa.

### Terjadi SHA conflict

Minta AI memuat ulang state terbaru dan menghitung ulang operasi. Jangan meminta force overwrite.

### Sumber tidak dapat dibuka

Minta AI mengganti kandidat fakta atau mencari sumber authoritative lain. Jangan menyimpan fakta yang belum terverifikasi.

### Post tidak muncul di ready queue

Jalankan:

```text
Periksa status post dan konsistensi ready queue. Jangan melakukan repair.
```

### Hasil chat berbeda dari repository

Anggap repository sebagai sumber utama. Minta AI menampilkan ulang post dari record terbaru.

## 15. Ringkasan Workflow Harian

1. Buat satu draft.
2. Baca keenam fakta.
3. Periksa sumber bila diperlukan.
4. Revisi wording atau ganti fakta.
5. Approve post.
6. Ambil clean copy dari ready queue.
7. Posting manual ke Facebook.
8. Tandai post sebagai `posted`.
