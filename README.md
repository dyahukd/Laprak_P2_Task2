# Task 2 : Organize and Analyze Anthony's Favorite Films

## **Deskripsi Soal**
Anthony sedang asyik menonton film favoritnya dari Netflix, namun seiring berjalannya waktu, koleksi filmnya semakin menumpuk. Ia pun memutuskan untuk membuat sistem agar film-film favoritnya bisa lebih terorganisir dan mudah diakses. Anthony ingin melakukan beberapa hal dengan lebih efisien dan serba otomatis.

> Film-film yang dimaksud adalah film-film yang ada di dalam file ZIP yang bisa diunduh dari **[Google Drive](https://drive.google.com/file/d/12GWsZbSH858h2HExP3x4DfWZB1jLdV-J/view?usp=drive_link)**.

Berikut adalah serangkaian tugas yang Anthony ingin capai untuk membuat pengalaman menonton filmnya jadi lebih menyenangkan:

### **a. One Click and Done!**

Pernahkah kamu merasa malas untuk mengelola file ZIP yang penuh dengan data film? Anthony merasa hal yang sama, jadi dia ingin semuanya serba instan dengan hanya satu perintah. Dengan satu perintah saja, Anthony bisa:

- Mendownload file ZIP yang berisi data film-film Netflix favoritnya.
- Mengekstrak file ZIP tersebut ke dalam folder yang sudah terorganisir.
- Menghapus file ZIP yang sudah tidak diperlukan lagi, supaya tidak memenuhi penyimpanan.

Buatlah skrip yang akan mengotomatiskan proses ini sehingga Anthony hanya perlu menjalankan satu perintah untuk mengunduh, mengekstrak, dan menghapus file ZIP.

### **b. Sorting Like a Pro**

Koleksi film Anthony semakin banyak dan dia mulai bingung mencari cara yang cepat untuk mengelompokkannya. Nah, Anthony ingin mengelompokkan film-filmnya dengan dua cara yang sangat mudah:

1. Berdasarkan huruf pertama dari judul film.
2. Berdasarkan tahun rilis (release year).

Namun, karena Anthony sudah mempelajari **multiprocessing**, dia ingin mengelompokkan kedua kategori ini secara paralel untuk menghemat waktu.

**Struktur Output:**

- **Berdasarkan Huruf Pertama Judul Film:**

  - Folder: `judul/`
  - Setiap file dinamai dengan huruf abjad atau angka, seperti `A.txt`, `B.txt`, atau `1.txt`.
  - Jika judul film tidak dimulai dengan huruf atau angka, film tersebut disimpan di file `#.txt`.

- **Berdasarkan Tahun Rilis:**
  - Folder: `tahun/`
  - Setiap file dinamai sesuai tahun rilis film, seperti `1999.txt`, `2021.txt`, dst.

Format penulisan dalam setiap file :

```
Judul Film - Tahun Rilis - Sutradara
```

Setiap proses yang berjalan akan mencatat aktivitasnya ke dalam satu file bernama **`log.txt`** dengan format:

```
[jam:menit:detik] Proses mengelompokkan berdasarkan [Abjad/Tahun]: sedang mengelompokkan untuk film [judul_film]
```

**Contoh Log:**

```
[14:23:45] Proses mengelompokkan berdasarkan Abjad: sedang mengelompokkan untuk film Avengers: Infinity War
[14:23:46] Proses mengelompokkan berdasarkan Tahun: sedang mengelompokkan untuk film Kung Fu Panda
```

### **c. The Ultimate Movie Report**

Sebagai penggemar film yang juga suka menganalisis, Anthony ingin mengetahui statistik lebih mendalam tentang film-film yang dia koleksi. Misalnya, dia ingin tahu berapa banyak film yang dirilis **sebelum tahun 2000** dan **setelah tahun 2000**.

Agar laporan tersebut mudah dibaca, Anthony ingin hasilnya disimpan dalam file **`report_ddmmyyyy.txt`**.

**Format Output dalam Laporan:**

```
i. Negara: <nama_negara>
Film sebelum 2000: <jumlah>
Film setelah 2000: <jumlah>

...
i+n. Negara: <nama_negara>
Film sebelum 2000: <jumlah>
Film setelah 2000: <jumlah>
```

Agar penggunaannya semakin mudah, Anthony ingin bisa menjalankan semua proses di atas melalui sebuah antarmuka terminal interaktif dengan pilihan menu seperti berikut:
1. Download File
2. Mengelompokkan Film
3. Membuat Report

Catatan:
- Dilarang menggunakan `system`
- Harap menggunakan thread dalam pengerjaan soal C

## **Kode Program**
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <pthread.h>
#include <ctype.h>
#include <sys/wait.h>
#include <sys/stat.h>
#include <time.h>

typedef struct {
    char judul[100];
    int tahun;
    char sutradara[100];
    char negara[100];
} Film;

typedef struct {
    char negara[50];
    int sebelum2000;
    int setelah2000;
} StatistikFilm;

Film daftarfilm[10000];
int jumlahfilm = 0;
StatistikFilm statistik[100];
int jumlahstatistik = 0;

pthread_mutex_t mutex_log = PTHREAD_MUTEX_INITIALIZER;
pthread_mutex_t mutex_statistik = PTHREAD_MUTEX_INITIALIZER;

void logproses(const char *keterangan, const char *judulfilm) {
    pthread_mutex_lock(&mutex_log);
    FILE *logfile = fopen("log.txt", "a");
    if (logfile) {
        time_t waktu = time(NULL);
        struct tm tm = *localtime(&waktu);
        fprintf(logfile, "[%02d:%02d:%02d] Proses mengelompokkan berdasarkan %s: sedang mengelompokkan untuk film %s\n",
                tm.tm_hour, tm.tm_min, tm.tm_sec, keterangan, judulfilm);
        fclose(logfile);
    }
    pthread_mutex_unlock(&mutex_log);
}

void *jalaninperintah(void *arg) {
    char **perintah = (char **)arg;
    pid_t id = fork();
    if (id == 0) {
        execvp(perintah[0], perintah);
        perror("Gagal menjalankan perintah.");
        exit(EXIT_FAILURE);
    } else if (id > 0) {
        int status;
        wait(&status);
    } else {
        perror("Fork gagal.");
    }
    return NULL;
}

void download_zip() {
    pthread_t t1, t2, t3;

    char *download[] = {
        "wget",
        "-O",
        "netflixData.zip",
        "https://drive.google.com/uc?export=download&id=12GWsZbSH858h2HExP3x4DfWZB1jLdV-J",
        NULL
    };
    char *ekstrak[] = {
        "unzip", "-o", "netflixData.zip", "-d", ".", NULL
    };
    char *hapus[] = {
        "rm", "netflixData.zip", NULL
    };

    printf("Mengunduh ZIP...\n");
    pthread_create(&t1, NULL, jalaninperintah, (void *)download);
    pthread_join(t1, NULL);

    printf("Mengekstrak ZIP...\n");
    pthread_create(&t2, NULL, jalaninperintah, (void *)ekstrak);
    pthread_join(t2, NULL);

    printf("Menghapus file ZIP...\n");
    pthread_create(&t3, NULL, jalaninperintah, (void *)hapus);
    pthread_join(t3, NULL);

    printf("Download dan ekstrak selesai.\n");
}

void baca_csv() {
    FILE *file = fopen("netflixData.csv", "r");
    if (!file) {
        perror("File tidak ditemukan");
        return;
    }

    char baris[2000];
    while (fgets(baris, sizeof(baris), file)) {
        char judul[100], sutradara[100], negara[100];
        int tahun;
        if (sscanf(baris, "%[^,],%[^,],%[^,],%d", judul, sutradara, negara, &tahun) == 4) {
            if (jumlahfilm < 10000) {
                strcpy(daftarfilm[jumlahfilm].judul, judul);
                strcpy(daftarfilm[jumlahfilm].sutradara, sutradara);
                strcpy(daftarfilm[jumlahfilm].negara, negara);
                daftarfilm[jumlahfilm].tahun = tahun;
                jumlahfilm++;
            }
        }
    }

    fclose(file);
}

void *kelompokjudul(void *arg) {
    mkdir("judul", 0777);

    for (int i = 0; i < jumlahfilm; i++) {
        char awal = '#';

        for (int j = 0; daftarfilm[i].judul[j] != '\0'; j++) {
            if (isalnum(daftarfilm[i].judul[j])) {
                awal = toupper(daftarfilm[i].judul[j]);
                break;
            } else if (!isspace(daftarfilm[i].judul[j])) {
                awal = '#';
                break;
            }
        }

        char nama_file[100];
        snprintf(nama_file, sizeof(nama_file), "judul/%c.txt", awal);

        FILE *f = fopen(nama_file, "a");
        if (f) {
            fprintf(f, "%s - %d - %s\n", daftarfilm[i].judul, daftarfilm[i].tahun, daftarfilm[i].sutradara);
            fclose(f);
        }

        logproses("Abjad", daftarfilm[i].judul);
    }
    return NULL;
}

void *kelompoktahun(void *arg) {
    mkdir("tahun", 0777);

    for (int i = 0; i < jumlahfilm; i++) {
        char nama_file[100];
        snprintf(nama_file, sizeof(nama_file), "tahun/%d.txt", daftarfilm[i].tahun);

        FILE *f = fopen(nama_file, "a");
        if (f) {
            fprintf(f, "%s - %d - %s\n", daftarfilm[i].judul, daftarfilm[i].tahun, daftarfilm[i].sutradara);
            fclose(f);
        }

        logproses("Tahun", daftarfilm[i].judul);
    }
    return NULL;
}

void *proses_statistik(void *arg) {
    for (int i = 0; i < jumlahfilm; i++) {
        pthread_mutex_lock(&mutex_statistik);
        int ditemukan = 0;
        for (int j = 0; j < jumlahstatistik; j++) {
            if (strcmp(statistik[j].negara, daftarfilm[i].negara) == 0) {
                if (daftarfilm[i].tahun < 2000)
                    statistik[j].sebelum2000++;
                else
                    statistik[j].setelah2000++;
                ditemukan = 1;
                break;
            }
        }
        if (!ditemukan) {
            strcpy(statistik[jumlahstatistik].negara, daftarfilm[i].negara);
            statistik[jumlahstatistik].sebelum2000 = (daftarfilm[i].tahun < 2000) ? 1 : 0;
            statistik[jumlahstatistik].setelah2000 = (daftarfilm[i].tahun >= 2000) ? 1 : 0;
            jumlahstatistik++;
        }
        pthread_mutex_unlock(&mutex_statistik);
    }
    return NULL;
}

void buat_laporan() {
    time_t waktu = time(NULL);
    struct tm tm = *localtime(&waktu);
    char nama_file[100];
    snprintf(nama_file, sizeof(nama_file), "report_%02d%02d%04d.txt",
             tm.tm_mday, tm.tm_mon + 1, tm.tm_year + 1900);

    FILE *f = fopen(nama_file, "w");
    if (!f) {
        perror("Gagal membuat file laporan");
        return;
    }

    for (int i = 0; i < jumlahstatistik; i++) {
        fprintf(f, "Negara: %s\nFilm sebelum 2000: %d\nFilm setelah 2000: %d\n\n",
                statistik[i].negara, statistik[i].sebelum2000, statistik[i].setelah2000);
    }

    fclose(f);
    printf("Laporan disimpan di %s\n", nama_file);
}

void menu() {
    int pilihan;
    do {
        printf("\n===== MENU =====\n");
        printf("1. One Click: Download, Ekstrak, dan Hapus File\n");
        printf("2. Sorting Film: Kelompokkan berdasarkan Judul dan Tahun\n");
        printf("3. Report Film: Buat Laporan Statistik\n");
        printf("0. Keluar\n");
        printf("Pilih: ");
        scanf("%d", &pilihan);
        getchar();

        switch (pilihan) {
            case 1:
                download_zip();
                break;
            case 2: {
                baca_csv();
                pthread_t t1, t2;
                pthread_create(&t1, NULL, kelompokjudul, NULL);
                pthread_create(&t2, NULL, kelompoktahun, NULL);
                pthread_join(t1, NULL);
                pthread_join(t2, NULL);
                printf("Pengelompokan selesai!\n");
                break;
            }
            case 3: {
                pthread_t t;
                pthread_create(&t, NULL, proses_statistik, NULL);
                pthread_join(t, NULL);
                buat_laporan();
                break;
            }
            case 0:
                printf("Keluar...\n");
                break;
            default:
                printf("Pilihan tidak valid!\n");
        }
    } while (pilihan != 0);
}

int main() {
    menu();
    return 0;
}
```

## **Penjelasan Kode Program**
```
#include <stdio.h>       
#include <stdlib.h>      
#include <string.h>      
#include <unistd.h>      
#include <pthread.h>     
#include <ctype.h>       
#include <sys/wait.h>    
#include <sys/stat.h>    
#include <time.h>
```     
- `<stdio.h>`: Menyediakan fungsi untuk operasi input/output, seperti `printf()`, `fopen()`, `fscanf()`, dan lain-lain.
- `<stdlib.h>`: Menyediakan fungsi untuk pengelolaan memori dinamis dan manipulasi angka, seperti `malloc()`, `free()`, dan `exit()`.
- `<string.h>`: Digunakan untuk manipulasi string, seperti `strcpy()`, `strlen()`, `strcmp()`.
- `<unistd.h>`: Menyediakan fungsi untuk pemrograman POSIX seperti `fork()`, `execvp()`, dan `getpid()`.
- `<pthread.h>`: Menyediakan fungsi untuk thread dalam C, seperti `pthread_create()`, `pthread_join()`, `pthread_mutex_lock()`, dll.
- `<ctype.h>`: Digunakan untuk memeriksa atau memanipulasi karakter, seperti `isalnum()`, `toupper()`, dll.
- `<sys/wait.h>`: Menyediakan fungsi untuk menangani proses anak (child process), seperti `wait()` dan status proses.
- `<sys/stat.h>`: Menyediakan fungsi untuk operasi terkait file dan direktori seperti `mkdir()`.
- `<time.h>`: Digunakan untuk menangani waktu, misalnya untuk mencatat waktu saat proses dijalankan dengan `time()` dan `localtime()`.

```
typedef struct {
    char judul[100];
    int tahun;
    char sutradara[100];
    char negara[100];
} Film;
```
- `Film`: Struktur data untuk menyimpan informasi tentang sebuah film, meliputi `judul` (nama film), `tahun` (tahun rilis), `sutradara` (nama sutradara), dan `negara` (negara asal).

```
typedef struct {
    char negara[50];
    int sebelum2000;
    int setelah2000;
} StatistikFilm;
```
- `StatistikFilm`: Struktur untuk menyimpan statistik film berdasarkan negara, yaitu jumlah film yang dirilis sebelum 2000 (`sebelum2000`) dan setelah 2000 (`setelah2000`).

```
Film daftarfilm[10000];
int jumlahfilm = 0;
StatistikFilm statistik[100];
int jumlahstatistik = 0;
```
- `daftarfilm`: Array untuk menyimpan daftar film yang dibaca dari file CSV.
- `jumlahfilm`: Menyimpan jumlah film yang sudah dimasukkan ke dalam array `daftarfilm`.
- `statistik`: Array untuk menyimpan statistik berdasarkan negara.
- `jumlahstatistik`: Menyimpan jumlah statistik negara yang tercatat.

```
pthread_mutex_t mutex_log = PTHREAD_MUTEX_INITIALIZER;
pthread_mutex_t mutex_statistik = PTHREAD_MUTEX_INITIALIZER;
```
- `mutex_log`: Mutex untuk mengamankan akses ke file log, menghindari kondisi bersaing saat penulisan log.
- `mutex_statistik`: Mutex untuk mengamankan akses ke data statistik film saat diupdate oleh beberapa thread secara bersamaan.

```
void logproses(const char *keterangan, const char *judulfilm) {
    pthread_mutex_lock(&mutex_log);
    FILE *logfile = fopen("log.txt", "a");
    if (logfile) {
        time_t waktu = time(NULL);
        struct tm tm = *localtime(&waktu);
        fprintf(logfile, "[%02d:%02d:%02d] Proses mengelompokkan berdasarkan %s: sedang mengelompokkan untuk film %s\n",
                tm.tm_hour, tm.tm_min, tm.tm_sec, keterangan, judulfilm);
        fclose(logfile);
    }
    pthread_mutex_unlock(&mutex_log);
}
```
- `logproses`: Fungsi untuk mencatat log saat memproses film. Menerima dua parameter: `keterangan` dan `judulfilm`.
- `pthread_mutex_lock`: Mengunci mutex supaya hanya satu thread yang bisa menulis ke file log dalam satu waktu.
- `logfile`: Membuka file bernama `log.txt` dalam mode append `"a"`, supaya data baru ditulis di akhir tanpa menghapus yang lama.
- `if (logfile)`: Mengecek apakah file berhasil dibuka. Kalau berhasil, lanjut menulis.
- `waktu`: Menyimpan waktu saat ini dalam format detik (epoch time).
- `tm`: Mengubah waktu ke dalam format waktu lokal (jam, menit, detik, dll).
- `fprintf`: Menulis log ke file dengan format waktu dan keterangan proses yang sedang berlangsung.
- `fclose`: Menutup file `log.txt` setelah selesai menulis.
- `pthread_mutex_unlock`: Membuka kunci mutex supaya thread lain bisa menulis ke log.

```
void *jalaninperintah(void *arg) {
    char **perintah = (char **)arg;
    pid_t id = fork();
    if (id == 0) {
        execvp(perintah[0], perintah);
        perror("Gagal menjalankan perintah.");
        exit(EXIT_FAILURE);
    } else if (id > 0) {
        int status;
        wait(&status);
    } else {
        perror("Fork gagal.");
    }
    return NULL;
}
```
- `jalaninperintah`: Fungsi untuk menjalankan perintah terminal pakai fork dan exec. Parameternya berupa `arg` yang nanti dikonversi ke array string (perintah).
- `perintah`: Mengubah argumen `arg` ke bentuk array string (misal: `{"wget", "link", NULL})`.
- `id`: Menyimpan hasil dari `fork()`, yang digunakan untuk membuat proses anak.
- `if (id == 0)`: Mengecek apakah ini proses anak (child process).
- `execvp`: Menjalankan perintah yang diberikan (misal: `wget`, `unzip`, `rm`, dll). Mengganti proses anak dengan proses baru.
- `perror`: Menampilkan pesan error kalau `execvp` gagal.
- `exit`: Keluar dari proses anak dengan status gagal.
- `else if (id > 0)`: Jika ini adalah proses induk (parent process).
- `status`: Variabel untuk menyimpan status keluaran dari proses anak.
- `wait`: Menunggu proses anak selesai sebelum lanjut.
- `else`: Jika `fork()` gagal (id < 0), artinya tidak bisa membuat proses baru.
- `perror`: Menampilkan pesan error kalau `fork` gagal.
- `return NULL`: Mengembalikan nilai NULL karena fungsi dipakai untuk thread `(void *)`.

```
void download_zip() {
    pthread_t t1, t2, t3;

    char *download[] = {
        "wget",
        "-O",
        "netflixData.zip",
        "https://drive.google.com/uc?export=download&id=12GWsZbSH858h2HExP3x4DfWZB1jLdV-J",
        NULL
    };
    char *ekstrak[] = {
        "unzip", "-o", "netflixData.zip", "-d", ".", NULL
    };
    char *hapus[] = {
        "rm", "netflixData.zip", NULL
    };

    printf("Mengunduh ZIP...\n");
    pthread_create(&t1, NULL, jalaninperintah, (void *)download);
    pthread_join(t1, NULL);

    printf("Mengekstrak ZIP...\n");
    pthread_create(&t2, NULL, jalaninperintah, (void *)ekstrak);
    pthread_join(t2, NULL);

    printf("Menghapus file ZIP...\n");
    pthread_create(&t3, NULL, jalaninperintah, (void *)hapus);
    pthread_join(t3, NULL);

    printf("Download dan ekstrak selesai.\n");
}
```
- Fungsi `download_zip`: Fungsi ini akan mengunduh file ZIP, mengekstrak, lalu menghapusnya.
- `pthread_t t1, t2, t3`: Variabel untuk menyimpan thread yang akan digunakan menjalankan perintah.
- `download`: Array string untuk perintah `wget` yang akan mengunduh file dan menyimpannya sebagai `netflixData.zip`.
- `ekstrak`: Array string untuk perintah `unzip`. Mengekstrak `netflixData.zip` ke direktori saat ini.
- `hapus`: Array string untuk perintah `rm`. Menghapus file ZIP setelah diekstrak.
- `printf("Mengunduh ZIP...\n")`: Menampilkan teks bahwa proses download akan dimulai.
- `pthread_create(&t1, NULL, jalaninperintah, (void *)download)`: Membuat thread `t1` untuk menjalankan perintah `download` pakai fungsi `jalaninperintah`.
- `pthread_join(t1, NULL)`: Menunggu thread `t1` selesai sebelum lanjut ke langkah berikutnya.
- `printf("Mengekstrak ZIP...\n")`: Menampilkan teks bahwa proses ekstrak akan dimulai.
- `pthread_create(&t2, NULL, jalaninperintah, (void *)ekstrak)`: Membuat thread `t2` untuk mengekstrak ZIP.
- `pthread_join(t2, NULL)`: Menunggu thread `t2` selesai.
- `printf("Menghapus file ZIP...\n")`: Menampilkan teks bahwa file ZIP akan dihapus.
- `pthread_create(&t3, NULL, jalaninperintah, (void *)hapus)`: Membuat thread `t3` untuk menghapus file ZIP.
- `pthread_join(t3, NULL)`: Menunggu thread `t3` selesai.
- `printf("Download dan ekstrak selesai.\n")`: Menampilkan teks bahwa semua proses telah selesai.

```
void baca_csv() {
    FILE *file = fopen("netflixData.csv", "r");
    if (!file) {
        perror("File tidak ditemukan");
        return;
    }

    char baris[2000];
    while (fgets(baris, sizeof(baris), file)) {
        char judul[100], sutradara[100], negara[100];
        int tahun;
        if (sscanf(baris, "%[^,],%[^,],%[^,],%d", judul, sutradara, negara, &tahun) == 4) {
            if (jumlahfilm < 10000) {
                strcpy(daftarfilm[jumlahfilm].judul, judul);
                strcpy(daftarfilm[jumlahfilm].sutradara, sutradara);
                strcpy(daftarfilm[jumlahfilm].negara, negara);
                daftarfilm[jumlahfilm].tahun = tahun;
                jumlahfilm++;
            }
        }
    }

    fclose(file);
}
```
- Fungsi `baca_csv`: Fungsi ini membaca file CSV dan menyimpan data ke array `daftarfilm`.
- `FILE *file = fopen("netflixData.csv", "r")`: Membuka file netflixData.csv untuk dibaca. Mode `r` berarti read (baca saja).
- `if (!file)`: Cek apakah file gagal dibuka (nilai file adalah NULL).
- `perror("File tidak ditemukam")`: Menampilkan pesan error ke layar jika file tidak ditemukan.
- `return`: Keluar dari fungsi `baca_csv()` tanpa melanjutkan proses.
- `char baris[2000]`: Buffer untuk menyimpan satu baris dari file CSV.
- `while (fgets(baris, sizeof(baris), file))`: Selama masih ada baris yang bisa dibaca dari file, lakukan perulangan.
- `char judul[100], sutradara[100], negara[100]; int tahun;`: Deklarasi variabel sementara untuk menyimpan data dari satu baris: judul, sutradara, negara, dan tahun.
- `if (sscanf(baris, "%[^,],%[^,],%[^,],%d", judul, sutradara, negara, &tahun) == 4)`: Membaca empat nilai dari baris, dipisah dengan koma. Pastikan semua 4 data berhasil terbaca.
- `if (jumlahfilm < 10000)`: Cek apakah jumlah film belum mencapai batas maksimal (10000 data).
- `strcpy(daftarfilm[jumlahfilm].judul, judul)`: Salin judul ke field judul pada array `daftarfilm`.
- `strcpy(daftarfilm[jumlahfilm].sutradara, sutradara)`: Salin `sutradara` ke field `sutradara` pada array `daftarfilm`.
- `strcpy(daftarfilm[jumlahfilm].negara, negara)`: Salin `negara` ke field `negara` pada array `daftarfilm`.
- `daftarfilm[jumlahfilm].tahun = tahun`: Simpan nilai `tahun` ke field `tahun` pada array `daftarfilm`.
- `jumlahfilm++`: Naikkan nilai `jumlahfilm` untuk menunjukkan bahwa satu data film berhasil disimpan.
- `fclose(file)`: Menutup file setelah selesai dibaca.

```
void *kelompokjudul(void *arg) {
    mkdir("judul", 0777);

    for (int i = 0; i < jumlahfilm; i++) {
        char awal = '#';

        for (int j = 0; daftarfilm[i].judul[j] != '\0'; j++) {
            if (isalnum(daftarfilm[i].judul[j])) {
                awal = toupper(daftarfilm[i].judul[j]);
                break;
            } else if (!isspace(daftarfilm[i].judul[j])) {
                awal = '#';
                break;
            }
        }

        char nama_file[100];
        snprintf(nama_file, sizeof(nama_file), "judul/%c.txt", awal);

        FILE *f = fopen(nama_file, "a");
        if (f) {
            fprintf(f, "%s - %d - %s\n", daftarfilm[i].judul, daftarfilm[i].tahun, daftarfilm[i].sutradara);
            fclose(f);
        }

        logproses("Abjad", daftarfilm[i].judul);
    }
    return NULL;
}
```
- `*kelompokjudul(void *arg)`: Fungsi untuk mensortir film berdasarkan judul.
- `mkdir("judul", 0777)`: Fungsi ini membuat direktori baru bernama `judul` dengan permission `0777`, yang berarti direktori dapat dibaca, ditulis, dan dieksekusi oleh siapa saja. Direktori ini akan digunakan untuk menyimpan file teks berdasarkan huruf pertama judul film.
- `for (int i = 0; i < jumlahfilm; i++)`: Loop ini akan iterasi untuk setiap film dalam array `daftarfilm`. Artinya, loop ini akan mengecek semua film yang ada di dalam array.
- `char awal = '#'`: Variabel awal digunakan untuk menyimpan karakter pertama dari judul film yang akan digunakan untuk mengelompokkan film. Jika tidak ada karakter yang valid, maka `#` akan digunakan sebagai pengganti (sebagai grup "lain-lain").
- `for (int j = 0; daftarfilm[i].judul[j] != '\0'; j++)`: Loop ini akan mengecek setiap karakter dalam judul film satu per satu sampai akhir string `('\0')`.
- `if (isalnum(daftarfilm[i].judul[j]))`: Fungsi `isalnum()` digunakan untuk memeriksa apakah karakter `daftarfilm[i].judul[j]` adalah alfanumerik (huruf atau angka). Jika ya, karakter tersebut akan menjadi huruf pertama yang digunakan untuk mengelompokkan film.
- `awal = toupper(daftarfilm[i].judul[j])`: Jika karakter pertama yang valid ditemukan, maka awal akan diubah menjadi huruf besar menggunakan `toupper()`.
- `break`: Setelah menemukan karakter pertama yang valid, proses pencarian dihentikan dengan break untuk menghindari pemeriksaan karakter lebih lanjut.
- `else if (!isspace(daftarfilm[i].judul[j]))`: Jika karakter yang ditemukan bukan huruf atau angka, tetapi juga bukan spasi, maka kelompokkan film ini ke dalam grup `#`.
- `awal = '#'`: Jika karakter tidak alfanumerik dan bukan spasi, artinya film akan dikelompokkan dalam kategori `#`, yaitu "lain-lain".
- `break`: Setelah menetapkan karakter kelompok, proses pencarian berhenti.
- `char nama_file[100]`: Variabel ini untuk menampung nama file yang akan dibuat untuk setiap kelompok.
- `snprintf(nama_file, sizeof(nama_file), "judul/%c.txt", awal)`: Fungsi `snprintf()` digunakan untuk membuat nama file berdasarkan karakter awal. File tersebut akan disimpan di dalam direktori `judul/` dengan nama file yang berisi huruf pertama dari judul film. Jika awal adalah `#`, maka file yang dibuat akan bernama `judul/#.txt`.
- `FILE *f = fopen(nama_file, "a")`: Membuka file dengan nama yang sudah dibuat sebelumnya dalam mode append `("a")`. Mode ini memastikan bahwa data baru akan ditambahkan ke akhir file tanpa menghapus isi yang ada.
- `if (f)`: Mengecek apakah `file` berhasil dibuka. Jika ya, maka lanjutkan ke langkah berikutnya.
- `fprintf(f, "%s - %d - %s\n", daftarfilm[i].judul, daftarfilm[i].tahun, daftarfilm[i].sutradara)`: Menulis informasi tentang `film` ke dalam file. Formatnya adalah judul film, tahun rilis, dan nama sutradara.
- `fclose(f)`: Menutup file setelah penulisan selesai.
- `logproses("Abjad", daftarfilm[i].judul)`: Fungsi ini dipanggil untuk mencatat log yang menunjukkan bahwa proses pengelompokan berdasarkan abjad untuk film tersebut telah selesai.
- `return NULL`: Fungsi `kelompokjudul` berakhir dan mengembalikan `NULL` karena fungsi ini digunakan oleh thread yang memiliki tipe `void*` sebagai return value.

