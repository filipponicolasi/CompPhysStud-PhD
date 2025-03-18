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
    with open("output.txt", "a") as file:
        file.write(f"N = {N}, Time: {elapsed_time:.6f} seconds, {test_result}\n")
        file.write("d = [ " + " ".join(f"{val:.2f}" for val in d[:10]) + " ... ]\n\n")

    return d

# Clear the output file before running
with open("output.txt", "w") as file:
    file.write("Vector Sum Results\n==================\n")

# Test for different N values
for N in [10, 10**6, 10**8]:
    vector_sum(N)

print("Computation completed. Check 'output.txt' for results.")
```
