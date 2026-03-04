# Modul Praktikum Pertemuan 3

# Paradigma Object-Oriented Programming (OOP)

## Tujuan Pembelajaran

Setelah mengikuti praktikum ini, mahasiswa diharapkan mampu:

1. Memahami perbedaan antara abstraksi *Class* dan perwujudan *Object*.  
2. Menerapkan enkapsulasi data menggunakan pembatasan akses `public` dan `private`.  
3. Menggunakan *Constructor* untuk menginisialisasi objek secara aman dan otomatis.  
4. Menyusun dan memanipulasi *Array of Objects* sebagai fondasi simulasi *batch processing*.  
5. Memahami konsep pewarisan (*Inheritance*) dan hak akses tingkat lanjut (`protected`).

# 1. Konsep Dasar Class dan Object

Pendekatan OOP menggeser cara pikir pemrograman dari sekadar "urutan perintah" menjadi "interaksi antar entitas".

- **Class:** Rancangan atau cetak biru (*blueprint*) abstrak. (Contoh: Konsep rancangan "Mobil").  
- **Object:** Wujud fisik yang menduduki memori berdasarkan cetak biru tersebut. (Contoh: "Mobil Toyota warna merah plat D 1234").

Dalam komputasi paralel, memecah 10.000 data ke dalam 10.000 objek independen jauh lebih aman untuk didistribusikan ke berbagai inti prosesor agar data tidak tertukar secara tidak sengaja.

## 1.1 Latihan Praktik: Dasar Pembuatan Class dan Object

Latihan ini akan mendemonstrasikan bagaimana kita mendefinisikan sebuah cetak biru, lalu membuat beberapa wujud nyatanya secara independen.

```cpp
#include <iostream>
#include <string>

// Mendefinisikan Class
class Mobil {  
public: // Sementara menggunakan public agar mudah diakses
    std::string merk;  
    std::string warna;  
    int tahun;

    // Perilaku (Method)  
    void nyalakanMesin() {  
        std::cout << "Mesin mobil " << merk << " (" << warna << ") menyala!" << std::endl;  
    }  
};

int main() {  
    // Membuat Object 1 dari cetak biru Mobil  
    Mobil mobil1;  
    mobil1.merk = "Toyota";  
    mobil1.warna = "Merah";  
    mobil1.tahun = 2022;

    // Membuat Object 2 dari cetak biru yang sama  
    Mobil mobil2;  
    mobil2.merk = "Honda";  
    mobil2.warna = "Hitam";  
    mobil2.tahun = 2020;

    // Memanggil perilaku dari masing-masing object
    mobil1.nyalakanMesin();  
    mobil2.nyalakanMesin();

    return 0;  
}
```

### Penjelasan Kode

- `class Mobil { ... };` mendefinisikan sebuah cetak biru. Ingat selalu bahwa deklarasi `class` diakhiri dengan tanda titik koma (`;`).  
- `Mobil mobil1;` membuat objek fisik (mengalokasikan memori) yang dinamakan `mobil1` berdasarkan cetak biru `Mobil`.  
- Operator titik (`.`), seperti pada `mobil1.merk`, digunakan untuk mengakses atribut (variabel) atau method (fungsi) spesifik milik objek tersebut.

### Instruksi Eksplorasi

1. Buatlah satu objek baru bernama `mobil3` di dalam `main()`. Isi atribut `merk`, `warna`, dan `tahun`-nya dengan mobil impian Anda. Kemudian panggil method `nyalakanMesin()` untuk objek baru tersebut dan jalankan programnya.

# 2. Atribut, Method, dan Enkapsulasi

Enkapsulasi adalah proses pembungkusan data agar terlindungi dari modifikasi ilegal oleh bagian program lain yang tidak berkepentingan. Di C++, kita menggunakan Hak Akses (*Access Modifiers*):

| Hak Akses | Deskripsi | Penggunaan Umum |
|------------|------------|-----------------|
| `private` | Hanya bisa dibaca/diubah oleh isi *Class* itu sendiri. Terkunci dari dunia luar. | Variabel / Data (Atribut). |
| `public` | Bisa dipanggil secara bebas dari mana saja (seperti dari fungsi `main`). | Fungsi / Pintu masuk interaksi (*Method*). |
| `protected` | Terkunci dari luar, tetapi diizinkan untuk diakses oleh *Class* Anak/Keturunannya. | Atribut pada pewarisan (*Inheritance*). |

## 2.1 Latihan Praktik: Class Partikel