```
void *kelompoktahun(void *arg) {
    mkdir("tahun", 0777);

    for (int i = 0; i < jumlahfilm; i++) {
        char nama_file[100];
        snprintf(nama_file, sizeof(nama_file), "tahun/%d.txt", daftarfilm[i].tahun);

        FILE *f = fopen(nama_file, "a");
        if (f) {
            fprintf(f, "%s - %d - %s\n", daftarfilm[i].judul, daftarfilm[i].tahun, daftarfilm[i].sutradara);
            fclose(f);
        }

        logproses("Tahun", daftarfilm[i].judul);
    }
    return NULL;
}
```
- `*kelompoktahun(void *arg)`: Fungsi untuk mensortir film berdasarkan tahun.
- `mkdir("tahun", 0777)`: Membuat direktori baru bernama `tahun` dengan permission `0777`, di mana setiap file yang akan dikelompokkan berdasarkan tahun rilis film akan disimpan.
- `for (int i = 0; i < jumlahfilm; i++)`: Loop ini berjalan untuk setiap `film` dalam array `daftarfilm`.
- `int tahun = daftarfilm[i].tahun`: Mendapatkan tahun rilis dari film saat ini.
- `char nama_file[100]`: Variabel untuk menampung nama file berdasarkan tahun.
- `snprintf(nama_file, sizeof(nama_file), "tahun/%d.txt", tahun)`: Membuat nama file berdasarkan tahun rilis film, dan menyimpannya dalam direktori `tahun/`.
- `FILE *f = fopen(nama_file, "a")`: Membuka file yang telah dibuat dalam mode append untuk menambah data film.
- `if (f)`: Memeriksa apakah file berhasil dibuka.
- `fprintf(f, "%s - %d - %s\n", daftarfilm[i].judul, daftarfilm[i].tahun, daftarfilm[i].sutradara)`: Menulis data film ke dalam file.
- `fclose(f)`: Menutup file setelah selesai menulis.
- `logproses("Tahun", daftarfilm[i].judul)`: Mencatat log bahwa film telah dikelompokkan berdasarkan tahun.
- `return NULL`: Mengembalikan `NULL` karena ini adalah fungsi thread.

