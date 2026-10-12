# MongoDB Logical Query Operators — Panduan Lengkap

Panduan berbahasa Indonesia untuk memahami **logical expressions / logical query operators** di MongoDB, dari dasar sampai pola query yang lebih kompleks. Contoh menggunakan `mongosh` dan collection `mahasiswa`.

> Fokus dokumen ini adalah operator logika MongoDB. Comparison operators seperti `$gt` dan `$in` digunakan sebagai bagian dari kondisi, tetapi bukan operator logika itu sendiri.

## Daftar Isi

1. [Persiapan](#1-persiapan)
2. [Cara Kerja Query MongoDB](#2-cara-kerja-query-mongodb)
3. [Empat Logical Query Operators](#3-empat-logical-query-operators)
4. [Implicit AND](#4-implicit-and)
5. [Menggabungkan Kondisi Kompleks](#5-menggabungkan-kondisi-kompleks)
6. [Logical Operators dan Field yang Tidak Ada](#6-logical-operators-dan-field-yang-tidak-ada)
7. [Logical Operators dengan Array](#7-logical-operators-dengan-array)
8. [Logical Operators dengan Nested Documents](#8-logical-operators-dengan-nested-documents)
9. [Kesalahan yang Sering Terjadi](#9-kesalahan-yang-sering-terjadi)
10. [Latihan](#10-latihan)
11. [Ringkasan Cepat](#11-ringkasan-cepat)
12. [Referensi Resmi](#12-referensi-resmi)

---

## 1. Persiapan

Pilih database yang digunakan:

```javascript
use belajarMongo
```

Lihat data di collection:

```javascript
db.mahasiswa.find()
```

Contoh dokumen yang digunakan dalam panduan ini:

```javascript
{
  nama: "Agung",
  umur: 19,
  jurusan: "Informatika",
  nilai: 85,
  aktif: true
}
```

Field yang digunakan:

- `nama`: nama mahasiswa.
- `umur`: umur mahasiswa.
- `jurusan`: jurusan mahasiswa.
- `nilai`: nilai mahasiswa.
- `aktif`: status aktif berupa boolean (`true` atau `false`).

Jika nama collection atau field di database kamu berbeda, sesuaikan query-nya.

## 2. Cara Kerja Query MongoDB

Bentuk dasar pencarian dokumen:

```javascript
db.mahasiswa.find({
  nilai: { $gt: 80 }
})
```

MongoDB mencari dokumen yang cocok dengan filter di dalam `find()`.

- `db.mahasiswa`: collection yang dipilih.
- `find()`: mengambil dokumen yang cocok dengan filter.
- `nilai`: field yang diperiksa.
- `$gt`: operator perbandingan *greater than*.
- `80`: nilai pembanding.

Logical query operators mengatur bagaimana satu atau beberapa kondisi digabungkan atau dinegasikan.

## 3. Empat Logical Query Operators

MongoDB memiliki empat logical query operators utama: `$and`, `$or`, `$not`, dan `$nor`.

### 3.1 `$and` — Semua kondisi harus cocok

`$and` mengembalikan dokumen yang cocok dengan **semua** kondisi di dalam array.

```javascript
db.mahasiswa.find({
  $and: [
    { umur: { $gte: 18 } },
    { nilai: { $gte: 80 } }
  ]
})
```

Artinya:
1. Umur minimal 18 tahun.
2. Nilai minimal 80.
3. Kedua kondisi harus terpenuhi.

Secara logika:

```text
umur >= 18 AND nilai >= 80
```

**Kapan digunakan?** Saat kamu perlu menyatakan beberapa kondisi harus benar secara bersamaan, terutama jika kondisi menggunakan ekspresi yang sama atau query membutuhkan struktur eksplisit.

### 3.2 `$or` — Minimal satu kondisi harus cocok

`$or` mengembalikan dokumen yang cocok dengan **setidaknya satu** kondisi.

```javascript
db.mahasiswa.find({
  $or: [
    { jurusan: "Informatika" },
    { nilai: { $gte: 90 } }
  ]
})
```

Artinya, dokumen akan cocok jika:
- jurusannya Informatika; atau
- nilainya minimal 90; atau
- kedua kondisi terpenuhi.

Secara logika:

```text
jurusan = "Informatika" OR nilai >= 90
```

**Kapan digunakan?** Saat ada beberapa alternatif yang dapat membuat sebuah dokumen lolos filter.

### 3.3 `$not` — Menegasikan kondisi suatu field

`$not` digunakan bersama ekspresi operator field. Operator ini mencocokkan dokumen yang tidak memenuhi ekspresi tersebut.

```javascript
db.mahasiswa.find({
  nilai: {
    $not: { $lt: 80 }
  }
})
```

Untuk data yang `nilai`-nya selalu ada dan berupa angka, query tersebut mencocokkan nilai 80 atau lebih.

Perlu dibedakan:
- `$not` adalah operator negasi pada level field.
- `$nor` adalah operator logika yang menerima array beberapa kondisi.

**Penting:** `$not` juga dapat mencocokkan dokumen yang field-nya tidak ada. Jika hanya ingin dokumen yang mempunyai field `nilai`, gunakan `$exists`:

```javascript
db.mahasiswa.find({
  nilai: {
    $exists: true,
    $not: { $lt: 80 }
  }
})
```

Jangan gunakan `$not` sebagai pengganti universal untuk semua negasi. Untuk kondisi “bukan salah satu nilai”, `$ne` atau `$nin` terkadang lebih mudah dibaca.

### 3.4 `$nor` — Tidak satu pun kondisi boleh cocok

`$nor` mengembalikan dokumen yang **tidak cocok dengan kondisi mana pun** dalam array.

```javascript
db.mahasiswa.find({
  $nor: [
    { jurusan: "Informatika" },
    { umur: { $gte: 20 } }
  ]
})
```

Artinya:
- jurusan bukan Informatika; dan
- umur tidak memenuhi kondisi minimal 20 tahun.

Untuk dokumen dengan field `umur` yang ada dan berupa angka, ini berarti umur kurang dari 20 tahun.

Secara logika:

```text
NOT (jurusan = "Informatika" OR umur >= 20)
```

Dengan hukum De Morgan, bentuk logika tersebut setara dengan:

```text
jurusan != "Informatika" AND umur < 20
```

**Kapan digunakan?** Saat kamu ingin mengecualikan dokumen yang cocok dengan satu atau beberapa kondisi.

---

## 4. Implicit AND

MongoDB secara default menggabungkan beberapa kondisi berbeda dalam satu objek filter menggunakan logika AND. Ini disebut **implicit AND**.

Contoh:

```javascript
db.mahasiswa.find({
  umur: { $gte: 18 },
  aktif: true
})
```

Artinya: umur minimal 18 **dan** status aktif harus benar.

Contoh ini setara dengan:

```javascript
db.mahasiswa.find({
  $and: [
    { umur: { $gte: 18 } },
    { aktif: true }
  ]
})
```

Biasanya bentuk implicit AND lebih singkat dan mudah dibaca.

### Kapan `$and` eksplisit diperlukan?

Salah satu kasusnya adalah ketika kamu perlu menerapkan beberapa ekspresi pada field yang sama dan tidak bisa menuliskannya sebagai satu pasangan key biasa tanpa konflik.

Contoh berikut tidak valid sebagai objek JavaScript biasa karena key `nilai` ditulis dua kali:

```javascript
// Hindari pola ini: key nilai ditulis dua kali.
db.mahasiswa.find({
  nilai: { $gt: 60 },
  nilai: { $lt: 90 }
})
```

Gunakan `$and`:

```javascript
db.mahasiswa.find({
  $and: [
    { nilai: { $gt: 60 } },
    { nilai: { $lt: 90 } }
  ]
})
```

Atau, untuk contoh perbandingan rentang ini, gabungkan operator pada field yang sama:

```javascript
db.mahasiswa.find({
  nilai: {
    $gt: 60,
    $lt: 90
  }
})
```

Kedua bentuk terakhir menyatakan nilai lebih dari 60 dan kurang dari 90.

---

## 5. Menggabungkan Kondisi Kompleks

Kamu dapat menggabungkan logical operators dalam satu query.

### 5.1 Kondisi wajib ditambah pilihan alternatif

Cari mahasiswa berumur minimal 18 tahun yang jurusannya Informatika **atau** nilainya minimal 90:

```javascript
db.mahasiswa.find({
  umur: { $gte: 18 },
  $or: [
    { jurusan: "Informatika" },
    { nilai: { $gte: 90 } }
  ]
})
```

Logikanya:

```text
umur >= 18 AND (jurusan = "Informatika" OR nilai >= 90)
```

Kondisi `umur` harus cocok, lalu minimal satu kondisi di `$or` harus cocok.

### 5.2 `$or` di dalam `$and`

Contoh: mahasiswa harus berumur minimal 18 tahun dan harus memenuhi salah satu dari dua kondisi: jurusan Informatika atau status aktif.

```javascript
db.mahasiswa.find({
  $and: [
    { umur: { $gte: 18 } },
    {
      $or: [
        { jurusan: "Informatika" },
        { aktif: true }
      ]
    }
  ]
})
```

Logikanya:

```text
umur >= 18 AND (jurusan = "Informatika" OR aktif = true)
```

### 5.3 Beberapa kelompok `$or`

Contoh: mahasiswa harus (jurusan Informatika atau Teknik Komputer) dan (nilai minimal 80 atau status aktif).

```javascript
db.mahasiswa.find({
  $and: [
    {
      $or: [
        { jurusan: "Informatika" },
        { jurusan: "Teknik Komputer" }
      ]
    },
    {
      $or: [
        { nilai: { $gte: 80 } },
        { aktif: true }
      ]
    }
  ]
})
```

Logikanya:

```text
(jurusan = "Informatika" OR jurusan = "Teknik Komputer")
AND
(nilai >= 80 OR aktif = true)
```

Untuk beberapa nilai pada field yang sama, `$in` biasanya lebih ringkas:

```javascript
db.mahasiswa.find({
  jurusan: { $in: ["Informatika", "Teknik Komputer"] },
  $or: [
    { nilai: { $gte: 80 } },
    { aktif: true }
  ]
})
```

### 5.4 Negasi dengan `$nor`

Cari mahasiswa yang bukan jurusan Informatika dan bukan mahasiswa berumur minimal 20 tahun:

```javascript
db.mahasiswa.find({
  $nor: [
    { jurusan: "Informatika" },
    { umur: { $gte: 20 } }
  ]
})
```

Ingat: `$nor` menolak dokumen yang cocok dengan kondisi pertama **atau** kondisi kedua. Dokumen yang dikembalikan tidak cocok dengan keduanya.

### 5.5 Memahami prioritas kondisi

Jangan mengandalkan intuisi saja ketika membaca query yang kompleks. Kelompokkan kondisi menggunakan `$and` dan `$or` agar maksud query terlihat jelas.

Contoh:

```javascript
db.mahasiswa.find({
  $and: [
    { aktif: true },
    {
      $or: [
        { nilai: { $gte: 90 } },
        { jurusan: "Teknik Komputer" }
      ]
    }
  ]
})
```

Strukturnya jelas: `aktif` wajib benar, lalu salah satu kondisi di dalam `$or` harus benar.

---

## 6. Logical Operators dan Field yang Tidak Ada

MongoDB membedakan field yang tidak ada, field bernilai `null`, dan field yang berisi nilai biasa. Perbedaan ini penting saat memakai operator negasi.

Misalnya, bayangkan ada tiga dokumen:

```javascript
{ nama: "Agung", nilai: 85 }
{ nama: "Budi", nilai: null }
{ nama: "Citra" }
```

Dokumen Agung memiliki nilai angka, Budi memiliki field `nilai` dengan nilai `null`, sedangkan Citra tidak memiliki field `nilai`.

### 6.1 Memeriksa keberadaan field dengan `$exists`

Dokumen yang memiliki field `nilai`, termasuk jika nilainya `null`:

```javascript
db.mahasiswa.find({
  nilai: { $exists: true }
})
```

Dokumen yang tidak memiliki field `nilai`:

```javascript
db.mahasiswa.find({
  nilai: { $exists: false }
})
```

### 6.2 Menggunakan `$ne`

```javascript
db.mahasiswa.find({
  nilai: { $ne: 80 }
})
```

`$ne` dapat mencocokkan dokumen yang nilai `nilai`-nya bukan 80 **dan** dokumen yang tidak memiliki field tersebut. Jika kamu hanya ingin dokumen yang memiliki field `nilai`:

```javascript
db.mahasiswa.find({
  nilai: {
    $exists: true,
    $ne: 80
  }
})
```

### 6.3 Menggunakan `$nin`

```javascript
db.mahasiswa.find({
  jurusan: { $nin: ["Informatika", "Teknik Komputer"] }
})
```

`$nin` juga dapat mencocokkan dokumen yang tidak memiliki field `jurusan`. Untuk membatasi hasil ke dokumen yang mempunyai field tersebut:

```javascript
db.mahasiswa.find({
  jurusan: {
    $exists: true,
    $nin: ["Informatika", "Teknik Komputer"]
  }
})
```

### 6.4 Menggunakan `$not`

```javascript
db.mahasiswa.find({
  nilai: { $not: { $gt: 80 } }
})
```

Ini menegasikan kondisi `nilai > 80`. Dokumen tanpa field `nilai` juga dapat cocok. Jika hanya ingin dokumen yang field `nilai`-nya ada:

```javascript
db.mahasiswa.find({
  nilai: {
    $exists: true,
    $not: { $gt: 80 }
  }
})
```

> Catatan: query dengan field `null` dan perbandingan angka dapat berperilaku berbeda dari field angka biasa. Tentukan dulu apakah kamu ingin memasukkan dokumen dengan `null` atau field yang hilang, lalu gunakan `$exists`, `$type`, atau filter lain sesuai kebutuhan.

---

## 7. Logical Operators dengan Array

Logical operators dapat digunakan dalam query yang memeriksa array. Namun, array memiliki aturan pencocokan tersendiri.

Contoh dokumen:

```javascript
{
  nama: "Agung",
  hobi: ["coding", "gaming", "musik"]
}
```

### 7.1 Mencari array yang mengandung nilai tertentu

```javascript
db.mahasiswa.find({
  hobi: "coding"
})
```

Untuk array sederhana, query tersebut cocok dengan dokumen yang array `hobi`-nya berisi `"coding"`.

### 7.2 Menggunakan `$or` untuk beberapa kemungkinan

```javascript
db.mahasiswa.find({
  $or: [
    { hobi: "coding" },
    { hobi: "gaming" }
  ]
})
```

Dokumen cocok jika array `hobi` mengandung `"coding"` atau `"gaming"`.

### 7.3 Mengharuskan dua nilai ada dalam array

```javascript
db.mahasiswa.find({
  $and: [
    { hobi: "coding" },
    { hobi: "gaming" }
  ]
})
```

Dokumen cocok jika array mengandung kedua nilai tersebut, tidak harus bersebelahan.

### 7.4 Memeriksa satu elemen array dengan beberapa syarat

Misalnya dokumen memiliki array nilai ujian:

```javascript
{
  nama: "Agung",
  ujian: [
    { mataKuliah: "Basis Data", nilai: 90 },
    { mataKuliah: "Jaringan", nilai: 70 }
  ]
}
```

Jika ingin mencari dokumen dengan **satu elemen array yang sama** memiliki `mataKuliah: "Basis Data"` dan `nilai >= 80`, gunakan `$elemMatch`:

```javascript
db.mahasiswa.find({
  ujian: {
    $elemMatch: {
      mataKuliah: "Basis Data",
      nilai: { $gte: 80 }
    }
  }
})
```

`$elemMatch` bukan logical query operator utama, tetapi sangat berguna saat membuat filter kompleks pada array.

Tanpa `$elemMatch`, kondisi pada field array bersarang bisa terpenuhi oleh elemen yang berbeda, tergantung struktur query. Gunakan `$elemMatch` ketika beberapa syarat harus berlaku pada **elemen array yang sama**.

---

## 8. Logical Operators dengan Nested Documents

Nested document adalah objek yang berada di dalam objek lain.

Contoh:

```javascript
{
  nama: "Agung",
  alamat: {
    kota: "Surabaya",
    provinsi: "Jawa Timur"
  }
}
```

### 8.1 Menggunakan dot notation

Cari mahasiswa yang tinggal di Surabaya:

```javascript
db.mahasiswa.find({
  "alamat.kota": "Surabaya"
})
```

### 8.2 Menggabungkan kondisi nested document

Cari mahasiswa yang tinggal di Surabaya dan berumur minimal 18 tahun:

```javascript
db.mahasiswa.find({
  "alamat.kota": "Surabaya",
  umur: { $gte: 18 }
})
```

Kondisi tersebut menggunakan implicit AND.

### 8.3 Menggabungkan alternatif pada nested field

```javascript
db.mahasiswa.find({
  $or: [
    { "alamat.kota": "Surabaya" },
    { "alamat.kota": "Malang" }
  ]
})
```

Dokumen cocok jika kota pada alamatnya Surabaya atau Malang.

---

## 9. Kesalahan yang Sering Terjadi

### Kesalahan 1: Salah membedakan `$gt` dan `$gte`

```javascript
{ nilai: { $gt: 75 } }
```

Nilai 75 **tidak** ikut.

```javascript
{ nilai: { $gte: 75 } }
```

Nilai 75 **ikut**.

### Kesalahan 2: Menganggap `$or` berarti semua kondisi

`$or` berarti setidaknya satu kondisi cocok. Untuk mewajibkan semua kondisi, gunakan implicit AND atau `$and`.

### Kesalahan 3: Menganggap `$nor` sama dengan `$or`

- `$or`: minimal satu kondisi cocok.
- `$nor`: tidak satu pun kondisi cocok.

### Kesalahan 4: Menulis key field yang sama berulang kali

Hindari:

```javascript
// Jangan tulis key nilai dua kali dalam objek yang sama.
db.mahasiswa.find({
  nilai: { $gt: 60 },
  nilai: { $lt: 90 }
})
```

Gunakan satu objek kondisi:

```javascript
db.mahasiswa.find({
  nilai: { $gt: 60, $lt: 90 }
})
```

Atau gunakan `$and` jika kondisi memang perlu ditulis sebagai ekspresi terpisah.

### Kesalahan 5: Melupakan field yang tidak ada

Query negatif seperti `$ne`, `$nin`, dan `$not` bisa mencocokkan dokumen yang field-nya tidak ada. Tambahkan `$exists: true` jika field wajib ada.

### Kesalahan 6: Query terlalu rumit tanpa struktur

Jika kondisi mulai bercabang, kelompokkan dengan `$and` dan `$or`, lalu tuliskan maksud logikanya sebagai komentar atau catatan sebelum menjalankan query.

---

## 10. Latihan

Kerjakan latihan berikut menggunakan collection `mahasiswa` yang sudah kamu buat.

### Level 1 — Dasar

**Soal 1.** Tampilkan mahasiswa yang berumur minimal 18 tahun dan berstatus aktif.

**Soal 2.** Tampilkan mahasiswa yang jurusannya Informatika atau nilai-nya minimal 90.

**Soal 3.** Tampilkan mahasiswa yang nilainya bukan 80.

**Soal 4.** Tampilkan mahasiswa yang bukan jurusan Informatika dan umurnya kurang dari 20 tahun menggunakan `$nor`.

### Level 2 — Gabungan kondisi

**Soal 5.** Tampilkan mahasiswa berumur minimal 18 tahun yang jurusannya Informatika atau nilainya minimal 90.

**Soal 6.** Tampilkan mahasiswa yang nilainya lebih dari 60 dan kurang dari 90.

**Soal 7.** Tampilkan mahasiswa yang jurusannya bukan Informatika maupun Teknik Komputer, dan field `jurusan` wajib ada.

### Level 3 — Konsep lanjutan

**Soal 8.** Buat dua dokumen uji: satu memiliki field `nilai: null`, satu lagi tidak memiliki field `nilai`. Cari dokumen yang memiliki field `nilai` menggunakan `$exists`.

**Soal 9.** Dengan dokumen yang memiliki array `hobi`, cari dokumen yang memiliki kedua nilai `"coding"` dan `"gaming"` menggunakan `$and`.

**Soal 10.** Dengan dokumen yang memiliki array `ujian`, cari mahasiswa yang memiliki satu elemen ujian dengan `mataKuliah: "Basis Data"` dan `nilai >= 80` menggunakan `$elemMatch`.

### Template untuk menjawab

```javascript
// Soal 1
db.mahasiswa.find({
  // tulis kondisi di sini
})
```

Coba jawab satu per satu dan periksa hasilnya dengan data di database kamu.

---

## 11. Ringkasan Cepat

| Operator / konsep | Makna |
|---|---|
| `$and` | Semua kondisi dalam daftar harus cocok |
| `$or` | Setidaknya satu kondisi dalam daftar harus cocok |
| `$not` | Menegasikan ekspresi suatu field |
| `$nor` | Tidak ada kondisi dalam daftar yang boleh cocok |
| Implicit AND | Beberapa kondisi dalam objek filter otomatis digabungkan dengan AND |
| `$exists` | Memeriksa apakah suatu field ada |
| `$elemMatch` | Mencocokkan satu elemen array yang memenuhi beberapa kondisi |
| Dot notation | Mengakses field di dalam nested document, misalnya `"alamat.kota"` |

### Rumus berpikir

1. Tentukan dokumen seperti apa yang ingin ditemukan.
2. Identifikasi field yang diperlukan.
3. Ubah setiap syarat menjadi kondisi query.
4. Gunakan AND jika semua syarat wajib terpenuhi.
5. Gunakan OR jika ada alternatif.
6. Gunakan NOT atau NOR untuk negasi dengan memperhatikan perilaku field yang hilang.
7. Uji query dengan dokumen yang mencakup nilai normal, `null`, dan field yang tidak ada bila kasus tersebut relevan.

---

## 12. Referensi Resmi

- [MongoDB Manual — Logical Query Operators](https://www.mongodb.com/docs/manual/reference/operator/query-logical/)
- [MongoDB Manual — Query Documents](https://www.mongodb.com/docs/manual/tutorial/query-documents/)
- [MongoDB Manual — Query and Projection Operators](https://www.mongodb.com/docs/manual/reference/operator/query/)
- [MongoDB Manual — `$elemMatch`](https://www.mongodb.com/docs/manual/reference/operator/query/elemMatch/)
- [MongoDB Manual — `$exists`](https://www.mongodb.com/docs/manual/reference/operator/query/exists/)

---

**Catatan:** README ini berfokus pada logical query operators dan pola pemakaiannya. Comparison operators, field operators, array operators, dan aggregation adalah topik terkait yang dapat dipelajari lebih lanjut secara terpisah.
