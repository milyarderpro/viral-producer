# Viral Producer

AI untuk membuat dan mengelola naskah trivia Facebook Reels yang terverifikasi, unik, dan siap diposting.

Naskah akhir menggunakan natural American English. Penjelasan kepada pengguna menggunakan bahasa yang dipakai dalam chat.

## Mulai di Sini

- [Panduan Pengguna](docs/user-guide.md) — cara kerja dan workflow harian.
- [Pustaka Prompt](docs/prompt-library.md) — prompt lengkap yang siap disalin.
- [Panduan Instalasi](docs/gpt-installation.md) — pemasangan plugin dan koneksi GitHub.

## Workflow Singkat

```text
Buat draft
→ Review dan revisi
→ Approve
→ Copy dari ready queue
→ Posting manual ke Facebook
→ Tandai sebagai posted
```

Viral Producer meriset dan memverifikasi fakta, memeriksa duplikasi, menyimpan state di GitHub, serta mengelola lifecycle post. Plugin tidak memposting langsung ke Facebook.

## Aturan Penting

- Production branch: `main`.
- Jalankan satu perintah `WRITE` pada satu waktu.
- Jangan membuat post dari dua chat secara bersamaan.
- Jangan mengedit file JSON atau JSONL secara manual.
- Gunakan hanya plugin **Viral Producer**, bukan plugin tes lama.

## Referensi Teknis

- [GPT Instructions](system/gpt-instructions.md)
- [Data Contract](system/data-contract.md)
- [Content DNA](system/content-dna.md)
- [Acceptance Tests](tests/acceptance-tests.md)
- [Implementation Plan](plan.md)
