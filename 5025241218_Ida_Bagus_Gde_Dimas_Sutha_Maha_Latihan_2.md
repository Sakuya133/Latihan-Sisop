# Trabowo & Peddy Movie Night
## Deskripsi:
Trabowo dan sahabatnya, Peddy, sedang menikmati malam minggu di rumah sambil mencari film seru untuk ditonton. Mereka menemukan sebuah file ZIP yang berisi gambar-gambar poster film yang sangat menarik. File tersebut dapat diunduh dari **[Google Drive](https://drive.google.com/file/d/1nP5kjCi9ReDk5ILgnM7UCnrQwFH67Z9B/view?usp=sharing)**. Karena penasaran dengan film-film tersebut, mereka memutuskan untuk membuat sistem otomatis guna mengelola semua file tersebut secara terstruktur dan efisien. Berikut adalah tugas yang harus dikerjakan untuk mewujudkan sistem tersebut:

### Soal:
### **a. Ekstraksi File ZIP**

Trabowo langsung mendownload file ZIP tersebut dan menyimpannya di penyimpanan lokal komputernya. Namun, karena file tersebut dalam bentuk ZIP, Trabowo perlu melakukan **unzip** agar dapat melihat daftar film-film seru yang ada di dalamnya.

### Jawaban:
File film di unzip menggunakan kode
# Tranbowo-a.c
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/wait.h>
#include <string.h>
// Function void untuk melakukan unzip, dengan parameter string, nama file yang menjadi tujuan unzip
void bjer(const char* zipFi) {
// Pembuatan fork
    pid_t pid = fork();
    if (pid == 0) {
        char *bruh[] = {"unzip", "-o", (char *)zipFi, "-d", ".", NULL};
        execvp(bruh[0], bruh); 
    } 
    else if (pid > 0) {
        int status;
        waitpid(pid, &status, 0); 
        if (status) {
            printf("DONE\n");
        } else {
            printf("GAGAL\n");
        }
    } 
}

int main() {
     char* zipFi = "film.zip";  
    bjer(zipFi);
    return 0;
}
```
Direktori sebelum film.zip di unzip
# foto
Direktori setelah film.zip di unzip
# foto
# output
# foto


### **b. Pemilihan Film Secara Acak**
### Soal: 
Setelah berhasil melakukan unzip, Trabowo iseng melakukan pemilihan secara acak/random pada gambar-gambar film tersebut untuk menentukan film pertama yang akan dia tonton malam ini.

**Format Output:**

```
Film for Trabowo & Peddy: ‘<no_namafilm_genre.jpg>’
```
### Jawaban:
Pemilihan film secara acak akan dilakukan oleh file
# Trabowo-b.c
```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <dirent.h>
#include <time.h>

void solve(char* dirBruh) {
    DIR* dir = opendir(dirBruh);
    struct dirent* bruh;
    char* files[1000];
    int JumFil = 0;
    while ((bruh = readdir(dir)) != NULL) {
        if (strcmp(bruh->d_name, ".") != 0 ) {
            files[JumFil++] = bruh->d_name;  
        }
    }
    closedir(dir);
    if (JumFil > 0) {
        srand(time(NULL));
        int ran = rand() % JumFil;  
        printf("Film for Trabowo & Peddy: ‘<%s>’\n", files[ran]);
    } else {
        printf("lah kok ga ada?\n");
    }
}

int main() {
    solve("film"); 
    return 0;
}
```
# output
# foto
# foto
# foto

### **c. Memilah Film Berdasarkan Genre**
### Soal
Karena Trabowo sangat perfeksionis dan ingin semuanya tertata rapi, dia memutuskan untuk mengorganisir film-film tersebut berdasarkan genre. Dia membuat 3 direktori utama di dalam folder `~/film`, yaitu:

- **FilmHorror**
- **FilmAnimasi**
- **FilmDrama**

Setelah itu, dia mulai memindahkan gambar-gambar film ke dalam folder yang sesuai dengan genrenya. Tetapi Trabowo terlalu tua untuk melakukannya sendiri, sehingga ia meminta bantuan Peddy untuk memindahkannya. Mereka membagi tugas secara efisien dengan mengerjakannya secara bersamaan (overlapping) dan membaginya sama banyak. Trabowo akan mengerjakan dari awal, sementara Peddy dari akhir. Misalnya, jika ada 10 gambar, Trabowo akan memulai dari gambar pertama, gambar kedua, dst dan Peddy akan memulai dari gambar kesepuluh, gambar kesembilan, dst. Lalu buatlah file “recap.txt” yang menyimpan log setiap kali mereka selesai melakukan task

Contoh format log :

```
[15-04-2025 13:44:59] Peddy: 50_toystory_animasi.jpg telah dipindahkan ke FilmAnimasi
```

Setelah memindahkan semua film, Trabowo dan Peddy juga perlu menghitung jumlah film dalam setiap kategori dan menuliskannya dalam file **`total.txt`**. Format dari file tersebut adalah:

```
Jumlah film horror: <jumlahfilm>
Jumlah film animasi: <jumlahfilm>
Jumlah film drama: <jumlahfilm>
Genre dengan jumlah film terbanyak: <namagenre>
```

### Trabowo-c.c
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <dirent.h>
#include <sys/stat.h>
#include <time.h>
// 3 dir 
// di saring
// trabowo dari depan, pedydy dari belakang
typedef struct {
    char filename[256];
    char genre[20];
} Movie;

void mkbjer(char *dir_name) {
    struct stat st = {0};
    if (stat(dir_name, &st) == -1) { if (mkdir(dir_name, 0700) != 0) { perror("bruh");} }}


void recap(char *worker, char *filename, char *genre) {
    FILE *logFile = fopen("recap.txt", "a");
    if (logFile == NULL) {
        perror("Fbruh");
        return;
    }

    time_t t = time(NULL);
    struct tm tm = *localtime(&t);
    fprintf(logFile, "[%02d-%02d-%04d %02d:%02d:%02d] %s: %s telah dipindahkan ke %s\n",
            tm.tm_mday, tm.tm_mon + 1, tm.tm_year + 1900, tm.tm_hour, tm.tm_min, tm.tm_sec, worker, filename, genre);
    fclose(logFile);
}
void pindah(char *file, char *desti) {
    char pathhh[512];
    char dir_path[512];
    char desti_path[512];

    snprintf(pathhh, sizeof(pathhh), "./film/%s", file);
    snprintf(desti_path, sizeof(desti_path), "./film/%s/%s", desti, file);
    snprintf(dir_path, sizeof(dir_path), "./film/%s", desti);
    mkbjer(dir_path); 
    if (rename(pathhh, desti_path) != 0) {
        perror("bruh");
    }
}
void saring(char *dir_path, Movie *muvi, int *count) {
    DIR *dir = opendir(dir_path);
    struct dirent *entry;
    if (dir == NULL) {
        perror("dir bruh");
        return;
    }
    while ((entry = readdir(dir)) != NULL) {
        if (entry->d_type == DT_REG) {  
            strcpy(muvi[*count].filename, entry->d_name);
            if (strstr(entry->d_name, "horror") != NULL) {
                strcpy(muvi[*count].genre, "FilmHorror");
            } else if (strstr(entry->d_name, "animasi") != NULL) {
                strcpy(muvi[*count].genre, "FilmAnimasi");
            } else if (strstr(entry->d_name, "drama") != NULL) {
                strcpy(muvi[*count].genre, "FilmDrama");
            } else {
                continue;  
            }
            (*count)++;
        }
    }
    closedir(dir);
}

int main() {
    Movie muvi[1000];
    int muviCou = 0;
    char *dir_path = "./film";
    saring(dir_path, muvi, &muviCou);
    int i = 0, j = muviCou - 1;
    int horrorCou = 0, animasiCou = 0, dramaCou = 0;
    while (i <= j) {
        if (strstr(muvi[i].genre, "FilmHorror") != NULL) {
            horrorCou++;
        } else if (strstr(muvi[i].genre, "FilmAnimasi") != NULL) {
            animasiCou++;
        } else if (strstr(muvi[i].genre, "FilmDrama") != NULL) {
            dramaCou++;
        }
        pindah(muvi[i].filename, muvi[i].genre);
        recap("Trabowo", muvi[i].filename, muvi[i].genre);
        i++;
        if (i <= j) {
            if (strstr(muvi[j].genre, "FilmHorror") != NULL) {
                horrorCou++;
            } else if (strstr(muvi[j].genre, "FilmAnimasi") != NULL) {
                animasiCou++;
            } else if (strstr(muvi[j].genre, "FilmDrama") != NULL) {
                dramaCou++;
            }
            pindah(muvi[j].filename, muvi[j].genre);
            recap("Peddy", muvi[j].filename, muvi[j].genre);
            j--;
        }
    }
    FILE *fileTOT = fopen("total.txt", "w");
    if (fileTOT == NULL) {
        perror("bruh");
        return 1;
    }
    fprintf(fileTOT, "Jumlah film horror: %d\n", horrorCou);
    fprintf(fileTOT, "Jumlah film animasi: %d\n", animasiCou);
    fprintf(fileTOT, "Jumlah film drama: %d\n", dramaCou);
    if (horrorCou >= animasiCou && horrorCou >= dramaCou) {
        fprintf(fileTOT, "Film terbanyak: FilmHorror\n");
    } else if (animasiCou >= horrorCou && animasiCou >= dramaCou) {
        fprintf(fileTOT, "Film terbanyak: FilmAnimasi\n");
    } else {
        fprintf(fileTOT, "Film terbanyak: FilmDrama\n");
    }
    fclose(fileTOT);
    return 0;
}
/*   /\_/\
*    (= ._.)
*   / >  \>
*/
// sorry gabut mas hehehehehhee
```
### **d. Pengarsipan Film**
### Soal
Setelah semua film tertata dengan rapi dan dikelompokkan dalam direktori masing-masing berdasarkan genre, Trabowo ingin mengarsipkan ketiga direktori tersebut ke dalam format **ZIP** agar tidak memakan terlalu banyak ruang di komputernya.

### Jawaban
File di zip dengan
# Trabowo-d.c
```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {
    char *zipFi = "filmdone.zip";
    char *bruh[] = {"zip", "-r", zipFi, "film", NULL};  

    pid_t pid = fork();

    if (pid == -1) {
        perror("fork failed");
        return 1;
    } else if (pid == 0) {
        execvp("zip", bruh);
        perror("gagal");
        return 1;
    } else {
        int status;
        waitpid(pid, &status, 0);
        if (WIFEXITED(status) && WEXITSTATUS(status) == 0) {
            printf("Zip DONE: %s\n", zipFi);
        } else {
            printf("GAGAL\n");
        }
    }

    return 0;
}

```
