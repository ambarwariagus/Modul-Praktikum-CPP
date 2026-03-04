# Modul Praktikum Pertemuan 4

# Struktur Data Modern dan File I/O

## Tujuan Pembelajaran

Setelah mengikuti praktikum ini, mahasiswa diharapkan mampu:

1. Mengimplementasikan `std::vector` sebagai struktur array dinamis yang modern dan aman.  
2. Menyusun simulasi pemrosesan batch menggunakan *Vector of Objects*.  
3. Memahami dan mengidentifikasi bahaya *Data Race* pada memori bersama (*Shared Memory*).  
4. Mengoperasikan baca/tulis (*Input/Output*) file eksternal teks menggunakan pustaka `<fstream>`.  
5. Mengevaluasi ketergantungan antar iterasi dalam perulangan (dependensi loop) sebagai kesiapan masuk ke komputasi paralel.

# 1. Standard Template Library (STL): std::vector

Array klasik (Pertemuan 2) memiliki sifat *fixed-size* (ukurannya kaku). Jika dialokasikan 5 elemen, ia tidak bisa menampung elemen ke-6.

C++ menyediakan `std::vector`, sebuah array dinamis modern yang memori internalnya menyesuaikan ukuran secara otomatis (*auto-resize*).

Berikut adalah beberapa fungsi dasar (method) yang sering digunakan pada `std::vector`:

| Method | Fungsi / Kegunaan |
|--------|-------------------|
| `.push_back(data)` | Menambah elemen baru di barisan paling belakang. |
| `.size()` | Mendapatkan jumlah elemen aktual yang ada di dalam vector. |
| `.pop_back()` | Membuang elemen paling belakang. |
| `.clear()` | Mengosongkan seluruh isi memori vector. |

## 1.1 Latihan Praktik: Membuat Vector Dinamis

```cpp
#include <iostream>
#include <vector> // Pustaka Wajib

int main() {  
    // Deklarasi array dinamis bertipe integer  
    std::vector<int> kumpulan_angka;

    // Menyuntikkan data tanpa batasan awal  
    kumpulan_angka.push_back(10);  
    kumpulan_angka.push_back(25);  
    kumpulan_angka.push_back(50);

    // .size() akan otomatis bernilai 3  
    for(int i = 0; i < kumpulan_angka.size(); i++) {  
        std::cout << "Data ke-" << i << ": " << kumpulan_angka[i] << std::endl;  
    }  
      
    return 0;  
}
```

### Penjelasan Kode

- Berbeda dengan array biasa `int arr[3]`, pada vektor kita cukup menulis `std::vector<int> nama_var;`. Alokasi memori dikelola penuh di belakang layar oleh C++.

### Instruksi Eksplorasi

1. Tambahkan perintah `kumpulan_angka.pop_back();` tepat di bawah `push_back(50);`, lalu jalankan program dan amati.
2. Tambahkan `kumpulan_angka.clear();` tepat sebelum loop `for`, lalu jalankan program dan amati.

# 2. Batch Processing Objek Dinamis

Menggabungkan konsep OOP (Pertemuan 3) dengan `vector`. Di ranah komputasi paralel seperti OpenMP atau MPI, titik injeksi komputasi paralel biasanya diletakkan persis di atas struktur iterasi vector ini.

## 2.1 Latihan Praktik: Pemrosesan Antrean Kalkulasi

```cpp
#include <iostream>
#include <vector>

class TugasKalkulasi {  
private:  
    int id_tugas;  
    double nilai_A;  
    double nilai_B;  
    double hasil;

public:  
    // Constructor menginisialisasi angka yang akan dihitung  
    TugasKalkulasi(int id, double a, double b) {  
        id_tugas = id;  
        nilai_A = a;  
        nilai_B = b;  
        hasil = 0.0;  
    }

    // Method komputasi (Di sinilah proses paralel nantinya bekerja)  
    void kerjakan() {  
        hasil = nilai_A + nilai_B;
    }

    void tampilkanHasil() {  
        std::cout << "Tugas " << id_tugas << " | "   
                  << nilai_A << " + " << nilai_B   
                  << " = " << hasil << std::endl;  
    }  
};

int main() {  
    std::vector<TugasKalkulasi> antrean;

    // Memasukkan antrean tugas dengan data yang berbeda-beda  
    antrean.push_back(TugasKalkulasi(1, 10.5, 5.0));  
    antrean.push_back(TugasKalkulasi(2, 20.0, 7.5));  
    antrean.push_back(TugasKalkulasi(3, 50.2, 10.1));

    // Proses massal (Sangat aman untuk diparalelkan)  
    for(int i = 0; i < antrean.size(); i++) {  
        antrean[i].kerjakan();  
        antrean[i].tampilkanHasil();  
    }

    return 0;  
}
```