```cpp
#include <iostream>

class Partikel {  
private:  
    // Atribut terkunci (tidak bisa diakses langsung dari main)  
    double posisi_x, posisi_y;

public:  
    // Fungsi untuk mengubah posisi (update atribut private)
    void aturPosisi(double x, double y) {  
        posisi_x = x;  
        posisi_y = y;  
    }  
      
    // Fungsi komputasi internal  
    void bergerak(double geser_x, double geser_y) {  
        posisi_x += geser_x;  
        posisi_y += geser_y;  
    }  
      
    void tampilkanPosisi() {  
        std::cout << "Posisi: (" << posisi_x << ", " << posisi_y << ")" << std::endl;  
    }  
};

int main() {  
    Partikel p1; // p1 adalah wujud nyata (Objek)  
      
    p1.aturPosisi(0.0, 0.0);  
    p1.bergerak(2.5, 3.0);  
    p1.tampilkanPosisi();  
      
    return 0;  
}
```

### Penjelasan Kode

- Pemisahan `private` dan `public` memastikan integritas model matematika dalam simulasi tetap terjaga (tidak ada fungsi luar yang merusak koordinat secara acak).  
- Sebuah fungsi di dalam *Object* dipanggil menggunakan notasi titik (`.`), contohnya: `p1.bergerak()`.

### Instruksi Eksplorasi

1. Di dalam fungsi `main()`, tambahkan baris `p1.posisi_x = 100.0;` tepat sebelum baris `return 0;`, lalu lakukan kompilasi dan amati bagaimana *compiler* **menolak** keras instruksi tersebut untuk membuktikan pembatasan akses `private`.  
2. Sekarang, ubah kata kunci `private:` di dalam rancangan `class Partikel` menjadi `public:`, lalu lakukan kompilasi ulang. Amati bahwa program kini berhasil dijalankan. Eksperimen ini menunjukkan bahayanya membiarkan data internal simulasi terbuka, di mana fungsi luar dapat merusak nilai koordinat tanpa pengawasan.

# 3. Constructor: Inisialisasi Otomatis

*Constructor* adalah fungsi istimewa yang **otomatis berjalan di detik pertama** objek dibuat. Tujuannya agar objek tidak pernah memiliki nilai *garbage* (sampah memori).

Syarat mutlak Constructor: Namanya harus persis sama dengan nama `Class`, dan ia tidak memiliki tipe kembalian (bahkan tidak memakai `void`).

## 3.1 Latihan Praktik: Menambahkan Constructor

```cpp
#include <iostream>

class Titik2D {  
private:  
    double x, y;

public:  
    // Constructor (Dipanggil otomatis saat objek dibuat)  
    Titik2D(double nilai_x, double nilai_y) {   
        x = nilai_x;  
        y = nilai_y;  
    }  
      
    void cetak() {   
        std::cout << "X: " << x << " | Y: " << y << std::endl;   
    }  
};

int main() {  
    // Objek dibuat sekaligus diberi nilai awal via Constructor  
    Titik2D titikA(10.5, 20.2);   
    titikA.cetak();  
      
    return 0;  
}
```

### Penjelasan Kode

- Baris `Titik2D titikA(10.5, 20.2);` sekaligus memesan memori dan memasukkan nilai awal.  
- Berkat *Constructor*, kita tidak perlu lagi memanggil fungsi seperti `titikA.aturPosisi(10.5, 20.2)` secara terpisah.

### Instruksi Eksplorasi

1. Ubah baris pembuatan objek di dalam `main()` menjadi sekadar `Titik2D titikA;` (tanpa memberikan nilai argumen sama sekali), lalu jalankan dan perhatikan bagaimana sistem akan menampilkan pesan *error* karena ia memaksa *programmer* untuk disiplin mendefinisikan status awal entitas.

# 4. Array of Objects

Mensimulasikan pebuatan banyak entitas dan memprosesnya secara massal (*batch processing*).

## 4.1 Latihan Praktik: Pemrosesan Sensor Massal

```cpp
#include <iostream>

class Sensor {  
private:  
    int id;   
    double nilai;

public:  
    // Constructor  
    Sensor(int id_sensor) {   
        id = id_sensor;   
        nilai = 0.0;   
    }  
      
    void rekam(double data) {   
        nilai = data;   
    }  
      
    void status() {   
        std::cout << "ID: " << id << " | Nilai: " << nilai << std::endl;   
    }  
};

int main() {  
    // Alokasi 3 objek Sensor ke dalam bentuk Array  
    Sensor jaringan[3] = { Sensor(101), Sensor(102), Sensor(103) };

    // Batch processing
    for (int i = 0; i < 3; i++) {  
        // Simulasi input data yang berbeda untuk tiap sensor  
        jaringan[i].rekam((i + 1) * 12.5);  
    }  
      
    // Output hasil  
    for (int i = 0; i < 3; i++) {  
        jaringan[i].status();  
    }  
      
    return 0;  
}
```

