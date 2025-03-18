# 1) Python code for vector sum:
```python
import numpy as np
import time

def vector_sum(N, a=3.0):
    """Computes d = a*x + y for vectors of dimension N."""
    x = np.full(N, 0.1)  # x = 0.1 for all elements
    y = np.full(N, 7.1)  # y = 7.1 for all elements
    
    start_time = time.time()  # Start timing
    d = a * x + y  # Compute vector sum
    elapsed_time = time.time() - start_time  # Compute elapsed time

    # Check if all elements of d are equal to 7.4
    expected_value = 7.4
    if np.allclose(d, expected_value, atol=1e-9):  # Allow small numerical errors
        test_result = "Test Passed"
    else:
        test_result = "Test Failed"

    print(f"N = {N}, Time taken: {elapsed_time:.6f} seconds, {test_result}")

    # Save results to a text file
    with open("vector_sum.txt", "a") as file:
        file.write(f"N = {N}, Time: {elapsed_time:.6f} seconds, {test_result}\n")
        file.write("d = [ " + " ".join(f"{val:.2f}" for val in d[:10]) + " ... ]\n\n")

    return d

# Clear the output file before running
with open("vector_sum.txt", "w") as file:
    file.write("Vector Sum Results\n==================\n")

# Test for different N values
for N in [10, 10**6, 10**8]:
    vector_sum(N)

print("Computation completed. Check 'vector_sum.txt' for results.")
```
# 2) C code for vector sum:

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
    FILE *file = fopen("vector_sum.txt", "w");
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
    
    printf("Computation completed. Check 'vector_sum.txt' for results.\n");

    return 0;
}

```
# 3) Python and C code for matrix multiplication:
## python:
```python
import numpy as np
import time

def matrix_multiplication(N):
    """Computes C = AB where A and B are NxN matrices filled with 3 and 7.1, respectively."""

    # Initialize matrices A and B
    A = np.full((N, N), 3.0)   # A filled with 3
    B = np.full((N, N), 7.1)   # B filled with 7.1

    start_time = time.time()  # Start timing

    # Perform matrix multiplication
    C = np.dot(A, B)  # Equivalent to C = AB

    elapsed_time = time.time() - start_time  # Compute elapsed time

    # Check if all elements of C are equal to 21.3
    expected_value = 21.3 * N
    if np.allclose(C, expected_value, atol=1e-9):  # Allow small numerical errors
        test_result = "Test Passed"
    else:
        test_result = "Test Failed"

    print(f"N = {N}, Time taken: {elapsed_time:.6f} seconds, {test_result}")

    # Save results to a text file
    with open("matrix_product.txt", "a") as file:
        file.write(f"N = {N}, Time: {elapsed_time:.6f} seconds, {test_result}\n")
        file.write("First row of C: [ " + " ".join(f"{val:.2f}" for val in C[0, :10]) + " ... ]\n\n")

    return C

# Clear the output file before running
with open("matrix_product.txt", "w") as file:
    file.write("Matrix Multiplication Results\n============================\n")

# Test for different N values
for N in [10, 100, 10000]:
    matrix_multiplication(N)

print("Computation completed. Check 'matrix_product.txt' for results.")
```



## C:
```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include <math.h>

// Function to compute matrix multiplication C = A * B
void matrix_multiplication(int N, FILE *file) {
    // Allocate memory for matrices A, B, and C
    double **A = (double **)malloc(N * sizeof(double *));
    double **B = (double **)malloc(N * sizeof(double *));
    double **C = (double **)malloc(N * sizeof(double *));

    if (A == NULL || B == NULL || C == NULL) {
        printf("Memory allocation failed for N = %d\n", N);
        return;
    }

    for (int i = 0; i < N; i++) {
        A[i] = (double *)malloc(N * sizeof(double));
        B[i] = (double *)malloc(N * sizeof(double));
        C[i] = (double *)malloc(N * sizeof(double));

        if (A[i] == NULL || B[i] == NULL || C[i] == NULL) {
            printf("Memory allocation failed for row %d\n", i);
            return;
        }
    }

    // Initialize matrices A and B
    for (int i = 0; i < N; i++) {
        for (int j = 0; j < N; j++) {
            A[i][j] = 3.0;
            B[i][j] = 7.1;
            C[i][j] = 0.0; // Initialize C with zeros
        }
    }

    // Start timing
    clock_t start = clock();

    // Compute C = A * B
    for (int i = 0; i < N; i++) {
        for (int j = 0; j < N; j++) {
            for (int k = 0; k < N; k++) {
                C[i][j] += A[i][k] * B[k][j];
            }
        }
    }

    // Stop timing
    double elapsed_time = (double)(clock() - start) / CLOCKS_PER_SEC;

    // Verify that all elements of C are equal to 21.3 * N
    double expected_value = 21.3 * N;
    int test_passed = 1; // Assume test passes
    for (int i = 0; i < N; i++) {
        for (int j = 0; j < N; j++) {
            if (fabs(C[i][j] - expected_value) > 1e-9) { // Allow small floating-point errors
                test_passed = 0; // Test failed
                break;
            }
        }
        if (!test_passed) break;
    }

    // Print results to console
    printf("N = %d, Time taken: %.6f seconds, %s\n",
           N, elapsed_time, test_passed ? "Test Passed" : "Test Failed");

    // Write results to file
    fprintf(file, "N = %d, Time: %.6f seconds, %s\n",
            N, elapsed_time, test_passed ? "Test Passed" : "Test Failed");

    fprintf(file, "First row of C: [ ");
    for (int j = 0; j < (N < 10 ? N : 10); j++) { // Print first 10 values
        fprintf(file, "%.2f ", C[0][j]);
    }
    fprintf(file, "... ]\n\n");

    // Free allocated memory
    for (int i = 0; i < N; i++) {
        free(A[i]);
        free(B[i]);
        free(C[i]);
    }
    free(A);
    free(B);
    free(C);
}

int main() {
    // Open the output file
    FILE *file = fopen("matrix_product.txt", "w");
    if (file == NULL) {
        printf("Error opening file!\n");
        return 1;
    }

    fprintf(file, "Matrix Multiplication Results\n============================\n");

    // Run for different values of N
    int N_values[] = {10, 100, 10000};
    for (int i = 0; i < 3; i++) {
        matrix_multiplication(N_values[i], file);
    }

    // Close the file
    fclose(file);

    printf("Computation completed. Check 'matrix_product.txt' for results.\n");

    return 0;
}
```
# Questions
Before answering the questions, I need to specify that, due to limited time, I have relied heavily on ChatGPT to write and understand code. I have started to grasp programming languages a little, but I need much more time to truly understand the code. So, I'm not sure if I did it correctly.

## i) 
Yes, I had issues with matrix computation. Specifically, the C program fails to compute for N=10,000. I looked into it a bit, and as far as I understand, the C code uses a basic triple-nested loop approach, which is not optimized. More importantly, the code does not take advantage of parallel computation with multithreading. In contrast, the Python code does this efficiently using highly optimized numerical libraries (via NumPy).
## ii) 
For the points I was able to compute (all except N=10,000 in the matrix calculation in C), yes.
- In Python, I used **`np.allclose()`**, which compares two arrays element by element with a given **tolerance** to handle floating-point precision errors.  
- In C, I created a **custom function (`is_close()`)** that uses `fabs(a - b) < TOLERANCE` to check if two floating-point numbers are **close enough** to be considered equal.  
