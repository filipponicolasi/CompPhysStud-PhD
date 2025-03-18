# Matrix and Vector Operations in Python and C

This repository contains four programs that perform matrix and vector computations in **Python** and **C**, testing their correctness and efficiency.

## **1. Vector Sum (Python & C)**
Computes the vector sum **d = a * x + y**, where:
- `a` is a scalar.
- `x` and `y` are vectors of size **N** (`N = 10, 10^6, 10^8`).
- Values: `a = 3`, `x = 0.1`, `y = 7.1`.

### **Test:**  
- **Python:** Uses `np.allclose()` to verify that all elements of `d` are `7.4`.  
- **C:** Uses a custom function `is_close()` with a tolerance check.

## **2. Matrix Multiplication (Python & C)**
Computes the matrix product **C = A * B**, where:
- `A` and `B` are **NxN** matrices (`N = 10, 100, 10,000`).
- Values: `A = 3`, `B = 7.1`.

### **Test:**  
- **Python:** Uses `np.allclose()` to check if all elements of `C` are `21.3 * N`.  
- **C:** Uses `is_close()` to verify correctness within a numerical tolerance.

## **Performance Considerations**
- Python benefits from **NumPy optimizations** for large arrays.  
- C provides **lower-level control**, but large matrix operations can be slow without optimizations (e.g., OpenMP).  

### **Usage**
Run the Python scripts directly.  
Compile and run C programs using:
```sh
gcc -o program program.c -lm
./program
