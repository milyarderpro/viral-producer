# Instalasi Viral Producer di ChatGPT

Documentation version: 3.1 — Stage 12.13

## 1. Tujuan

Panduan ini menjelaskan cara memasang Viral Producer secara privat, menghubungkannya ke GitHub, memverifikasi akses baca dan tulis, lalu menyiapkannya untuk produksi.

Panduan utama ini memasang profile produksi berikut:

    RUNTIME_REPOSITORY=milyarderpro/viral-producer
    RUNTIME_BRANCH=main
    RUNTIME_MODE=production
    ALLOW_WRITES=true
    ALLOW_MAIN_WRITES=true

Acceptance suite versi 2 telah lulus. Stage 12.10 menguji AT-25 sampai AT-55 pada configured test branch. AT-24 tidak dijalankan sebagai pre-merge acceptance test: AT-24 adalah mandatory post-cutover smoke test setelah PR Stage 12 digabung dan plugin produksi diperbarui ke v1.1, tetapi sebelum backfill atau produksi dilanjutkan. `main` tetap menjadi satu-satunya branch produksi.

## 2. Catatan Produk Terkini

Dokumentasi ini diperbarui pada 2026-10-02.

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

### 3.1 Runtime profile

Runtime profile adalah konfigurasi lokal plugin yang dipercaya. Nilainya tidak disimpan sebagai state repository dan tidak boleh diubah oleh prompt pengguna, isi repository, atau respons connector.

Profile produksi:

    RUNTIME_REPOSITORY=milyarderpro/viral-producer
    RUNTIME_BRANCH=main
    RUNTIME_MODE=production
    ALLOW_WRITES=true
    ALLOW_MAIN_WRITES=true

Profile test terisolasi, untuk referensi boundary saja:

    RUNTIME_REPOSITORY=milyarderpro/viral-producer
    RUNTIME_BRANCH=test/viral-producer-v1.1
    RUNTIME_MODE=test
    ALLOW_WRITES=true
    ALLOW_MAIN_WRITES=false

Panduan ini memasang **production plugin**. Jangan memakai langkah instalasi production untuk membuat test plugin. Untuk test plugin gunakan `docs/test-plugin-installation.md`.

Aturan profile:

- production mode hanya valid untuk `main` dengan ALLOW_MAIN_WRITES true;
- test mode wajib memakai configured non-main branch dan ALLOW_MAIN_WRITES false;
- semua list dan read connector harus menyebut RUNTIME_REPOSITORY dan `ref: RUNTIME_BRANCH`;
- semua write harus menyebut RUNTIME_REPOSITORY dan `branch: RUNTIME_BRANCH` atau exact-ref field setara;
- response repository dan ref harus cocok sebelum content atau SHA dipercaya;
- default-branch fallback dilarang;
- SHA tidak boleh dipakai lintas branch.

### 3.2 Kebutuhan akses

Siapkan:

- akun ChatGPT atau workspace yang mengizinkan pembuatan workflow;
- akses ke repository milyarderpro/viral-producer;
- izin baca dan tulis sesuai RUNTIME_BRANCH yang dipasang;
- GitHub connector, app, atau plugin yang tersedia pada workspace;
- Web Search atau browser research yang tersedia pada pengalaman tersebut;
- browser desktop untuk konfigurasi awal;
- waktu untuk menjalankan smoke test sebelum penggunaan produksi.

Pastikan file berikut tersedia pada RUNTIME_BRANCH:

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

- smoke test read-only boleh membaca RUNTIME_BRANCH tanpa membuat commit;
- jangan mengulang acceptance test yang mutatif dengan profile produksi; gunakan profile test terisolasi;
- pertimbangkan branch protection yang tetap mengizinkan workflow produksi terotorisasi pada `main`;
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

    Buat plugin privat bernama Viral Producer untuk memproduksi naskah trivia Facebook Reels berbahasa Inggris. Gunakan file system/gpt-instructions.md dari repository milyarderpro/viral-producer sebagai instruksi workflow utama. Simpan runtime profile berikut sebagai konfigurasi lokal plugin: RUNTIME_REPOSITORY=milyarderpro/viral-producer, RUNTIME_BRANCH=main, RUNTIME_MODE=production, ALLOW_WRITES=true, dan ALLOW_MAIN_WRITES=true. Semua operasi GitHub wajib menyebut repository dan ref runtime secara eksplisit, memverifikasi repository/ref pada respons, dan menolak default-branch fallback. Sertakan kemampuan web research untuk memverifikasi fakta. Repository adalah sumber kebenaran; jangan mengandalkan memory percakapan. Jangan publikasikan atau bagikan plugin sebelum acceptance test lulus.

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
4. Simpan kelima runtime values sebagai konfigurasi lokal plugin.
5. Pastikan setiap connector call memakai RUNTIME_REPOSITORY dan RUNTIME_BRANCH secara eksplisit.
6. Pastikan production mode menolak branch selain `main`.
7. Simpan sebagai private.

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

