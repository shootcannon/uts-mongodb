# MongoDB Projection — Panduan Lengkap

Panduan belajar **projection** di MongoDB menggunakan `mongosh`. Projection membantu kita menentukan field mana yang ingin ditampilkan dari hasil query.

## Daftar Isi

- [1. Apa Itu Projection?](#1-apa-itu-projection)
- [2. Sintaks Dasar](#2-sintaks-dasar)
- [3. Inclusion Projection](#3-inclusion-projection)
- [4. Exclusion Projection](#4-exclusion-projection)
- [5. Field `_id`](#5-field-_id)
- [6. Projection dengan Filter](#6-projection-dengan-filter)
- [7. Projection pada Nested Documents](#7-projection-pada-nested-documents)
- [8. Projection pada Array](#8-projection-pada-array)
- [9. Projection dengan Operator Lanjutan](#9-projection-dengan-operator-lanjutan)
- [10. Kesalahan Umum](#10-kesalahan-umum)
- [11. Latihan](#11-latihan)
- [12. Ringkasan](#12-ringkasan)
- [13. Referensi Resmi](#13-referensi-resmi)

---

## 1. Apa Itu Projection?

Projection adalah fitur MongoDB untuk memilih field yang dikembalikan dalam hasil query.

Misalnya sebuah dokumen mahasiswa:

```javascript
{
  _id: ObjectId("..."),
  nama: "Agung",
  umur: 19,
  jurusan: "Informatika",
  nilai: 85,
  aktif: true
}
```

Kalau hanya membutuhkan nama dan nilai, projection dapat membuat hasil query lebih ringkas.

```javascript
db.mahasiswa.find(
  {},
  { nama: 1, nilai: 1 }
)
```

Hasilnya kurang lebih:

```javascript
{
  _id: ObjectId("..."),
  nama: "Agung",
  nilai: 85
}
```

`_id` masih muncul karena MongoDB menyertakannya secara default. Untuk menghilangkannya, tulis `_id: 0`.

> Projection mengatur field yang dikembalikan, bukan menghapus field dari dokumen asli di database.

## 2. Sintaks Dasar

Bentuk umum:

```javascript
db.collection.find(
  { filter },
  { projection }
)
```

Contoh:

```javascript
db.mahasiswa.find(
  { aktif: true },
  { nama: 1, jurusan: 1, _id: 0 }
)
```

- `{ aktif: true }` adalah **filter**: hanya mahasiswa aktif yang dicari.
- `{ nama: 1, jurusan: 1, _id: 0 }` adalah **projection**: hanya nama dan jurusan yang ditampilkan, tanpa `_id`.

Ingat perbedaannya:
- **Filter** menentukan dokumen mana yang cocok.
- **Projection** menentukan field mana yang dikembalikan dari dokumen yang cocok.

## 3. Inclusion Projection

Inclusion berarti memilih field yang ingin ditampilkan. Gunakan nilai `1`.

```javascript
db.mahasiswa.find(
  {},
  {
    nama: 1,
    umur: 1
  }
)
```

Hanya field `nama`, `umur`, dan `_id` yang akan ditampilkan.

Untuk menyembunyikan `_id`:

```javascript
db.mahasiswa.find(
  {},
  {
    nama: 1,
    umur: 1,
    _id: 0
  }
)
```

### Contoh lain

Hanya menampilkan nama, jurusan, dan nilai:

```javascript
db.mahasiswa.find(
  {},
  {
    nama: 1,
    jurusan: 1,
    nilai: 1,
    _id: 0
  }
)
```

Field lain seperti `umur` dan `aktif` tidak ditampilkan.

## 4. Exclusion Projection

Exclusion berarti menyembunyikan field tertentu. Gunakan nilai `0`.

Contoh: sembunyikan field `aktif`:

```javascript
db.mahasiswa.find(
  {},
  {
    aktif: 0
  }
)
```

Semua field lain akan tetap ditampilkan, kecuali `aktif`.

Contoh: sembunyikan `aktif` dan `jurusan`:

```javascript
db.mahasiswa.find(
  {},
  {
    aktif: 0,
    jurusan: 0
  }
)
```

### Perbedaan inclusion dan exclusion

| Tujuan | Contoh | Hasil |
|---|---|---|
| Memilih field | `{ nama: 1, nilai: 1 }` | Hanya field yang dipilih, plus `_id` secara default |
| Menyembunyikan field | `{ aktif: 0 }` | Semua field kecuali `aktif` |
| Memilih field tanpa `_id` | `{ nama: 1, _id: 0 }` | Hanya `nama` |
| Menyembunyikan beberapa field | `{ aktif: 0, jurusan: 0 }` | Semua field kecuali keduanya |

**Aturan penting:** biasanya jangan mencampur inclusion (`1`) dan exclusion (`0`) dalam projection yang sama. Pengecualian utamanya adalah `_id`, yang boleh disembunyikan dalam inclusion projection.

Valid:

```javascript
db.mahasiswa.find(
  {},
  { nama: 1, nilai: 1, _id: 0 }
)
```

Tidak valid sebagai projection biasa:

```javascript
// Jangan mencampur inclusion dan exclusion seperti ini.
db.mahasiswa.find(
  {},
  { nama: 1, aktif: 0 }
)
```

Pilih salah satu pendekatan: tentukan field yang ingin ditampilkan, atau tentukan field yang ingin disembunyikan.

## 5. Field `_id`

MongoDB menambahkan field `_id` pada dokumen dan secara default menyertakannya dalam hasil query.

Contoh:

```javascript
db.mahasiswa.find(
  {},
  { nama: 1 }
)
```

Hasil tetap mencakup `_id` dan `nama`.

Jika `_id` tidak diperlukan:

```javascript
db.mahasiswa.find(
  {},
  { nama: 1, _id: 0 }
)
```

Kamu juga bisa melakukan exclusion terhadap `_id` saja:

```javascript
db.mahasiswa.find(
  {},
  { _id: 0 }
)
```

## 6. Projection dengan Filter

Filter dan projection sering digunakan bersamaan.

### Contoh 1: Mahasiswa aktif, tampilkan nama saja

```javascript
db.mahasiswa.find(
  { aktif: true },
  { nama: 1, _id: 0 }
)
```

### Contoh 2: Nilai minimal 80, tampilkan nama dan nilai

```javascript
db.mahasiswa.find(
  { nilai: { $gte: 80 } },
  { nama: 1, nilai: 1, _id: 0 }
)
```

### Contoh 3: Jurusan Informatika, tampilkan nama dan umur

```javascript
db.mahasiswa.find(
  { jurusan: "Informatika" },
  { nama: 1, umur: 1, _id: 0 }
)
```

Urutan berpikirnya:
1. Tentukan dokumen yang ingin dicari melalui filter.
2. Tentukan field yang ingin ditampilkan melalui projection.

## 7. Projection pada Nested Documents

Nested document adalah objek yang berada di dalam objek lain.

Contoh dokumen:

```javascript
{
  nama: "Agung",
  alamat: {
    kota: "Surabaya",
    provinsi: "Jawa Timur",
    kodePos: "602xx"
  }
}
```

### 7.1 Menampilkan seluruh objek `alamat`

```javascript
db.mahasiswa.find(
  {},
  { nama: 1, alamat: 1, _id: 0 }
)
```

### 7.2 Menampilkan hanya field `alamat.kota`

Gunakan dot notation:

```javascript
db.mahasiswa.find(
  {},
  {
    nama: 1,
    "alamat.kota": 1,
    _id: 0
  }
)
```

Hasilnya kurang lebih:

```javascript
{
  nama: "Agung",
  alamat: {
    kota: "Surabaya"
  }
}
```

Perhatikan penggunaan tanda kutip pada `"alamat.kota"`.

## 8. Projection pada Array

Projection juga bisa mengatur bagian tertentu dari array yang ditampilkan.

Contoh dokumen:

```javascript
{
  nama: "Agung",
  hobi: ["coding", "gaming", "musik"]
}
```

### 8.1 Menampilkan seluruh array

```javascript
db.mahasiswa.find(
  {},
  { nama: 1, hobi: 1, _id: 0 }
)
```

Field `hobi` ditampilkan beserta seluruh elemennya.

### 8.2 Menggunakan `$slice`

`$slice` membatasi jumlah elemen array yang dikembalikan dalam hasil.

Ambil dua elemen pertama:

```javascript
db.mahasiswa.find(
  {},
  {
    nama: 1,
    hobi: { $slice: 2 },
    _id: 0
  }
)
```

Ambil dua elemen terakhir:

```javascript
db.mahasiswa.find(
  {},
  {
    nama: 1,
    hobi: { $slice: -2 },
    _id: 0
  }
)
```

Ambil dua elemen mulai dari indeks 1:

```javascript
db.mahasiswa.find(
  {},
  {
    nama: 1,
    hobi: { $slice: [1, 2] },
    _id: 0
  }
)
```

Pada bentuk `[1, 2]`, angka pertama adalah posisi awal (*skip*) dan angka kedua adalah jumlah elemen yang dikembalikan.

### 8.3 Projection dengan `$elemMatch`

`$elemMatch` pada projection mengembalikan **elemen array pertama** yang cocok dengan kondisi projection.

Contoh dokumen:

```javascript
{
  nama: "Agung",
  ujian: [
    { mataKuliah: "Basis Data", nilai: 70 },
    { mataKuliah: "Jaringan", nilai: 90 }
  ]
}
```

Query:

```javascript
db.mahasiswa.find(
  {},
  {
    nama: 1,
    ujian: {
      $elemMatch: { nilai: { $gte: 80 } }
    },
    _id: 0
  }
)
```

Projection tersebut mengembalikan elemen `ujian` pertama yang memenuhi `nilai >= 80`, bukan seluruh array.

**Jangan tertukar:** `$elemMatch` pada filter menentukan apakah dokumen cocok; `$elemMatch` pada projection menentukan elemen array yang ditampilkan.

## 9. Projection dengan Operator Lanjutan

### 9.1 Menggunakan `$meta`

Projection dapat menampilkan metadata tertentu, misalnya skor pencarian teks dari text index.

Contoh konseptual:

```javascript
db.artikel.find(
  { $text: { $search: "mongodb" } },
  {
    judul: 1,
    skor: { $meta: "textScore" }
  }
).sort({
  skor: { $meta: "textScore" }
})
```

Contoh ini membutuhkan collection `artikel` dengan field teks yang sesuai dan **text index**. Ini bukan contoh yang bisa langsung dijalankan pada collection `mahasiswa` tanpa persiapan.

### 9.2 Menggunakan `$project` pada aggregation

Projection dalam `find()` berbeda dengan stage `$project` pada aggregation pipeline.

Contoh:

```javascript
db.mahasiswa.aggregate([
  {
    $project: {
      _id: 0,
      nama: 1,
      nilai: 1
    }
  }
])
```

`$project` digunakan sebagai tahap dalam aggregation pipeline. Selain memilih field, `$project` juga dapat membentuk field baru dengan ekspresi aggregation.

Contoh membuat field baru:

```javascript
db.mahasiswa.aggregate([
  {
    $project: {
      _id: 0,
      nama: 1,
      nilai: 1,
      lulus: { $gte: ["$nilai", 70] }
    }
  }
])
```

Hasilnya mencakup field `lulus` yang bernilai `true` jika `nilai >= 70`, atau `false` jika tidak.

## 10. Kesalahan Umum

### Kesalahan 1: Mengira projection menghapus data

Projection hanya menentukan field yang dikembalikan pada hasil query. Projection tidak mengubah dokumen asli di database.

### Kesalahan 2: Lupa `_id`

```javascript
db.mahasiswa.find({}, { nama: 1 })
```

Hasil tetap menyertakan `_id`. Gunakan `{ nama: 1, _id: 0 }` jika `_id` tidak dibutuhkan.

### Kesalahan 3: Mencampur inclusion dan exclusion

Hindari:

```javascript
db.mahasiswa.find({}, { nama: 1, aktif: 0 })
```

Gunakan inclusion:

```javascript
db.mahasiswa.find({}, { nama: 1, _id: 0 })
```

Atau gunakan exclusion:

```javascript
db.mahasiswa.find({}, { aktif: 0 })
```

### Kesalahan 4: Salah menempatkan projection

Projection adalah argumen kedua dari `find()`:

```javascript
db.mahasiswa.find(
  { aktif: true },
  { nama: 1, _id: 0 }
)
```

Argumen pertama adalah filter; argumen kedua adalah projection.

## 11. Latihan

Kerjakan menggunakan collection `mahasiswa` kamu.

### Level 1 — Dasar

1. Tampilkan hanya field `nama` untuk semua mahasiswa, tanpa `_id`.
2. Tampilkan `nama` dan `jurusan`, tanpa `_id`.
3. Sembunyikan field `aktif` dari semua hasil.
4. Cari mahasiswa aktif, lalu tampilkan hanya `nama` dan `nilai`.

### Level 2 — Filter dan nested documents

5. Cari mahasiswa dengan nilai minimal 80, tampilkan `nama` dan `nilai` saja tanpa `_id`.
6. Jika dokumen memiliki `alamat.kota`, tampilkan `nama` dan hanya `alamat.kota`.
7. Jika dokumen memiliki array `hobi`, tampilkan `nama` dan dua elemen pertama dari `hobi`.

### Level 3 — Lanjutan

8. Dengan dokumen yang memiliki array `ujian`, gunakan projection `$elemMatch` untuk menampilkan elemen pertama dengan `nilai >= 80`.
9. Buat aggregation yang hanya menampilkan `nama` dan `nilai`, tanpa `_id`.
10. Buat aggregation yang menampilkan `nama`, `nilai`, dan field baru `lulus` yang bernilai `true` jika `nilai >= 70`.

## 12. Ringkasan

| Konsep | Contoh | Fungsi |
|---|---|---|
| Inclusion | `{ nama: 1, nilai: 1 }` | Memilih field yang ditampilkan |
| Exclusion | `{ aktif: 0 }` | Menyembunyikan field tertentu |
| Hilangkan `_id` | `{ nama: 1, _id: 0 }` | Menampilkan `nama` tanpa `_id` |
| Nested field | `{ "alamat.kota": 1 }` | Memilih field di dalam objek |
| `$slice` | `{ hobi: { $slice: 2 } }` | Membatasi elemen array yang dikembalikan |
| `$elemMatch` | `{ ujian: { $elemMatch: { nilai: { $gte: 80 } } } }` | Mengembalikan elemen array pertama yang cocok |
| `$project` | Stage dalam `aggregate()` | Memilih, membentuk, atau membuat field hasil aggregation |

### Rumus mudah diingat

- `1` = sertakan field.
- `0` = sembunyikan field.
- `_id` disertakan secara default.
- Inclusion dan exclusion tidak boleh dicampur dalam projection biasa, kecuali pengecualian khusus `_id`.
- Filter menentukan dokumen; projection menentukan field yang dikembalikan.

## 13. Referensi Resmi

- [MongoDB Manual — Project Fields to Return from Query](https://www.mongodb.com/docs/manual/tutorial/project-fields-from-query-results/)
- [MongoDB Manual — Projection Operators](https://www.mongodb.com/docs/manual/reference/operator/projection/)
- [MongoDB Manual — `$slice` Projection Operator](https://www.mongodb.com/docs/manual/reference/operator/projection/slice/)
- [MongoDB Manual — `$elemMatch` Projection Operator](https://www.mongodb.com/docs/manual/reference/operator/projection/elemMatch/)
- [MongoDB Manual — `$project` Aggregation Stage](https://www.mongodb.com/docs/manual/reference/operator/aggregation/project/)

---

README ini berfokus pada projection MongoDB. Simpan sebagai `README.md` di repository khusus projection, atau gunakan nama seperti `mongodb-projection.md` jika digabungkan ke repository catatan belajar MongoDB.
