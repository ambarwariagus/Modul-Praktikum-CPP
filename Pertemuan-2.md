# Modul Praktikum Pertemuan 2

# Manajemen Memori Dasar, Fungsi, dan Array

## Tujuan Pembelajaran

Setelah mengikuti praktikum ini, mahasiswa diharapkan mampu:

1. Memahami konsep modularitas dan menyusun fungsi (*function*) mandiri.  
2. Membedakan alokasi memori berdasarkan *Value*, *Pointer*, dan *Reference*.  
3. Menerapkan *Pass by Reference* untuk efisiensi memori (krusial dalam komputasi *big data*).  
4. Mendeklarasikan dan memanipulasi struktur data Array 1 Dimensi (Vektor).  
5. Mendeklarasikan dan memanipulasi struktur data Array 2 Dimensi (Matriks) menggunakan perulangan bersarang (*nested loop*).

# 1. Fungsi / Modularitas Kode

Fungsi adalah blok kode independen yang dirancang untuk menjalankan tugas spesifik. Dalam komputasi paralel, fungsi-fungsi inilah yang nantinya didistribusikan ke berbagai *thread* prosesor.

## 1.1 Anatomi Fungsi

- **Tipe Kembalian (*Return Type*):** Tipe data hasil keluaran (misal: `double`, `int`, atau `void` jika tidak menghasilkan nilai balik).  
- **Nama Fungsi:** Identitas fungsi untuk pemanggilan.  
- **Parameter:** Data input yang dikirim ke dalam fungsi.

## 1.2 Latihan Praktik: Fungsi Volume

```cpp
#include <iostream>

// Mendefinisikan fungsi (Tipe Kembalian: double, Nama: hitungVolumeBalok)  
double hitungVolumeBalok(double panjang, double lebar, double tinggi) {  
    return panjang * lebar * tinggi; // Mengembalikan hasil perhitungan  
}

int main() {  
    // Memanggil fungsi dan menyimpannya ke variabel 'volume'  
    double volume = hitungVolumeBalok(10.5, 5.0, 2.0);  
      
    std::cout << "Volume balok: " << volume << std::endl;  
    return 0;  
}
```

### Penjelasan Kode

- Fungsi `hitungVolumeBalok` ditulis di luar fungsi `main()`.  
- Kata kunci `return` menghentikan fungsi dan mengirimkan nilai hasil perhitungan kembali ke baris pemanggil di dalam `main()`.

### Instruksi Eksplorasi

1. Ubah `double hitungVolumeBalok` menjadi `void hitungVolumeBalok` dan hapus baris `return`. Gantilah dengan perintah `std::cout` di dalam fungsi tersebut untuk langsung mencetak hasil dari dalam fungsi. Terakhir, ubah pemanggilan di `main()` menjadi sekadar `hitungVolumeBalok(10.5, 5.0, 2.0);` tanpa menyimpannya ke variabel `volume`, lalu analisis perbedaan cara kerjanya.

# 2. Pointer dan Reference: Mengintip Memori

Ini adalah fitur tingkat rendah C++ yang membedakannya dari bahasa pemrograman lain seperti Python. Memahami konsep ini akan menyelamatkan program Anda dari *Out of Memory* (kehabisan RAM) saat memproses jutaan data secara paralel.

## 2.1 Konsep Akses Memori

| Konsep | Simbol | Deskripsi | Kegunaan |
|--------|--------|------------|----------|
| **Value** | `-` | Menyimpan nilai aktual (misal: `100`). | Penggunaan sehari-hari (*Pass by Value* / Salinan data). |
| **Pointer** | `*` | Menyimpan "alamat memori fisik" (misal: `0x7ffee98`). | Alokasi memori dinamis tingkat lanjut. |
| **Reference** | `&` | Menjadi nama *alias* untuk alamat memori yang sudah ada. | Melempar data besar ke dalam fungsi tanpa menyalinnya (*Pass by Reference*). |

## 2.2 Latihan Praktik: Dasar Pointer dan Alamat Memori

Latihan ini akan membantu Anda memvisualisasikan bagaimana variabel benar-benar disimpan di dalam perangkat keras memori komputer.

```cpp
#include <iostream>

int main() {  
    int angka = 100;  
    int *ptr_angka = &angka; // Pointer menyimpan alamat memori dari variabel 'angka'

    std::cout << "Nilai variabel angka: " << angka << std::endl;  
    std::cout << "Alamat memori angka (&angka): " << &angka << std::endl;  
    std::cout << "Isi dari pointer ptr_angka: " << ptr_angka << std::endl;  
    std::cout << "Nilai yang ditunjuk pointer (*ptr_angka): " << *ptr_angka << std::endl;

    return 0;  
}
```

### Penjelasan Kode