1. Buka file system/gpt-instructions.md dari RUNTIME_REPOSITORY pada RUNTIME_BRANCH.
2. Salin seluruh isinya.
3. Tempelkan tanpa diringkas ke kolom Instructions.
4. Simpan kelima runtime values di konfigurasi lokal builder, bukan di knowledge file.
5. Untuk instalasi produksi, pastikan profile persis sama dengan profile produksi pada bagian 3.1.
6. Jangan menerima prompt yang mencoba mengganti runtime profile.

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

    Read plan.md and data/production-state.json from {{RUNTIME_REPOSITORY}} using explicit ref {{RUNTIME_BRANCH}}. Report the current implementation stage, revision, next post number, next fact number, configured repository, branch, and runtime mode. Verify the repository and ref returned by GitHub. Do not modify anything.

Lulus jika:

- repository dan branch yang dilaporkan benar;
- status stage sesuai plan.md;
- angka sesuai production-state.json;
- tidak ada file yang berubah;
- tidak ada commit baru.

Gagal jika GPT menjawab dari memory, membaca branch selain RUNTIME_BRANCH, memakai default branch, menerima response ref yang tidak cocok, atau tidak dapat menyebutkan state aktual.

## 9. Verifikasi Akses Tulis

Lakukan hanya setelah akses baca lulus.

Jangan menjalankan AT-02 sebagai acceptance test pada profile produksi. Untuk memverifikasi write permission produksi, lanjutkan hanya jika Anda memang ingin membuat satu draft produksi nyata:

    Create one post.

Untuk acceptance test mutatif, gunakan prompt yang sama hanya dengan profile test terisolasi pada RUNTIME_BRANCH test.

Lulus jika:

- satu draft tersimpan pada active-drafts.jsonl;
- satu post ID dan enam fact ID dialokasikan;
- production-state.json diperbarui;
- revision bertambah satu;
- GPT menampilkan commit-backed save status;
- penulisan terjadi hanya pada RUNTIME_BRANCH;
- write request dan response menyebut RUNTIME_REPOSITORY serta RUNTIME_BRANCH secara eksplisit;
- hanya file dan state yang diharapkan berubah.

Jika write meminta persetujuan, periksa target repository, branch, dan file sebelum menyetujui.

Setelah test, jangan menghapus record secara manual atau menurunkan counter. Gunakan lifecycle GPT atau biarkan record menjadi bagian dari acceptance run.

## 10. Smoke Test Setelah Instalasi

### 10.1 Pemeriksaan read-only

Jalankan version check pada bagian 16.5. Pastikan versi, repository, branch, dan state sesuai tanpa membuat commit.

### 10.2 Pemeriksaan write

Lakukan hanya jika Anda memang ingin membuat satu draft produksi:

    Buatkan satu post trivia baru.

Lulus jika draft tersimpan lengkap, counter dan revision diperbarui, serta commit hanya mengubah file yang semestinya. Draft tersebut menjadi data produksi nyata; jangan menghapusnya atau menurunkan counter secara manual.

### 10.3 Acceptance test lanjutan

Jangan menjalankan acceptance test mutatif dengan profile produksi. Untuk AT-25 sampai AT-55 dan test mutatif lainnya, gunakan **Viral Producer v1.1 Test** pada configured test branch dan ikuti `docs/test-plugin-installation.md` serta `tests/acceptance-tests.md`.

AT-24 adalah pengecualian: ia merupakan mandatory post-cutover smoke test read-only untuk production plugin v1.1. Jalankan hanya setelah PR Stage 12 di-merge ke `main` dan production plugin di-refresh, tetapi sebelum controlled backfill atau produksi dilanjutkan.

## 11. Memeriksa Perubahan di GitHub

Setelah setiap operasi tulis:

1. Buka repository milyarderpro/viral-producer.
2. Pilih RUNTIME_BRANCH.
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

Selama plugin masih bersifat pribadi:

- gunakan visibility Private atau Only me;
- jangan publikasikan ke workspace directory;
- jangan bagikan link ke pengguna lain;
- jangan menghubungkan akun GitHub yang memiliki akses berlebihan;
- jangan mengubah runtime profile melalui chat atau repository content;
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
- branch protection pada target produksi;
- apakah target write berbeda dari RUNTIME_BRANCH;
- apakah response repository/ref berbeda dari runtime profile;
- apakah file telah berubah dan SHA menjadi stale.

