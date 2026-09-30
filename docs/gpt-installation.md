# Instalasi Viral Producer di ChatGPT

## 1. Tujuan

Panduan ini menjelaskan cara memasang Viral Producer secara privat, menghubungkannya ke GitHub, memverifikasi akses baca dan tulis, lalu menyiapkannya untuk acceptance test.

Repository produksi:

    milyarderpro/viral-producer

Branch pengujian:

    build/viral-content-gpt

Jangan mengubah branch menjadi main sebelum seluruh acceptance test lulus dan pull request disetujui.

## 2. Catatan Produk Terkini

Dokumentasi ini diperbarui pada 2026-09-30.

OpenAI sedang memindahkan workflow Custom GPT ke Plugin. Plugin menggabungkan skill sebagai instruksi workflow dan app sebagai koneksi ke layanan seperti GitHub. Ketersediaan pembuatan, instalasi, dan migrasi bergantung pada akun, workspace, serta izin administrator.

Referensi resmi:

- [Moving your custom GPT workflows to plugins](https://learn.chatgpt.com/docs/migrate-custom-gpts)
- [Build plugins](https://learn.chatgpt.com/docs/build-plugins)
- [Install and use plugins](https://learn.chatgpt.com/docs/plugins)
- [Workspace connections](https://learn.chatgpt.com/docs/enterprise/shared-connections)

Gunakan urutan pilihan berikut:

1. Jika Plugin Creator dan Plugins tersedia, gunakan Jalur A.
2. Jika GPT Builder masih tersedia, gunakan Jalur B.
3. Jika keduanya tidak tersedia, minta administrator mengaktifkan fitur yang sesuai.
4. Bangun MCP atau Action khusus hanya jika koneksi GitHub yang tersedia tidak mendukung operasi file yang dibutuhkan.

Kedua jalur tetap menggunakan file repository yang sama sebagai sumber kebenaran.

## 3. Persiapan

Siapkan:

- akun ChatGPT atau workspace yang mengizinkan pembuatan workflow;
- akses ke repository milyarderpro/viral-producer;
- izin baca dan tulis pada branch build/viral-content-gpt;
- GitHub connector, app, atau plugin yang tersedia pada workspace;
- Web Search atau browser research yang tersedia pada pengalaman tersebut;
- browser desktop untuk konfigurasi awal;
- waktu untuk menjalankan smoke test sebelum penggunaan produksi.

Pastikan file berikut sudah tersedia pada branch pengujian:

    system/gpt-instructions.md
    system/content-dna.md
    system/data-contract.md
    data/production-state.json
    data/active-drafts.jsonl
    data/facts/*.jsonl
    output/ready-to-post.md
    tests/acceptance-tests.md

## 4. Membatasi Akses GitHub

Gunakan prinsip akses minimum.

Pilihan terbaik:

1. Gunakan akun GitHub khusus untuk Viral Producer.
2. Berikan akun tersebut akses hanya ke milyarderpro/viral-producer.
3. Jika layar otorisasi GitHub menyediakan pilihan repository, pilih hanya viral-producer.
4. Jangan memberikan akses organisasi atau repository lain jika tidak diperlukan.
5. Jangan menempelkan personal access token ke chat, instructions, knowledge file, atau repository.

Jika koneksi berbasis user account tidak menyediakan pembatasan per repository, gunakan akun GitHub khusus yang hanya memiliki akses ke repository ini.

Izin repository tidak selalu dapat dibatasi per branch. Perlindungan branch dan instruksi GPT menjadi lapisan tambahan:

- seluruh pengujian menulis ke build/viral-content-gpt;
- main tidak boleh ditulis selama instalasi dan acceptance test;
- pertimbangkan branch protection pada main;
- gunakan satu writer aktif sesuai data contract.

Dokumentasi resmi menyarankan pengujian akses baca dan aksi tulis secara terpisah serta memeriksa akun GitHub yang benar-benar dipakai oleh koneksi.

## 5. Menghubungkan GitHub

Nama menu dapat sedikit berbeda menurut akun atau workspace.

1. Buka Plugins, Apps, atau pengaturan workspace.
2. Cari GitHub.
3. Pilih Connect atau Install.
4. Masuk menggunakan akun GitHub yang sudah dibatasi.
5. Periksa identitas akun dan permission yang diminta.
6. Jika tersedia, pilih hanya milyarderpro/viral-producer.
7. Selesaikan otorisasi.
8. Catat nama koneksi agar tidak tertukar dengan koneksi GitHub lain.

Pada workspace terkelola, administrator mungkin harus menambahkan GitHub melalui Workspace connections dan mengizinkannya untuk role Anda.

Jika repository tidak muncul:

- pastikan akun GitHub tersebut dapat membuka repository;
- periksa apakah koneksi tersedia untuk role Anda;
- periksa allowed actions pada GitHub connection;
- reconnect setelah permission GitHub berubah;
- jangan memperluas akses ke semua repository hanya untuk melewati error.

## 6. Jalur A — Plugin, Direkomendasikan

Gunakan jalur ini jika Plugin Creator tersedia.

### 6.1 Membuka Plugin Creator

1. Buka percakapan baru di ChatGPT atau ChatGPT Work.
2. Ketik @ lalu pilih Plugin Creator.
3. Pastikan Anda berada di workspace yang benar.
4. Mulai dengan plugin privat.

### 6.2 Prompt pembuatan

Kirim prompt berikut:

    Buat plugin privat bernama Viral Producer untuk memproduksi naskah trivia Facebook Reels berbahasa Inggris. Gunakan file system/gpt-instructions.md dari repository milyarderpro/viral-producer sebagai instruksi workflow utama. Sertakan koneksi GitHub untuk membaca dan menulis repository tersebut pada branch build/viral-content-gpt, serta kemampuan web research untuk memverifikasi fakta. Repository adalah sumber kebenaran; jangan mengandalkan memory percakapan. Jangan publikasikan atau bagikan plugin sebelum acceptance test lulus.

Jika Plugin Creator meminta file instruksi, unduh atau lampirkan:

    system/gpt-instructions.md

File berikut dapat dilampirkan sebagai referensi bila Plugin Creator memerlukannya, tetapi versi GitHub tetap menjadi sumber terbaru:

    system/content-dna.md
    system/data-contract.md
    dataset-reference.md

Jangan mengunggah snapshot data aktif sebagai knowledge statis. production-state.json, active-drafts.jsonl, fact indexes, archives, dan ready queue harus selalu dibaca langsung dari GitHub agar tidak kedaluwarsa.

### 6.3 Menambahkan GitHub dan research

Di konfigurasi plugin:

1. Tambahkan GitHub connection yang sudah diotorisasi.
2. Jelaskan bahwa GitHub digunakan untuk pembacaan dan penulisan file.
3. Pastikan web research tersedia untuk verifikasi fakta.
4. Periksa bahwa instruksi menyebut repository dan branch pengujian secara eksplisit.
5. Simpan sebagai private.

Jika kemampuan web search tidak tersedia pada pengalaman tersebut, jangan gunakan plugin untuk produksi fakta. Pindah ke pengalaman yang mendukung research atau tambahkan tool research yang sesuai.

### 6.4 Memanggil plugin

Gunakan pemanggilan eksplisit selama pengujian:

    @Viral Producer Show production status. Do not modify anything.

Setelah pemilihan eksplisit berhasil, uji juga permintaan natural tanpa @. Pemilihan skill otomatis harus diuji terpisah karena plugin yang terpasang tidak selalu dipilih pada setiap permintaan.

## 7. Jalur B — GPT Builder, Jika Masih Tersedia

Gunakan jalur ini hanya jika akun atau workspace masih menyediakan pembuatan atau pengeditan GPT.

### 7.1 Membuka builder

1. Buka area GPTs atau My GPTs.
2. Pilih Create, New GPT, atau Edit GPT.
3. Buka tab Configure jika builder menyediakan mode percakapan dan konfigurasi.
4. Jangan publikasikan ke GPT Store.

### 7.2 Identitas GPT

Gunakan:

Name:

    Viral Producer

Description:

    Produces verified, non-duplicate English trivia scripts for Facebook Reels using a GitHub-backed production database.

### 7.3 Instructions

1. Buka file system/gpt-instructions.md dari branch build/viral-content-gpt.
2. Salin seluruh isinya.
3. Tempelkan tanpa diringkas ke kolom Instructions.
4. Pastikan bagian Runtime Configuration masih menunjuk ke build/viral-content-gpt.
5. Jangan mengganti branch menjadi main.

Jika builder menolak panjang instruksi, jangan memangkas aturan data, deduplikasi, verifikasi, atau write safety. Gunakan Jalur A agar instruksi dapat menjadi skill yang lengkap.

### 7.4 Capabilities dan app

1. Aktifkan Web Search atau kemampuan browser research yang tersedia.
2. Tambahkan GitHub App atau GitHub connection.
3. Hubungkan akun GitHub terbatas yang sudah disiapkan.
4. Pastikan GPT dapat menggunakan GitHub untuk tindakan baca dan tulis.
5. Jangan menambahkan Action khusus bila GitHub App sudah memenuhi kebutuhan.
6. Simpan versi awal sebagai private atau only me.

### 7.5 Conversation starters

Tambahkan empat contoh berikut:

    Buatkan satu post trivia baru.

    Buatkan tiga post trivia khusus Amerika Serikat.

    Tampilkan konten berikutnya yang siap diposting.

    Audit database fakta tanpa mengubah data.

Conversation starters hanya membantu pengguna memulai. Aturan sebenarnya tetap berasal dari gpt-instructions.md dan repository.

## 8. Verifikasi Akses Baca

Mulai percakapan baru dengan Viral Producer.

Prompt:

    Read plan.md and data/production-state.json from milyarderpro/viral-producer on branch build/viral-content-gpt. Report the current implementation stage, revision, next post number, and next fact number. Do not modify anything.

Lulus jika:

- repository dan branch yang dilaporkan benar;
- status stage sesuai plan.md;
- angka sesuai production-state.json;
- tidak ada file yang berubah;
- tidak ada commit baru.

Gagal jika GPT menjawab dari memory, membaca main, atau tidak dapat menyebutkan state aktual.

## 9. Verifikasi Akses Tulis

Lakukan hanya setelah akses baca lulus.

Gunakan test AT-02 dari tests/acceptance-tests.md:

    Create one post.

Lulus jika:

- satu draft tersimpan pada active-drafts.jsonl;
- satu post ID dan enam fact ID dialokasikan;
- production-state.json diperbarui;
- revision bertambah satu;
- GPT menampilkan commit-backed save status;
- penulisan terjadi pada build/viral-content-gpt;
- main tetap tidak berubah.

Jika write meminta persetujuan, periksa target repository, branch, dan file sebelum menyetujui.

Setelah test, jangan menghapus record secara manual atau menurunkan counter. Gunakan lifecycle GPT atau biarkan record menjadi bagian dari acceptance run.

## 10. Smoke Test Sebelum Acceptance Suite

Jalankan berurutan:

### 10.1 Read-only

Gunakan AT-01. Pastikan tidak ada commit.

### 10.2 Write

Gunakan AT-02. Catat post ID yang dihasilkan.

### 10.3 Duplicate

Gunakan AT-05 dan AT-06. Pastikan duplicate exact dan paraphrase ditolak tanpa write.

### 10.4 Lifecycle

Gunakan post hasil AT-02:

1. AT-08 untuk revisi wording.
2. AT-09 untuk mengganti satu fakta.
3. AT-10 untuk approval.
4. AT-11 untuk ready queue dan percakapan baru.
5. AT-12 untuk menandai posted.
6. AT-13 untuk retry idempotent.

Setelah smoke test lulus, jalankan seluruh tests/acceptance-tests.md pada Stage 10.

## 11. Memeriksa Perubahan di GitHub

Setelah setiap operasi tulis:

1. Buka repository milyarderpro/viral-producer.
2. Pilih branch build/viral-content-gpt.
3. Periksa commit terbaru.
4. Pastikan file yang berubah sesuai operasi.
5. Buka file dan validasi hasilnya.

Untuk create draft, periksa:

    data/production-state.json
    data/active-drafts.jsonl

Untuk approval, periksa:

    data/production-state.json
    data/active-drafts.jsonl
    output/ready-to-post.md

Untuk mark posted, periksa:

    data/production-state.json
    data/active-drafts.jsonl
    data/facts/*.jsonl
    data/posts/YYYY-MM.jsonl
    output/ready-to-post.md

Jangan menilai keberhasilan hanya dari jawaban chat. Repository adalah bukti akhirnya.

## 12. Menjaga Versi Pertama Tetap Privat

Sebelum Stage 10 selesai:

- gunakan visibility Private atau Only me;
- jangan publikasikan ke workspace directory;
- jangan bagikan link ke pengguna lain;
- jangan menghubungkan akun GitHub yang memiliki akses berlebihan;
- jangan pindahkan runtime branch ke main;
- jangan menjalankan beberapa writer bersamaan kecuali saat test konflik terkontrol.

Setelah seluruh test lulus, review permission kembali sebelum sharing.

## 13. Troubleshooting

### Repository tidak ditemukan

Periksa:

- akun GitHub yang terhubung;
- akses akun tersebut ke milyarderpro/viral-producer;
- scope repository pada otorisasi;
- workspace role dan availability GitHub connection;
- nama repository dan owner.

Reconnect setelah permission berubah.

### Bisa membaca tetapi tidak bisa menulis

Periksa:

- apakah GitHub connection mengizinkan write actions;
- apakah workspace meminta approval untuk tindakan tulis;
- permission akun GitHub pada repository;
- branch protection pada build/viral-content-gpt;
- apakah GPT mencoba menulis main;
- apakah file telah berubah dan SHA menjadi stale.

Uji read dan write secara terpisah. Keberhasilan read tidak membuktikan write permission.

### Menulis ke main

Hentikan pengujian. Jangan lanjutkan lifecycle.

Periksa Runtime Configuration pada gpt-instructions.md dan konfigurasi plugin/GPT. Dokumentasikan commit yang salah, lalu pulihkan melalui proses GitHub yang dapat diaudit. Jangan memakai perintah destruktif atau menimpa history.

### Web research tidak tersedia

Jangan membuat draft produksi. Fakta baru wajib diverifikasi dengan web research. Aktifkan tool yang sesuai atau pindah ke pengalaman yang mendukungnya.

### GPT mengatakan tersimpan tetapi tidak ada commit

Anggap operasi gagal.

- periksa branch;
- refresh repository;
- periksa izin write;
- minta GPT menjalankan consistency audit;
- jangan menandai konten sebagai ready atau posted.

### Konflik SHA atau partial failure

Mulai dengan:

    Audit the repository after the reported failure. Repair only deterministic state allowed by data-contract.md.

GPT harus membaca ulang file dan SHA terbaru. Jangan meminta GPT memaksa overwrite.

### Copy chat berbeda dari ready-to-post.md

Jalankan audit. active-drafts.jsonl adalah sumber kebenaran dan ready-to-post.md harus diregenerasi secara deterministik.

### Instruksi terlalu panjang untuk builder

Jangan memangkas bagian keselamatan atau kontrak data. Gunakan Plugin Creator dan jadikan instruksi sebagai skill, atau gunakan integrasi MCP terkontrol.

## 14. Fallback MCP atau Action

Gunakan fallback hanya bila GitHub connection tidak dapat:

- membaca file dari branch yang ditentukan;
- mengganti file menggunakan blob SHA;
- membuat file arsip bulanan;
- mengonfirmasi commit;
- menangani write conflict dengan aman.

Urutan fallback:

1. Plugin dengan GitHub app yang tersedia.
2. Plugin dengan MCP server GitHub yang dibatasi.
3. Legacy Custom GPT Action hanya jika builder masih mendukungnya dan kompatibilitasnya dengan konfigurasi yang dipilih sudah diverifikasi.
4. Backend khusus sebagai pilihan terakhir.

MCP atau backend khusus harus mengimplementasikan data-contract.md, bukan memberi akses GitHub mentah tanpa guardrail.

Minimum operasi yang diperlukan:

- fetch file dan blob SHA;
- create file;
- replace file menggunakan expected SHA;
- list atau read monthly archives;
- return commit confirmation;
- reject stale writes.

Jangan membangun fallback sebelum test membuktikan koneksi bawaan tidak cukup.

## 15. Checklist Instalasi

Instalasi siap untuk Stage 10 jika semua jawaban adalah ya:

- [ ] Workflow dibuat sebagai Plugin atau GPT privat.
- [ ] Instruksi lengkap terpasang.
- [ ] Repository benar.
- [ ] Runtime branch adalah build/viral-content-gpt.
- [ ] GitHub memakai akun dengan akses minimum.
- [ ] Akses read berhasil.
- [ ] Akses write berhasil.
- [ ] Web research tersedia.
- [ ] Main tidak berubah.
- [ ] Conversation starters atau saved prompts tersedia.
- [ ] tests/acceptance-tests.md dapat dibuka.
- [ ] Hasil smoke test dicatat.
- [ ] Tidak ada integrity error yang belum selesai.

## 16. Handoff ke Stage 10

Setelah instalasi, jangan langsung mengubah runtime branch ke main.

Mulai Stage 10 dengan:

    Run AT-01 from tests/acceptance-tests.md and report the evidence. Do not run later tests yet.

Jalankan test satu per satu, periksa GitHub setelah setiap write, lalu isi Results Table pada tests/acceptance-tests.md.

Jika satu test gagal:

1. hentikan test yang bergantung padanya;
2. dokumentasikan perilaku aktual dan commit;
3. perbaiki instruksi atau kontrak pada branch pengujian;
4. pasang ulang versi terbaru;
5. ulangi test yang terdampak;
6. lanjutkan hanya setelah hasilnya lulus.