```
void *proses_statistik(void *arg) {
    for (int i = 0; i < jumlahfilm; i++) {
        pthread_mutex_lock(&mutex_statistik);
        int ditemukan = 0;
        for (int j = 0; j < jumlahstatistik; j++) {
            if (strcmp(statistik[j].negara, daftarfilm[i].negara) == 0) {
                if (daftarfilm[i].tahun < 2000)
                    statistik[j].sebelum2000++;
                else
                    statistik[j].setelah2000++;
                ditemukan = 1;
                break;
            }
        }
        if (!ditemukan) {
            strcpy(statistik[jumlahstatistik].negara, daftarfilm[i].negara);
            statistik[jumlahstatistik].sebelum2000 = (daftarfilm[i].tahun < 2000) ? 1 : 0;
            statistik[jumlahstatistik].setelah2000 = (daftarfilm[i].tahun >= 2000) ? 1 : 0;
            jumlahstatistik++;
        }
        pthread_mutex_unlock(&mutex_statistik);
    }
    return NULL;
}
```
- `void *proses_statistik(void *arg)`: Fungsi untuk menghitung statistik film berdasarkan negara dan tahun. Dijalankan oleh thread.
- `for (int i = 0; i < jumlahfilm; i++)`: Loop untuk memproses setiap data film yang ada di array `daftarfilm`.
- `pthread_mutex_lock(&mutex_statistik)`: Mengunci `mutex_statistik` agar proses ini tidak bentrok dengan thread lain.
- int ditemukan = 0`: Variabel untuk menandai apakah negara film ini sudah ada di statistik.
- `for (int j = 0; j < jumlahstatistik; j++)`: Loop untuk mencari apakah negara film sudah ada di array statistik.
- `if (strcmp(statistik[j].negara, daftarfilm[i].negara) == 0)`: Bandingkan nama negara di statistik dengan negara film saat ini.
- `if (daftarfilm[i].tahun < 2000) { statistik[j].sebelum2000++; }`: Jika tahun film sebelum 2000, tambahkan 1 ke hitungan `sebelum2000`.
- `else { statistik[j].setelah2000++; }`: Jika tidak, tambahkan 1 ke hitungan `setelah2000`.
- `ditemukan = 1`: Tandai bahwa negara ini sudah ditemukan.
- `break`: Keluar dari loop karena sudah ketemu.
- `if (!ditemukan)`: Jika negara belum ada di statistik...
- `strcpy(statistik[jumlahstatistik].negara, daftarfilm[i].negara)`: Salin nama negara ke indeks statistik baru.
- `statistik[jumlahstatistik].sebelum2000 = (daftarfilm[i].tahun < 2000) ? 1 : 0`: Isi `sebelum2000` sesuai dengan tahun film. Jika <2000, isi 1. Kalau tidak, isi 0.
- `statistik[jumlahstatistik].setelah2000 = (daftarfilm[i].tahun >= 2000) ? 1 : 0`: Isi `setelah2000` sesuai dengan tahun film. Jika >=2000, isi 1. Kalau tidak, isi 0.
- `jumlahstatistik++`: Tambahkan jumlah data statistik.
- `pthread_mutex_unlock(&mutex_statistik)`: Buka kembali kunci mutex agar thread lain bisa mengakses data.
- `return NULL`: Kembalikan `NULL` karena tipe fungsi adalah `void*`.

```
void buat_laporan() {
    time_t waktu = time(NULL);
    struct tm tm = *localtime(&waktu);
    char nama_file[100];
    snprintf(nama_file, sizeof(nama_file), "report_%02d%02d%04d.txt",
             tm.tm_mday, tm.tm_mon + 1, tm.tm_year + 1900);

    FILE *f = fopen(nama_file, "w");
    if (!f) {
        perror("Gagal membuat file laporan");
        return;
    }

    for (int i = 0; i < jumlahstatistik; i++) {
        fprintf(f, "Negara: %s\nFilm sebelum 2000: %d\nFilm setelah 2000: %d\n\n",
                statistik[i].negara, statistik[i].sebelum2000, statistik[i].setelah2000);
    }

    fclose(f);
    printf("Laporan disimpan di %s\n", nama_file);
}
```
- `void buat_laporan()`: Fungsi untuk membuat file laporan statistik film.
- `time_t waktu = time(NULL)`: Ambil waktu saat ini (sekarang).
- `struct tm tm = *localtime(&waktu)`: Ubah waktu ke format lokal (tanggal, bulan, tahun).
- `char nama_file[100]`: Deklarasi array untuk nama file laporan.
- `snprintf(nama_file, sizeof(nama_file), "report_%02d%02d%04d.txt", tm.tm_mday, tm.tm_mon + 1, tm.tm_year + 1900)`: Buat nama file dengan format: `report_ddmmyyyy.txt`.
- `FILE *f = fopen(nama_file, "w")`: Buka file untuk ditulis. Jika belum ada, akan dibuat.
- `if (!f)`: Jika gagal membuka/membuat file...
- `perror("Gagal membuat file laporan")`: Tampilkan pesan error ke terminal.
- `return`: Keluar dari fungsi.
- `for (int i = 0; i < jumlahstatistik; i++)`: Loop untuk menulis semua data statistik negara.
- `fprintf(f, "Negara: %s\nFilm sebelum 2000: %d\nFilm setelah 2000: %d\n\n", statistik[i].negara, statistik[i].sebelum2000, statistik[i].setelah2000)`: Tulis informasi negara, jumlah film sebelum dan setelah tahun 2000 ke dalam file.
- `fclose(f)`: Tutup file setelah selesai ditulis.
- `printf("Laporan disimpan di %s\n", nama_file)`: Tampilkan lokasi (nama) file laporan ke terminal.

```
void menu() {
    int pilihan;
    do {
        printf("\n===== MENU =====\n");
        printf("1. One Click: Download, Ekstrak, dan Hapus File\n");
        printf("2. Sorting Film: Kelompokkan berdasarkan Judul dan Tahun\n");
        printf("3. Report Film: Buat Laporan Statistik\n");
        printf("0. Keluar\n");
        printf("Pilih: ");
        scanf("%d", &pilihan);
        getchar();

        switch (pilihan) {
            case 1:
                download_zip();
                break;
            case 2: {
                baca_csv();
                pthread_t t1, t2;
                pthread_create(&t1, NULL, kelompokjudul, NULL);
                pthread_create(&t2, NULL, kelompoktahun, NULL);
                pthread_join(t1, NULL);
                pthread_join(t2, NULL);
                printf("Pengelompokan selesai!\n");
                break;
            }
            case 3: {
                pthread_t t;
                pthread_create(&t, NULL, proses_statistik, NULL);
                pthread_join(t, NULL);
                buat_laporan();
                break;
            }
            case 0:
                printf("Keluar...\n");
                break;
            default:
                printf("Pilihan tidak valid!\n");
        }
    } while (pilihan != 0);
}
```
- `void menu()`: Fungsi utama untuk menampilkan menu interaktif ke pengguna.
- `int pilihan`: Variabel untuk menyimpan pilihan menu dari pengguna.
- `do`: Mulai perulangan menu, akan terus berjalan sampai pengguna memilih keluar
- ```
        printf("\n===== MENU =====\n");
        printf("1. One Click: Download, Ekstrak, dan Hapus File\n");
        printf("2. Sorting Film: Kelompokkan berdasarkan Judul dan Tahun\n");
        printf("3. Report Film: Buat Laporan Statistik\n");
        printf("0. Keluar\n");
  ```
Tampilkan daftar pilihan menu ke layar.
- `printf("Pilih: "); scanf("%d", &pilihan); getchar()`: Ambil input angka dari pengguna dan simpan di `pilihan`. Gunakan `getchar()` untuk menyerap karakter newline.
- `switch (pilihan)`: Periksa nilai `pilihan` menggunakan `switch`.
- `case 1: { download_zip(); break; }`: Jika pilih 1, jalankan fungsi `download_zip()` untuk unduh, ekstrak, dan hapus file ZIP.
- `case 2: { baca_csv();`: Jika pilih 2, baca data film dari file CSV.
- `pthread_t t1, t2`: Deklarasi dua thread.
- `pthread_create(&t1, NULL, kelompokjudul, NULL); pthread_create(&t2, NULL, kelompoktahun, NULL);`: Jalankan fungsi `kelompokjudul()` dan `kelompoktahun()` secara bersamaan menggunakan thread.
- `pthread_join(t1, NULL); pthread_join(t2, NULL);`: Tunggu sampai kedua thread selesai.
- `printf("Pengelompokan selesai!\n")`: Tampilkan pesan bahwa pengelompokan selesai.
- `case 3: { pthread_t t;`: Jika pilih 3, buat satu thread.
- `pthread_create(&t, NULL, proses_statistik, NULL); pthread_join(t, NULL);`: Jalankan fungsi `proses_statistik()` di thread, lalu tunggu sampai selesai.
- `buat_laporan()`: Setelah statistik selesai, buat file laporan.
- `case 0: { printf("Keluar...\n")`: Jika pilih 0, tampilkan pesan keluar dan hentikan loop.
- `default: { printf("Pilihan tidak valid!\n"); }`: Jika input tidak sesuai dengan pilihan yang tersedia, maka cetak bahwa pilihan tidak valid.
- `while (pilihan != 0)`: Ulangi menu selama pengguna belum memilih 0.

```
int main() {
    menu();
    return 0;1
}
```
`int main()`: Fungsi utama program.
`menu()`: Panggil fungsi `menu()`.
`return 0`: Selesai menjalankan program.

## **Hasil Program**














