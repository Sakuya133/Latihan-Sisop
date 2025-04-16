1. Identitas diri:

   - 5025241218
   - Ida Bagus Gde Dimas Sutha Maha
   - E
   - E12
## 1. Process

**Deskripsi:**

Buat satu program C yang melakukan tiga proses secara berurutan:

a. Membuat folder baru bernama `halo`.

b. Membuat file kosong bernama `hai.txt` di dalam folder `halo`.

c. Mengcopy file `hai.txt` tersebut ke luar folder, sehingga hasilnya:

- Ada `hai.txt` di dalam folder `halo/`
- Ada juga salinan `hai.txt` di luar folder `halo` (di direktori yang sama dengan program ini dijalankan)

**Hasil akhir struktur direktori:**

```
Direktori/
│
├── program.c
├── halo/
│   └── hai.txt
└── hai.txt  ← hasil copy dari halo/hai.txt
```

**Catatan:**

- Semua proses ditulis dalam **satu file C**.
- Program dijalankan **sekali saja**.

---
### Code.c
```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/stat.h> 
#include <sys/types.h>
#include <string.h>
#include <errno.h>

int main() {
    const char *folder_name = "halo";
    const char *file_inside = "halo/hai.txt";
    const char *file_outside = "hai.txt";

   
    if (mkdir(folder_name, 0777) == -1) {
        if (errno == EEXIST) {
            printf("Folder '%s' sudah ada.\n", folder_name);
        } else {
            perror("Gagal membuat folder");
            return 1;
        }
    } else {
        printf("Folder '%s' berhasil dibuat.\n", folder_name);
    }

    
    FILE *fp = fopen(file_inside, "w");
    if (fp == NULL) {
        perror("Gagal membuat file hai.txt di dalam folder");
        return 1;
    }
    fclose(fp);
    printf("File '%s' berhasil dibuat.\n", file_inside);

    
    FILE *src = fopen(file_inside, "r");
    FILE *dst = fopen(file_outside, "w");

    if (src == NULL || dst == NULL) {
        perror("Gagal membuka file untuk menyalin");
        return 1;
    }

    char buffer[1024];
    size_t bytes;
    while ((bytes = fread(buffer, 1, sizeof(buffer), src)) > 0) {
        fwrite(buffer, 1, bytes, dst);
    }

    fclose(src);
    fclose(dst);

    printf("Sukses copy '%s'\n", file_outside);

    return 0;
}

```
# Penjelasan
```c
const char *folder_name = "halo";
const char *file_inside = "halo/hai.txt";
const char *file_outside = "hai.txt";
```
Variabel string yang menyimpan file dan folder
- Folder yang akan dibuat di direktori: "halo
- File dibuat dalam "halo/hai.txt"
- File hasil salinan yang akan diletakkan di luar folder: "hai.txt"

```c
if (mkdir(folder_name, 0777) == -1) {
    if (errno == EEXIST) {
        printf("Folder '%s' sudah ada.\n", folder_name);
    } else {
        perror("Gagal membuat folder");
        return 1;
    }
} else {
    printf("Folder '%s' berhasil dibuat.\n", folder_name);
}

```
- Fungsi mkdir digunakan untuk membuat folder dengan izin akses penuh (0777). Kalau alasannya karena folder sudah ada, maka ditampilkan pesan bahwa folder sudah ada.
Kalau alasannya lain, maka ditampilkan error dari perror. Kalau berhasil, ditampilkan bahwa folder berhasil dibuat.
## 2. Thread


Buat satu program dengan **3 thread**, masing-masing bertugas:

a. **Thread 1:** Menulis angka 1–100 secara berurutan ke file `count.txt`.

b. **Thread 2:** Menulis `"Saya pintar mengerjakan thread"` ke file `print.txt`.

c. **Thread 3:** Menulis semua angka **genap** dari 1–100 ke file `count_2.txt`.

**Catatan:**

- Jalankan program **sebanyak 3 kali**.
- Catat **urutan thread** yang selesai duluan ke file `log.txt`.

**Contoh isi `log.txt`:**

