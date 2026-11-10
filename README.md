# Otomasi Meeting-to-Task dengan n8n

Workflow n8n yang mengubah transkrip meeting menjadi daftar tugas terstruktur. Sebelum disimpan, daftar tugas harus disetujui manusia lewat email. Tugas yang disetujui disimpan ke Google Sheets, dan setiap proses dicatat di log.

> Proyek demo dengan data fiktif. Bukan pekerjaan klien.

*English summary: an n8n workflow that extracts action items from a meeting transcript with an LLM, requires human approval by email, saves approved tasks to Google Sheets, and logs every run.*

## Demo

[Tonton demo (2–3 menit)](TEMPEL_LINK_VIDEO_DI_SINI)

![Kanvas workflow](docs/workflow.png)

## Masalah yang diselesaikan

Tugas hasil meeting sering terlambat dicatat atau terlewat. Ringkasan buatan AI juga bisa keliru. Workflow ini mengekstrak tugas secara otomatis, tetapi tidak menyimpan apa pun sebelum manusia menyetujuinya.

## Siapa yang terlibat

- **Pengisi form:** orang yang memegang transkrip meeting, misalnya notulen. Ia menempel transkrip ke form.
- **Penyetuju:** orang yang bertanggung jawab atas hasil meeting, misalnya manajer proyek. Ia menerima email berisi ringkasan dan daftar tugas, lalu memilih setuju atau tolak.
- **Penanggung jawab tugas:** orang yang namanya disebut di transkrip. Namanya dicatat AI di kolom `owner`.

Pengisi form dan penyetuju boleh orang yang sama.

## Cara kerja

Form → LLM → Validasi → Persetujuan manusia → Google Sheets → Log

1. **Form:** pengisi form memasukkan judul meeting, tanggal, dan transkrip.
2. **LLM (Gemini):** model membuat ringkasan dan daftar tugas dalam format JSON. Setiap tugas berisi task, owner, tenggat, prioritas, kutipan sumber, dan penanda `needs_review`.
3. **Validasi:** node Code memeriksa struktur JSON. Hasil yang kosong atau rusak ditolak.
4. **Persetujuan manusia:** penyetuju menerima email dan memilih setuju atau tolak.
5. **Penyimpanan:** tugas yang disetujui ditambahkan ke sheet `Tugas`.
6. **Log:** hasil setiap proses ditulis ke sheet `Log`.

Prompt melarang model menebak. Jika owner atau tenggat tidak disebut di transkrip, nilainya diisi "tidak disebutkan". Tugas yang diragukan model diberi tanda "PERLU DICEK" di email persetujuan.

## Cakupan

Proyek ini mengambil bagian tengah dari rantai otomasi meeting berikut:

Meeting → Transkrip → Ringkasan → Action Items → Penugasan → Follow-up

Status setiap tahap:

- **Meeting → Transkrip:** belum. Transkrip ditempel manual lewat form.
- **Ringkasan:** sudah.
- **Action Items:** sudah, berupa JSON terstruktur yang divalidasi.
- **Penugasan:** sebagian. Nama penanggung jawab dicatat di Sheets, tetapi penanggung jawab belum dikabari otomatis setelah tugas disetujui.
- **Follow-up:** belum.

Dua keputusan desain utama:

- **Persetujuan manusia:** AI hanya mengusulkan tugas. Penyimpanan baru terjadi setelah penyetuju menyetujui.
- **Log proses:** setiap proses dicatat, baik yang sukses, gagal, maupun ditolak. Log ini dipakai untuk memantau performa workflow.

## Penanganan error

Yang terjadi saat error:

- **Panggilan LLM gagal atau timeout:** dicoba ulang maksimal 4 kali (jeda 5 detik), lalu dicatat sebagai "gagal LLM".
- **Hasil LLM kosong atau tidak valid:** dicatat sebagai "gagal validasi" beserta pesan errornya.
- **Penyetuju menolak atau tidak merespons:** dicatat sebagai "ditolak atau tidak ada respon".
- **Penyimpanan ke Sheets gagal:** dicatat sebagai "gagal simpan".

## Cara memasang

1. Import file `Demo_AI_Meeting-to-Task.json` ke n8n.
2. Buat tiga credential dan pasang ke node yang sesuai:
   - **Header Auth** untuk node LLM. Isi nama header `x-goog-api-key` dan nilai berupa API key Gemini milikmu.
   - **Gmail OAuth2** untuk node persetujuan.
   - **Google Sheets OAuth2** untuk node penyimpanan dan log.
3. Buat satu spreadsheet dengan dua sheet:
   - `Tugas`, baris 1: `task, owner, due_date, priority, source_quote, needs_review, meeting_title, meeting_date`
   - `Log`, baris 1: `waktu, judul_meeting, status, jumlah_action_items, detail`
4. Ganti `GANTI_DENGAN_ID_SPREADSHEET` dengan ID spreadsheet-mu. Ganti `GANTI_DENGAN_EMAIL_PENYETUJU` dengan email penyetuju.
5. Buka form, tempel transkrip **fiktif**, lalu jalankan.

Daftar nilai yang perlu disiapkan ada di file `.env.example`.

## Teknologi

n8n, Gemini API, Gmail, Google Sheets

## Keterbatasan dan rencana

- Hanya satu penyetuju, dan persetujuan hanya lewat email.
- Baru diuji dengan transkrip fiktif.
- Rencana pengembangan: konversi audio ke transkrip, pengiriman tugas ke penanggung jawab setelah disetujui (misalnya lewat email atau Slack), dan pengingat follow-up terjadwal.
