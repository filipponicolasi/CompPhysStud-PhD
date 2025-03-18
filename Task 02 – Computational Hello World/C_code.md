```c

#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <math.h>

// Function to compute d = a*x + y
void vector_sum(long long N, double a, FILE *file) {
    double *x = (double *)malloc(N * sizeof(double)); // Allocate memory for x
    double *y = (double *)malloc(N * sizeof(double)); // Allocate memory for y
    double *d = (double *)malloc(N * sizeof(double)); // Allocate memory for d

    if (x == NULL || y == NULL || d == NULL) {
        printf("Memory allocation failed for N = %lld\n", N);
        return;
    }

    // Initialize x and y
    for (long long i = 0; i < N; i++) {
        x[i] = 0.1;
        y[i] = 7.1;
    }

    // Start timing
    clock_t start = clock();

    // Compute d = a*x + y
    for (long long i = 0; i < N; i++) {
        d[i] = a * x[i] + y[i];
    }

    // Stop timing
    double elapsed_time = (double)(clock() - start) / CLOCKS_PER_SEC;

    // Check if all elements of d are equal to 7.4
    int test_passed = 1;  // Assume test passes
    for (long long i = 0; i < N; i++) {
        if (fabs(d[i] - 7.4) > 1e-9) { // Allow small floating-point errors
            test_passed = 0;  // Test failed
            break;
        }
    }

    // Print results to console
    printf("N = %lld, Time taken: %.6f seconds, %s\n",
           N, elapsed_time, test_passed ? "Test Passed" : "Test Failed");

    // Write results to file
    fprintf(file, "N = %lld, Time: %.6f seconds, %s\n",
            N, elapsed_time, test_passed ? "Test Passed" : "Test Failed");

    fprintf(file, "d = [ ");
    for (int i = 0; i < (N < 10 ? N : 10); i++) {  // Print first 10 values
        fprintf(file, "%.2f ", d[i]);
    }
    fprintf(file, "... ]\n\n");

    // Free allocated memory
    free(x);
    free(y);
    free(d);
}

int main() {
    // Open the output file
    FILE *file = fopen("output.txt", "w");
    if (file == NULL) {
        printf("Error opening file!\n");
        return 1;
    }

    fprintf(file, "Vector Sum Results\n==================\n");

    // Run for different values of N
    long long N_values[] = {10, 1000000, 100000000};
    for (int i = 0; i < 3; i++) {
        vector_sum(N_values[i], 3.0, file);
    }

    // Close the file
    fclose(file);
    
    printf("Computation completed. Check 'output.txt' for results.\n");

    return 0;
}

```
