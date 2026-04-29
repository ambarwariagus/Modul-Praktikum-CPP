# Modul Pengantar Komputasi Paralel dengan MPI (Message Passing Interface)

## Deskripsi

Modul ini berisi pengantar praktis pemrograman paralel menggunakan standar MPI dengan bahasa C. Materi disusun mulai dari inisialisasi lingkungan dasar hingga operasi tingkat lanjut. Modul ini dirancang dengan pendekatan analogi dunia nyata agar konsep komputasi paralel mudah dipahami oleh mahasiswa dari berbagai disiplin ilmu, tidak hanya dari Ilmu Komputer.

---

# Pendahuluan: Apa itu Komputasi Paralel dan MPI?

Bayangkan Anda harus mengecat tembok sepanjang 100 meter. Jika Anda mengerjakannya sendiri (Komputasi Sekuensial), butuh waktu 10 hari. Namun, jika Anda mempekerjakan 10 orang untuk mengecat bagian yang berbeda secara bersamaan, tugas tersebut bisa selesai dalam 1 hari.

Inilah inti dari **Komputasi Paralel**: memecah masalah besar menjadi bagian-bagian kecil yang dikerjakan secara bersamaan.

---

# Dua Model Utama Memori Komputasi Paralel

Sebelum kita membahas MPI secara spesifik, penting untuk memahami bagaimana para "pekerja" (prosesor) ini menyimpan dan berbagi informasi (data) di dalam sistem. Secara umum, arsitektur paralel dibagi menjadi dua model memori utama:

---

## 1. Shared Memory Model (Memori Berbagi)

### Konsep

Semua prosesor mengakses satu ruang memori fisik (RAM) yang sama. Perubahan data yang dilakukan oleh satu prosesor seketika itu juga dapat dilihat oleh prosesor lainnya tanpa perlu saling mengirim file.

### Karakteristik

- Sangat cepat untuk berbagi data
- Rawan tabrakan (*race condition*) jika dua prosesor mencoba mengubah data di alamat memori yang sama persis pada waktu bersamaan
- Skalabilitas terbatas pada kapasitas satu papan induk (*motherboard*)

### Analogi

Beberapa koki memasak bersama di satu dapur besar dan menggunakan rak bumbu yang sama. Jika stoples garam kosong, semua koki yang melihat rak tersebut akan langsung tahu secara otomatis.

---

## 2. Distributed Memory Model (Memori Terdistribusi)

### Konsep

Setiap prosesor memiliki memori lokal (RAM) sendiri-sendiri secara privat. Prosesor A tidak bisa langsung membaca atau mengubah isi memori Prosesor B. Untuk bertukar data, mereka secara eksplisit harus saling mengirim "pesan" berupa paket data melalui jaringan komunikasi (seperti LAN atau kabel Infiniband).

### Karakteristik

- Sangat *scalable* (bisa menggabungkan ribuan komputer biasa menjadi satu superkomputer)
- Tidak ada tabrakan memori secara langsung
- Proses komunikasi lebih lambat karena data harus dibungkus dan dikirim melewati jaringan kabel fisik

### Analogi

Koki-koki memasak di restoran (dapur) yang berada di kota berbeda. Jika Koki A di Jakarta butuh bumbu racikan rahasia dari Koki B di Bandung, ia tidak bisa langsung mengambilnya. Koki B harus membungkus bumbu tersebut ke dalam paket dan mengirimkannya lewat kurir (pesan) ke Jakarta.

---

# MPI (Message Passing Interface)

MPI (*Message Passing Interface*) adalah sebuah standar atau aturan pemrograman yang dirancang khusus untuk bekerja pada **Distributed Memory Model**.

MPI menjadi bahasa pengantar (kurir) yang memfasilitasi bagaimana komputer-komputer terpisah ini bisa saling berkoordinasi, mengoper data, dan memecahkan satu tugas besar secara serempak seolah-olah mereka adalah satu entitas.

---

# Daftar Isi

