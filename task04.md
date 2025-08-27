```c
/*
 * Task 04 – Play with discrete math (soluzione in C)
 *
 * Requisiti dalla traccia (riassunto):
 * - f(x) = e^x * cos(x)
 * - INPUT: N (numero di punti), x_inf, x_sup
 * - OUTPUT:
 *     (1) Scrivere un file con N righe e 2 colonne: x   f(x)
 *         (campionamento uniforme in [x_inf, x_sup], estremi inclusi)
 *     (2) Stampare a terminale il valore dell’integrale:
 *             I = ∫_0^{π/2} f(x) dx
 *         con 16 cifre decimali, usando (ad es.) la regola dei trapezi.
 *
 * NOTA IMPORTANTE (richiesta dall’utente):
 * - NON rispondere/affrontare per ora le domande 1) 2) 3) 4) della traccia.
 *
 * COMPILAZIONE:
 *     gcc -O2 -Wall -Wextra -std=c11 task04.c -o task04 -lm
 *
 * ESECUZIONE:
 *   a) Con argomenti da riga di comando:
 *        ./task04 N x_inf x_sup [output_filename]
 *      Esempio:
 *        ./task04 1000 0.0 1.0 fx_samples.txt
 *
 *   b) Interattivo (senza argomenti):
 *        ./task04
 *      Il programma chiede N, x_inf, x_sup e usa "fx_samples.txt" come default.
 *
 * DETTAGLI DI IMPLEMENTAZIONE:
 * - Il campionamento genera N punti uniformemente spaziati su [x_inf, x_sup],
 *   includendo *sempre* x_inf (i=0) e x_sup (i=N-1). Se N=1, scrive un solo punto x_inf,
 *   ma per l’integrale la regola dei trapezi richiede almeno 2 punti: in tal caso
 *   il programma segnala errore.
 * - L’integrale I viene calcolato su [0, π/2] con la regola dei trapezi usando M=N
 *   nodi uniformi (M>=2). Questo collega la “risoluzione numerica” dell’integrale
 *   a N, come spesso si fa nelle esercitazioni. Volendo, si potrebbe svincolare M da N.
 * - Stampa dell’integrale con 16 cifre decimali: printf("%.16f", I).
 * - Si fa uso di double (64 bit) e funzioni di <math.h> (exp, cos).
 */

#include <stdio.h>    // printf, fprintf, scanf, FILE, fopen, fclose
#include <stdlib.h>   // strtod, strtoll, EXIT_SUCCESS/EXIT_FAILURE
#include <string.h>   // strlen, strcpy, etc. (qui usato minimamente)
#include <errno.h>    // errno per robustezza nel parsing
#include <math.h>     // exp, cos, acos

/* ----------------------------------------------------------------------------
 * COSTANTI & UTILITIES
 * ----------------------------------------------------------------------------
 */

/*
 * Definizione di π. Evitiamo di fare affidamento su M_PI (non standard C11),
 * e la otteniamo in modo portabile come acos(-1.0). La rendiamo 'const' così
 * il compilatore può ottimizzare.
 */
static double get_pi(void) {
    return acos(-1.0);   // valore di π con precisione double
}

/*
 * f(x) = e^x * cos(x)  —  funzione obiettivo della traccia.
 * inline per permettere al compilatore di ottimizzare eventuali chiamate ripetute.
 */
static inline double f(double x) {
    return exp(x) * cos(x);
}

/* ----------------------------------------------------------------------------
 * REGOLA DEI TRAPEZI
 * ----------------------------------------------------------------------------
 *
 * Calcola numericamente ∫_a^b f(x) dx usando la regola dei trapezi con M nodi
 * uniformemente spaziati.
 *
 * - M è il numero di nodi (punti di campionamento), non il numero di sottointervalli.
 *   Quindi il numero di sottointervalli è (M - 1). Per la regola dei trapezi è
 *   necessario M >= 2.
 * - Spaziatura: h = (b - a) / (M - 1)
 * - Formula:
 *      I ≈ h * [ 0.5*f(x0) + f(x1) + f(x2) + ... + f(x_{M-2}) + 0.5*f(x_{M-1}) ]
 *   dove x0=a e x_{M-1}=b.
 */
static double trapezoidal_integral(double a, double b, long long M) {
    if (M < 2) {
        // Caso non valido per la regola dei trapezi: servono almeno due nodi
        return NAN;
    }

    const double h = (b - a) / (double)(M - 1);

    // Valutazione f agli estremi con peso 1/2
    double sum = 0.5 * f(a) + 0.5 * f(b);

    // Somma dei termini interni, ciascuno con peso 1
    for (long long i = 1; i < M - 1; ++i) {
        const double xi = a + h * (double)i;
        sum += f(xi);
    }

    // Moltiplica per la spaziatura per ottenere l’approssimazione dell’integrale
    return h * sum;
}

/* ----------------------------------------------------------------------------
 * SCRITTURA DEL FILE DI CAMPIONAMENTO (x, f(x))
 * ----------------------------------------------------------------------------
 *
 * Scrive su 'filename' un file testuale con N righe e 2 colonne:
 *   colonna 1: x_i
 *   colonna 2: f(x_i)
 * dove i=0..N-1 e x_i = x_inf + i * (x_sup - x_inf)/(N-1)  (se N>=2).
 *
 * Formato: due numeri in doppia precisione per riga, separati da TAB,
 * con 16 cifre decimali per coerenza con la richiesta di stampa “ad alta precisione”.
 *
 * NOTE:
 * - Se N==1, scriviamo una sola riga con x_inf e f(x_inf).
 * - In caso di errori (apertura file, ecc.), restituiamo un codice non nullo.
 */
static int write_samples(const char *filename, long long N, double x_inf, double x_sup) {
    FILE *fp = fopen(filename, "w");
    if (!fp) {
        perror("Errore nell'aprire il file di output");
        return 1;
    }

    if (N <= 0) {
        fprintf(stderr, "N deve essere >= 1 per generare il file di campioni.\n");
        fclose(fp);
        return 2;
    }

    if (N == 1) {
        const double x0 = x_inf;
        const double fx0 = f(x0);
        // Due colonne, 16 cifre decimali, separate da TAB
        fprintf(fp, "%.16f\t%.16f\n", x0, fx0);
        fclose(fp);
        return 0;
    }

    // N >= 2: campionamento uniforme includendo gli estremi
    const double h = (x_sup - x_inf) / (double)(N - 1);

    for (long long i = 0; i < N; ++i) {
        const double xi  = x_inf + h * (double)i;
        const double fxi = f(xi);
        // Stili possibili: CSV con virgola, TSV con TAB, spazi. Qui usiamo TAB.
        fprintf(fp, "%.16f\t%.16f\n", xi, fxi);
    }

    fclose(fp);
    return 0;
}

/* ----------------------------------------------------------------------------
 * PARSING ROBUSTO DEGLI ARGOMENTI
 * ----------------------------------------------------------------------------
 *
 * Funzioni di utilità per convertire stringhe in interi (long long) e double,
 * con controllo elementare degli errori (overflow, input non numerici).
 */

/* Converte una stringa in long long (per N). Ritorna 0 in caso di successo, !=0 in errore. */
static int parse_ll(const char *s, long long *out) {
    if (!s || !out) return 1;
    errno = 0;
    char *end = NULL;
    long long val = strtoll(s, &end, 10);
    if (errno != 0 || end == s || *end != '\0') {
        return 2; // errore di conversione
    }
    *out = val;
    return 0;
}

/* Converte una stringa in double. Ritorna 0 in caso di successo, !=0 in errore. */
static int parse_double(const char *s, double *out) {
    if (!s || !out) return 1;
    errno = 0;
    char *end = NULL;
    double val = strtod(s, &end);
    if (errno != 0 || end == s || *end != '\0') {
        return 2; // errore di conversione
    }
    *out = val;
    return 0;
}

/* ----------------------------------------------------------------------------
 * MAIN
 * ----------------------------------------------------------------------------
 *
 * Flusso del programma:
 *   1) Legge N, x_inf, x_sup dagli argomenti o, se assenti, da stdin.
 *   2) Scrive il file con N righe (x, f(x)) su [x_inf, x_sup].
 *   3) Calcola I = ∫_0^{π/2} f(x) dx mediante regola dei trapezi con M=N nodi.
 *   4) Stampa I a terminale con 16 cifre decimali.
 *
 * NOTA: Per esplicita richiesta, NON calcola né stampa errori relativi/assoluti,
 *       né risponde alle domande 1)–4) della traccia.
 */
int main(int argc, char **argv) {
    // Valori di input
    long long N = 0;       // numero di punti (>=1 per il file; >=2 per l'integrale a trapezi)
    double x_inf = 0.0;    // estremo inferiore per il campionamento
    double x_sup = 0.0;    // estremo superiore per il campionamento

    // Nome file di output (di default, se non passato da CLI)
    const char *default_out = "fx_samples.txt";
    const char *out_name = default_out;

    /* --------------- Lettura degli argomenti --------------- */
    if (argc == 4 || argc == 5) {
        // Uso: ./task04 N x_inf x_sup [output_filename]
        if (parse_ll(argv[1], &N) != 0 || N <= 0) {
            fprintf(stderr, "Errore: N deve essere un intero >= 1.\n");
            return EXIT_FAILURE;
        }
        if (parse_double(argv[2], &x_inf) != 0) {
            fprintf(stderr, "Errore nel parsing di x_inf.\n");
            return EXIT_FAILURE;
        }
        if (parse_double(argv[3], &x_sup) != 0) {
            fprintf(stderr, "Errore nel parsing di x_sup.\n");
            return EXIT_FAILURE;
        }
        if (argc == 5) {
            out_name = argv[4]; // Nome del file specificato dall'utente
        }
    } else if (argc == 1) {
        // Modalità interattiva: chiediamo i tre valori
        printf("Inserisci N (intero >= 1): ");
        if (scanf("%lld", &N) != 1 || N <= 0) {
            fprintf(stderr, "Errore: N deve essere un intero >= 1.\n");
            return EXIT_FAILURE;
        }
        printf("Inserisci x_inf (double): ");
        if (scanf("%lf", &x_inf) != 1) {
            fprintf(stderr, "Errore nel leggere x_inf.\n");
            return EXIT_FAILURE;
        }
        printf("Inserisci x_sup (double): ");
        if (scanf("%lf", &x_sup) != 1) {
            fprintf(stderr, "Errore nel leggere x_sup.\n");
            return EXIT_FAILURE;
        }
        // out_name resta il default ("fx_samples.txt")
        printf("Userò il file di output predefinito: %s\n", out_name);
    } else {
        // Guida rapida all'uso
        fprintf(stderr,
                "Uso: %s N x_inf x_sup [output_filename]\n"
                "oppure, senza argomenti, input interattivo da stdin.\n", argv[0]);
        return EXIT_FAILURE;
    }

    /* --------------- Scrittura del file di campioni (x, f(x)) --------------- */
    if (write_samples(out_name, N, x_inf, x_sup) != 0) {
        fprintf(stderr, "Errore nella generazione del file dei campioni.\n");
        return EXIT_FAILURE;
    }

    /* --------------- Calcolo numerico dell'integrale I --------------- */
    const double a = 0.0;               // limite inferiore di integrazione
    const double b = 0.5 * get_pi();    // limite superiore (π/2)

    if (N < 2) {
        /*
         * Nota: Per la regola dei trapezi sono necessari almeno 2 nodi.
         * Abbiamo comunque scritto il file (per N=1), ma non possiamo calcolare I.
         */
        fprintf(stderr, "Attenzione: N=%lld < 2; impossibile calcolare l'integrale con la regola dei trapezi.\n", N);
        fprintf(stderr, "Aumenta N a >= 2 e riesegui per ottenere I.\n");
        return EXIT_FAILURE;
    }

    // Usiamo M=N nodi per l'integrale, uniformemente in [0, π/2]
    const double I = trapezoidal_integral(a, b, N);

    // Stampa con 16 cifre decimali come richiesto dalla traccia
    // (NOTA: nessun calcolo o stampa di errori qui.)
    printf("I = %.16f\n", I);

    /*
     * Fine: il programma ha
     *  - scritto su 'out_name' le N coppie (x, f(x)) su [x_inf, x_sup]
     *  - calcolato e stampato I = ∫_0^{π/2} f(x) dx con la regola dei trapezi
     *    usando M=N nodi, con 16 cifre decimali
     */
    return EXIT_SUCCESS;
}

```
