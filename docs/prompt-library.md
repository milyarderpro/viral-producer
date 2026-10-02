# Pustaka Prompt Viral Producer

Prompt di bawah ini untuk **production plugin Viral Producer** pada branch `main`, kecuali bagian administrator yang secara eksplisit menyebut test plugin.

- `READ-ONLY`: tidak mengubah repository.
- `WRITE`: dapat membuat commit pada branch runtime.
- Ganti contoh `P-000009`, topic, negara, tanggal, atau metric sesuai kebutuhan.
- Jalankan satu prompt `WRITE` pada satu waktu.

Lihat juga [Panduan Pengguna](user-guide.md).

## 1. Pemeriksaan Awal

### Version dan runtime profile — READ-ONLY

```text
Periksa instruction version, editorial version, test specification version, repository, branch, runtime mode, ALLOW_WRITES, ALLOW_MAIN_WRITES, dan production state. Jangan mengubah apa pun.
```

### Ringkasan produksi — READ-ONLY

```text
Tampilkan revision, next Post ID, next Fact ID, jumlah active post, jumlah ready post, publishing-plan revision, dan performance sample size. Jangan mengubah repository.
```

## 2. Membuat Post

### Rotasi default — WRITE

```text
Buatkan satu post baru menggunakan rotasi format, topic, dan country focus default. Simpan sebagai draft hanya setelah seluruh source, duplicate, cooldown, editorial, audit, dan publishing-package gate lulus.
```

### Themed — WRITE

```text
Buatkan satu post themed animals-nature dengan country focus GLOBAL. Simpan hanya jika seluruh gate lulus.
```

### Mixed — WRITE

```text
Buatkan satu post mixed trivia dengan country focus GLOBAL. Gunakan sedikitnya empat fact topics dan maksimal dua fakta dari topic yang sama.
```

### Country-specific mixed — WRITE

```text
Buatkan satu post mixed untuk Australia. Setiap fakta harus secara eksplisit mendukung scope Australia dan seluruh gate biasa tetap berlaku.
```

### Beberapa post — WRITE

```text
Buatkan tiga post menggunakan rotasi default. Validasi seluruh batch, lalu persist secara serial satu logical operation pada satu waktu. Jangan melakukan parallel repository writes.
```

## 3. Named Series

Gunakan hanya bila memang ingin seri bertema yang dapat meminta cooldown override terbatas.

```text
Buat satu post untuk seri bernama “Grand Canyon Week”. Gunakan series override hanya bila diperlukan untuk subject/cluster cooldown; duplicate claim, source, scope, safety, dan quality gate tetap tidak boleh dilewati.
```

## 4. Menampilkan Post dan Sumber

### Tampilkan post — READ-ONLY

```text
Tampilkan P-000009 beserta status, effective post format, topic, country focus, ON-SCREEN SCRIPT, dan FACEBOOK CAPTION. Jangan mengubah repository.
```

### Sumber — READ-ONLY

```text
Tampilkan seluruh sumber P-000009 dan jelaskan klaim serta qualifier yang didukung setiap sumber. Jangan melakukan riset baru dan jangan mengubah repository.
```

### Audit satu post — READ-ONLY

```text
Audit P-000009 untuk source support, scope, permanent duplicate, cooldown evidence, operator diversity, viral strength, quality rationale, generation audit, caption, dan hashtags. Jangan mengubah repository.
```

## 5. Revisi

### Wording-only — WRITE

```text
Buat Fact 3 pada P-000009 lebih punchy tanpa mengubah canonical claim, angka, lokasi, waktu, scope, qualifier, atau Fact ID. Recheck publishing package dan pertahankan caption/hashtags byte-for-byte jika masih valid.
```

### Ganti fakta — WRITE

```text
Ganti Fact 4 pada P-000009 dengan fakta yang benar-benar berbeda. Riset dan verifikasi penggantinya, jalankan permanent duplicate dan cooldown check, lalu perbarui audit serta publishing package yang terdampak.
```

## 6. Fast Approval

### Approve — WRITE

```text
Approve P-000009 menggunakan Fast Approval. Validasi hanya evidence yang sudah tersimpan; jangan membuka sumber, jangan Web Search, jangan rescore, jangan alokasikan ID baru, dan jangan regenerasi caption/hashtags yang masih valid.
```

### Reject — WRITE

```text
Tolak P-000009 dan ikuti lifecycle rejection sesuai data contract. Jangan menggunakan kembali ID yang sudah dikonsumsi.
```

## 7. Ready Queue dan Publishing Package

### Post berikutnya — READ-ONLY

```text
Tampilkan post berikutnya yang siap saya copy dengan ON-SCREEN SCRIPT dan FACEBOOK CAPTION dalam dua blok terpisah. Jangan mengubah repository.
```

### Seluruh ready queue — READ-ONLY

```text
Tampilkan seluruh ready queue sesuai urutan repository. Jangan mengubah status, schedule, atau queue.
```

### Parity satu ready post — READ-ONLY

```text
Periksa P-000009: pastikan active record, ready queue, caption, hashtag order, dan dua surface copy-ready sama persis. Jangan mengubah repository.
```

## 8. Smart Recommendation

### Rekomendasi berikutnya — READ-ONLY