### Instruksi Eksplorasi

1. Hapus ketiga baris `push_back` manual tersebut, lalu gantilah menggunakan struktur perulangan `for` agar program mampu membangkitkan dan memasukkan secara otomatis 100 tugas kalkulasi secara dinamis.

> **Petunjuk:** Anda bisa menggunakan nilai `i` dari loop pembentuk antrean sebagai bagian dari perhitungan, misalnya `antrean.push_back(TugasKalkulasi(i, i * 2.5, i * 1.5));`.

# 3. Data Race (Kecelakaan Memori)

Ini adalah materi konseptual terpenting sebelum mempelajari komputasi paralel. *Data Race* adalah salah satu kesalahan paling fatal di komputasi paralel. Hal ini terjadi ketika:

1. Banyak *Core* prosesor mengakses satu lokasi variabel memori yang sama persis secara bersamaan.  
2. Operasinya melibatkan modifikasi atau pengubahan data (*Write*).  
3. Tidak ada mekanisme antrean/kunci sinkronisasi yang diterapkan.

## 3.1 Latihan Praktik: Reduction

Jika iterasi di bawah ini dipaksa berjalan secara paralel tanpa teknik penguncian (*Lock/Reduction*), hasilnya akan menjadi tidak terprediksi.

```cpp
#include <iostream>
#include <vector>

int main() {  
    std::vector<int> data = {10, 20, 30, 40};  
    int total = 0; // <-- TARGET BENTROKAN (DATA RACE)

    // Simulasi: Jika 4 Core berebut mengubah variabel 'total' bersamaan  
    for(int i = 0; i < data.size(); i++) {  
        total = total + data[i];   
    }

    std::cout << "Total Akumulasi: " << total << std::endl;  
    return 0;  
}
```

### Analisis Konseptual

- Bayangkan Core A membaca `total = 0`, lalu bersiap menambah `10`.  
- Pada milidetik yang sama, Core B membaca `total = 0`, bersiap menambah `20`.  
- Core A menulis `10` ke memori. Sepersekian detik kemudian Core B menimpanya dengan `20`.  
- Angka `10` hilang. Hasil akhirnya bukan `30`, melainkan `20`. Inilah bahayanya memori bersama (*shared state*) yang dibiarkan terbuka saat proses paralel berlangsung.

### Instruksi Eksplorasi

1. Pindahkan deklarasi `int total = 0;` ke *dalam* blok loop `for` (sehingga menjadi variabel lokal loop). Kompilasi dan jalankan program. Mengapa output perhitungannya menjadi kacau?

# 4. Operasi File I/O: Membaca & Menulis File

Di proyek sains dan pengolahan data riil, data eksperimen (jutaan baris) tidak diketik secara manual, melainkan dibaca dari file eksternal (seperti `.txt` atau `.csv`), dan hasil perhitungannya pun disimpan kembali ke dalam file.

C++ menyediakan pustaka `<fstream>` (*File Stream*) untuk menangani hal ini:

| Pustaka (Class) | Mode Operasi | Fungsi / Kegunaan |
|-----------------|-------------|-------------------|
| `std::ofstream` | *Output* (O) | Digunakan khusus untuk **menulis** data ke dalam file eksternal. |
| `std::ifstream` | *Input* (I) | Digunakan khusus untuk **membaca** data dari file eksternal. |

## 4.1 Latihan Praktik: Siklus Tulis dan Baca Eksternal

```cpp
#include <iostream>
#include <fstream> // Pustaka akses file
#include <string>  // Pustaka teks

int main() {  
    // --- 1. PROSES MENULIS FILE ---  
    std::ofstream fileTulis("data_sensor.txt"); 

    if (fileTulis.is_open()) {  
        fileTulis << "100.5\n"; // \n untuk pindah baris dalam file  
        fileTulis << "200.0\n";  
        fileTulis << "150.5\n";  
        fileTulis.close(); // Wajib ditutup setelah selesai digunakan!  
        std::cout << "=> Data tersimpan di 'data_sensor.txt'\n";  
    } else {  
        std::cout << "Gagal membuat file.\n";  
    }

    // --- 2. PROSES MEMBACA FILE ---  
    std::ifstream fileBaca("data_sensor.txt");  
    std::string baris_teks;  
      
    std::cout << "\n--- Membaca isi file ---" << std::endl;  
    if (fileBaca.is_open()) {  
        // Terus membaca selama masih ada baris baru di dalam file  
        while (std::getline(fileBaca, baris_teks)) {  
            std::cout << "Isi file: " << baris_teks << std::endl;  
        }  
        fileBaca.close();  
    } else {  
        std::cout << "File tidak ditemukan!\n";  
    }

    return 0;  
}
```