1. Dasar-Dasar MPI
2. Komunikasi Point-to-Point (Blocking)
3. Komunikasi Point-to-Point (Non-Blocking)
4. Komunikasi Kolektif (Broadcast, Scatter, & Gather)
5. Derived Datatypes (Tipe Data Bentukan)
6. Operasi Komunikator (Splitting Communicators)
7. Studi Kasus: Penjumlahan Array Paralel
8. Tugas: Implementasi Paralel Algoritme Sorting

---

# 1. Dasar-Dasar MPI

## Teori Singkat

Program MPI bekerja menggunakan model **SPMD (Single Program Multiple Data)**.

Artinya, Anda hanya menulis satu buah kode program, tetapi kode tersebut akan dijalankan (diduplikasi) secara serentak oleh banyak prosesor. Karena kodenya sama, program perlu cara untuk membedakan "siapa saya" dan "berapa banyak rekan saya".

Oleh karena itu, langkah pertama selalu menginisialisasi lingkungan MPI untuk mendata total proses yang berpartisipasi (*size*) beserta identitas unik (*rank*) masing-masing prosesor.

---

## Penjelasan Konsep

- `MPI_Init`: Menginisialisasi lingkungan eksekusi MPI. Tidak ada instruksi atau komunikasi paralel yang boleh dilakukan sebelum fungsi ini dipanggil oleh sistem.
- `MPI_Comm_size`: Mendapatkan total jumlah proses (*instances* dari program) yang dialokasikan oleh sistem untuk berjalan bersamaan.
- `MPI_Comm_rank`: Mendapatkan identifier (ID) unik untuk setiap proses, biasanya berupa angka dari `0` hingga `size - 1`. ID ini krusial sebagai penanda alur logika agar program tahu bagian data mana yang harus dikerjakan oleh proses tersebut (misal: proses dengan Rank 0 bertindak sebagai pengatur utama).

---

## Kode Program

```c
#include <stdio.h>
#include <mpi.h>

int main(int argc, char** argv) {
    int rank, size;

    // 1. Menginisialisasi lingkungan eksekusi MPI
    MPI_Init(&argc, &argv);

    // 2. Mendapatkan total jumlah proses (instances) dalam MPI_COMM_WORLD
    MPI_Comm_size(MPI_COMM_WORLD, &size);

    // 3. Mendapatkan identifier (ID) unik / rank untuk proses ini
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);

    printf("Siap bertugas! Saya proses dengan Rank %d dari total %d proses.\n", rank, size);

    // 4. Terminasi lingkungan eksekusi MPI dan membersihkan memori
    MPI_Finalize();
    return 0;
}
```

---

# 2. Anatomi Pesan MPI dan Tipe Data

## Teori Singkat

Sebelum masuk ke mekanisme pengiriman data spesifik, penting untuk memahami bagaimana data dibungkus di dalam MPI.

Setiap pesan yang melintasi jaringan dalam MPI terdiri dari dua bagian utama:

---

## 1. Payload (Data Aktual)

Ini adalah isi memori yang sesungguhnya ingin Anda kirimkan, seperti nilai variabel, teks, atau array.

---

## 2. Metadata (Amplop Pesan)

Ini adalah informasi ekstra yang menjamin pesan sampai ke tempat dan program yang tepat.

Metadata mencakup parameter seperti:

- ID pengirim
- ID tujuan
- Label pesan (*tag*)
- Jaringan komunikasi (*communicator*)

---

## Struktur Fungsi Pengiriman dan Penerimaan

Konsep payload dan metadata ini tertuang secara eksplisit dalam definisi (*signature*) fungsi komunikasi MPI dasar berikut:

---

### Definisi Fungsi untuk Mengirim Pesan

```c
int MPI_Send(
    const void *buf,       // (Payload) Pointer ke memori data yang akan dikirim
    int count,             // (Payload) Jumlah elemen data yang dikirim
    MPI_Datatype datatype, // (Payload) Tipe data MPI dari elemen tersebut
    int dest,              // (Metadata) ID/Rank proses tujuan
    int tag,               // (Metadata) Label/ID unik untuk pesan ini
    MPI_Comm comm          // (Metadata) Ruang lingkup komunikasi (misal: MPI_COMM_WORLD)
);
```