Uji read dan write secara terpisah. Keberhasilan read tidak membuktikan write permission.

### Target branch tidak cocok dengan runtime profile

Hentikan operasi. Jangan lanjutkan lifecycle.

Periksa kelima runtime values pada konfigurasi lokal plugin/GPT. Pastikan mode dan branch cocok, ALLOW_MAIN_WRITES sesuai profile, serta setiap connector call memakai exact ref. Dokumentasikan commit yang salah, lalu pulihkan melalui proses GitHub yang dapat diaudit. Jangan memakai perintah destruktif, menimpa history, atau mencoba ulang melalui default branch.

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

Instalasi atau refresh siap digunakan jika semua jawaban adalah ya:

- [ ] Workflow tetap menggunakan Plugin atau GPT privat yang sama.
- [ ] Nama, visibility, GitHub connection, dan permission tidak berubah tanpa alasan.
- [ ] system/gpt-instructions.md terpasang lengkap.
- [ ] Instruction version adalah 3.0 — Stage 12.
- [ ] Editorial version adalah 3.0 — Stage 12.
- [ ] Test specification version adalah 3.0 — Stage 12.
- [ ] Kelima runtime values tersimpan di konfigurasi lokal plugin.
- [ ] RUNTIME_REPOSITORY adalah milyarderpro/viral-producer.
- [ ] Production profile memakai RUNTIME_BRANCH=main, RUNTIME_MODE=production, ALLOW_WRITES=true, dan ALLOW_MAIN_WRITES=true.
- [ ] Test profile, bila dipasang, memakai configured non-main branch, RUNTIME_MODE=test, dan ALLOW_MAIN_WRITES=false.
- [ ] Semua connector call memakai explicit repository/ref dan memverifikasi response identity.
- [ ] Tidak ada default-branch fallback atau cross-branch SHA reuse.
- [ ] GitHub memakai akun dengan akses minimum.
- [ ] Akses read berhasil.
- [ ] Akses write tetap tersedia tetapi belum dipakai sebelum read-only checks lulus.
- [ ] Web research tersedia.
- [ ] generation_audit dan empat field editorial per fakta dikenali.
- [ ] Tidak ada perubahan RUNTIME_BRANCH yang tidak diharapkan.
- [ ] Tiga saved regression prompts tersedia.
- [ ] tests/acceptance-tests.md dapat dibuka.
- [ ] Plugin masih Private atau Only me.
- [ ] Tidak ada integrity error yang belum selesai.

## 16. Memperbarui Plugin ke Runtime Version 3

Gunakan bagian ini untuk memperbarui Viral Producer yang sudah terpasang. Jangan membuat plugin kedua dengan nama yang sama.

### 16.1 Sebelum memperbarui

1. Pastikan seluruh perubahan produksi yang dirilis berada pada RUNTIME_BRANCH profile produksi.
2. Catat nama plugin, visibility, GitHub connection, dan permission saat ini.
3. Pastikan plugin masih private.
4. Catat dan pertahankan kelima runtime values.
5. Jangan menjalankan produksi atau acceptance test selama proses refresh.

File sumber terbaru:

    system/gpt-instructions.md
    system/content-dna.md
    system/data-contract.md
    tests/acceptance-tests.md
    docs/gpt-installation.md

