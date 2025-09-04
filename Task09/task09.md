# Task09
## Preliminary considerations
To execute `task09.jl` I implemented a configuration file that parametrizes:
- the number of elements (`N`),
- the number of elements per chunk (`chunksize`)
### config_file:
```yaml
N : 100000000 #if you change N remeber to change N also in the input strings here!
chunksize : 1000
scalar_a : 3
vector_x_path : "vector_N100000000_x.dat" #in the actual dir
vector_y_path : "vector_N100000000_y.dat" #in the actual dir
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
- implement the **parallel version** using Julia threads, ensuring both versions share more or less the same memory layout;
- measure the **total execution time** of the computation kernel (a single overall timing is sufficient);
- verify correctness by comparing the parallel result with the serial reference.

## Serial DAXPY calculation (serial_DAXPY_calculation.jl)

For the serial DAXPY I use **broadcasting**. To measure runtime I use the `@elapsed` macro, which returns the wall-clock time for the wrapped expression. In this setup, `@elapsed` includes not only the arithmetic but also the **allocation** of `d` and the **one-time JIT compilation** cost (Julia compiles the broadcasting expression on first use, then reuses the compiled code thereafter). To minimize JIT effects, perform a warm-up run; to exclude allocation, pre-allocate `d` and use in-place broadcasting: `d .= a .* x .+ y`.

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
chunksize = config_file["chunksize"]
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

#Broadcasting serial calculation and timing 
t = @elapsed begin
d = a .* x .+ y
end 

# Write the result to a file
output = "$(prefix)$(N)_d_serial.dat"
open(output, "w") do f
        for i in d
            println(f, i)
        end
end 

#Print total compute time
println("Total_compute_time = $(t) s")
```