### Definisi Fungsi untuk Menerima Pesan

```c
int MPI_Recv(
    void *buf,             // (Payload) Pointer ke memori penampung data
    int count,             // (Payload) Kapasitas maksimal elemen yang bisa ditampung
    MPI_Datatype datatype, // (Payload) Tipe data MPI yang diharapkan
    int source,            // (Metadata) ID/Rank proses pengirim (bisa spesifik atau ANY_SOURCE)
    int tag,               // (Metadata) Label pesan yang ditunggu (bisa spesifik atau ANY_TAG)
    MPI_Comm comm,         // (Metadata) Ruang lingkup komunikasi
    MPI_Status *status     // Output: Objek penyimpan status detail penerimaan (ukuran aktual, dll)
);
```

---

## Tipe Data Dasar MPI

Agar sistem dengan arsitektur berbeda dapat saling memahami ukuran memori saat bertukar *payload*, MPI memetakan tipe data standar C ke tipe data miliknya sendiri.

---

### Tabel Tipe Data Dasar MPI

| Tipe Data C     | Tipe Data MPI       | Deskripsi |
|---|---|---|
| `char` | `MPI_CHAR` | Karakter 8-bit |
| `int` | `MPI_INT` | Bilangan bulat (*integer*) standar |
| `short` | `MPI_SHORT` | Bilangan bulat pendek |
| `long` | `MPI_LONG` | Bilangan bulat panjang |
| `float` | `MPI_FLOAT` | Angka pecahan (*floating-point*) presisi tunggal (32-bit) |
| `double` | `MPI_DOUBLE` | Angka pecahan (*floating-point*) presisi ganda (64-bit) |
| `unsigned char` | `MPI_UNSIGNED_CHAR` | Karakter tanpa tanda baca (positif) |
| `unsigned int` | `MPI_UNSIGNED` | Integer tanpa tanda baca (positif) |

---

# 3. Komunikasi Point-to-Point (Blocking)

## Teori Singkat

Ini adalah komunikasi langsung antara dua pihak spesifik (Pengirim A ke Penerima B).

Sifatnya **Blocking** (*menahan*). Artinya, program akan berhenti sementara di baris kode tersebut sampai urusan serah-terima data benar-benar selesai dan aman.

---

## Analogi (Serah Terima Dokumen Fisik)

### Pengirim (`MPI_Send`)

Anda mengantarkan dokumen langsung ke meja rekan Anda. Anda akan mematung berdiri dan tidak akan kembali bekerja sampai rekan Anda mengambil dokumen itu dari tangan Anda.

### Penerima (`MPI_Recv`)

Rekan Anda diam di mejanya, tidak mengerjakan apa-apa, murni hanya menunggu Anda datang membawa dokumen.

---

## Penerapan Nyata

Digunakan pada sistem transaksi kritis (seperti basis data perbankan terdistribusi) di mana sinkronisasi dan kepastian penerimaan data jauh lebih penting daripada kecepatan.

Jika uang belum diterima server B, server A tidak boleh melanjutkan proses.

---

## Kode Program

```c
#include <stdio.h>
#include <mpi.h>

int main(int argc, char** argv) {
    int rank, dokumen;
    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);

    if (rank == 0) {
        dokumen = 100;

        // Proses diblokir hingga dokumen berpindah tangan ke Proses 1
        // Parameter: (data, jumlah data, tipe, tujuan, label pesan, jaringan)
        MPI_Send(&dokumen, 1, MPI_INT, 1, 0, MPI_COMM_WORLD);

        printf("Proses 0: Dokumen bernilai %d berhasil dikirim.\n", dokumen);
    } 
    else if (rank == 1) {
        MPI_Status status;

        // Proses masuk wait state menunggu dokumen dari Proses 0
        // Parameter: (tempat_simpan, jumlah, tipe, sumber, label, jaringan, status)
        MPI_Recv(&dokumen, 1, MPI_INT, 0, 0, MPI_COMM_WORLD, &status);

        printf("Proses 1: Dokumen bernilai %d diterima dan siap diproses.\n", dokumen);
    }

    MPI_Finalize();
    return 0;
}
```

