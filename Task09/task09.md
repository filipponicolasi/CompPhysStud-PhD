# Task09
## Preliminary considerations
To execute all the `task09.jl` tasks I implemented the configuration file as:
### config_file:
```yaml
N : 100000000 #if you change N remeber to change N also in the input strings here!
scalar_a : 3
vector_x_path : "vector_N100000000_x.dat" #in the actual dir
vector_y_path : "vector_N100000000_y.dat" #in the actual dir
prefix_output : "vector_N" #in the actual dir
 ```
In this task we parallelize the DAXPY calculation by creating threads inside a single process.  
Each thread executes a subset of the iterations.  
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
- implement the **parallel version** using Julia threads, ensuring both versions share more or less the same memory layout;
- measure the **total execution time** of the computation kernel (a single overall timing is sufficient);
- verify correctness by comparing the parallel result with the serial reference.

## Serial DAXPY calculation (serial_DAXPY_calculation.jl)

For the serial DAXPY I avoid broadcasting (`d = a .* x .+ y`). Instead, I use the same explicit `for` loop as i will use in the threaded version, so both implementations share the same computational structure (same per-element work and memory access pattern). This makes the timing comparison fair.
Runtime is measured with the `@elapsed` macro, which returns the wall-clock time of the wrapped expression.  
With this setup the timing includes not only the arithmetic but also the memory allocation and the one-time JIT compilation cost, since Julia compiles the daxpy expression on first use and then reuses the compiled code on subsequent runs.

### serial_DAXPY_calculation.jl
```julia
# serial_DAXPY_calculation.jl

using YAML 



# Check for correct number of arguments
if length(ARGS) != 1
    println("Usage: julia <program_name> <config_file>")
    exit(1)
end

#Load configuration from YAML file
config_file = YAML.load_file(ARGS[1]) 
a = config_file["scalar_a"]
xpath = config_file["vector_x_path"]
ypath = config_file["vector_y_path"]
prefix = config_file["prefix_output"]
N = config_file["N"]
x = Float64[]    
y = Float64[]

#read input vectors from files
open(xpath, "r") do f
    for i in eachline(f)
        push!(x, parse(Float64, i))
    end
end
open(ypath, "r") do f
    for i in eachline(f)
        push!(y, parse(Float64, i))
    end
end
if length(x) != length(y) || length(y) != N 
    println("Error: Vectors must have the same length and equal to N")
    exit(1)
end

#Serial calculation and timing 
d_serial = Vector{Float64}(undef, N)
t = @elapsed begin
    for i in 1:N
        d_serial[i] = a * x[i] + y[i]
    end
end

# Write the result to a file
output = "$(prefix)$(N)_d_serial.dat"
open(output, "w") do f
        for i in d_serial
            println(f, i)
        end
end 

#Print total compute time
println("Total_compute_time = $(t) s")
```

## Parallel DAXPY calculation (parallel_DAXPY_calculation.jl)
In Julia, parallel for-loops use the native `Base.Threads` module. If we iterate over `1:N` and prefix the loop with `@threads` (after `using Base.Threads` package), Julia statically partitions the `N` iterations among the threads configured at startup (e.g., `julia -t <num_threads> task09.jl <config_file>`). For DAXPY there is no need to create manual chunks: simply write `@threads for i in 1:N ... end` and Julia will distribute the iterations across the selected threads. 

If we start Julia with 5 threads, the loop of 1000000 iterations is partitioned into 5 contiguous blocks, so each thread executes about $1000000/5 = 200000$ iterations. At the end of the `@threads for ... end` block there is an implicit barrier, so execution continues only after all threads have finished.

Also in this case, `@elapsed` is used in order to include not only the arithmetic but also memory allocation and the one-time JIT compilation cost.
### parallel_daxpy_calculation.jl
```julia
#parallel_DAXPY_calculation.jl

using YAML 
using Base.Threads #for multithreading


# Check for correct number of arguments
if length(ARGS) != 1
    println("Usage: julia <program_name> <config_file>")
    exit(1)
end

#Load configuration from YAML file
config_file = YAML.load_file(ARGS[1]) 
a = config_file["scalar_a"]
xpath = config_file["vector_x_path"]
ypath = config_file["vector_y_path"]
prefix = config_file["prefix_output"]
N = config_file["N"]
x = Float64[]    
y = Float64[]

#read input vectors from files
open(xpath, "r") do f
    for i in eachline(f)
        push!(x, parse(Float64, i))
    end
end
open(ypath, "r") do f
    for i in eachline(f)
        push!(y, parse(Float64, i))
    end
end
if length(x) != length(y) || length(y) != N 
    println("Error: Vectors must have the same length and equal to N")
    exit(1)
end

#Parallel calculation using Threads and timing
d_parallel = Vector{Float64}(undef, N)
t = @elapsed begin
    @threads  for i in 1:N
               d_parallel[i] = a * x[i] + y[i]
              end
end

# Write the result to a file
output = "$(prefix)$(N)_d_parallel.dat"
open(output, "w") do f
        for i in d_chunk
            println(f, i)
        end
end 

#Print total compute time
println("Total_compute_time = $(t) s")
```
### Results
Total compute time for serial calculation:\
$= 0.098 s$
Total compute time for parallel calculation:
1  thread: $= 0.115 s$
2  thread: $= 0.096 s$
3  thread: $= 0.082 s$
4  thread: $= 0.073 s$
5  thread: $= 0.068 s$
6  thread: $= 0.062 s$
7  thread: $= 0.058 s$
8  thread: $= 0.060 s$
9  thread: $= 0.063 s$
10 thread: $= 0.068 s$
10--->20 threads between \\approx 0.060 0.070

With one thread, the `@threads` macro adds setup and waiting overhead, so it’s slower than the plain serial loop. As we increase the thread count, that overhead gets spread out and things speed up—until the program becomes limited by how fast data can be moved to and from memory. At that point (around 5–8 threads, ≈0.06–0.07 s) it plateaus. Beyond that, adding more threads doesn’t help because the memory transfer rate is already maxed out (small timing wiggles can be normal due to scheduling and the mix of P- and E-cores).






