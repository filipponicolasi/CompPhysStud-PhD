# Guide: Writing and Running a Python Program on AlmaLinux 9 with Docker

This guide will help you create, run, and save the output of a simple Python program that computes the scalar product `z = a*x + y` and writes the result to a text file.

## Step 1: Create the Python Script  
Open the terminal and use `nano` to create the file `program.py`:
```sh
nano program.py
```
Now copy and paste the following code into the file:

```python
# Define values
a = 3.0  # Scalar value
N = 20   # Dimension of vectors

# Initialize vectors
x = [1.0] * N
y = [4.0] * N

# Compute z = a * x + y
z = [a * x[i] + y[i] for i in range(N)]

# Write the result to a text file
with open("output.txt", "w") as file:
    file.write("z = [ " + " ".join(f"{val:.2f}" for val in z) + " ]\n")

print("Computation completed. Check output.txt for results.")
```

Save the file:  
- Press **CTRL + X** to exit `nano`.  
- Press **Y** to confirm saving the file.  
- Press **Enter** to save it with the same name (`program.py`).

## Step 2: Run the Python Program  
Run the program using the following command:
```sh
python3 program.py
```
This will generate an output file named `output.txt`.

## Step 3: View the Result  
To see the contents of the `output.txt` file, use the command:
```sh
cat output.txt
```
You should get an output similar to this:
```
z = [ 7.00 7.00 7.00 7.00 7.00 7.00 7.00 7.00 7.00 7.00 7.00 7.00 7.00 7.00 7.00 7.00 7.00 7.00 7.00 7.00 ]
```

## Conclusion  
You have successfully created and executed a Python program on AlmaLinux 9 inside Docker!   
Now you can modify it to experiment with different values or save it to GitHub.