---

# 4. Komunikasi Point-to-Point (Non-Blocking)

## Teori Singkat

Dalam komputasi tingkat tinggi, menunggu (*blocking*) adalah pemborosan waktu prosesor.

Komunikasi **Non-Blocking (Asinkron)** memisahkan inisiasi pengiriman data dari proses validasinya. Proses akan "menitipkan" data untuk dikirim, lalu langsung lanjut mengerjakan baris kode berikutnya (*overlapping computation and communication*).

---

## Analogi (Pesan Antar Makanan)

Memesan makanan lewat aplikasi ojol.

Anda menekan tombol "Pesan" (`MPI_Isend` / `MPI_Irecv`), lalu Anda meninggalkan HP untuk menyapu rumah (komputasi di latar belakang). Anda tidak diam di depan pintu.

Anda baru menghentikan pekerjaan dan ke depan pintu saat kurir benar-benar tiba (`MPI_Wait`).

---

## Struktur Fungsi Pengiriman dan Penerimaan (Non-Blocking)

Dalam mode **non-blocking**, fungsi hanya menginisiasi operasi dan langsung kembali (*return*) ke program utama tanpa menunggu proses selesai.

Hal ini memungkinkan terjadinya *overlap* antara komunikasi dan komputasi.

---

### Definisi Fungsi untuk Mengirim Pesan (Non-Blocking)

```c
int MPI_Isend(
    const void *buf,       // (Payload) Pointer ke memori data yang akan dikirim
    int count,             // (Payload) Jumlah elemen data yang dikirim
    MPI_Datatype datatype, // (Payload) Tipe data MPI dari elemen tersebut
    int dest,              // (Metadata) ID/Rank proses tujuan
    int tag,               // (Metadata) Label/ID unik untuk pesan ini
    MPI_Comm comm,         // (Metadata) Ruang lingkup komunikasi
    MPI_Request *request   // (Handle) Output: Objek pelacak status operasi (tiket)
);
```

---

### Definisi Fungsi untuk Menerima Pesan (Non-Blocking)

```c
int MPI_Irecv(
    void *buf,             // (Payload) Pointer ke memori penampung data
    int count,             // (Payload) Kapasitas maksimal elemen yang bisa ditampung
    MPI_Datatype datatype, // (Payload) Tipe data MPI yang diharapkan
    int source,            // (Metadata) ID/Rank proses pengirim
    int tag,               // (Metadata) Label pesan yang ditunggu
    MPI_Comm comm,         // (Metadata) Ruang lingkup komunikasi
    MPI_Request *request   // (Handle) Output: Objek pelacak status operasi (tiket)
);
```

---

### Definisi Fungsi untuk Menunggu Penyelesaian

Karena fungsi di atas langsung kembali, Anda harus menggunakan fungsi berikut untuk memastikan data sudah benar-benar terkirim atau diterima sebelum *buffer* digunakan kembali.

```c
int MPI_Wait(
    MPI_Request *request,  // (Handle) Objek request yang diperoleh dari Isend/Irecv
    MPI_Status *status     // (Output) Detail penerimaan (serupa dengan MPI_Recv)
);
```

---

## Penerapan Nyata

Sangat umum pada pemrosesan video streaming *real-time* atau *Computer Vision*.

CPU menghitung kecerahan gambar blok A, sementara secara bersamaan perangkat jaringan (NIC) mengunduh gambar blok B di latar belakang tanpa mengganggu CPU.

---

## Kode Program