```text
Rekomendasikan ready post terbaik untuk diposting berikutnya. Hormati planned slot jika ada dan jelaskan evidence pemilihan secara singkat. Jangan approve, edit, schedule, dequeue, atau mark posted.
```

## 9. Content Calendar

### Jadwal tujuh hari — WRITE

```text
Susun jadwal posting tujuh hari, dua post per hari. Gunakan ready posts saja dan default 12:00 serta 19:00 WIB jika jam tidak saya tentukan.
```

### Jadwal satu post — WRITE

```text
Jadwalkan P-000009 besok pukul 19.00 WIB.
```

### Tampilkan calendar — READ-ONLY

```text
Tampilkan content calendar dalam waktu Asia/Jakarta dan verifikasi parity dengan publishing plan. Jangan mengubah repository.
```

### Pindahkan schedule — WRITE

```text
Pindahkan P-000009 ke 4 Oktober 2026 pukul 12.00 WIB. Tolak jika slot terisi atau post tidak lagi ready.
```

## 10. Setelah Posting

### Mark posted — WRITE

```text
Saya sudah memposting P-000009 ke Facebook. Tandai sebagai posted, arsipkan publishing package yang sama persis, selesaikan planned slot bila ada, lalu verifikasi archive, fact ledgers, active drafts, ready queue, calendar, dan production state.
```

### Verifikasi — READ-ONLY

```text
Periksa P-000009 setelah posting: pastikan hanya ada satu archive record, enam published facts, tidak ada active/ready copy, package archive sama dengan final ready package, dan slot terjadwal sudah completed bila ada. Jangan mengubah repository.
```

## 11. Performance Feedback

### Catat performa — WRITE

```text
Catat performa P-000009: 1.2M views, 84K reactions, 2,300 comments, 15K shares, average watch time 8.4 seconds, retention 42%, dan 3,200 followers gained.
```

Minimal satu metric harus diberikan. Hanya archived posted post yang eligible.

### Ringkasan — READ-ONLY

```text
Tampilkan performance summary dan sample size. Verifikasi summary terhadap raw snapshots tanpa melakukan repair.
```

### Analisis — READ-ONLY

```text
Analisis performance berdasarkan topic, country, post format, dan operator. Tampilkan post_count tiap bucket dan terapkan batas sample size 15/20. Jangan mengubah repository.
```

## 12. Audit dan Recovery

### Audit lengkap — READ-ONLY

```text
Audit complete repository state: JSON/JSONL, IDs, signatures, lifecycle, ready queue, publishing package, cooldown evidence, performance data/summary, publishing plan, content calendar, archive/fact linkage, counters, dan unresolved partial operation. Jangan memperbaiki apa pun.
```

### Partial operation — READ-ONLY

```text
Periksa apakah ada partial lifecycle, performance, schedule, atau derived-data operation yang belum selesai. Laporkan confirmed writes dan tindakan recovery tanpa melakukan write.
```

### Deterministic recovery — WRITE

```text
Lakukan hanya deterministic recovery yang diizinkan data contract untuk inconsistency yang sudah dikonfirmasi. Gunakan state/SHA terbaru, jangan mengalokasikan ID baru kecuali contract memang mensyaratkannya, dan verifikasi seluruh postcondition.
```

## 13. Konflik SHA

### Diagnosis — READ-ONLY

```text
Operasi sebelumnya mengalami SHA conflict atau hasil tidak jelas. Baca ulang target branch dan tentukan write mana yang sudah berhasil. Jangan mengulang write dan jangan mengubah file.
```

### Restart logical operation — WRITE

```text
Muat ulang seluruh state dan SHA terbaru, hitung ulang logical operation dari awal, pertahankan competing writer dan ID yang sudah dikonsumsi, lalu lanjutkan hanya jika data contract mengizinkan.
```

## 14. Administrator — Acceptance Test

Acceptance test mutatif **tidak** dijalankan dengan production plugin.

Gunakan plugin **Viral Producer v1.1 Test** pada `test/viral-producer-v1.1` untuk AT-25 sampai AT-55 dan test mutatif lainnya sesuai specification.

### Menjalankan test pada test plugin

```text
Baca tests/acceptance-tests.md dari configured test branch dan jalankan hanya AT-XX sesuai precondition-nya. Jangan jalankan test lain.
```

### AT-24 — production post-cutover smoke test

AT-24 adalah pengecualian karena menguji production runtime. Jalankan **hanya** setelah:

1. PR Stage 12 di-merge ke `main`;
2. production plugin diperbarui ke v1.1.0;
3. production writers dipause;
4. chat baru dimulai dan version/profile check read-only lulus.

```text
Jalankan hanya AT-24 sesuai tests/acceptance-tests.md. Jangan mengubah repository.
```

AT-24 harus PASS sebelum controlled backfill atau produksi dilanjutkan.

## 15. Prompt yang Harus Dihindari

Hindari permintaan ambigu seperti:

```text
Perbaiki semuanya.
```

```text
Posting sekarang.
```

```text
Pakai branch lain untuk kali ini.
```

Untuk operasi record-specific, sebutkan Post ID dan tindakan yang jelas. Untuk read-only, tambahkan “Jangan mengubah repository.”
