# Code for 1)
```c

#include <stdio.h>
#include <stdlib.h>

int main() {
    int N;

    printf("Questo programma genera 2 vettori (x e y);\nIl modulo di ogni componente di X sarà 0.1 mentre il modulo di ogni componente di Y sarà 7.1.\nOra sta a te definire il numero di componenti e quindi la dimensionalità N dei due vettori.\nScegli N e digitalo qui sotto, poi premi invio:\n");

    // Controllo che l'input sia un intero valido
    if (scanf("%d", &N) != 1) {
        printf("Errore: input non valido. Devi inserire un intero positivo.\n");
        return 1;
    }

    // Verifica che N sia un valore positivo
    if (N <= 0) {
        printf("Errore: N deve essere un numero positivo.\n");
        return 1;
    }

    // Generazione dinamica dei nomi dei file usando sprintf
    char NOME_FILE_1[100], NOME_FILE_2[100];  // Array per memorizzare i nomi dei file
    sprintf(NOME_FILE_1, "Vector_X_dim%d.dat", N);
    sprintf(NOME_FILE_2, "Vector_Y_dim%d.dat", N);

    FILE *f1 = fopen(NOME_FILE_1, "w");
    if (f1 == NULL) {
        printf("Errore nell'apertura del file %s!\n", NOME_FILE_1);
        return 1;
    }

    double *vector1 = (double*)malloc(N * sizeof(double));
    if (vector1 == NULL) {
        printf("Errore nell'allocazione della memoria!\n");
        fclose(f1);
        return 1;
    }

    // Riempimento del vettore 1 con 0.1
    for (int i = 0; i < N; i++) {
        vector1[i] = 0.1;
        fprintf(f1, "%.1f ", vector1[i]);
    }

    fclose(f1);
    printf("File %s creato con successo!\n", NOME_FILE_1);

    FILE *f2 = fopen(NOME_FILE_2, "w");
    if (f2 == NULL) {
        printf("Errore nell'apertura del file %s!\n", NOME_FILE_2);
        free(vector1);
        return 1;
    }

    double *vector2 = (double*)malloc(N * sizeof(double));
    if (vector2 == NULL) {
        printf("Errore nell'allocazione della memoria!\n");
        free(vector1);
        fclose(f2);
        return 1;
    }

    // Riempimento del vettore 2 con 7.1
    for (int i = 0; i < N; i++) {
        vector2[i] = 7.1;
        fprintf(f2, "%.1f ", vector2[i]);
    }

    fclose(f2);
    printf("File %s creato con successo!\n", NOME_FILE_2);

    // Libero la memoria
    free(vector1);
    free(vector2);

    return 0;
}
```
# Code for 2)
```c
```