```c
#include <stdio.h>
#include <mpi.h>
#include <unistd.h> // Untuk simulasi komputasi dengan fungsi sleep()

int main(int argc, char** argv) {
    int rank, paket;
    MPI_Request request; // Resi pengiriman
    MPI_Status status;

    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);

    if (rank == 0) {
        paket = 500;

        // Titip paket ke kurir jaringan, langsung lanjut ke baris bawah
        MPI_Isend(&paket, 1, MPI_INT, 1, 0, MPI_COMM_WORLD, &request);
        
        printf("Proses 0: Melakukan komputasi lain (background) mengatur matriks...\n");
        sleep(1); // Simulasi kerja berat selama 1 detik
        
        // Cek resi. Memaksa program menunggu sampai transfer benar-benar selesai
        MPI_Wait(&request, &status); 
    } 
    else if (rank == 1) {
        // Beritahu sistem untuk siap menerima paket, tapi lanjut kerja dulu
        MPI_Irecv(&paket, 1, MPI_INT, 0, 0, MPI_COMM_WORLD, &request);
        
        printf("Proses 1: Mengeksekusi instruksi independen sambil menunggu data...\n");
        sleep(2); // Simulasi kerja berat selama 2 detik
        
        // Cek resi penerimaan. Berhenti bekerja sampai paket benar-benar tiba
        MPI_Wait(&request, &status);

        printf("Proses 1: Buffer sinkron. Paket bernilai %d siap digunakan.\n", paket);
    }

    MPI_Finalize();
    return 0;
}
```

---

# 5. Komunikasi Kolektif (Broadcast, Scatter, & Gather)

## Teori Singkat

Instruksi komunikasi yang melibatkan seluruh prosesor sekaligus dalam satu grup.

Operasi ini adalah tulang punggung dari algoritma **Divide and Conquer (Pecah dan Taklukkan)**.

---

## Jenis Operasi Komunikasi Kolektif

### Broadcast (`MPI_Bcast`)

Menyalin (*meng-copy-paste*) satu data dari *Root* ke semua anggota.

### Scatter (`MPI_Scatter`)

Mendistribusikan atau memotong-motong array besar menjadi potongan kecil dan membagikannya secara merata.

### Gather (`MPI_Gather`)

Kebalikan dari *Scatter*. Mengumpulkan hasil potongan kecil dari semua orang menjadi satu array utuh kembali di *Root*.

---

## Analogi (Ruang Kelas)

### Bcast

Guru memakai megafon mengumumkan besok libur (satu info disalin ke pikiran semua murid).

### Scatter

Guru membagikan setumpuk 40 kertas ujian secara membagi rata ke 4 ketua barisan (masing-masing 10 kertas).

### Gather

Ujian selesai, 4 ketua barisan mengumpulkan kertas dan menumpuknya kembali menjadi 40 kertas di meja guru.

---

## Penerapan Nyata

Arsitektur **Artificial Intelligence (Distributed Deep Learning)** menggunakan *Broadcast* untuk menyalin parameter atau bobot model awal ke ribuan GPU.

Algoritma **MapReduce (Big Data)** menggunakan *Scatter* untuk membagi miliaran baris teks ke berbagai server, dan *Gather* untuk menyatukan hasil frekuensi katanya.

---

## Kode Program (Contoh Scatter & Gather)

```c
#include <stdio.h>
#include <mpi.h>

#define TOTAL_DATA 8 

int main(int argc, char** argv) {
    int rank;
    int data_global[TOTAL_DATA]; // Memori utuh, relevan di proses Root (0)
    int data_lokal[2];           // Buffer potongan untuk setiap pekerja

    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);

    // 1. Root menginisialisasi kumpulan data mentah
    if (rank == 0) {
        for (int i = 0; i < TOTAL_DATA; i++) {
            data_global[i] = (i + 1) * 10; // [10, 20, 30, 40, 50, 60, 70, 80]
        }
    }

    // 2. SCATTER: Memotong data_global jadi bagian per 2 elemen ke data_lokal
    MPI_Scatter(data_global, 2, MPI_INT, data_lokal, 2, MPI_INT, 0, MPI_COMM_WORLD);

    printf("Proses %d memegang sub-array: [%d, %d]\n", rank, data_lokal[0], data_lokal[1]);

    // [Di titik ini: Komputasi Paralel terjadi. Contoh: tiap proses mengalikan isi lokal dengan 2]

    // 3. GATHER: Menyatukan data_lokal dari semua orang kembali ke data_global di Root (0)
    MPI_Gather(data_lokal, 2, MPI_INT, data_global, 2, MPI_INT, 0, MPI_COMM_WORLD);

    if (rank == 0) {
        printf("Proses 0 (Root): Seluruh data berhasil digabungkan kembali.\n");
    }

    MPI_Finalize();
    return 0;
}
```

