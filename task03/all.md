# Code for 1)
```c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[]) {
    if (argc != 3) {
        printf("Uso corretto: %s <N> <prefix_output>\n", argv[0]);
        return 1;
    }

    int N = atoi(argv[1]);
    char *prefix = argv[2];

    if (N <= 0) {
        printf("Errore: N deve essere un intero positivo.\n");
        return 1;
    }

    // Generazione nomi dei file nella cartella corrente
    char nome_x[200], nome_y[200];
    sprintf(nome_x, "%sN%d_x.dat", prefix, N);
    sprintf(nome_y, "%sN%d_y.dat", prefix, N);

    // Allocazione
    double *x = malloc(N * sizeof(double));
    double *y = malloc(N * sizeof(double));

    if (x == NULL || y == NULL) {
        printf("Errore nell'allocazione della memoria.\n");
        free(x); free(y);
        return 1;
    }

    // Riempimento
    for (int i = 0; i < N; i++) {
        x[i] = 0.1;
        y[i] = 7.1;
    }

    // Scrittura file X
    FILE *fx = fopen(nome_x, "w");
    if (fx == NULL) {
        printf("Errore nell'apertura del file %s\n", nome_x);
        free(x); free(y);
        return 1;
    }
    for (int i = 0; i < N; i++) fprintf(fx, "%.1f ", x[i]);
    fclose(fx);

    // Scrittura file Y
    FILE *fy = fopen(nome_y, "w");
    if (fy == NULL) {
        printf("Errore nell'apertura del file %s\n", nome_y);
        free(x); free(y);
        return 1;
    }
    for (int i = 0; i < N; i++) fprintf(fy, "%.1f ", y[i]);
    fclose(fy);

    free(x); free(y);

    printf("File creati con successo:\n- %s\n- %s\n", nome_x, nome_y);
    return 0;
}

```
# Code for 2)
```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_LINE 256

// Funzione per contare i numeri in un file
int conta_elementi(FILE *f) {
    double temp;
    int count = 0;
    while (fscanf(f, "%lf", &temp) == 1) {
        count++;
    }
    rewind(f);
    return count;
}

int main(int argc, char *argv[]) {
    if (argc != 2) {
        printf("Uso: %s <config_file.cfg>\n", argv[0]);
        return 1;
    }

    // Variabili da riempire col contenuto del .cfg
    char file_x[100], file_y[100], prefix_output[100];
    int N = -1;
    double a = 0.0;

    // Apertura file cfg
    FILE *fcfg = fopen(argv[1], "r");
    if (fcfg == NULL) {
        printf("Errore nell'apertura del file di configurazione.\n");
        return 1;
    }

    char riga[MAX_LINE];
    while (fgets(riga, MAX_LINE, fcfg)) {
        // Rimuovo eventuale newline
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

    // Verifica dei parametri
    if (N <= 0 || a == 0.0 || strlen(file_x) == 0 || strlen(file_y) == 0 || strlen(prefix_output) == 0) {
        printf("Errore nei dati di configurazione.\n");
        return 1;
    }

    // Apertura file x e y
    FILE *fx = fopen(file_x, "r");
    FILE *fy = fopen(file_y, "r");

    if (fx == NULL || fy == NULL) {
        printf("Errore nell'apertura dei file x o y.\n");
        if (fx) fclose(fx);
        if (fy) fclose(fy);
        return 1;
    }

    // Controllo effettiva dimensione dei file
    int n_x = conta_elementi(fx);
    int n_y = conta_elementi(fy);

    if (n_x != N || n_y != N) {
        printf("Errore: i file non contengono %d valori.\n", N);
        fclose(fx); fclose(fy);
        return 1;
    }

    // Allocazione memoria
    double *x = malloc(N * sizeof(double));
    double *y = malloc(N * sizeof(double));
    double *d = malloc(N * sizeof(double));

    if (!x || !y || !d) {
        printf("Errore allocazione memoria.\n");
        fclose(fx); fclose(fy);
        free(x); free(y); free(d);
        return 1;
    }

    // Lettura vettori
    for (int i = 0; i < N; i++) {
        fscanf(fx, "%lf", &x[i]);
        fscanf(fy, "%lf", &y[i]);
    }
    fclose(fx); fclose(fy);

    // Calcolo: d = a * x + y
    for (int i = 0; i < N; i++) {
        d[i] = a * x[i] + y[i];
    }

    // Nome file output
    char nome_file_d[150];
    sprintf(nome_file_d, "%sN%d_d.dat", prefix_output, N);

    // Scrittura file output
    FILE *fd = fopen(nome_file_d, "w");
    if (fd == NULL) {
        printf("Errore nella creazione del file di output.\n");
        free(x); free(y); free(d);
        return 1;
    }

    for (int i = 0; i < N; i++) {
        fprintf(fd, "%.2f ", d[i]);
    }

    fclose(fd);
    free(x); free(y); free(d);

    printf("File di output creato: %s\n", nome_file_d);
    return 0;
}

```
