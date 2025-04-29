# Laprak_P2_Task2

**Kode Program**
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