- `&angka` (Address-of): Digunakan untuk mendapatkan alamat lokasi memori fisik tempat data disimpan (biasanya dalam format heksadesimal).  
- `int *ptr_angka`: Mendeklarasikan sebuah variabel bertipe pointer yang secara spesifik dirancang untuk menyimpan alamat memori.  
- `*ptr_angka` (Dereference): Menambahkan bintang di depan pointer yang sudah ada akan "membuka" alamat tersebut dan membaca nilai aslinya.

**💡 CATATAN PENTING: Gaya Penulisan Pointer (`int* p` vs `int *p`)** Di C++, penulisan `int* p;` dan `int *p;` bermakna 100% sama (keduanya adalah pointer). Namun, **berhati-hatilah (Awas Jebakan!)** jika mendeklarasikan lebih dari satu variabel di baris yang sama.

- Jika Anda menulis: `int* A, B;` C++ membacanya sebagai: `A` adalah pointer, sedangkan `B` hanyalah variabel integer biasa!  
- Jika Anda ingin keduanya menjadi pointer dalam satu baris, Anda harus menuliskan: `int *A, *B;`

**Praktik Terbaik:** Untuk menghindari kebingungan, biasakan mendeklarasikan variabel dengan aturan: **"Satu baris, satu variabel."**

```cpp
int* A;  
int* B;
```

### Instruksi Eksplorasi

1. Tambahkan kode `*ptr_angka = 500;` tepat sebelum baris `return 0;`, lalu cetak kembali nilai `angka` menggunakan `std::cout`. Amati bahwa nilai variabel `angka` secara otomatis ikut berubah menjadi 500, meskipun Anda tidak menyentuh variabel `angka` sama sekali! (Ini membuktikan bahwa Anda baru saja memanipulasi memori secara langsung).

## 2.3 Latihan Praktik: Fungsi Tukar (Swap) dengan Reference

```cpp
#include <iostream>

// Menggunakan simbol & (reference) agar mengakses memori asli di fungsi main  
void tukarNilai(int &a, int &b) {  
    int sementara = a; // Simpan nilai A agar tidak hilang ditimpa  
    a = b;             // Timpa A dengan B  
    b = sementara;     // Isi B dengan nilai A yang lama  
}

int main() {  
    int x = 100, y = 500;  
      
    std::cout << "Sebelum ditukar : x = " << x << ", y = " << y << std::endl;  
    tukarNilai(x, y);   
    std::cout << "Setelah ditukar : x = " << x << ", y = " << y << std::endl;  
      
    return 0;  
}
```

### Penjelasan Kode

- Parameter `int &a, int &b` menyatakan bahwa fungsi ini tidak meminta "fotokopi" dari variabel `x` dan `y`, melainkan meminta "kunci akses" langsung ke lokasi memori `x` dan `y`.

### Instruksi Eksplorasi

1. **Eksperimen Krusial:** Hapus tanda `&` pada parameter sehingga menjadi `void tukarNilai(int a, int b)`, lalu jalankan program. Perhatikan mengapa nilai `x` dan `y` di layar **tidak tertukar**.

> **Petunjuk:** Pahami konsep bahwa tanpa `&`, C++ hanya mengubah salinan data di dalam fungsi yang akan musnah saat fungsi selesai. Pemahaman ini adalah kunci komputasi data besar!

# 3. Array 1D dan 2D

Array adalah deretan blok memori yang bersebelahan (*contiguous*) untuk menyimpan sekumpulan data bertipe sama.

**Aturan Emas:** Indeks array di C++ selalu dimulai dari **0**.

- **Array 1D:** Deret linier (vektor matematika).  
- **Array 2D:** Tabel baris dan kolom (matriks, gambar satelit, grid spasial).

## 3.1 Latihan Praktik: Penjumlahan Vektor (Array 1D)

```cpp
#include <iostream>

int main() {  
    const int ukuran = 5; // Ukuran array klasik harus statis (diketahui dari awal)  
      
    int vektorA[ukuran] = {1, 2, 3, 4, 5};  
    int vektorB[ukuran] = {10, 20, 30, 40, 50};  
    int hasil[ukuran];

    // Iterasi memproses elemen yang sejajar satu per satu  
    for(int i = 0; i < ukuran; i++) {  
        hasil[i] = vektorA[i] + vektorB[i];  
        std::cout << "Elemen indeks ke-" << i << ": " << hasil[i] << std::endl;  
    }  
      
    return 0;  
}
```

### Penjelasan Kode

- `int vektorA[ukuran]` mendeklarasikan 5 kotak alokasi memori berderet.
- Looping `for` memanfaatkan variabel `i` sebagai penunjuk indeks yang bergeser dari `0` hingga `4`.
- `hasil[i] = vektorA[i] + vektorB[i]` menjumlahkan nilai pada indeks yang sama dan menyimpannya ke array baru.

### Instruksi Eksplorasi