---

# 6. Derived Datatypes (Tipe Data Bentukan)

## Teori Singkat

Standar MPI hanya mengenali tipe data dasar seperti Integer, Float, dan Char.

Jika kita memiliki struktur data yang kompleks (misal: `struct` C) atau memori yang lompat-lompat (seperti mengambil 1 kolom vertikal dari matriks), mengirimkannya secara sepotong-sepotong akan sangat lambat karena *overhead* jaringan (biaya per satu kali pengiriman sangat mahal).

**Derived Datatypes** mengatasi ini dengan membuat "peta" baru. Kita membungkus data kompleks tersebut menjadi satu entitas baru, sehingga bisa dikirim dalam satu instruksi pengiriman.

---

## Analogi (Mengepak Koper)

Mengirim 5 kemeja dan 3 celana panjang melalui kurir secara terpisah membutuhkan 8 kali biaya ongkos kirim.

Solusi cerdasnya:

- Masukkan semuanya ke dalam satu koper besar
- Daftarkan wujud koper itu (*commit*)
- Kirim dalam satu resi/paket (`MPI_Send`)
- Buang kardus pembungkusnya setelah sampai (*free*)

---

## Penerapan Nyata

Pemodelan Spasial 3D, *Point Cloud*, dan Simulasi Angin (*CFD*).

Sistem ini bertukar titik koordinat kompleks (X, Y, Z, Suhu, Tekanan) antar komputer.

Dibandingkan mengirim 5 variabel terpisah berjuta-juta kali, mereka membungkusnya dalam tipe kustom dan dikirim satu paket.

---

## Kode Program

```c
#include <stdio.h>
#include <mpi.h>

int main(int argc, char** argv) {
    int rank;
    int isi_koper[5];
    MPI_Datatype tipe_koper; 

    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);

    // 1. Membuat koper: Definisikan blok memori berisi 5 integer berurutan
    MPI_Type_contiguous(5, MPI_INT, &tipe_koper);
    
    // 2. Mendaftarkan tipe data agar jaringan tahu bentuknya (Commit)
    MPI_Type_commit(&tipe_koper);

    if (rank == 0) {
        for (int i = 0; i < 5; i++) 
            isi_koper[i] = i * 5; 
        
        // Mengirim 1 entitas bentukan (bukan 5 buah MPI_INT terpisah)
        MPI_Send(isi_koper, 1, tipe_koper, 1, 0, MPI_COMM_WORLD);

        printf("Proses 0: Struktur memori kustom berhasil ditransfer.\n");
    } 
    else if (rank == 1) {
        MPI_Status status;

        MPI_Recv(isi_koper, 1, tipe_koper, 0, 0, MPI_COMM_WORLD, &status);

        printf("Proses 1: Struktur diterima. Elemen ke-3: %d\n", isi_koper[2]);
    }

    // 3. Menghancurkan peta memori (Garbage Collection / Hemat RAM)
    MPI_Type_free(&tipe_koper);
    
    MPI_Finalize();
    return 0;
}
```

---

# 7. Operasi Komunikator (Splitting Communicators)

## Teori Singkat

Secara default, semua prosesor yang menyala berada di dalam satu grup raksasa bernama `MPI_COMM_WORLD`.

Namun, masalah kompleks seringkali membutuhkan pembagian tim.

**Splitting** memecah satu ruang topologi besar menjadi sub-sub grup independen.

Pengelompokan dipilah menggunakan parameter `color`.

Prosesor yang memiliki "warna" sama akan masuk ke grup yang sama. Urutan absensi (*Rank*) di grup baru diatur ulang dari nol.

---

## Analogi (Kelompok Belajar Studi Wisata)

Satu kelas berisi 40 siswa (Grup Utama) dipecah menjadi dua kelompok studi wisata:

- Bus A (Warna 0)
- Bus B (Warna 1)

Setelah berpisah bus (`MPI_Comm_split`), siswa di dalam Bus A akan berhitung absensi dari nomor urut 1 hingga 20 secara internal, tanpa mencampuri absen Bus B.

