# Dashboard Capaian PUD

Dashboard interaktif untuk monitoring capaian Program Unit Diikuti (PUD) — Astra Honda Motor · PQM.
Data disuntik **live** dari Spreadsheet **Database Pengerjaan PUD** dan otomatis di-refresh tiap 5 menit.

## Akses Publik (setelah deploy)

Setelah workflow deploy berhasil, dashboard bisa dibuka semua orang di:

```
https://pqmahm-sketch.github.io/Dashboard-PUD/
```

Cukup buka URL di atas di browser — tidak perlu install apa pun, tidak perlu login GitHub.

---

## Cara Deploy (sekali setup saja)

### 1. Pastikan Spreadsheet Database Pengerjaan PUD publik untuk dibaca

Dashboard membaca data lewat endpoint publik Google Sheets (`gviz/tq`). Supaya bisa diakses tanpa login:

1. Buka spreadsheet **Database Pengerjaan PUD** di Google Sheets.
2. Klik **Share** (kanan atas).
3. Di bagian *General access*, ubah jadi **"Anyone with the link"** — role **Viewer**.
4. Klik **Done**.

Alternatif yang setara: **File → Share → Publish to web → Publish** (biarkan default "Entire Document · Web page").

> ⚠️ Ingat: siapa pun yang tahu URL dashboard bisa lihat data. Kalau datanya sensitif, pertimbangkan hosting internal.

Sheet ID yang dipakai dashboard sudah di-hard-code di `index.html`:

```js
const SHEET_ID = '1h0x6iGnYsquSueq8gKy9pIEXNZ0HqwYcq3BlSgTcu6w';
const SHEET_GID = '0';   // sheet pertama
```

Kalau nanti pindah spreadsheet, cukup ubah dua konstanta itu.

### 2. Aktifkan GitHub Pages di repo ini

1. Buka repo di GitHub → **Settings** → **Pages** (sidebar kiri).
2. Di bagian **Build and deployment · Source**, pilih **GitHub Actions**.
3. Simpan.

Setelah itu, setiap push ke branch `main` akan otomatis men-trigger workflow `.github/workflows/deploy-pages.yml` yang men-deploy `index.html` ke Pages.

### 3. Merge branch feature ke main

Deployment mengambil isi branch `main`. Kalau perubahan masih di branch feature (mis. `claude/dashboard-capaian-pud-feedback-*`), merge dulu ke `main` — bisa via Pull Request atau langsung push.

### 4. Cek workflow

Buka tab **Actions** di repo untuk lihat progres deploy. Kalau job "Deploy Dashboard Capaian PUD to GitHub Pages" sukses (bulet hijau), dashboard sudah live di URL di atas.

---

## Cara Kerja Live-Sync

- Saat halaman dibuka, `index.html` men-inject `<script>` ke `https://docs.google.com/spreadsheets/d/<SHEET_ID>/gviz/tq?...` (JSONP).
- Response berupa JSON dipetakan ke struktur internal `RAW_DATA` lewat `mapGvizRowsToRaw()`.
- Auto-refresh tiap **5 menit** (`REFRESH_INTERVAL_MS`).
- Tombol **↻ Refresh Data** di header untuk refresh manual.

### Format kolom spreadsheet (wajib)

| Kolom | Isi                          | Dipakai sebagai              |
|-------|------------------------------|-------------------------------|
| A     | Nama PUD (mis. "PUD CBR250 RR") | `r.n` / `r.f` (label PUD)   |
| B     | Kode MD Distribusi           | `r.md`                        |
| C     | No. Rangka                   | `r.rk` (wajib — baris kosong di sini di-skip) |
| D     | No. Mesin                    | `r.ms`                        |
| E     | Tanggal Lapor                | `r.tl` (basis grouping bulan)|
| F     | Kode MD Pelapor              | `r.mdp`                       |
| G     | Kode AHASS                   | `r.ah`                        |
| H     | Nama AHASS                   | `r.nah`                       |
| I     | Tanggal Pemeriksaan          | `r.tp`                        |
| J     | Tanggal Pengerjaan           | `r.tk`                        |
| K     | Status                       | `r.st` — **blank = Belum Dikerjakan**, terisi = Sudah Diproses (Pengerjaan / Deklarasi) |

Kalau nanti struktur kolom berubah, sesuaikan `mapGvizRowsToRaw()` di dalam `index.html`.

---

## Struktur Repo

```
Dashboard-PUD/
├── index.html                          ← file dashboard (entry point Pages)
├── .github/
│   └── workflows/
│       └── deploy-pages.yml            ← workflow deploy otomatis
└── README.md
```

---

## Troubleshooting

**Dashboard buka tapi angka 0 semua & pill status merah "Gagal memuat data".**
Spreadsheet belum di-share publik atau Sheet ID salah. Cek langkah 1 di atas.

**Angka KPI "Sudah Diproses" ≠ garis kumulatif.**
Seharusnya sudah tidak terjadi lagi — logika di `renderTrend()` menjamin keduanya sama. Kalau muncul lagi, cek console browser untuk error di `mapGvizRowsToRaw()` (kemungkinan format kolom sumber berubah).

**Deploy gagal di tab Actions.**
Pastikan Pages sudah di-set ke source **GitHub Actions** (bukan "Deploy from a branch"). Cek Settings → Pages.