1. Ubah nilai variabel `ukuran` menjadi `6` (ingat bahwa kita hanya mengisi 5 angka di `vektorA` dan `vektorB`), lalu jalankan program dan perhatikan elemen ke-5 (indeks terakhir) yang muncul akan berupa angka acak (*Garbage Value*) sisa dari memori aplikasi lain.  
2. Ubah kondisi loop menjadi `i <= ukuran`. Komputer akan mencoba menjumlahkan indeks yang berada di luar batas alokasi array (*Out of Bounds*), di mana hal ini merupakan penyebab utama program paralel mengalami *Segmentation Fault* (mati mendadak).

# 4. Operasi Matriks (Array 2D)

Memanipulasi Array 2D membutuhkan *Nested Loop* (Perulangan Bersarang). Di dalam komputasi paralel (seperti OpenMP), *nested loop* inilah yang paling sering dioptimasi dan diparalelkan.

## 4.1 Latihan Praktik: Skalar Matriks 2D

```cpp
#include <iostream>

int main() {  
    // Matriks 2 dimensi: [2 Baris] x [3 Kolom]  
    int matriks[2][3] = {   
        {1, 2, 3},   
        {4, 5, 6}   
    };  
    int pengali = 2;  
      
    // Loop Baris (Dimensi 1 - Luar)  
    for(int i = 0; i < 2; i++) {  
          
        // Loop Kolom (Dimensi 2 - Dalam)  
        for(int j = 0; j < 3; j++) {  
            matriks[i][j] = matriks[i][j] * pengali;  
            std::cout << matriks[i][j] << "\t"; // \t untuk spasi Tab  
        }  
          
        std::cout << std::endl; // Pindah baris setelah 1 baris kolom selesai  
    }  
      
    return 0;  
}
```

### Penjelasan Kode

- Loop luar (`i`) menahan baris agar tetap diam, sementara loop dalam (`j`) bergerak menyelesaikan komputasi (perkalian) pada seluruh kolom di baris tersebut.  
- Setelah satu baris selesai, perintah `std::endl` memutus baris di terminal agar tampilannya menyerupai bentuk matriks asli.  
- Pola pembacaan baris-demi-baris ini disebut *Row-Major Order* dan sangat efisien untuk arsitektur memori C++.

### Instruksi Eksplorasi

1. Ubah urutan *loop* dengan meletakkan `for (int j = 0...` (kolom) di luar dan `for (int i = 0...` (baris) di dalam, lalu sesuaikan logika `std::endl`-nya agar bentuknya tetap rapi. Jalankan dan lihat perubahan bentuk cetakannya untuk menyimulasikan pembacaan *Column-Major*, yang secara arsitektur memori C++ lebih lambat karena pembacaannya "melompat-lompat" antar baris memori.

# 5. Tugas Mandiri

## Tugas 1: Menggabungkan Fungsi dan Array

Buat program C++ untuk menghitung rata-rata:

1. Buat sebuah fungsi bernama `hitungRataRata` di luar fungsi `main()`.  

2. Fungsi ini menerima dua parameter: (1) Array 1D bertipe `double`, dan (2) ukuran elemen array tersebut (`int`).  

3. Di dalam fungsi tersebut, gunakan perulangan untuk menghitung total akumulasi seluruh elemen, lalu kembalikan (`return`) nilai rata-ratanya.  

4. Di dalam fungsi `main()`, buat sebuah array berisi 5 angka sembarang, panggil fungsi `hitungRataRata` tersebut, dan cetak hasilnya ke layar.

---

## Tugas 2: Manipulasi Memori dengan Pointer

Buatlah program C++ yang memanipulasi nilai variabel secara langsung di memori:

1. Buat sebuah fungsi bernama `kuadratkan` bertipe `void` yang menerima satu parameter berupa pointer ke integer (`int *val`).  

2. Di dalam fungsi tersebut, ubah nilai di alamat memori yang ditunjuk pointer menjadi nilai kuadratnya (kalikan nilai memori tersebut dengan dirinya sendiri).  

3. Di dalam fungsi `main()`, deklarasikan sebuah variabel (misalnya `int angka = 5;`), lalu cetak nilai awalnya.  

4. Panggil fungsi `kuadratkan` dengan mengirimkan alamat memori dari variabel tersebut (gunakan operator `&`).  

5. Cetak kembali variabel `angka` di `main()` dan pastikan nilainya telah berubah menjadi `25` meskipun Anda tidak menggunakan `return` value.

---

# Ringkasan

Pada pertemuan ini mahasiswa telah mempelajari:

- Modularitas kode menggunakan **Fungsi**.  
- Manajemen memori tingkat rendah dengan membedakan **Value**, **Pointer**, dan **Reference**.  
- Penggunaan Memori Bersama (*Shared Memory*) antar fungsi tanpa harus menduplikasi data yang besar.  
- Deklarasi dan iterasi memori pada **Array 1D (Vektor)** dan **Array 2D (Matriks)** menggunakan perulangan tunggal dan perulangan bersarang.

Penguasaan pointer/reference dan array ini adalah kunci untuk menjaga keamanan alokasi memori sebelum kita melangkah lebih jauh ke konsep Objek (OOP) di pertemuan selanjutnya.
