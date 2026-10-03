# Viral Producer

Viral Producer adalah workflow AI privat untuk membuat, mengelola, menjadwalkan, dan mengevaluasi naskah trivia Facebook Reels yang terverifikasi. Naskah akhir menggunakan natural American English; penjelasan operasional mengikuti bahasa pengguna.

## Mulai di Sini

- [Panduan Pengguna](docs/user-guide.md) — workflow produksi harian.
- [Pustaka Prompt](docs/prompt-library.md) — prompt siap salin untuk operasi produksi.
- [Panduan Instalasi Production Plugin](docs/gpt-installation.md) — memasang atau memperbarui plugin **Viral Producer** pada `main`.
- [Panduan Test Plugin](docs/test-plugin-installation.md) — khusus acceptance test terisolasi pada `test/viral-producer-v1.1`.
- [Acceptance Tests](tests/acceptance-tests.md) — spesifikasi dan evidence pengujian.

## Workflow Produksi 1.1

```text
Buat draft themed atau mixed
→ Review/revisi
→ Fast Approval
→ Pilih atau jadwalkan ready post
→ Copy ON-SCREEN SCRIPT + FACEBOOK CAPTION
→ Posting manual ke Facebook
→ Tandai posted
→ Catat dan analisis performa
```

Fitur Stage 12 mencakup:

- Fast Approval dari evidence yang sudah tersimpan, tanpa riset atau rescoring baru;
- post `themed` dan `mixed`, dengan rotasi default jangka panjang 75% / 25%;
- subject dan angle cooldown untuk mengurangi pengulangan;
- publishing package lengkap: caption satu kalimat dan 4–6 hashtag;
- smart recommendation dan content calendar dengan waktu operasional Asia/Jakarta;
- pencatatan performa posted post dan ringkasan deterministik;
- kompatibilitas dengan record lama tanpa migrasi implisit.

Plugin tidak memposting langsung ke Facebook.

## Production dan Test Tidak Boleh Tertukar

**Production plugin — Viral Producer**

- repository: `milyarderpro/viral-producer`;
- branch: `main`;
- mode: `production`;
- dipakai untuk workflow harian.

**Test plugin — Viral Producer v1.1 Test**

- branch: `test/viral-producer-v1.1`;
- mode: `test`;
- dipakai hanya untuk acceptance test;
- tidak boleh membaca/menulis `main`;
- test data tidak pernah di-merge ke feature branch atau `main`.

Jangan menggunakan test plugin untuk produksi dan jangan menjalankan acceptance test mutatif dengan production plugin.

## Cutover Stage 12

Stage 12.10 sudah menyelesaikan gate pre-cutover: AT-25 sampai AT-55 PASS pada branch test beserta final consistency audit. AT-24 berbeda: ia adalah **mandatory post-cutover smoke test** untuk production plugin v1.1.

Urutan AT-24 wajib:

```text
Merge PR Stage 12 ke main
→ Refresh production plugin ke v1.1
→ Mulai chat baru
→ Jalankan AT-24 read-only
→ AT-24 PASS
→ Baru jalankan controlled backfill
→ Final production consistency audit
→ Resume produksi
```

AT-24 bukan pre-merge gate, tetapi tetap wajib untuk Definition of Done Stage 12.

## Aturan Operasional

- Production branch selalu `main`.
- Jalankan satu operasi `WRITE` pada satu waktu.
- Jangan membuat post dari dua chat secara bersamaan.
- Jangan mengedit JSON/JSONL produksi secara manual.
- Repository dan runtime profile adalah sumber kebenaran; jangan mengandalkan state dari percakapan lama.
- Setiap GitHub operation harus memakai repository/ref eksplisit sesuai runtime profile.
- Body kosong dengan SHA non-empty tidak boleh dianggap sebagai database kosong; plugin wajib mengambil blob lengkap.
- Active file besar memakai continuity guard dan Git Data API non-forced agar record lama tidak terhapus.

## Referensi Teknis

- [GPT Instructions](system/gpt-instructions.md)
- [Data Contract](system/data-contract.md)
- [Content DNA](system/content-dna.md)
- [Acceptance Tests](tests/acceptance-tests.md)
- [Implementation Plan](plan.md)