### Penjelasan Kode

- `std::ofstream` akan langsung merakit file baru jika file tersebut belum ada di komputer Anda. Jika sudah ada, isinya akan ditimpa.  
- Fungsi `getline()` membaca file baris demi baris.

### Instruksi Eksplorasi

1. Ubah nama file di bagian ke-2 (proses membaca file) menjadi `"file_salah.txt"`, lalu jalankan program dan amati.
2. Buka *File Explorer*, cari file `data_sensor.txt` yang baru saja dibuat, lalu tambahkan angka `999.9` secara manual menggunakan Notepad dan simpan file tersebut. Berikan komentar pada blok kode ke-1 (proses menulis), lalu jalankan ulang program dan amati hasilnya.

## 4.2 Latihan Praktik: Pemrosesan Data Eksternal dengan Vector

Latihan ini menggabungkan seluruh pemahaman dasar (OOP, Vector, Loop, File I/O) ke dalam satu alur kerja utuh yang sering dijumpai di komputasi sains. Kita akan membaca data mentah posisi dari file `.txt`, menyimpannya ke memori, melakukan kalkulasi matematika secara massal, lalu menyimpan hasilnya kembali ke file baru.

**Skenario:** Kita memiliki ribuan titik partikel. Kita ingin menghitung jarak setiap partikel tersebut ke titik pusat menggunakan rumus Phytagoras.

### Langkah 1: Persiapan Data Mentah

Buatlah sebuah file teks baru secara manual bernama `koordinat_mentah.txt` di folder yang sama dengan program C++ Anda. Isi file tersebut dengan data tabel berikut (termasuk baris *header*-nya):

| Posisi_X | Posisi_Y |
|----------|----------|
| 10.0     | 5.0      |
| -3.0     | 4.0      |
| 12.5     | -8.2     |
| 0.0      | 15.0     |
| 25.4     | 11.1     |
| -8.8     | -8.8     |

### Langkah 2: Kode Pemrosesan C++

Ketikkan dan jalankan kode C++ berikut. Program ini akan membaca *file* yang baru saja Anda buat secara otomatis.

```cpp
#include <iostream>
#include <fstream>
#include <vector>
#include <cmath>  // Pustaka fungsi matematika (std::sqrt)
#include <string> // Pustaka string untuk membaca teks header

// 1. Blueprint Objek  
class Partikel {  
private:  
    double x, y;  
    double jarak_pusat;

public:
    Partikel(double pos_x, double pos_y) {  
        x = pos_x;
        y = pos_y;
        jarak_pusat = 0.0;
    }

    // Fungsi hitung jarak
    void hitungJarak() {
        jarak_pusat = std::sqrt((x * x) + (y * y));
    }  
};

int main() {  
    // A. FASE I/O BACA
    std::vector<Partikel> kumpulan_partikel;  
    std::ifstream fileBaca("koordinat_mentah.txt");  
      
    std::string header1, header2; // Variabel sementara penampung teks  
    double temp_x, temp_y;

    if (fileBaca.is_open()) {
        
        fileBaca >> header1 >> header2;

        while (fileBaca >> temp_x >> temp_y) {   
            kumpulan_partikel.push_back(Partikel(temp_x, temp_y));
        }  
        fileBaca.close();  
    } else {  
        std::cout << "Gagal! Pastikan file koordinat_mentah.txt sudah Anda buat." << std::endl;  
        return 1; // Menghentikan program jika file tidak ada  
    }

    // B. FASE KOMPUTASI (Batch Processing)  
    // Di sinilah pragma paralel seperti OpenMP nantinya
    for (int i = 0; i < kumpulan_partikel.size(); i++) {  
        kumpulan_partikel[i].hitungJarak();  
    }

    // C. FASE I/O TULIS
    std::ofstream fileHasil("hasil_jarak.txt");  
    if (fileHasil.is_open()) {  
        for (int i = 0; i < kumpulan_partikel.size(); i++) {  
            fileHasil << "Partikel " << i + 1  
                      << " | Posisi: (" << kumpulan_partikel[i].x << ", " << kumpulan_partikel[i].y << ") "  
                      << "| Jarak: " << kumpulan_partikel[i].jarak_pusat << "\n";  
        }  
        fileHasil.close();  
        std::cout << "=> Selesai! Buka file 'hasil_jarak.txt' untuk melihat kalkulasinya." << std::endl;  
    }

    return 0;  
}
```

