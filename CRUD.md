# MongoDB CRUD — Panduan Lengkap

Panduan belajar **CRUD** di MongoDB menggunakan `mongosh`. CRUD adalah singkatan dari **Create**, **Read**, **Update**, dan **Delete** — empat operasi dasar untuk mengelola data di database.

## Daftar Isi

- [1. Apa Itu CRUD?](#1-apa-itu-crud)
- [2. Persiapan — Setup Collection](#2-persiapan--setup-collection)
- [3. Create — Menyimpan Data](#3-create--menyimpan-data)
- [4. Read — Membaca Data](#4-read--membaca-data)
- [5. Update — Mengubah Data](#5-update--mengubah-data)
- [6. Delete — Menghapus Data](#6-delete--menghapus-data)
- [7. Filter Lanjutan](#7-filter-lanjutan)
- [8. Opsi Tambahan](#8-opsi-tambahan)
- [9. Operasi Bulk](#9-operasi-bulk)
- [10. Kesalahan Umum](#10-kesalahan-umum)
- [11. Latihan](#11-latihan)
- [12. Ringkasan](#12-ringkasan)
- [13. Referensi Resmi](#13-referensi-resmi)

---

## 1. Apa Itu CRUD?

CRUD adalah empat operasi fundamental dalam pengelolaan data:

| Singkatan | Operasi | Perintah MongoDB |
|---|---|---|
| **C** | Create | `insertOne()`, `insertMany()` |
| **R** | Read | `find()`, `findOne()` |
| **U** | Update | `updateOne()`, `updateMany()`, `replaceOne()` |
| **D** | Delete | `deleteOne()`, `deleteMany()` |

Semua operasi ini bekerja pada **collection** — kumpulan dokumen di MongoDB, mirip seperti tabel di database relasional.

> Dokumen di MongoDB disimpan dalam format BSON (Binary JSON). Setiap dokumen memiliki field `_id` unik yang ditambahkan secara otomatis jika tidak disertakan.

---

## 2. Persiapan — Setup Collection

### Memilih atau membuat database

```javascript
use tokoBuku
```

Jika database `tokoBuku` belum ada, MongoDB akan membuatnya secara otomatis saat pertama kali data dimasukkan.

### Melihat database aktif

```javascript
db
```

### Melihat daftar collection

```javascript
show collections
```

### Contoh dokumen yang digunakan sepanjang panduan ini

```javascript
{
  _id: ObjectId("..."),
  judul: "Belajar MongoDB",
  penulis: "Agung",
  harga: 85000,
  stok: 10,
  kategori: "Teknologi",
  tersedia: true,
  tags: ["database", "nosql", "backend"],
  diterbitkan: new Date("2024-01-15")
}
```

---

## 3. Create — Menyimpan Data

### 3.1 `insertOne()` — Menyimpan satu dokumen

```javascript
db.buku.insertOne({
  judul: "Belajar MongoDB",
  penulis: "Agung",
  harga: 85000,
  stok: 10,
  kategori: "Teknologi",
  tersedia: true,
  tags: ["database", "nosql", "backend"],
  diterbitkan: new Date("2024-01-15")
})
```

Hasil:

```javascript
{
  acknowledged: true,
  insertedId: ObjectId("664f1a2b3c4d5e6f7a8b9c0d")
}
```

- `acknowledged: true` berarti operasi berhasil.
- `insertedId` adalah `_id` dokumen yang baru dibuat.

### 3.2 `insertMany()` — Menyimpan banyak dokumen sekaligus

```javascript
db.buku.insertMany([
  {
    judul: "Clean Code",
    penulis: "Robert C. Martin",
    harga: 120000,
    stok: 5,
    kategori: "Teknologi",
    tersedia: true,
    tags: ["programming", "best-practice"]
  },
  {
    judul: "Sapiens",
    penulis: "Yuval Noah Harari",
    harga: 95000,
    stok: 8,
    kategori: "Sejarah",
    tersedia: true,
    tags: ["sejarah", "manusia"]
  },
  {
    judul: "Atomic Habits",
    penulis: "James Clear",
    harga: 78000,
    stok: 0,
    kategori: "Self-Help",
    tersedia: false,
    tags: ["produktivitas", "kebiasaan"]
  }
])
```

Hasil:

```javascript
{
  acknowledged: true,
  insertedCount: 3,
  insertedIds: {
    '0': ObjectId("..."),
    '1': ObjectId("..."),
    '2': ObjectId("...")
  }
}
```

### 3.3 Menyimpan dengan `_id` kustom

MongoDB memperbolehkan kamu menentukan `_id` sendiri:

```javascript
db.buku.insertOne({
  _id: "buku-001",
  judul: "Panduan Git",
  penulis: "Budi",
  harga: 60000
})
```

> **Perhatian:** `_id` harus unik. Menyimpan dokumen dengan `_id` yang sudah ada akan menghasilkan error `duplicate key`.

### 3.4 `ordered` pada `insertMany()`

Secara default, jika satu dokumen gagal dimasukkan (misalnya `_id` duplikat), MongoDB akan berhenti dan tidak memasukkan dokumen berikutnya.

Untuk melanjutkan meskipun ada yang gagal:

```javascript
db.buku.insertMany(
  [ /* array dokumen */ ],
  { ordered: false }
)
```

---

## 4. Read — Membaca Data

### 4.1 `find()` — Mengambil semua dokumen

```javascript
db.buku.find()
```

Menampilkan semua dokumen di collection `buku`.

### 4.2 `findOne()` — Mengambil satu dokumen pertama

```javascript
db.buku.findOne()
```

Mengembalikan satu dokumen pertama yang ditemukan.

### 4.3 `find()` dengan filter

Cari buku dengan kategori "Teknologi":

```javascript
db.buku.find({ kategori: "Teknologi" })
```

Cari buku yang tersedia:

```javascript
db.buku.find({ tersedia: true })
```

Cari buku dengan harga tepat 85000:

```javascript
db.buku.find({ harga: 85000 })
```

### 4.4 `findOne()` dengan filter

```javascript
db.buku.findOne({ judul: "Sapiens" })
```

Mengembalikan dokumen pertama yang cocok, atau `null` jika tidak ditemukan.

### 4.5 Projection pada `find()`

Tampilkan hanya `judul` dan `harga`, tanpa `_id`:

```javascript
db.buku.find(
  {},
  { judul: 1, harga: 1, _id: 0 }
)
```

Sembunyikan field `stok` dan `tags`:

```javascript
db.buku.find(
  {},
  { stok: 0, tags: 0 }
)
```

### 4.6 Menghitung dokumen

Menghitung semua dokumen:

```javascript
db.buku.countDocuments()
```

Menghitung dengan filter:

```javascript
db.buku.countDocuments({ tersedia: true })
```

### 4.7 Sorting hasil query

Urutkan dari harga termurah ke termahal (`1` = ascending):

```javascript
db.buku.find().sort({ harga: 1 })
```

Urutkan dari harga termahal ke termurah (`-1` = descending):

```javascript
db.buku.find().sort({ harga: -1 })
```

Urutkan berdasarkan dua field:

```javascript
db.buku.find().sort({ kategori: 1, harga: -1 })
```

### 4.8 Membatasi dan melewati hasil

Ambil 3 dokumen saja:

```javascript
db.buku.find().limit(3)
```

Lewati 2 dokumen pertama (berguna untuk paginasi):

```javascript
db.buku.find().skip(2)
```

Kombinasi untuk paginasi (halaman 2, 3 item per halaman):

```javascript
db.buku.find().skip(3).limit(3)
```

### 4.9 `distinct()` — Nilai unik pada field

```javascript
db.buku.distinct("kategori")
```

Mengembalikan daftar kategori yang ada tanpa duplikat, misalnya: `["Teknologi", "Sejarah", "Self-Help"]`.

---

## 5. Update — Mengubah Data

### 5.1 `updateOne()` — Mengubah satu dokumen

Sintaks dasar:

```javascript
db.collection.updateOne(
  { filter },
  { $set: { field: nilai } }
)
```

Ubah harga buku "Sapiens":

```javascript
db.buku.updateOne(
  { judul: "Sapiens" },
  { $set: { harga: 99000 } }
)
```

Ubah beberapa field sekaligus:

```javascript
db.buku.updateOne(
  { judul: "Sapiens" },
  {
    $set: {
      harga: 99000,
      stok: 15,
      tersedia: true
    }
  }
)
```

Hasil:

```javascript
{
  acknowledged: true,
  matchedCount: 1,
  modifiedCount: 1
}
```

- `matchedCount`: jumlah dokumen yang cocok dengan filter.
- `modifiedCount`: jumlah dokumen yang benar-benar diubah.

### 5.2 `updateMany()` — Mengubah banyak dokumen

Tandai semua buku dengan stok 0 sebagai tidak tersedia:

```javascript
db.buku.updateMany(
  { stok: 0 },
  { $set: { tersedia: false } }
)
```

Tambahkan field baru pada semua dokumen:

```javascript
db.buku.updateMany(
  {},
  { $set: { diskon: 0 } }
)
```

### 5.3 Operator update yang sering digunakan

#### `$set` — Mengatur nilai field

```javascript
db.buku.updateOne(
  { judul: "Clean Code" },
  { $set: { harga: 110000 } }
)
```

#### `$unset` — Menghapus field dari dokumen

```javascript
db.buku.updateOne(
  { judul: "Clean Code" },
  { $unset: { diskon: "" } }
)
```

Nilai pada `$unset` tidak berpengaruh; gunakan `""` atau `1`.

#### `$inc` — Menambah atau mengurangi nilai numerik

Tambah stok sebanyak 5:

```javascript
db.buku.updateOne(
  { judul: "Belajar MongoDB" },
  { $inc: { stok: 5 } }
)
```

Kurangi stok sebanyak 2:

```javascript
db.buku.updateOne(
  { judul: "Belajar MongoDB" },
  { $inc: { stok: -2 } }
)
```

#### `$mul` — Mengalikan nilai numerik

Naikkan harga semua buku sebesar 10%:

```javascript
db.buku.updateMany(
  {},
  { $mul: { harga: 1.1 } }
)
```

#### `$rename` — Mengubah nama field

```javascript
db.buku.updateMany(
  {},
  { $rename: { "penulis": "author" } }
)
```

#### `$min` dan `$max` — Update kondisional

`$min` hanya mengubah jika nilai baru lebih kecil dari nilai saat ini:

```javascript
db.buku.updateOne(
  { judul: "Sapiens" },
  { $min: { harga: 80000 } }
)
```

`$max` hanya mengubah jika nilai baru lebih besar:

```javascript
db.buku.updateOne(
  { judul: "Sapiens" },
  { $max: { stok: 20 } }
)
```

### 5.4 Operator update untuk array

#### `$push` — Menambahkan elemen ke array

```javascript
db.buku.updateOne(
  { judul: "Belajar MongoDB" },
  { $push: { tags: "tutorial" } }
)
```

#### `$pull` — Menghapus elemen dari array

```javascript
db.buku.updateOne(
  { judul: "Belajar MongoDB" },
  { $pull: { tags: "backend" } }
)
```

#### `$addToSet` — Menambahkan elemen hanya jika belum ada

```javascript
db.buku.updateOne(
  { judul: "Belajar MongoDB" },
  { $addToSet: { tags: "nosql" } }
)
```

Berbeda dengan `$push`, `$addToSet` tidak akan menambahkan duplikat.

### 5.5 `replaceOne()` — Mengganti seluruh dokumen

`replaceOne()` mengganti seluruh isi dokumen (kecuali `_id`):

```javascript
db.buku.replaceOne(
  { judul: "Atomic Habits" },
  {
    judul: "Atomic Habits",
    penulis: "James Clear",
    harga: 85000,
    stok: 12,
    kategori: "Self-Help",
    tersedia: true
  }
)
```

> **Hati-hati:** `replaceOne()` akan menghapus semua field yang tidak disertakan dalam dokumen pengganti. Gunakan `$set` jika hanya ingin mengubah field tertentu.

### 5.6 Opsi `upsert`

`upsert: true` akan membuat dokumen baru jika tidak ada yang cocok dengan filter:

```javascript
db.buku.updateOne(
  { judul: "Buku Baru" },
  {
    $set: {
      judul: "Buku Baru",
      penulis: "Penulis X",
      harga: 70000
    }
  },
  { upsert: true }
)
```

Jika tidak ditemukan, dokumen baru akan dibuat. Jika ditemukan, dokumen yang ada akan diperbarui.

### 5.7 `findOneAndUpdate()` — Update sekaligus mengambil dokumen

Secara default mengembalikan dokumen **sebelum** diperbarui:

```javascript
db.buku.findOneAndUpdate(
  { judul: "Sapiens" },
  { $set: { harga: 105000 } }
)
```

Mengembalikan dokumen **setelah** diperbarui:

```javascript
db.buku.findOneAndUpdate(
  { judul: "Sapiens" },
  { $set: { harga: 105000 } },
  { returnDocument: "after" }
)
```

---

## 6. Delete — Menghapus Data

### 6.1 `deleteOne()` — Menghapus satu dokumen

Hapus satu buku berdasarkan judul:

```javascript
db.buku.deleteOne({ judul: "Buku Baru" })
```

Hasil:

```javascript
{
  acknowledged: true,
  deletedCount: 1
}
```

### 6.2 `deleteMany()` — Menghapus banyak dokumen

Hapus semua buku yang tidak tersedia:

```javascript
db.buku.deleteMany({ tersedia: false })
```

Hapus semua dokumen di collection (**berbahaya!**):

```javascript
db.buku.deleteMany({})
```

### 6.3 `findOneAndDelete()` — Hapus sekaligus mengambil dokumen

Menghapus dan mengembalikan dokumen yang dihapus:

```javascript
db.buku.findOneAndDelete({ judul: "Panduan Git" })
```

### 6.4 Menghapus collection

Menghapus seluruh collection beserta semua dokumennya:

```javascript
db.buku.drop()
```

### 6.5 Menghapus database

Menghapus database aktif saat ini:

```javascript
db.dropDatabase()
```

> **Perhatian:** Operasi `drop()` dan `dropDatabase()` tidak bisa dibatalkan. Pastikan kamu berada di database yang tepat sebelum menjalankannya.

---

## 7. Filter Lanjutan

Filter yang lebih dari sekadar kecocokan nilai biasa.

### 7.1 Operator perbandingan

| Operator | Arti | Contoh |
|---|---|---|
| `$eq` | Sama dengan | `{ harga: { $eq: 85000 } }` |
| `$ne` | Tidak sama dengan | `{ kategori: { $ne: "Sejarah" } }` |
| `$gt` | Lebih besar dari | `{ harga: { $gt: 80000 } }` |
| `$gte` | Lebih besar atau sama dengan | `{ harga: { $gte: 80000 } }` |
| `$lt` | Lebih kecil dari | `{ stok: { $lt: 5 } }` |
| `$lte` | Lebih kecil atau sama dengan | `{ stok: { $lte: 5 } }` |

Contoh penggunaan:

```javascript
// Buku dengan harga antara 70000 dan 100000
db.buku.find({ harga: { $gte: 70000, $lte: 100000 } })

// Buku dengan stok lebih dari 0
db.buku.find({ stok: { $gt: 0 } })
```

### 7.2 Operator `$in` dan `$nin`

`$in` — cocok jika nilainya ada dalam daftar:

```javascript
db.buku.find({ kategori: { $in: ["Teknologi", "Self-Help"] } })
```

`$nin` — cocok jika nilainya **tidak** ada dalam daftar:

```javascript
db.buku.find({ kategori: { $nin: ["Sejarah"] } })
```

### 7.3 Operator logika

`$and` — semua kondisi harus terpenuhi:

```javascript
db.buku.find({
  $and: [
    { harga: { $gte: 80000 } },
    { tersedia: true }
  ]
})
```

Versi lebih ringkas (secara implisit menggunakan AND):

```javascript
db.buku.find({ harga: { $gte: 80000 }, tersedia: true })
```

`$or` — salah satu kondisi cukup:

```javascript
db.buku.find({
  $or: [
    { kategori: "Teknologi" },
    { harga: { $lt: 70000 } }
  ]
})
```

`$not` — membalik kondisi:

```javascript
db.buku.find({ harga: { $not: { $gt: 100000 } } })
```

`$nor` — tidak ada kondisi yang terpenuhi:

```javascript
db.buku.find({
  $nor: [
    { kategori: "Sejarah" },
    { tersedia: false }
  ]
})
```

### 7.4 Operator elemen

`$exists` — memeriksa apakah field ada:

```javascript
// Dokumen yang memiliki field 'diskon'
db.buku.find({ diskon: { $exists: true } })

// Dokumen yang tidak memiliki field 'diskon'
db.buku.find({ diskon: { $exists: false } })
```

`$type` — memeriksa tipe data field:

```javascript
db.buku.find({ harga: { $type: "number" } })
db.buku.find({ judul: { $type: "string" } })
```

### 7.5 Filter pada array

Cari dokumen yang arraynya mengandung nilai tertentu:

```javascript
db.buku.find({ tags: "nosql" })
```

Cari dokumen yang arraynya mengandung **semua** nilai yang disebutkan:

```javascript
db.buku.find({ tags: { $all: ["database", "nosql"] } })
```

Cari dokumen berdasarkan ukuran array:

```javascript
db.buku.find({ tags: { $size: 3 } })
```

### 7.6 `$regex` — Pencarian teks dengan ekspresi reguler

Cari buku yang judulnya mengandung kata "MongoDB" (case-insensitive):

```javascript
db.buku.find({ judul: { $regex: "MongoDB", $options: "i" } })
```

Cari buku yang judulnya dimulai dengan "Belajar":

```javascript
db.buku.find({ judul: { $regex: "^Belajar" } })
```

---

## 8. Opsi Tambahan

### 8.1 `.sort()`, `.limit()`, `.skip()` — Chaining

Metode-metode ini dapat digabungkan:

```javascript
db.buku.find({ tersedia: true })
  .sort({ harga: -1 })
  .skip(0)
  .limit(5)
```

Query di atas mengambil 5 buku dengan harga tertinggi yang masih tersedia.

### 8.2 Membuat index

Index mempercepat query, terutama pada collection besar.

Membuat index pada field `judul`:

```javascript
db.buku.createIndex({ judul: 1 })
```

Membuat index unik pada field `judul`:

```javascript
db.buku.createIndex({ judul: 1 }, { unique: true })
```

Melihat daftar index:

```javascript
db.buku.getIndexes()
```

Menghapus index:

```javascript
db.buku.dropIndex({ judul: 1 })
```

### 8.3 `explain()` — Menganalisis performa query

```javascript
db.buku.find({ kategori: "Teknologi" }).explain("executionStats")
```

Berguna untuk memahami apakah query menggunakan index atau melakukan *full collection scan*.

---

## 9. Operasi Bulk

`bulkWrite()` memungkinkan menggabungkan beberapa operasi dalam satu pemanggilan:

```javascript
db.buku.bulkWrite([
  {
    insertOne: {
      document: {
        judul: "Desain Sistem",
        penulis: "Alex Xu",
        harga: 130000,
        stok: 3,
        tersedia: true
      }
    }
  },
  {
    updateOne: {
      filter: { judul: "Sapiens" },
      update: { $set: { harga: 100000 } }
    }
  },
  {
    deleteOne: {
      filter: { stok: 0 }
    }
  }
])
```

Hasil:

```javascript
{
  acknowledged: true,
  insertedCount: 1,
  modifiedCount: 1,
  deletedCount: 1,
  ...
}
```

`bulkWrite()` lebih efisien dibandingkan menjalankan operasi satu per satu, karena mengurangi round-trip ke server.

---

## 10. Kesalahan Umum

### Kesalahan 1: Lupa kurung kurawal pada filter kosong

```javascript
// Salah — akan error
db.buku.find()  // ini benar
db.buku.updateMany({ $set: { diskon: 0 } })  // salah! filter tidak ada

// Benar
db.buku.updateMany({}, { $set: { diskon: 0 } })
```

### Kesalahan 2: Menggunakan `replaceOne()` padahal hanya ingin mengubah satu field

```javascript
// Salah — menghapus semua field lain!
db.buku.replaceOne(
  { judul: "Sapiens" },
  { harga: 99000 }
)

// Benar — gunakan $set
db.buku.updateOne(
  { judul: "Sapiens" },
  { $set: { harga: 99000 } }
)
```

### Kesalahan 3: `deleteMany({})` pada collection yang salah

Selalu periksa database dan collection aktif sebelum operasi destruktif:

```javascript
db  // cek database aktif
show collections  // cek collection yang ada
db.buku.deleteMany({})  // baru jalankan jika sudah yakin
```

### Kesalahan 4: `updateOne()` tanpa operator update

```javascript
// Salah — tidak menggunakan operator seperti $set
db.buku.updateOne(
  { judul: "Sapiens" },
  { harga: 99000 }  // ini akan menggantikan seluruh dokumen!
)

// Benar
db.buku.updateOne(
  { judul: "Sapiens" },
  { $set: { harga: 99000 } }
)
```

### Kesalahan 5: Mencari `_id` dengan string biasa

`_id` bawaan MongoDB bertipe `ObjectId`, bukan string:

```javascript
// Salah
db.buku.findOne({ _id: "664f1a2b3c4d5e6f7a8b9c0d" })

// Benar
db.buku.findOne({ _id: ObjectId("664f1a2b3c4d5e6f7a8b9c0d") })
```

### Kesalahan 6: `find()` tidak iterasi di environment tertentu

Di `mongosh`, `find()` langsung ditampilkan. Di kode aplikasi (misalnya Node.js), hasilnya adalah cursor yang perlu di-*iterate* atau dikonversi:

```javascript
// Di Node.js dengan driver resmi
const hasil = await db.collection("buku").find({ tersedia: true }).toArray()
```

---

## 11. Latihan

Kerjakan menggunakan collection `buku` yang sudah dibuat di atas.

### Level 1 — Dasar

1. Tambahkan satu buku baru dengan `insertOne()`.
2. Tampilkan semua buku yang ada.
3. Cari buku berdasarkan kategori "Teknologi".
4. Ubah harga salah satu buku menggunakan `updateOne()` dan `$set`.
5. Hapus buku berdasarkan judulnya.

### Level 2 — Filter dan Update

6. Cari semua buku dengan harga antara 70.000 dan 100.000.
7. Tampilkan semua buku yang tersedia, diurutkan dari harga termahal, hanya 3 hasil pertama.
8. Naikkan stok semua buku kategori "Teknologi" sebanyak 3 menggunakan `$inc`.
9. Tambahkan tag "rekomendasi" ke semua buku dengan harga di atas 100.000 menggunakan `$addToSet`.
10. Tampilkan daftar kategori unik yang ada di collection.

### Level 3 — Lanjutan

11. Gunakan `upsert: true` untuk memperbarui buku yang mungkin belum ada.
12. Gunakan `findOneAndUpdate()` dan kembalikan dokumen setelah diperbarui.
13. Lakukan `bulkWrite()` yang menggabungkan operasi insert, update, dan delete.
14. Cari buku yang judulnya mengandung kata tertentu menggunakan `$regex`.
15. Buat index pada field `kategori` dan lihat pengaruhnya dengan `explain()`.

---

## 12. Ringkasan

### Create

| Perintah | Fungsi |
|---|---|
| `insertOne({})` | Menyimpan satu dokumen |
| `insertMany([])` | Menyimpan banyak dokumen sekaligus |

### Read

| Perintah | Fungsi |
|---|---|
| `find()` | Mengambil semua dokumen |
| `find({ filter })` | Mengambil dokumen yang cocok filter |
| `findOne({ filter })` | Mengambil satu dokumen pertama |
| `.sort({ field: 1|-1 })` | Mengurutkan hasil |
| `.limit(n)` | Membatasi jumlah hasil |
| `.skip(n)` | Melewati n dokumen pertama |
| `countDocuments()` | Menghitung jumlah dokumen |
| `distinct("field")` | Nilai unik pada field |

### Update

| Perintah | Fungsi |
|---|---|
| `updateOne(filter, update)` | Mengubah satu dokumen |
| `updateMany(filter, update)` | Mengubah banyak dokumen |
| `replaceOne(filter, doc)` | Mengganti seluruh dokumen |
| `findOneAndUpdate(...)` | Update dan kembalikan dokumen |

### Operator Update

| Operator | Fungsi |
|---|---|
| `$set` | Mengatur nilai field |
| `$unset` | Menghapus field |
| `$inc` | Menambah/mengurangi angka |
| `$mul` | Mengalikan angka |
| `$rename` | Mengubah nama field |
| `$push` | Menambah elemen ke array |
| `$pull` | Menghapus elemen dari array |
| `$addToSet` | Menambah elemen unik ke array |

### Delete

| Perintah | Fungsi |
|---|---|
| `deleteOne({ filter })` | Menghapus satu dokumen |
| `deleteMany({ filter })` | Menghapus banyak dokumen |
| `findOneAndDelete(filter)` | Hapus dan kembalikan dokumen |
| `drop()` | Menghapus seluruh collection |
| `dropDatabase()` | Menghapus seluruh database |

### Rumus mudah diingat

- **C**reate → `insert`
- **R**ead → `find`
- **U**pdate → `update` dengan operator seperti `$set`
- **D**elete → `delete`
- Filter selalu menjadi **argumen pertama** pada operasi update dan delete.
- Gunakan `$set` untuk update agar tidak mengganti seluruh dokumen.
- Selalu periksa database dan collection aktif sebelum operasi destruktif.

---

## 13. Referensi Resmi

- [MongoDB Manual — Insert Documents](https://www.mongodb.com/docs/manual/tutorial/insert-documents/)
- [MongoDB Manual — Query Documents](https://www.mongodb.com/docs/manual/tutorial/query-documents/)
- [MongoDB Manual — Update Documents](https://www.mongodb.com/docs/manual/tutorial/update-documents/)
- [MongoDB Manual — Delete Documents](https://www.mongodb.com/docs/manual/tutorial/remove-documents/)
- [MongoDB Manual — Query and Projection Operators](https://www.mongodb.com/docs/manual/reference/operator/query/)
- [MongoDB Manual — Update Operators](https://www.mongodb.com/docs/manual/reference/operator/update/)
- [MongoDB Manual — `bulkWrite()`](https://www.mongodb.com/docs/manual/reference/method/db.collection.bulkWrite/)

---

README ini berfokus pada operasi CRUD MongoDB. Simpan sebagai `README.md` di repository khusus CRUD, atau gunakan nama seperti `mongodb-crud.md` jika digabungkan ke repository catatan belajar MongoDB.
