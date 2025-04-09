
# Solution to DAXPY Task — All 5 Questions

---

## 1) Code to Generate Two Vectors (x and y)

```c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[]) {
    if (argc != 3) {
        printf("Correct usage: %s <N> <prefix_output>\n", argv[0]);
        return 1;
    }

    int N = atoi(argv[1]);
    char *prefix = argv[2];

    if (N <= 0) {
        printf("Error: N must be a positive integer.\n");
        return 1;
    }

    char nome_x[200], nome_y[200];
    sprintf(nome_x, "%sN%d_x.dat", prefix, N);
    sprintf(nome_y, "%sN%d_y.dat", prefix, N);

    double *x = malloc(N * sizeof(double));
    double *y = malloc(N * sizeof(double));

    if (!x || !y) {
        printf("Memory allocation error.\n");
        free(x); free(y);
        return 1;
    }

    for (int i = 0; i < N; i++) {
        x[i] = 0.1;
        y[i] = 7.1;
    }

    FILE *fx = fopen(nome_x, "w");
    FILE *fy = fopen(nome_y, "w");

    if (!fx || !fy) {
        printf("Error opening output files.\n");
        if (fx) fclose(fx);
        if (fy) fclose(fy);
        free(x); free(y);
        return 1;
    }

    for (int i = 0; i < N; i++) fprintf(fx, "%.1f ", x[i]);
    for (int i = 0; i < N; i++) fprintf(fy, "%.1f ", y[i]);

    fclose(fx); fclose(fy);
    free(x); free(y);

    printf("Files created: %s and %s\n", nome_x, nome_y);
    return 0;
}
```

**Run example**:
```bash
./program1 10 vector_
```

---

## 2) Code to Compute d = a * x + y using Config File

### Example `config.cfg`:
```ini
file_x = vector_N10_x.dat
file_y = vector_N10_y.dat
N = 10
a = 3
prefix_output = vector_
```

### C Code:
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_LINE 256

int conta_elementi(FILE *f) {
    double temp;
    int count = 0;
    while (fscanf(f, "%lf", &temp) == 1) count++;
    rewind(f);
    return count;
}

int main(int argc, char *argv[]) {
    if (argc != 2) {
        printf("Usage: %s <config_file.cfg>\n", argv[0]);
        return 1;
    }

    char file_x[100], file_y[100], prefix_output[100];
    int N = -1;
    double a = 0.0;

    FILE *fcfg = fopen(argv[1], "r");
    if (!fcfg) {
        printf("Error opening config file.\n");
        return 1;
    }

    char riga[MAX_LINE];
    while (fgets(riga, MAX_LINE, fcfg)) {
        riga[strcspn(riga, "\n")] = '\0';
        sscanf(riga, "file_x = %s", file_x) == 1 ||
        sscanf(riga, "file_y = %s", file_y) == 1 ||
        sscanf(riga, "N = %d", &N) == 1 ||
        sscanf(riga, "a = %lf", &a) == 1 ||
        sscanf(riga, "prefix_output = %s", prefix_output) == 1;
    }
    fclose(fcfg);

    FILE *fx = fopen(file_x, "r");
    FILE *fy = fopen(file_y, "r");
    if (!fx || !fy) {
        printf("Error opening input files.\n");
        return 1;
    }

    double *x = malloc(N * sizeof(double));
    double *y = malloc(N * sizeof(double));
    double *d = malloc(N * sizeof(double));
    if (!x || !y || !d) {
        printf("Memory allocation failed.\n");
        return 1;
    }

    for (int i = 0; i < N; i++) {
        fscanf(fx, "%lf", &x[i]);
        fscanf(fy, "%lf", &y[i]);
    }
    fclose(fx); fclose(fy);

    for (int i = 0; i < N; i++) d[i] = a * x[i] + y[i];

    char nome_file_d[200];
    sprintf(nome_file_d, "%sN%d_d.dat", prefix_output, N);
    FILE *fd = fopen(nome_file_d, "w");
    for (int i = 0; i < N; i++) fprintf(fd, "%.2f ", d[i]);
    fclose(fd);
    free(x); free(y); free(d);

    printf("Output file created: %s\n", nome_file_d);
    return 0;
}
```

**Run example**:
```bash
./calcola_daxpy config.cfg
```

---

## 3) Makefile to Compile Both Programs

```make
CC = gcc
CFLAGS = -Wall -O2

