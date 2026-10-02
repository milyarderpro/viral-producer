# Panduan Pengguna Viral Producer

Panduan ini untuk **production plugin Viral Producer** pada branch `main`. Untuk acceptance test gunakan [Viral Producer v1.1 Test](test-plugin-installation.md), bukan plugin produksi.

Naskah Facebook menggunakan natural American English. Penjelasan operasional dapat menggunakan bahasa Indonesia.

Butuh prompt siap salin? Buka [Pustaka Prompt](prompt-library.md).

## 1. Alur Produksi

```text
Buat draft
→ Review/revisi
→ Fast Approval
→ Pilih/jadwalkan ready post
→ Copy script + caption
→ Posting manual
→ Tandai posted
→ Catat performa
```

Viral Producer tidak memposting langsung ke Facebook.

## 2. Pemeriksaan Awal

Mulai dari chat baru dengan production plugin **Viral Producer**.

```text
Periksa versi sistem, runtime profile, branch, dan status repository. Jangan mengubah apa pun.
```

Runtime produksi yang benar:

- repository `milyarderpro/viral-producer`;
- branch `main`;
- mode `production`;
- `ALLOW_WRITES=true`;
- `ALLOW_MAIN_WRITES=true`.

Jika plugin menunjukkan branch test atau mode test, jangan lanjutkan operasi produksi.

## 3. Membuat Post

Setiap post memiliki enam fakta dan publishing package. AI melakukan riset, source checking, duplicate prevention, cooldown, editorial audit, lalu menyimpan draft hanya jika seluruh gate lulus.

### Format default

Jika format tidak ditentukan, sistem menjaga target jangka panjang:

- 75% `themed`;
- 25% `mixed`.

`themed` memakai satu fact topic untuk enam fakta. `mixed` memakai sedikitnya empat fact topic dan maksimal dua fakta dari topic yang sama.

### Contoh

```text
Buatkan satu post baru menggunakan rotasi default. Simpan sebagai draft hanya jika seluruh gate lulus.
```

```text
Buatkan satu post mixed trivia dengan country focus GLOBAL.
```

```text
Buatkan satu post geography-history untuk US.
```

Fact topic yang tersedia:

- `geography-history`
- `animals-nature`
- `body-science`
- `food-home`
- `inventions-records`
- `practical`

Country focus: `US`, `CA`, `UK`, `AU`, atau `GLOBAL`.

## 4. Subject dan Angle Cooldown

Selain permanent duplicate prevention, post baru memakai cooldown untuk mengurangi pengulangan subjek dan cluster yang terlalu dekat.

- klaim yang sama tetap dilarang permanen;
- subjek yang baru dipakai dapat ditolak sementara walaupun klaimnya berbeda;
- sistem membandingkan active reservations, batch saat ini, dan 20 post terarsip terbaru;
- record lama tanpa `subject_key` tetap dapat dibaca melalui fallback tanpa ditulis ulang.

Named-series override hanya boleh dipakai jika Anda secara eksplisit meminta seri bernama. Override tidak pernah membolehkan duplicate claim, sumber lemah, scope drift, atau kegagalan quality gate.

## 5. Memeriksa dan Merevisi Post

Tampilkan post tanpa perubahan:

```text
Tampilkan P-000009 beserta status, format, topic, country focus, ON-SCREEN SCRIPT, dan FACEBOOK CAPTION. Jangan mengubah repository.
```

Tampilkan sumber:

```text
Tampilkan seluruh sumber P-000009 dan jelaskan klaim yang didukung. Jangan mengubah repository.
```

### Revisi wording

Gunakan jika klaim faktual tetap sama.

```text
Buat Fact 3 pada P-000009 lebih punchy tanpa mengubah canonical claim, angka, lokasi, waktu, scope, qualifier, atau Fact ID.
```

### Ganti fakta

Gunakan jika klaim berubah.

```text
Ganti Fact 4 pada P-000009 dengan fakta yang benar-benar berbeda. Riset dan verifikasi penggantinya, lalu perbarui audit yang diwajibkan.
```

Perubahan fakta, topic, country focus, atau format akan memicu recheck publishing package. Wording-only revision boleh mempertahankan caption/hashtag jika semuanya masih valid.

## 6. Fast Approval

Approval di Version 1.1 adalah **Fast Approval**. Untuk draft yang lengkap, approval memvalidasi evidence yang sudah tersimpan dan **tidak**:

- membuka sumber;
- menjalankan Web Search;
- meriset atau memverifikasi fakta dari awal;
- menjalankan global deduplication baru;
- rescore quality;
- membangun ulang `generation_audit`;
- mengalokasikan ID baru;
- meregenerasi caption atau hashtag yang sudah valid.

Prompt:

```text
Approve P-000009 jika record tersimpan memenuhi seluruh Fast Approval gate dan masukkan ke ready queue.
```

Draft legacy atau tidak lengkap harus berhenti sebelum write dan memerlukan revisi/upgrade terpisah.

## 7. Publishing Package dan Ready Queue

Setiap post Version 1.1 baru menyimpan:

- **ON-SCREEN SCRIPT** — hook, enam fakta, dan CTA;
- **FACEBOOK CAPTION** — satu kalimat caption, satu baris kosong, lalu 4–6 hashtag dalam urutan tersimpan.

Hashtag tidak pernah masuk ke on-screen script.

Ambil post berikutnya:

```text
Tampilkan post berikutnya yang siap saya copy. Jangan mengubah repository.
```