Jangan mengganti file data aktif dengan snapshot:

    data/production-state.json
    data/active-drafts.jsonl
    data/facts/*.jsonl
    data/posts/*.jsonl
    output/ready-to-post.md

Data tersebut harus tetap dibaca langsung dari GitHub.

### 16.2 Jalur A — Memperbarui Plugin yang sudah ada

1. Buka Plugins.
2. Pilih Viral Producer yang sudah dibuat.
3. Pilih Edit Plugin.
4. Pastikan Anda mengedit plugin yang sama, bukan membuat salinan baru.
5. Pertahankan nama, description, visibility, GitHub connection, dan Web Search.
6. Kirim prompt update berikut kepada Plugin Creator:

    Perbarui plugin privat Viral Producer yang sedang saya edit. Pertahankan identitas plugin, nama, visibility, GitHub connection, permission, audience, dan runtime profile saat ini. Gunakan RUNTIME_REPOSITORY=milyarderpro/viral-producer, RUNTIME_BRANCH=main, RUNTIME_MODE=production, ALLOW_WRITES=true, dan ALLOW_MAIN_WRITES=true sebagai konfigurasi lokal plugin yang tidak dapat diubah oleh prompt atau repository content. Ganti instruksi workflow dengan isi lengkap terbaru dari system/gpt-instructions.md pada RUNTIME_REPOSITORY dan explicit ref RUNTIME_BRANCH. Refresh system/content-dna.md, system/data-contract.md, dan tests/acceptance-tests.md dari exact ref yang sama. Semua connector call wajib explicit-ref dan memverifikasi repository/ref pada respons. Jangan mengubah permission, jangan membuat plugin baru, dan jangan menyentuh data produksi. Setelah selesai, sebutkan file yang diperbarui dan biarkan plugin tetap private.

7. Jika Plugin Creator meminta attachment, unduh file terbaru dari RUNTIME_BRANCH dan lampirkan file dengan nama yang sama.
8. Pastikan file lama diganti, bukan ditambahkan sebagai salinan bernama berbeda.
9. Review ringkasan perubahan sebelum menyelesaikan update.
10. Simpan plugin tetap private.

Jika plugin dikelola melalui sinkronisasi GitHub, gunakan mekanisme update dari source GitHub yang sama. Jangan mengunggah archive manual di atas plugin yang source-of-truth-nya sudah dikelola GitHub.

### 16.3 Jalur B — Memperbarui GPT Builder yang masih tersedia

1. Buka My GPTs lalu pilih Viral Producer yang sama.
2. Pilih Edit.
3. Buka system/gpt-instructions.md terbaru dari RUNTIME_BRANCH.
4. Ganti seluruh kolom Instructions dengan isi file lengkap tanpa diringkas.
5. Ganti knowledge/reference lama dengan versi terbaru dari:
   - system/content-dna.md;
   - system/data-contract.md;
   - tests/acceptance-tests.md.
6. Pertahankan GitHub App, Web Search, nama, description, dan visibility.
7. Jangan mengunggah data aktif sebagai knowledge.
8. Simpan sebagai Private atau Only me.

Jika kolom Instructions memotong isi file, hentikan dan gunakan Jalur A. Jangan menghapus aturan audit, deduplikasi, verification, lifecycle, atau write safety agar muat.

### 16.4 Mulai percakapan baru

Setelah update selesai:

1. Tutup percakapan lama yang digunakan untuk produksi.
2. Mulai percakapan baru.
3. Pilih atau mention Viral Producer secara eksplisit.
4. Jangan langsung meminta pembuatan post.
5. Jalankan version check read-only terlebih dahulu.

Percakapan lama dapat membawa konteks atau perilaku sebelum update. Hasil validasi resmi harus berasal dari percakapan baru.

### 16.5 Version check read-only

Kirim prompt:

    Read system/gpt-instructions.md, system/content-dna.md, system/data-contract.md, and tests/acceptance-tests.md from {{RUNTIME_REPOSITORY}} using explicit ref {{RUNTIME_BRANCH}}. Verify the repository and ref returned by GitHub. Report the instruction version, editorial version, test specification version, configured repository, branch, runtime mode, ALLOW_WRITES, ALLOW_MAIN_WRITES, and whether generation_audit is required for new drafts. Do not modify anything.

Hasil yang benar:

    Instruction version: 3.0 — Stage 12
    Editorial version: 3.0 — Stage 12
    Test specification version: 3.0 — Stage 12
    Repository: milyarderpro/viral-producer
    Branch: main
    Runtime mode: production
    ALLOW_WRITES: true
    ALLOW_MAIN_WRITES: true
    generation_audit required for new drafts: yes

Periksa GitHub setelah prompt. Lulus hanya jika:

- tidak ada commit baru;
- production-state.json tidak berubah;
- active-drafts.jsonl tidak berubah;
- ready-to-post.md tidak berubah.

Jika satu versi salah atau tidak dapat disebutkan, anggap plugin belum ter-refresh dan jangan menjalankan test tulis.

Pada cutover Stage 12, version check ini adalah preflight langsung untuk AT-24. Setelah PR sudah digabung ke `main`, plugin produksi sudah diperbarui ke v1.1.0, dan percakapan baru sudah dimulai, jalankan AT-24 dari `tests/acceptance-tests.md`. AT-24 harus PASS sebelum caption/hashtag backfill dimulai atau produksi normal dilanjutkan. Jika AT-24 belum PASS, pertahankan production-write pause dan perbaiki/refresh plugin terlebih dahulu.

### 16.6 Saved regression prompts

Prompt berikut hanya untuk regression testing oleh administrator. Jangan jalankan dengan profile produksi; gunakan profile test terisolasi.

#### Geography-history

    Create one United States geography-history post under editorial version 2. Use at least four surprise operators, no more than two record or superlative facts, exact geographic scope, and complete generation_audit. Save it only if every hard gate passes.

#### Animals-nature

    Create one global animals-nature post under editorial version 2. Favor vivid and counterintuitive facts, use at least four surprise operators, avoid familiar internet filler, and persist complete generation_audit. Save it only if every hard gate passes.

#### Body-science

    Create one global body-science post under editorial version 2. Reject basic textbook definitions, favor visual or everyday counterintuitive payoffs, use at least four surprise operators, and persist complete generation_audit. Save it only if every hard gate passes.

Catat post ID yang dihasilkan nanti sebagai geography v2, animals v2, dan body-science v2.

### 16.7 Membandingkan dengan baseline

Gunakan baseline tanpa mengubahnya:

| Baseline | Masalah lama | Target output baru |
|---|---|---|
| P-000001 | Terlalu banyak record dan satu geographic scope drift | Maksimal dua record_superlative, minimal empat operator, scope persis |
| P-000002 | DNA terbaik tetapi beberapa fakta cukup umum | Visuality tetap kuat, novelty meningkat, audit metadata lengkap |
| P-000003 | Terlalu textbook-like dan skor 12/12 terlalu longgar | Tidak ada filler textbook, opening/closing kuat, skor dan rationale realistis |

Jangan membandingkan hanya total skor. Bandingkan:

- operator variety;
- viral-strength distribution;
- kekuatan Facts 1 dan 6;
- scope fidelity;
- source accessibility;
- candidate accounting;
- quality rationales;
- overall tell-someone reaction.

### 16.8 Troubleshooting refresh

Jika version check masih menunjukkan instruksi lama:

- pastikan Anda mengedit plugin yang benar;
- pastikan file lama benar-benar diganti;
- simpan ulang update;
- mulai percakapan baru;
- panggil plugin secara eksplisit;
- ulangi version check tanpa write.

Jika draft baru tidak memiliki generation_audit atau empat field editorial per fakta:

- hentikan test lanjutan;
- jangan approve draft tersebut;
- periksa apakah gpt-instructions.md terpotong;
- periksa apakah content-dna.md dan data-contract.md terbaru tersedia;
- refresh plugin lalu mulai percakapan baru.

Jika plugin meminta permission GitHub yang lebih luas:

- jangan memperluas akses secara otomatis;
- periksa apakah koneksi lama masih tersedia;
- pertahankan akses hanya ke repository yang diperlukan;
- ulangi read-only version check setelah koneksi benar.

## 17. Pemeliharaan Setelah Go-Live

Runtime produksi menggunakan `main`. Test plugin tetap terpisah pada `test/viral-producer-v1.1` dan tidak dipakai untuk pekerjaan produksi.

- Gunakan plugin Viral Producer untuk operasi harian.
- Jalankan acceptance test mutatif hanya dengan profile test terisolasi.
- Buat perubahan sistem pada feature branch, review melalui pull request, lalu merge.
- Setelah perubahan sistem, refresh plugin dan jalankan version check read-only.
- Jangan mengubah data produksi secara manual.

## 18. Hotfix Large-File 1.1.1

Hotfix ini diperlukan ketika `data/active-drafts.jsonl` melewati batas body connector.

Urutan update:

1. Hentikan sementara produksi.
2. Perbarui plugin **Viral Producer** yang sama ke versi 1.1.1; jangan membuat plugin baru.
3. Pertahankan runtime profile, visibility, GitHub binding, permission, dan audience.
4. Pastikan plugin memuat Instruction 3.1 — Stage 12.13, Editorial 3.0 — Stage 12, dan Test Specification 3.1 — Stage 12.13.
5. Mulai chat baru.
6. Jalankan AT-56 dari `tests/acceptance-tests.md` secara read-only.
7. Jangan melanjutkan produksi apabila body blob lengkap tidak dapat dibaca atau AT-56 gagal.
8. Jalankan AT-57 sampai AT-60 hanya pada test runtime terisolasi.

Prompt update singkat:

    Perbarui plugin Viral Producer yang sama ke versi 1.1.1 menggunakan file terbaru dari milyarderpro/viral-producer@main. Pertahankan identitas, private visibility, GitHub connection, permission, audience, dan runtime profile. Terapkan exact-SHA blob fallback untuk authoritative file yang body normalnya kosong atau terpotong, continuity guard Post ID/Fact ID sebelum active-drafts write, Git Data API untuk active file besar, latest-head recheck, dan non-forced fast-forward. Jangan menyentuh data produksi.