---

## Penerapan Nyata

Simulasi Cuaca Kompleks (*Multi-Physics*).

Sebuah superkomputer memiliki 10.000 prosesor.

Daripada semuanya mengerjakan hal yang sama, sistem membelahnya:

- 5.000 prosesor (Komunikator A) difokuskan khusus memprediksi arah angin badai
- 5.000 lainnya (Komunikator B) menghitung tinggi gelombang laut secara bersamaan

---

## Kode Program

```c
#include <stdio.h>
#include <mpi.h>

int main(int argc, char** argv) {
    int rank, rank_lokal;
    MPI_Comm sub_komunikator; // Variabel penyimpan Grup Baru

    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);

    // Aturan partisi: Rank genap -> Subnet 0, ganjil -> Subnet 1
    int color = rank % 2; 

    // Eksekusi pemecahan topologi grup
    MPI_Comm_split(MPI_COMM_WORLD, color, rank, &sub_komunikator);

    // Mengambil identitas absen lokal HANYA di dalam ruang grup yang baru
    MPI_Comm_rank(sub_komunikator, &rank_lokal);

    printf("Topologi Global [ID: %d] masuk ke Subnet %d -> ID Lokal Baru: %d\n", 
           rank, color, rank_lokal);

    // Bebaskan grup dari memori setelah selesai digunakan
    MPI_Comm_free(&sub_komunikator);

    MPI_Finalize();
    return 0;
}
```

---

# 8. Studi Kasus: Penjumlahan Array Paralel

## Konsep

Alih-alih satu prosesor kelelahan menjumlahkan array berisi ratusan elemen angka, kita membagi tugas tersebut ke prosesor (`N`) yang tersedia.

---

## Pecah Beban

Jika ada 100 elemen dan 4 prosesor, masing-masing prosesor hanya perlu menghitung 25 angka saja secara paralel.

---

## Reduksi (`MPI_Reduce`)

Ini adalah fungsi khusus.

Alih-alih melakukan *Gather* (mengumpulkan sisa angka) lalu dijumlahkan manual oleh *Root*, fungsi `MPI_Reduce` secara otomatis menarik data dari semua pekerja sekaligus menggabungkannya dengan operasi matematika spesifik, seperti:

- Penjumlahan → `MPI_SUM`
- Nilai maksimum → `MPI_MAX`
- dan operasi kolektif lainnya

Semua proses tersebut terjadi dalam perjalanan pulang ke *Root*.

---

## Kode Program Lengkap

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <mpi.h>

#define TOTAL_DATA 100

