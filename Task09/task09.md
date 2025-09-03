# Task09
## Preliminary considerations
To execute `task09.jl` I implemented a configuration file that parametrizes:
- the number of elements (`N`),
- the number of elements per chunk (`chunksize`)
### config_file:
```yaml
N : 1000000 #if you change N remeber to change N also in the input strings here!
chunksize : 1000
scalar_a : 3
vector_x_path : "vector_N1000000_x.dat" #in the actual dir
vector_y_path : "vector_N1000000_y.dat" #in the actual dir
prefix_output : "vector_N" #in the actual dir
 ```
In this task we parallelize the DAXPY calculation by creating threads inside a single process.  
Each thread executes a subset (chunk) of the iterations.  
Since we are working in Julia, we cannot use the **OpenMP** model (used in C/C++/Fortran); instead, we use the native `Base.Threads` module, which provides equivalent constructs.

Furthermore, we cannot set the number of threads in the config file, because Julia cannot change it at runtime.  
Therefore, we must set it before running the program; a quick option is to pass it on the command line:
```bash
julia -t <number_of_threads> task09.jl <config_file>
```

### Hardware notes
For the experiments I use an **Intel i7-14700**, which has 20 physical cores:
- **8 performance cores (P-cores)** supporting hyper-threading → 16 logical threads  
  Base: ~2.10 GHz, Turbo: ~5.3–5.4 GHz
- **12 efficiency cores (E-cores)** → 12 logical threads  
  Base: ~1.50 GHz, Turbo: ~4.2 GHz

In total this CPU exposes **28 logical threads**.

### Methodology
To compare the parallel method with the serial one, we will:
- define a **serial DAXPY calculation** (based on Task03) that serves as the reference baseline;
- implement the **parallel version** using Julia threads, ensuring both versions share the same memory layout and I/O structure;
- measure the **total execution time** of the computation kernel (a single overall timing is sufficient);
- verify correctness by comparing the parallel result with the serial reference.
