# Task09: OpenMP
## Preliminary considerations
To execute all the `task09.jl` tasks I implemented the configuration file as:
### config_file:
```yaml
N : 1000000 #if you change N remeber to change N also in the input strings here!
scalar_a : 3
vector_x_path : "vector_N1000000_x.dat" #in the actual dir
vector_y_path : "vector_N1000000_y.dat" #in the actual dir
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
    println("Usage: julia -t <num_threads> <program_name> <config_file>")
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
        for i in d_parallel
            println(f, i)
        end
end 

#Print total compute time
println("Total_compute_time = $(t) s")
```
### Results
Calculations were computed for $N = 10^{6}$

Total compute time for serial calculation: $0.098 s$

Total compute time for parallel calculation: \
1  thread: $= 0.115$ s\
2  thread: $= 0.096$ s\
3  thread: $= 0.082$ s\
4  thread: $= 0.073$ s\
5  thread: $= 0.068$ s\
6  thread: $= 0.062$ s\
7  thread: $= 0.058$ s\
8  thread: $= 0.060$ s\
9  thread: $= 0.063$ s\
10 thread: $= 0.068$ s\
10-->20 threads between $\\approx 0.060 - 0.070$ s

With one thread, the `@threads` macro adds setup and waiting overhead, so it’s slower than the plain serial loop. As we increase the thread count, that overhead gets spread out and things speed up—until the program becomes limited by how fast data can be moved to and from memory. At that point (around 5–8 threads, ≈0.06–0.07 s) it plateaus. Beyond that, adding more threads doesn’t help because the memory transfer rate is already maxed out (small timing wiggles can be normal due to scheduling and the mix of P- and E-cores).

# MPI
In Julia, MPI-based distributed parallelism is provided by the **MPI.jl** package. With MPI you launch P processes of the same program; each process has a rank (an ID from 0 to P−1) and its own private memory.

In our case we use **P = 4** to split the work over $N$ elements. **Rank 0** reads the input vectors `x` and `y`, splits them into contiguous blocks, and sends each block to the corresponding process using **Scatterv**. Every process (Rank 1,2,3) then computes its local result `d_loc = a * x_loc + y_loc`. Finally, **rank 0** collects all local pieces and reconstructs the global vector `d` using **Gatherv**.

MPI is not the same as threads: threads share the same memory while with MPI, processes do not share memory and communicate only by sending messages.
```julia
# MPI_DAXPY_calculation.jl
# Usage:
#   mpirun -np 4 julia --project -t 1 MPI_DAXPY_calculation.jl config_file.yaml

using YAML
using MPI

MPI.Init()
try
    # --- MPI basics ---
    comm  = MPI.COMM_WORLD
    rank  = MPI.Comm_rank(comm)
    nproc = MPI.Comm_size(comm)

    # --- Read config only on rank 0 ---
    N_local = 0
    a_local = 0.0
    xpath   = ""
    ypath   = ""
    prefix  = ""

    if rank == 0
        if length(ARGS) != 1
            println("Usage: mpirun -np <P> julia --project -t 1 MPI_DAXPY_calculation.jl <config_file>")
            MPI.Abort(comm, 1)
        end
        cfg     = YAML.load_file(ARGS[1])
        N_local = cfg["N"]
        a_local = cfg["scalar_a"]
        xpath   = cfg["vector_x_path"]
        ypath   = cfg["vector_y_path"]
        prefix  = cfg["prefix_output"]
    end

    # --- Broadcast scalars to all ranks (functional style) ---
    N = MPI.bcast(N_local, 0, comm)     # Int on every rank
    a = MPI.bcast(a_local, 0, comm)     # Float64 on every rank

    # --- Rank 0 reads input vectors; others keep `nothing` as send-buffer ---
    x_full = nothing
    y_full = nothing
    if rank == 0
        x = Float64[]
        y = Float64[]
        open(xpath, "r") do f
            for ln in eachline(f); push!(x, parse(Float64, ln)); end
        end
        open(ypath, "r") do f
            for ln in eachline(f); push!(y, parse(Float64, ln)); end
        end
        if isempty(x) || isempty(y)
            println("Error: one of the input files is empty")
            MPI.Abort(comm, 1)
        end
        if length(x) != length(y) || length(y) != N
            println("Error: vectors must have the same length and equal to N")
            MPI.Abort(comm, 1)
        end
        x_full = x
        y_full = y
    end

    # --- Block partition (counts) ---
    base = N ÷ nproc
    rmd  = N % nproc
    counts  = [i <= rmd ? base + 1 : base for i in 1:nproc]   # Vector{Int}
    counts32 = Int32.(counts)                                  # MPI.jl expects Int32
    local_n = counts[rank + 1]

    # --- Local buffers + Scatterv ---
    x_loc = Vector{Float64}(undef, local_n)
    y_loc = Vector{Float64}(undef, local_n)
    d_loc = Vector{Float64}(undef, local_n)

    # MPI.jl 0.20.x: Scatterv!/Gatherv! take counts (Int32); no displacements arg
    MPI.Scatterv!(rank == 0 ? x_full : nothing, x_loc, counts32, 0, comm)
    MPI.Scatterv!(rank == 0 ? y_full : nothing, y_loc, counts32, 0, comm)

    # --- Compute-only timing (global = max over ranks) ---
    MPI.Barrier(comm)
    t_local = @elapsed begin
        @inbounds @simd for i in 1:local_n
            d_loc[i] = a * x_loc[i] + y_loc[i]
        end
    end
    t_max = Ref(t_local)
    MPI.Allreduce!(t_max, MPI.MAX, comm)

    # --- Gather result on rank 0 and write to disk ---
    d_full = rank == 0 ? Vector{Float64}(undef, N) : nothing
    MPI.Gatherv!(d_loc, rank == 0 ? d_full : nothing, counts32, 0, comm)

    if rank == 0
        output = "$(prefix)$(N)_d_mpi.dat"
        open(output, "w") do f
            for v in d_full
                println(f, v)
            end
        end
        println("Total_compute_time = $(t_max[]) s")
    end

finally
    MPI.Finalize()
end
```