all: genera_vettori daxpy_from_cfg

genera_vettori: genera_vettori.c
	$(CC) $(CFLAGS) -o genera_vettori genera_vettori.c

daxpy_from_cfg: daxpy_from_cfg.c
	$(CC) $(CFLAGS) -o daxpy_from_cfg daxpy_from_cfg.c

clean:
	rm -f genera_vettori daxpy_from_cfg
```

Run:
```bash
make
```

---

## 4) Using Binary Files Instead of Text

### Modifications:

- Write with `fwrite(...)` and open files in `"wb"` mode.
- Read with `fread(...)` and open files in `"rb"` mode.

**No other logic changes needed**, except removing text parsing.

---

## 5) Using GSL: `gsl_vector_axpby`

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <gsl/gsl_vector.h>
#include <gsl/gsl_blas.h>

#define MAX_LINE 256

int main(int argc, char *argv[]) {
    if (argc != 2) {
        printf("Uso: %s <config_file.cfg>\n", argv[0]);
        return 1;
    }

    char file_x[100] = "", file_y[100] = "", prefix_output[100] = "";
    int N = -1;
    double a = 0.0;

    FILE *fcfg = fopen(argv[1], "r");
    if (!fcfg) {
        printf("Errore apertura file di configurazione.\n");
        return 1;
    }

    char riga[MAX_LINE];
    while (fgets(riga, MAX_LINE, fcfg)) {
        riga[strcspn(riga, "\n")] = '\0';

        if (strncmp(riga, "file_x", 6) == 0) {
            sscanf(riga, "file_x = %s", file_x);
        } else if (strncmp(riga, "file_y", 6) == 0) {
            sscanf(riga, "file_y = %s", file_y);
        } else if (strncmp(riga, "N", 1) == 0) {
            sscanf(riga, "N = %d", &N);
        } else if (strncmp(riga, "a", 1) == 0) {
            sscanf(riga, "a = %lf", &a);
        } else if (strncmp(riga, "prefix_output", 13) == 0) {
            sscanf(riga, "prefix_output = %s", prefix_output);
        }
    }
    fclose(fcfg);

    if (N <= 0 || a == 0.0 || strlen(file_x) == 0 || strlen(file_y) == 0 || strlen(prefix_output) == 0) {
        printf("Errore nei dati di configurazione.\n");
        return 1;
    }

    FILE *fx = fopen(file_x, "rb");
    FILE *fy = fopen(file_y, "rb");

    if (!fx || !fy) {
        printf("Errore apertura file x o y.\n");
        if (fx) fclose(fx);
        if (fy) fclose(fy);
        return 1;
    }

    // Alloco vettori GSL
    gsl_vector *vx = gsl_vector_alloc(N);
    gsl_vector *vy = gsl_vector_alloc(N);
    gsl_vector *vd = gsl_vector_alloc(N);

    if (!vx || !vy || !vd) {
        printf("Errore allocazione memoria GSL.\n");
        fclose(fx); fclose(fy);
        gsl_vector_free(vx);
        gsl_vector_free(vy);
        gsl_vector_free(vd);
        return 1;
    }

    // Lettura da file binari dentro i vettori GSL
    fread(vx->data, sizeof(double), N, fx);
    fread(vy->data, sizeof(double), N, fy);
    fclose(fx); fclose(fy);

    // Calcolo: vd = a * vx + 1.0 * vy
    gsl_vector_memcpy(vd, vy);  // vd ← vy
    gsl_blas_daxpy(a, vx, vd);  // vd ← a*vx + vd

    // Scrittura su file binario
    char nome_file_d[150];
    sprintf(nome_file_d, "%sN%d_d.dat", prefix_output, N);
    FILE *fd = fopen(nome_file_d, "wb");

    if (!fd) {
        printf("Errore nella creazione del file di output.\n");
        gsl_vector_free(vx);
        gsl_vector_free(vy);
        gsl_vector_free(vd);
        return 1;
    }

    fwrite(vd->data, sizeof(double), N, fd);
    fclose(fd);

    gsl_vector_free(vx);
    gsl_vector_free(vy);
    gsl_vector_free(vd);

    printf("File di output creato (usando GSL): %s\n", nome_file_d);
    return 0;
}

```

Compile with:
```bash
gcc -o daxpy_gsl daxpy_gsl.c -lgsl -lgslcblas -lm
```

---