### Penjelasan Kode

- Array `jaringan` tidak berisi angka satuan, melainkan berisi tiga objek `Sensor` yang sepenuhnya mandiri.  
- Karena masing-masing objek `Sensor` mengurung datanya sendiri, iterasi `for` ini sangat aman untuk didistribusikan ke prosesor *multi-core* (Core 1 memproses indeks 0, Core 2 memproses indeks 1, tanpa terjadi bentrokan).

### Instruksi Eksplorasi

1. Hapus nilai bawaan `nilai = 0.0;` pada `Constructor`, kemudian hapus (atau beri *comment*) pada loop `for` pertama yang bertugas merekam data, lalu jalankan program untuk melihat bahwa loop kedua akan mencetak angka acak, yang membuktikan fatalnya jika entitas simulasi tidak diinisialisasi secara benar.

# 5. Pewarisan (Inheritance) & Hak Akses protected

Pewarisan memungkinkan Class Baru (Anak) mengambil dan memperluas sifat Class Lama (Induk) untuk menghindari penulisan ulang kode. Di sinilah hak akses `protected` berperan: jika kita menggunakan `private`, Anak tidak akan bisa melihat atribut Induknya sendiri.

## 5.1 Latihan Praktik: Sensor Spesifik

```cpp
#include <iostream>

// Class Induk (Base Class)  
class SensorDasar {  
protected:   
    // Menggunakan protected agar bisa diwariskan ke Anak  
    int id_sensor;  
    double nilai_bacaan;

public:  
    SensorDasar(int id) {  
        id_sensor = id;  
        nilai_bacaan = 0.0;  
    }  

    void rekam(double data) {   
        nilai_bacaan = data;   
    }  
};

// Class Anak (Derived Class) mewarisi SensorDasar  
class SensorSuhu : public SensorDasar {  
public:  
    // Constructor Anak meneruskan data ke Constructor Induk  
    SensorSuhu(int id) : SensorDasar(id) {}

    // Method khusus milik Anak  
    void kalibrasiDanCetak() {  
        // Anak BISA memanggil nilai_bacaan secara langsung berkat 'protected'  
        double kalibrasi = nilai_bacaan + 1.5;   
        std::cout << "Sensor Suhu ID " << id_sensor   
                  << " | Kalibrasi: " << kalibrasi << " C\n";  
    }  
};

int main() {  
    SensorSuhu s1(99);       // Membentuk objek dari Class Anak  
    s1.rekam(35.0);          // Memanggil fungsi warisan dari Induk  
    s1.kalibrasiDanCetak();  // Memanggil fungsi spesifik dari Anak  
      
    return 0;  
}
```

### Penjelasan Kode

- `class SensorSuhu : public SensorDasar` menandakan bahwa `SensorSuhu` merupakan pengembangan dari `SensorDasar`. Ia mewarisi semua sifat induknya (termasuk fungsi `rekam`).  
- Fungsi `kalibrasiDanCetak` dapat mengakses variabel `nilai_bacaan` karena di set sebagai `protected`.

### Instruksi Eksplorasi

1. Ubah kata `protected` pada struktur `SensorDasar` menjadi `private`, lalu lakukan kompilasi dan amati bagaimana *compiler* langsung menolak mengeksekusi program karena *Class* `SensorSuhu` dianggap mencoba mencuri data dari induknya sendiri!

# 6. Tugas Mandiri

Buatlah sebuah program C++ sederhana dengan spesifikasi berikut:

1. Buat *Class* bernama `RekeningBank`.  
2. Miliki atribut `private` bernama `saldo` (tipe `double`).  
3. Sediakan *Constructor* untuk mengisi saldo awal saat objek dibuat.  
4. Sediakan method `public`: `setor(double jumlah)` dan `tarik(double jumlah)`.  
5. Pada method `tarik`, tambahkan logika kondisi `if-else`. Jika nominal penarikan melebihi isi `saldo`, cetak pesan `"Penarikan Gagal"` dan batalkan pemotongan. Jika saldonya cukup, potong `saldo` sesuai nominal dan cetak sisa saldonya.

# Ringkasan

Pada pertemuan ini mahasiswa telah mempelajari:

- Pergeseran paradigma dari sekuensial menjadi **Berbasis Objek (OOP)**.  
- Pengamanan data komputasi melalui mekanisme enkapsulasi **Private** dan **Public**.  
- Otomatisasi pengaturan nilai memori awal melalui **Constructor**.  
- Eksekusi massal objek mandiri sebagai fondasi *parallel tasking* menggunakan **Array of Objects**.  
- Hierarki pembagian sifat objek melalui konsep **Inheritance** dan hak akses **Protected**.