int main(int argc, char** argv) {
    int rank, size;
    int data[TOTAL_DATA];
    int sub_total = 0;      // Pegangan perhitungan lokal tiap pekerja
    int total_global = 0;   // Pegangan hasil akhir (Hanya berguna di Root)

    MPI_Init(&argc, &argv);
    MPI_Comm_rank(MPI_COMM_WORLD, &rank);
    MPI_Comm_size(MPI_COMM_WORLD, &size);

    // 1. Root melakukan inisialisasi kumpulan data
    if (rank == 0) {
        srand(time(NULL));

        // Isi dengan angka random 0-9
        for(int i = 0; i < TOTAL_DATA; i++) {
            data[i] = rand() % 10; 
        }
    }

    // 2. Broadcast: Sebar salinan seluruh data ke memori tiap prosesor
    MPI_Bcast(data, TOTAL_DATA, MPI_INT, 0, MPI_COMM_WORLD);

    // 3. Hitung jatah/partisi indeks yang harus dikerjakan prosesor ini
    int beban = TOTAL_DATA / size;
    int sisa = TOTAL_DATA % size;
    
    int index_mulai = rank * beban;
    int index_akhir = index_mulai + beban;
    
    // Prosesor terakhir kebagian tugas menyapu sisa data ganjil
    // (jika pembagian tidak bulat)
    if (rank == size - 1) {
        index_akhir += sisa;
    }

    // 4. Komputasi Paralel:
    // Masing-masing proses bekerja HANYA di area indeksnya
    for(int i = index_mulai; i < index_akhir; i++) {
        sub_total += data[i];
    }

    printf("[Rank %d] Menjumlahkan array indeks %d s/d %d. Sub-total saya = %d\n", 
           rank, index_mulai, index_akhir - 1, sub_total);

    // 5. REDUCE: Operasi matematis kolektif
    // Menarik variabel sub_total dari semua proses,
    // dijumlahkan (MPI_SUM),
    // hasilnya disimpan di variabel total_global pada Proses 0
    MPI_Reduce(
        &sub_total,
        &total_global,
        1,
        MPI_INT,
        MPI_SUM,
        0,
        MPI_COMM_WORLD
    );

    // 6. Root mencetak hasil paripurna
    if (rank == 0) {
        printf("----------------------------------\n");
        printf("Hasil Akhir (Penjumlahan Paralel): %d\n", total_global);
    }

    MPI_Finalize();
    return 0;
}
```

---

# 9. Tugas: Implementasi Paralel Algoritme Sorting

## Deskripsi Tugas

Sebagai evaluasi pemahaman Anda terhadap konsep komputasi paralel menggunakan MPI yang telah dipelajari, Anda ditugaskan untuk mengimplementasikan sebuah algoritme pengurutan data (*sorting*) secara paralel.

Algoritme yang disarankan untuk digunakan antara lain:

- Parallel Merge Sort
- Odd-Even Transposition Sort
- Algoritme sorting lain yang sesuai untuk komputasi terdistribusi

---

## Spesifikasi Masukan (Input)

Program harus dapat membaca sekumpulan data angka acak bernilai pecahan (*floating-point*, gunakan tipe data `float` atau `double`) yang bersumber dari sebuah file eksternal (misalnya `.txt` atau `.csv`).

Uji ketahanan dan kinerja program Anda dengan tiga skenario ukuran *dataset* masukan berikut:

- 10.000 angka pecahan acak
- 100.000 angka pecahan acak
- 1.000.000 angka pecahan acak

---

## Ketentuan Program

### 1. Inisialisasi (Proses Root)

Proses *Root* (misalnya Rank 0) bertanggung jawab tunggal untuk:

- Membaca data dari file teks
- Mencatat waktu mulai eksekusi (menggunakan fungsi `MPI_Wtime()`)
- Membagikan data tersebut (*Scatter*) secara merata ke proses-proses lainnya di dalam jaringan

---

### 2. Komputasi Paralel (Semua Proses)

Setiap proses akan menerima potongan array dan melakukan instruksi pengurutan pada bagiannya masing-masing secara independen.

---

### 3. Penggabungan (Proses Root)

Proses *Root* mengumpulkan kembali (*Gather*) potongan array yang sudah diurutkan dari tiap proses, kemudian melakukan tahapan penggabungan (*merge*) terakhir untuk memastikan keseluruhan elemen array terurut dengan benar dari terkecil ke terbesar.

---

### 4. Validasi

Proses *Root* harus:

- Mencetak 10 angka pertama dari array yang telah selesai diurutkan
- Mencetak 10 angka terakhir dari array yang telah selesai diurutkan
- Menampilkan hasil ke layar konsol sebagai bukti validasi bahwa algoritme berjalan dengan benar
- Mencatat waktu akhir komputasi

---

## Laporan yang Diharapkan

Buatlah laporan singkat yang memuat poin-poin berikut:

---

### 1. Metodologi

Jelaskan strategi paralelisasi atau desain algoritme *sorting* yang Anda pilih.

---

### 2. Perbandingan Kinerja

Sajikan tabel perbandingan waktu eksekusi (dalam satuan detik) antara:

- Program berjalan sekuensial (1 prosesor)
- Program paralel menggunakan:
  - 2 prosesor
  - 4 prosesor
  - 8 prosesor

untuk ketiga variasi ukuran masukan di atas.

---

### 3. Analisis

Hitung nilai **Speedup** yang didapatkan, dan berikan kesimpulan mengenai efektivitas algoritme Anda saat dihadapkan pada data berjumlah besar dibandingkan dengan data berjumlah kecil.
