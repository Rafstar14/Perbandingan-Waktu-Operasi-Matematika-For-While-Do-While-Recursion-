# Benchmark: For vs While vs Do While vs Recursion

Program C++ untuk membandingkan kecepatan empat struktur kendali (`for`, `while`, `do while`, dan rekursi) dalam menjalankan operasi matematika dasar: penjumlahan, pengurangan, perkalian, dan pembagian.

## Struktur Project

```
.
├── user_bench.cpp      # Benchmark utama: 4 metode x 4 operasi, 10 juta iterasi
└── README.md
```

## Cara Menjalankan

Butuh compiler C++17 (GCC/Clang/MSVC).

```bash
# Tanpa optimasi compiler
g++ -std=c++17 -O0 user_bench.cpp -o bench_O0
./bench_O0

# Dengan optimasi compiler
g++ -std=c++17 -O2 user_bench.cpp -o bench_O2
./bench_O2
```

> Flag `-O0`/`-O2` memengaruhi hasil secara signifikan — lihat bagian **Catatan** di bawah.

## Metode yang Diuji

| Metode | Keterangan |
|---|---|
| `for` | Perulangan standar dengan jumlah iterasi diketahui |
| `while` | Perulangan dengan pengecekan kondisi di awal |
| `do while` | Perulangan dengan pengecekan kondisi di akhir |
| Rekursi | Pendekatan *divide and conquer*, tanpa `for`/`while` sama sekali |

Setiap metode menjalankan 10.000.000 operasi yang saling berantai (hasil operasi sebelumnya jadi input berikutnya), dengan nilai awal `100.0`, agar compiler tidak bisa menyederhanakan atau menghapus perhitungan.

## Ringkasan Hasil (`-O2`, 10 juta operasi)

| Operasi | For / While / Do While | Rekursi |
|---|---|---|
| Penjumlahan (+) | 6,2 ms | 18,9 ms |
| Pengurangan (−) | 6,2 ms | 18,9 ms |
| Perkalian (×) | 9,3 ms | 16,1 ms |
| Pembagian (÷) | 43,5 ms | 43,5 ms |

**Kesimpulan singkat:**
- `for`, `while`, dan `do while` selalu setara — compiler menghasilkan instruksi mesin yang identik untuk ketiganya.
- Rekursi lebih lambat untuk operasi berlatensi rendah (+, −, ×) karena biaya pemanggilan fungsi (menyimpan parameter, alamat kembali, lompat, kembali).
- Pembagian adalah operasi paling mahal di antara keempatnya, jauh lebih memengaruhi waktu eksekusi dibanding jenis struktur kendali yang dipakai.
- Pada pembagian, selisih rekursi vs perulangan hilang — latensi pembagian yang tinggi "menyembunyikan" biaya pemanggilan fungsi.

Penjelasan lengkap beserta analisis teknis tiap temuan ada di laporan terpisah (lihat bagian Dokumentasi).

## Catatan

- **Jangan pakai nilai awal `0` untuk perkalian/pembagian** — hasilnya akan selalu `0`, dan compiler ber-optimasi (`-O2`) bisa menghapus seluruh loop sehingga waktu yang terukur jadi `0.000 ms` dan tidak valid.
- Angka pengali `1.0000001` pada perkalian/pembagian dipilih agar terjadi perubahan kecil tiap langkah; setelah 10 juta iterasi, hasilnya mendekati kelipatan bilangan *e* (≈ 2,71828), bukan kebetulan.
- Hasil di atas hanya satu kali pengukuran per metode; pada percobaan ulang, variasi waktu antar-run bisa mencapai ~19% untuk metode yang sama, jadi untuk pengukuran yang lebih ketat sebaiknya tiap metode diulang beberapa kali dan diambil nilai tercepat atau rata-ratanya.
