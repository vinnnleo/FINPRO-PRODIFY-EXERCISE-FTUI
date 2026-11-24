# Tenggat — Papan Deadline dengan Pengingat

Aplikasi to-do berbasis **Next.js (App Router)** yang menaruh setiap tugas di **garis waktu** sesuai jam deadline-nya, dengan garis merah "sekarang" yang terus bergerak. Di bawahnya, tugas dikelompokkan menurut sisa waktu (*Lewat deadline*, *24 jam ke depan*, *Minggu ini*, *Nanti*), dan aplikasi mengirim pengingat lewat **Web Notifications API** serta notifikasi di dalam halaman.

Final project bootcamp **Prodify — EXERCISE FTUI**.

---

## Fitur

- **Garis waktu 7 hari** — tugas diletakkan tepat di jam deadline-nya; label disusun otomatis supaya tidak bertumpuk. Klik label untuk mengedit.
- **Ringkasan dalam satu kalimat** — jumlah tugas aktif, yang lewat deadline, dan tugas berikutnya beserta sisa waktunya.
- **Daftar berdasarkan sisa waktu** — tugas otomatis pindah kelompok seiring waktu berjalan, dengan hitung mundur per detik.
- **CRUD lengkap** — tambah cepat (judul + deadline), edit lewat panel samping, tandai selesai, hapus (dengan konfirmasi).
- **Prioritas & kategori** — Rendah/Sedang/Tinggi; Kuliah, Organisasi, Lomba, Pribadi, Lainnya.
- **Pencarian & filter kategori**.
- **Dua jenis pengingat**:
  - *Pengingat awal* — 5 menit s.d. 1 hari sebelum deadline (bisa juga dimatikan).
  - *Deadline tiba* — saat waktu deadline tercapai.
- **Pengingat dijadwalkan ulang otomatis** jika deadline atau waktu pengingat diedit.
- **Judul tab** menampilkan jumlah tugas yang terlewat, mis. `(2) Tenggat`.
- **Validasi di server** — input yang salah ditolak dengan pesan error yang jelas (HTTP 400/404).
- Data tersimpan di `tasks.json` (ditulis lewat file sementara agar tidak rusak).

## Teknologi

| Bagian | Yang dipakai |
| --- | --- |
| Framework | Next.js 16 (App Router), React 19 |
| Styling | Tailwind CSS v4 |
| HTTP client | Axios |
| Notifikasi | Web Notifications API |
| Penyimpanan | Node.js `fs` → `tasks.json` |

## Struktur Folder

```
app/
  layout.js                 layout utama + font Bricolage Grotesque
  page.js                   halaman utama → <Board />
  globals.css               token warna + font
  components/
    Board.js                state utama, polling pengingat, tata letak halaman
    Timeline.js             garis waktu 7 hari + penyusunan label
    TaskForm.js             form tambah cepat & edit lengkap (dipakai ulang)
    TaskRow.js              satu baris tugas + hitung mundur
    Toasts.js               notifikasi di dalam halaman
  api/
    tasks/route.js          GET (list, filter, cari), POST
    tasks/[id]/route.js     GET, PATCH, DELETE
    reminders/route.js      GET pengingat yang harus muncul sekarang
    reminders/ack/route.js  POST tandai pengingat sudah dikirim
lib/
  store.js                  baca/tulis tasks.json, validasi, logika pengingat
  time.js                   format waktu, hitung mundur, pembagian kolom
  constants.js              kategori & prioritas
tasks.json                  data
```

## Cara Menjalankan

```bash
npm install
npm run dev
```

Buka `http://localhost:3000` (disarankan Chrome atau Edge).

## Cara Kerja Pengingat

1. Setiap 10 detik, halaman memanggil `GET /api/reminders`.
2. Server memeriksa setiap tugas yang belum selesai:
   - jika `sekarang ≥ deadline` dan belum dikirim → pengingat **due**
   - jika `sekarang ≥ deadline − remindBefore` dan belum dikirim → pengingat **early**
3. Halaman menampilkan toast + notifikasi browser, lalu memanggil `POST /api/reminders/ack` supaya pengingat yang sama tidak muncul lagi.

### Tips menguji

1. Klik **Nyalakan notifikasi** lalu pilih **Allow**.
2. Di Windows, pastikan notifikasi browser aktif (*Settings → System → Notifications*) dan *Do Not Disturb* mati.
3. Biarkan tab tetap terbuka (pengingat dicek dari halaman).
4. Uji cepat: buat tugas dengan deadline **±7 menit dari sekarang** dan pengingat **5 menit sebelumnya** (buka "Tambah catatan, kategori, prioritas, pengingat"). Pengingat awal muncul ±2 menit kemudian, lalu notifikasi deadline saat waktunya tiba.

## API

| Method | Endpoint | Keterangan |
| --- | --- | --- |
| `GET` | `/api/tasks` | Semua tugas, urut deadline. Query opsional: `status=active\|done`, `q=kata` |
| `POST` | `/api/tasks` | Tambah tugas. Body: `title`, `dueAt`, `notes?`, `remindBefore?`, `priority?`, `category?` |
| `GET` | `/api/tasks/:id` | Detail satu tugas |
| `PATCH` | `/api/tasks/:id` | Ubah sebagian field (termasuk `completed`) |
| `DELETE` | `/api/tasks/:id` | Hapus tugas (404 jika tidak ada) |
| `GET` | `/api/reminders` | Pengingat yang harus ditampilkan sekarang |
| `POST` | `/api/reminders/ack` | Tandai pengingat terkirim. Body: `id`, `type` (`early`/`due`) |

Contoh data tugas:

```json
{
  "id": "6f1c…",
  "title": "Laporan Praktikum Modul 4",
  "notes": "Analisis data + kesimpulan",
  "category": "Kuliah",
  "priority": "high",
  "dueAt": "2026-11-28T15:00:00.000Z",
  "remindBefore": 60,
  "completed": false,
  "completedAt": null,
  "earlySent": false,
  "dueSent": false,
  "createdAt": "2026-11-24T12:00:00.000Z",
  "updatedAt": "2026-11-24T12:00:00.000Z"
}
```
