# 1) $\vec{d}=a\vec{x} + \vec{y}$&nbsp;&nbsp;&nbsp;&nbsp;with Julia
```julia
a = 3
n_1 = 10
n_2 = 10^6
n_3 = 10^8
x_1 = fill(0.1, n_1)
x_2 = fill(0.1, n_2)
x_3 = fill(0.1, n_3)
y_1 = fill(7.1, n_1)
y_2 = fill(7.1, n_2)
y_3 = fill(7.1, n_3)
d_1 = a*x_1 + y_1
d_2 = a*x_2 + y_2
d_3 = a*x_3 + y_3
#println("Tutti gli elementi per N = 10 sono corretti: $(all(d_1 .== 7.4))")    
    # .== compara ogni elemento
    # all verifica che tutte le comparazioni siano true, se così sputa fuori "true" altrimenti "false"
#println("Tutti gli elementi per N = 10^6 sono corretti: $(all(d_2 .== 7.4))")
#println("Tutti gli elementi per N = 10^8 sono corretti: $(all(d_3 .== 7.4))")
#facendo girare il codice il test sulle componenti definiva: "false".
#questo perché 0.1 e 7.1 in Float64, che qui Julia utilizza di default, non hanno rappresentazione finita in binario
#soluzione è usare all(isapprox.(d, 7.4; atol=1e-12)) con appunto tolleranza più larga paragonata alla precisione di un Float64.
#altra soluzione è usare BigFloat come tipo per i decimali. Es: x = fill(big(0.1), n). Alta precisione, ma lento e pesante per N grandi!!!
# oppure razionali (1//10, 71//10, 74//10) per una verifica “esatta” ma poco performante. Ogni elemento non è più un singolo numero in hardware, ma una coppia “numeratore/denominatore” arbitrariamente grandi. Anche qui, super preciso, ma costosissimo in termini di tempo e memoria.
println("Tutti gli elementi per N = 10 sono corretti: $(all(isapprox.(d_1, 7.4; atol=1e-12)))")
println("Tutti gli elementi per N = 10^6 sono corretti: $(all(isapprox.(d_2, 7.4; atol=1e-12)))")
println("Tutti gli elementi per N = 10^8 sono corretti: $(all(isapprox.(d_3, 7.4; atol=1e-12)))")
```
# 2) $\vec{d}=a\vec{x} + \vec{y}$&nbsp;&nbsp;&nbsp;&nbsp;with C
```c
#include <stdio.h>
#include <stdlib.h>
#include <math.h>

#define EPSILON 1e-12

int main() {
 long vector_sizes[] = {10, 1000000, 100000000};
    int num_vector_sizes = 3;
    for (int idx = 0; idx < num_vector_sizes; ++idx) {
        long N = vector_sizes[idx];
        // Alloca memoria per i vettori x, y e d (lunghezza N ciascuno)
        double *x = malloc(N * sizeof(double));
        double *y = malloc(N * sizeof(double));
        double *d = malloc(N * sizeof(double));
        if (x == NULL || y == NULL || d == NULL) {
            fprintf(stderr, "Allocazione memoria fallita per N = %ld\n", N);
            return 1;
        }
        // Inizializza x con 0.1 e y con 7.1
        for (long i = 0; i < N; ++i) {
            x[i] = 0.1;
            y[i] = 7.1;
        }
        // Calcola d = a*x + y con lo scalare a = 3
        double a = 3.0;
        for (long i = 0; i < N; ++i) {
            d[i] = a * x[i] + y[i];
        }
        // Verifica che tutti gli elementi di d siano circa uguali a 7.4 (con tolleranza assoluta)
        double expected_val = 7.4;
        int all_correct_vec = 1;
        for (long i = 0; i < N; ++i) {
            // Controllo differenza assoluta: |d[i] - expected_val| <= EPSILON
            if (fabs(d[i] - expected_val) > EPSILON) {
                all_correct_vec = 0;
                break;
            }
        }
        // Stampa "true" se tutti gli elementi sono corretti entro la tolleranza, altrimenti "false"
        printf("%s\n", all_correct_vec ? "true" : "false");
        // Libera la memoria allocata
        free(x);
        free(y);
        free(d);
    }

    return 0;
}
```