### Penjelasan Kode

- `<cmath>`: Memuat fungsi matematika esensial C++, seperti `std::sqrt()` untuk akar kuadrat.  
- `fileBaca >> header1 >> header2;`: Teknik untuk membersihkan atau melewati ("*skip*") baris pertama *file txt* yang biasanya berisi teks judul kolom sebelum memasuki perulangan untuk membaca kumpulan angka.  
- `while (fileBaca >> temp_x >> temp_y)`: Berbeda dengan `getline()` yang membaca satu baris penuh sebagai teks biasa, operator `>>` dengan berguna untuk mendeteksi angka dan memisahkannya berdasarkan spasi secara langsung ke dalam variabel *double*.  
- **Mengapa ini aman untuk paralel?** Karena saat loop `for (int i = 0 ...)` berjalan, setiap iterasi memodifikasi objek `Partikel` miliknya sendiri di dalam *vector*. Indeks `0` tidak pernah berpotensi mengubah data milik indeks `1`, sehingga 100% aman dari *Data Race*.

### Instruksi Eksplorasi

1. **Uji Elastisitas Vector:** Tambahkan 5 baris titik koordinat angka acak tambahan secara manual ke dalam file teks `koordinat_mentah.txt` Anda, simpan perubahannya, lalu jalankan ulang program C++ tanpa mengubah kode sama sekali. Amati di file `hasil_jarak.txt`.
2. **Modifikasi Komputasi:** Ubah isi method `hitungJarak()` agar menghitung jarak partikel terhadap sebuah titik acuan baru, misalnya titik koordinat `(5.0, 5.0)`. Kompilasi ulang program dan periksa perubahannya di file output hasil.

# 5. Tugas Mandiri

Buatlah sebuah program C++ lengkap yang mensimulasikan proses kalibrasi data sensor dan menyimpan hasilnya ke dalam file eksternal.

Program Anda harus memenuhi langkah-langkah berikut secara berurutan:

1. **Inisialisasi Struktur Data**
   - Deklarasikan sebuah `std::vector<double>` bernama `sensor` dalam keadaan kosong.

2. **Pengisian Data Awal**
   - Gunakan method `.push_back()` untuk memasukkan tiga nilai awal berikut ke dalam vector:
     - `100.5`
     - `200.0`
     - `150.5`

3. **Proses Kalibrasi (Batch Processing)**
   - Gunakan perulangan `for` untuk mengakses setiap elemen dalam vector.
   - Kalikan setiap nilai dengan `1.1` untuk mensimulasikan kenaikan kalibrasi sebesar 10%.
   - Simpan kembali hasil perhitungannya ke elemen vector yang sama.

4. **Penyimpanan ke File Eksternal**
   - Buka file output menggunakan `std::ofstream`.
   - Beri nama file output: `hasil_kalibrasi.txt`.
   - Pastikan file berhasil dibuka sebelum menulis data.

5. **Menulis Hasil ke File**
   - Gunakan perulangan `for` untuk menuliskan seluruh nilai hasil kalibrasi ke dalam file.
   - Setiap nilai ditulis pada baris terpisah.
   - Tutup file setelah proses penulisan selesai.

### Output yang Diharapkan

File `hasil_kalibrasi.txt` harus berisi tiga nilai baru yang sudah dikalibrasi (bertambah 10% dari nilai awal).

# Ringkasan

Pada pertemuan ini mahasiswa telah mempelajari:

- Penggunaan struktur data dinamis modern melalui **std::vector**.  
- Integrasi **OOP** dengan *Vector of Objects* untuk simulasi *batch processing*.  
- Identifikasi risiko komputasi paralel melalui konsep **Data Race** pada memori bersama (*Shared Memory*).  
- Implementasi mekanisme **File Input/Output** menggunakan pustaka `<fstream>`.  
- Evaluasi dependensi perulangan sebagai fondasi kesiapan menuju **Parallel Computing**.

Penguasaan struktur data dinamis, pemrosesan massal, dan manajemen I/O ini menjadi landasan penting sebelum memasuki implementasi komputasi paralel secara eksplisit menggunakan OpenMP, MPI, atau CUDA.
