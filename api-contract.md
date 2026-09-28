# API Contract — Library API

## User Stories

- Sebagai pengunjung, saya ingin melihat daftar buku agar dapat memilih buku yang tersedia.
- Sebagai petugas, saya ingin menambahkan buku agar koleksi terbaru tersedia untuk anggota.

## Resource Dictionary — Book

| Field | Type | Required saat create | Akses | Aturan | Contoh |
|---|---|---|---|---|---|
| id | integer | Tidak | Read-only | Dibuat server, auto-increment | 15 |
| title | string | Ya | Read/write | 3–200 karakter | Clean Code |
| isbn | string | Ya | Read/write | Unik, 10 atau 13 digit | 9780132350884 |
| author_id | integer | Ya | Write | Author harus tersedia di database | 3 |
| available | boolean | Tidak | Read-only | Dihitung server (default: true) | true |
| created_at | string | Tidak | Read-only | ISO 8601 datetime format (YYYY-MM-DDTHH:mm:ssZ) | 2026-09-25T09:30:00Z |

## Endpoint Matrix

| Kebutuhan | Method | Endpoint | Success | Error |
|---|---|---|---|---|
| Daftar resource | GET | /api/books | 200 | — |
| Membuat resource | POST | /api/books | 201 | 422 |
| Detail resource | GET | /api/books/{book} | 200 | 404 |
| Mengubah resource | PATCH | /api/books/{book} | 200 | 404, 422 |
| Menghapus resource | DELETE | /api/books/{book} | 204 | 404 |

## Request dan Response Contract

### Create Request

```http
POST /api/books
Content-Type: application/json
Accept: application/json
```

```json
{
  "title": "Clean Code",
  "isbn": "9780132350884",
  "author_id": 3
}
```

![Create Request](GAMBAR/Create.png)

### `201 Created`

```json
{
  "data": {
    "id": 15,
    "title": "Clean Code",
    "isbn": "9780132350884",
    "author_id": 3,
    "available": true,
    "created_at": "2026-09-25T09:30:00Z"
  }
}
```

![201 Created](GAMBAR/201%20create.png)

### `200 OK`

```json
{
  "data": [
    {
      "id": 15,
      "title": "Clean Code",
      "isbn": "9780132350884",
      "available": true
    }
  ]
}
```

![200 OK](GAMBAR/200%20OK.png)

### `404 Not Found`

```json
{
  "message": "Book not found",
  "errors": null
}
```

![404 Not Found](GAMBAR/404%20not%20found.png)

### `422 Unprocessable Content`

```json
{
  "message": "The given data was invalid.",
  "errors": {
    "title": [
      "The title field is required."
    ],
    "isbn": [
      "The isbn has already been taken."
    ]
  }
}
```

![422 Unprocessable Content](GAMBAR/422.png)

## Design Decisions

- Memakai PATCH karena client mengubah sebagian field dari resource buku tanpa harus mengirimkan ulang seluruh data.
- Field available bersifat read-only karena status ketersediaan dihitung dan ditentukan oleh sistem server berdasarkan sirkulasi peminjaman.

## Refleksi

1. Keputusan Kontrak yang Paling Berdampak

Bagian yang paling berpengaruh adalah aturan pembungkusan hasil data dari server. Karena aplikasi pembaca sudah disetting untuk mengambil data dari wadah bernama "data" saat berhasil dan wadah "errors" saat gagal, perubahan kecil pada nama wadah atau format jawaban akan membuat aplikasi bingung dan gagal menampilkan informasi.

2. Risiko Perubahan Tipe Data

Mengubah jenis data setelah aplikasi pembaca selesai dibuat dapat menyebabkan aplikasi mendadak menutup sendiri atau rusak (crash). Misalnya, jika kode nomor buku yang tadinya disepakati dalam bentuk teks tiba-tiba dikirim sebagai angka murni, sistem pembaca tidak akan bisa memprosesnya karena perbedaan format tersebut.

3. Penerapan di Laravel

Tabel alamat URL akan digunakan untuk menentukan jalur dan perintah API di Laravel. Aturan pengisian data dipakai untuk memastikan input dari pengguna sudah benar—seperti memastikan judul buku tidak boleh kosong. Sementara itu, format jawaban server dipakai oleh Laravel untuk menyusun ulang data dari basis data agar bentuk akhirnya rapi dan sesuai kesepakatan awal sebelum dikirim.
