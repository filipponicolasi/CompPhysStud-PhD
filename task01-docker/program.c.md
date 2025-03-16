# Guide: Writing and Running a C Program on AlmaLinux 9 with Docker

This guide will help you create, compile, and run a simple C program to compute the scalar product `z = a*x + y` and save the result to a text file.

## Step 1: Create the Program File  
Open the terminal and use `nano` to create the file `program.c`:
```sh
nano program.c
```
Now copy and paste the following code into the file:

```c
#include <stdio.h>

#define N 20  // Define the size of the vectors

int main() {
    float a = 3.0;  // Scalar value
    float x[N], y[N], z[N];  // Vectors

    // Initialize x with 1s and y with 4s
    for (int i = 0; i < N; i++) {
        x[i] = 1.0;
        y[i] = 4.0;
    }

    // Compute z = a * x + y
    for (int i = 0; i < N; i++) {
        z[i] = a * x[i] + y[i];
    }

    // Write the result to a text file
    FILE *file = fopen("output.txt", "w");
    if (file == NULL) {
        printf("Error opening file!\n");
        return 1;
    }

    fprintf(file, "z = [ ");
    for (int i = 0; i < N; i++) {
        fprintf(file, "%.2f ", z[i]);
    }
    fprintf(file, "]\n");

    fclose(file);
    printf("Computation completed. Check output.txt for results.\n");

    return 0;
}
```

Save the file:  
- Press **CTRL + X** to exit `nano`.  
- Press **Y** to confirm saving the file.  
- Press **Enter** to save it with the same name (`program.c`).

## Step 2: Compile the Program  
Compile the C code using `gcc`:
```sh
gcc program.c -o program
```
If the compilation is successful, an executable file named `program` will be created.

## Step 3: Run the Program  
Run the program using the following command:
```sh
./program
```
This will generate an output file named `output.txt`.

## Step 4: View the Result  
To see the contents of the `output.txt` file, use the command:
```sh
cat output.txt
```
You should get an output similar to this:
```
z = [ 7.00 7.00 7.00 7.00 7.00 7.00 7.00 7.00 7.00 7.00 7.00 7.00 7.00 7.00 7.00 7.00 7.00 7.00 7.00 7.00 ]
```

## Conclusion  
You have successfully created a C program on AlmaLinux 9 inside Docker!