# 3.1) $C = AB \Rightarrow c_{ij} = \sum_{k=1}^{N} a_{ik} \, b_{kj}$&nbsp;&nbsp;&nbsp;&nbsp;with Julia
```julia
n_1 = 10
n_2 = 100
n_3 = 10000
A_1 = fill(3.0, n_1, n_1)
#fill(3, n, n) crea Matrix{Int}; moltiplicando con Matrix{Float64}, ovvero B, Julia deve convertire, creando copie temporanee. Meglio tipizzare da subito!
#quindi fill(3.0, n, n)
B_1 = fill(7.1, n_1, n_1)
C_1 = A_1 * B_1
A_2 = fill(3.0, n_2, n_2)
B_2 = fill(7.1, n_2, n_2)
C_2 = A_2 * B_2
A_3 = fill(3.0, n_3, n_3)
B_3 = fill(7.1, n_3, n_3)
C_3 = A_3 * B_3
# Nota: per N=10000 ogni matrice Float64 ≈ 0.8 GB; A,B,C ~ 2.4+ GB totali.
# Calcolo O(N^3): molto pesante; farlo solo se la macchina lo consente.
#credo ci sia un piccolo refuso nella consegna, gli elementi di C = N*21.3
#println("Tutti gli elementi di C per N = 10 sono corretti: ",(all(isapprox.(C_1, 21.3; rtol=1e-12)))) -> false
#println("Tutti gli elementi di C per N = 100 sono corretti: ",(all(isapprox.(C_2, 21.3; rtol=1e-12)))) -> false
#println("Tutti gli elementi di C per N = 10000 sono corretti: ",(all(isapprox.(C_3, 21.3; rtol=1e-12)))) -> false
#uso rtol e non atol, perché moltiplico un Float64 che ha errore sulla 15'esima ad N, e quindi dovrei variare di volta in volta il paragone con atol
# isapprox usa: |x - y| ≤ atol + rtol*max(|x|,|y|)
# qui mettiamo rtol=1e-12 (atol implicito = 0), così la tolleranza scala con N
println("Tutti gli elementi di C per N = 10 sono corretti: ",(all(isapprox.(C_1, 21.3*n_1; rtol=1e-12))))
println("Tutti gli elementi di C per N = 100 sono corretti: ",(all(isapprox.(C_2, 21.3*n_2; rtol=1e-12))))
println("Tutti gli elementi di C per N = 10000 sono corretti: ",(all(isapprox.(C_3, 21.3*n_3; rtol=1e-12))))
```
# 3.1) $C = AB \Rightarrow c_{ij} = \sum_{k=1}^{N} a_{ik} \, b_{kj}$&nbsp;&nbsp;&nbsp;&nbsp;with C
```c
#include <stdio.h>
#include <stdlib.h>
#include <math.h>

#define EPSILON 1e-12

int main() {
    int matrix_sizes[] = {10, 100, 10000};
    int num_matrix_sizes = 3;
    for (int idx = 0; idx < num_matrix_sizes; ++idx) {
        int N = matrix_sizes[idx];
        // Alloca memoria per le matrici A, B e C (dimensione N x N ciascuna)
        double *A = malloc((size_t)N * N * sizeof(double));
        double *B = malloc((size_t)N * N * sizeof(double));
        double *C = malloc((size_t)N * N * sizeof(double));
        if (A == NULL || B == NULL || C == NULL) {
            fprintf(stderr, "Allocazione memoria fallita per N = %d\n", N);
            return 1;
        }
        // Inizializza A e B con costanti 3.0 e 7.1 rispettivamente
        long total_elements = (long)N * N;
        for (long i = 0; i < total_elements; ++i) {
            A[i] = 3.0;
            B[i] = 7.1;
        }
        // Calcola la moltiplicazione C = A * B
        for (int i = 0; i < N; ++i) {
            for (int j = 0; j < N; ++j) {
                double sum = 0.0;
                for (int k = 0; k < N; ++k) {
                    sum += A[i * N + k] * B[k * N + j];
                }
                C[i * N + j] = sum;
            }
        }
        // Verifica che tutti gli elementi di C siano circa uguali a N * 21.3 (con tolleranza relativa)
        double expected_value = N * 21.3;
        int all_correct = 1;
        for (long i = 0; i < total_elements; ++i) {
            double c_val = C[i];
            // Controllo differenza relativa: |c_val - expected| <= EPSILON * max(|c_val|, |expected|)
            if (fabs(c_val - expected_value) > EPSILON * fmax(fabs(c_val), fabs(expected_value))) {
                all_correct = 0;
                break;
            }
        }
        // Stampa "true" se tutti gli elementi sono corretti entro la tolleranza, altrimenti "false"
        printf("%s\n", all_correct ? "true" : "false");
        // Libera la memoria allocata
        free(A);
        free(B);
        free(C);
    }
    return 0;
}
```

# Answers 
## i) Did you find any problems in running the codes for some N. If so, do you have an idea why?
Yes. The main difficulty arose in the third exercise (matrix multiplication) for $N = 10000$.  

- **Time complexity:** matrix multiplication has $O(N^3)$ cost. For $N = 10000$, this is on the order of $10^{12}$ operations, which is infeasible on a standard machine.  
- **Memory requirements:** each \(10000 \times 10000\) `Float64` matrix needs about 0.8 GB. Since we allocate three matrices (A, B, C), the total memory usage is ~2.4 GB, which can easily exhaust RAM.  

Because of this, the program became extremely slow and practically unusable for that size.
## ii) Where you able to test correctly the sum and product of points 1-3? If so, how? If not, what was the problem?
At first, no. Direct comparison using `==` (in Julia) or equality tests in C failed because floating-point numbers do not represent decimal values like 0.1 or 7.1 exactly in binary. This leads to small rounding errors.  

To solve this:  
- In **Julia**, I used `isapprox` with a relative tolerance (`rtol=1e-12`), so the comparison scales with the magnitude of the numbers.  
- In **C**, I implemented the same logic using a relative error check:  
  $|x - y| \leq \varepsilon \cdot \max(|x|, |y|)$ 
  with $\varepsilon = 10^{-12}$.  

With these modifications, all tests passed correctly.
 