Output copy-ready harus memakai dua blok teks terpisah dan sama dengan record/ready queue.

## 8. Smart Recommendation

Untuk memilih ready post tanpa mengubah state:

```text
Rekomendasikan post terbaik untuk diposting berikutnya. Jangan mengubah repository.
```

Jika ada planned slot, sistem memprioritaskan slot terawal. Jika tidak, pemilihan mempertimbangkan rotasi topic/country/format, cooldown, operator overlap, quality, umur ready post, dan performance hanya jika ambang sampelnya sudah cukup.

Recommendation tidak approve, mengedit, menjadwalkan, dequeue, atau menandai post sebagai posted.

## 9. Content Calendar

Scheduling hanya memakai post berstatus `ready`.

### Jadwal tujuh hari

```text
Susun jadwal posting tujuh hari, dua post per hari.
```

Tanpa jam eksplisit, default adalah 12:00 dan 19:00 WIB mulai hari lokal penuh berikutnya. Timestamp disimpan dalam UTC, sedangkan tampilan kalender menggunakan Asia/Jakarta.

### Jadwalkan satu post

```text
Jadwalkan P-000009 besok pukul 19.00 WIB.
```

### Tampilkan kalender

```text
Tampilkan content calendar. Jangan mengubah repository.
```

### Pindahkan jadwal

```text
Pindahkan P-000009 ke 4 Oktober 2026 pukul 12.00 WIB.
```

Scheduling dan move tidak mengubah isi post, ready queue, production revision, ID, atau counter.

## 10. Setelah Posting ke Facebook

Setelah konten benar-benar dipublikasikan:

```text
Saya sudah memposting P-000009 ke Facebook. Tandai sebagai posted dan verifikasi archive, published facts, active drafts, ready queue, publishing plan, calendar, dan production state.
```

Jika post memiliki planned slot, slot menjadi `completed` dan hilang dari active content calendar.

Jangan menandai post sebagai posted sebelum benar-benar dipublikasikan.

## 11. Mencatat Performa

Performance hanya boleh dicatat untuk post yang sudah berada di immutable posted archive.

Metric yang didukung:

- views;
- reactions;
- comments;
- shares;
- average watch time;
- retention percent;
- followers gained.

Contoh:

```text
Catat performa P-000009: 1.2M views, 84K reactions, 2,300 comments, dan 15K shares.
```

Pencatatan performa tidak mengubah content lifecycle, fact ledger, ready queue, production revision, ID, atau counter.

## 12. Ringkasan dan Analisis Performa

Ringkasan:

```text
Tampilkan ringkasan performa konten. Jangan mengubah repository.
```

Analisis:

```text
Analisis pola performa berdasarkan topic, country, format, dan operator. Jangan mengubah repository.
```

Aturan interpretasi:

- kurang dari 15 measured posts: deskriptif saja;
- 15–19: directional observation dengan hati-hati;
- 20 atau lebih: performance boleh menjadi tie-breaker, tetapi tidak boleh melemahkan factual/editorial gate.

## 13. Reject dan Audit

Reject:

```text
Tolak P-000009 dan ikuti lifecycle rejection sesuai data contract.
```

ID yang sudah dikonsumsi tidak digunakan kembali.

Audit read-only:

```text
Audit database, ready queue, publishing package, performance summary, publishing plan, content calendar, dan lifecycle consistency. Jangan memperbaiki apa pun.
```

Jika ditemukan masalah, lihat temuan terlebih dahulu. Repair hanya dilakukan ketika targetnya jelas dan contract mengizinkan.

## 14. Kompatibilitas Record Lama

Record lama tetap dapat dibaca tanpa migrasi otomatis.

- missing `post_format` berarti effective `themed`;
- missing `subject_key` memakai read-time fallback;
- archived legacy post boleh tidak memiliki caption/hashtag;
- active legacy post yang menunggu controlled backfill tidak dianggap corrupt;
- inspection/audit tidak boleh diam-diam menambah field lama.

## 15. Production Plugin vs Test Plugin

Untuk produksi harian gunakan **Viral Producer** pada `main`.

Jangan gunakan **Viral Producer v1.1 Test** untuk produksi. Test plugin hanya untuk branch `test/viral-producer-v1.1`, dan test data tidak pernah dibawa kembali ke `main`.

Acceptance test mutatif tidak dijalankan dengan production plugin.

## 16. Catatan Administrator: AT-24

AT-24 bukan bagian workflow harian. Ia adalah mandatory post-cutover smoke test Stage 12.12.

Urutan wajib:

1. PR Stage 12 sudah di-merge ke `main`.
2. Production plugin diperbarui ke v1.1.0.
3. Mulai chat baru dan lakukan version/profile check read-only.
4. Jalankan AT-24.
5. Jika AT-24 PASS, baru lakukan controlled caption/hashtag backfill.
6. Jalankan final production consistency audit.
7. Resume produksi.

Jika AT-24 belum PASS, produksi tetap dipause dan backfill tidak dimulai.

## 17. Aturan Penggunaan

- Jalankan satu operasi write pada satu waktu.
- Jangan membuat post dari dua chat secara bersamaan.
- Jangan force overwrite saat SHA conflict.
- Jangan edit JSON/JSONL secara manual.
- Gunakan repository state terbaru, bukan ingatan percakapan.
- Untuk perintah read-only, tulis jelas: “Jangan mengubah repository.”