```
1. Thread 1
2. Thread 1
3. Thread 1
```
> Setiap kali program dijalankan, bisa saja urutan thread yang selesai berbeda-beda.
##### Thread.c
```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>

pthread_mutex_t log_mutex;
int finished_count = 0;


void log_thread_finish(const char *thread_name) {
    pthread_mutex_lock(&log_mutex);

    FILE *log = fopen("log.txt", "a");
    if (log == NULL) {
        perror("Gagal membuka log.txt");
        exit(1);
    }
    fprintf(log, "%d. %s\n", ++finished_count, thread_name);
    fclose(log);

    pthread_mutex_unlock(&log_mutex);
}


void *thread1_func(void *arg) {
    FILE *f = fopen("count.txt", "w");
    if (!f) {
        perror("Gagal membuka count.txt");
        pthread_exit(NULL);
    }
    for (int i = 1; i <= 100; i++) {
        fprintf(f, "%d\n", i);
        usleep(1000);
    }
    fclose(f);
    log_thread_finish("Thread 1");
    pthread_exit(NULL);
}

void *thread2_func(void *arg) {
    FILE *f = fopen("print.txt", "w");
    if (!f) {
        perror("Gagal membuka print.txt");
        pthread_exit(NULL);
    }
    fprintf(f, "Saya pintar mengerjakan thread\n");
    fclose(f);
    log_thread_finish("Thread 2");
    pthread_exit(NULL);
}


void *thread3_func(void *arg) {
    FILE *f = fopen("count_2.txt", "w");
    if (!f) {
        perror("Gagal membuka count_2.txt");
        pthread_exit(NULL);
    }
    for (int i = 2; i <= 100; i += 2) {
        fprintf(f, "%d\n", i);
        usleep(1000);
    }
    fclose(f);
    log_thread_finish("Thread 3");
    pthread_exit(NULL);
}

int main() {
    pthread_t t1, t2, t3;

    
    finished_count = 0;

   
    pthread_mutex_init(&log_mutex, NULL);

    
    pthread_create(&t1, NULL, thread1_func, NULL);
    pthread_create(&t2, NULL, thread2_func, NULL);
    pthread_create(&t3, NULL, thread3_func, NULL);

    
    pthread_join(t1, NULL);
    pthread_join(t2, NULL);
    pthread_join(t3, NULL);

    
    pthread_mutex_destroy(&log_mutex);

    return 0;
}

```
## 3. Thread dengan Mutex

Buatlah satu program yang:

a. Membuat **5 thread**, masing-masing menghitung dari 1 sampai 3.

b. Setiap thread menulis hasil hitungannya ke file `log.txt` dengan format:

```
thread {id} count {angka}
```

c. Gunakan `pthread_mutex` untuk mencegah **race condition** saat menulis ke file.

> **Catatan:** Saat satu thread sedang counting dan menulis ke file, thread lain **harus menunggu**.

**Contoh isi `log.txt`:**

```
thread 3 count 1
thread 3 count 2
thread 3 count 3
thread 1 count 1
...
thread 5 count 3
```
### Code.c
```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>

#define THREAD_COUNT 5

pthread_mutex_t file_mutex;

void *counting_thread(void *arg) {
    int id = *(int *)arg;

    // Lock mutex agar hanya satu thread yang mengakses file
    pthread_mutex_lock(&file_mutex);

    FILE *log = fopen("log.txt", "a");
    if (!log) {
        perror("Gagal membuka log.txt");
        pthread_mutex_unlock(&file_mutex);
        pthread_exit(NULL);
    }

    for (int i = 1; i <= 3; i++) {
        fprintf(log, "thread %d count %d\n", id, i);
        fflush(log); // Memastikan langsung ditulis ke file
        sleep(1);    // Delay biar efek counting terlihat
    }

    fclose(log);
    pthread_mutex_unlock(&file_mutex);

    pthread_exit(NULL);
}

int main() {
    pthread_t threads[THREAD_COUNT];
    int thread_ids[THREAD_COUNT];

    pthread_mutex_init(&file_mutex, NULL);

    // Buat thread
    for (int i = 0; i < THREAD_COUNT; i++) {
        thread_ids[i] = i + 1;
        pthread_create(&threads[i], NULL, counting_thread, &thread_ids[i]);
    }

    // Tunggu semua thread selesai
    for (int i = 0; i < THREAD_COUNT; i++) {
        pthread_join(threads[i], NULL);
    }

    pthread_mutex_destroy(&file_mutex);

    return 0;
}

```
---
