
```julia

using YAML 


if length(ARGS) != 1
    println("Usage: julia task3_2.jl <config_file>")
    exit(1)
end
config_file = YAML.load_file(ARGS[1]) 
a = config_file["scalar_a"]
xpath = config_file["vector_x_path"]
ypath = config_file["vector_y_path"]
prefix = config_file["prefix_output"]
N = config_file["N"]
x = Float64[]    
y = Float64[]
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
if length(x) != length(y) && length(y) != N #piccola modifica per tenere più ordine
    println("Error: Vectors must have the same length and equal to N")
    exit(1)
end

#andiamo ad implementare qui i chunks


#funzione intera in broadcasting:
d = a .* x .+ y

output = "$(prefix)$(N)_d.dat"
open(output, "w") do f
    for i in d
        println(f, i)
    end
end


# numero di chunk
chunksize = 1000
Nchunks = cld(N , chunksize)
d_chunk = Vector{Float64}(undef, N) #inizializzo il vettore d_chunk

for i in 0:(Nchunks -1)
    start = i * chunksize + 1
    stop = (i + 1) * chunksize
    if stop > N
        stop = N
    end
    
    for j in start:stop
        d_chunk[j] = a * x[j] + y[j]
    end
end


# inserisco i risultati in file .dat

output1 = "$(prefix)$(N)_d.dat"
open(output1, "w") do f
    for i in d
        println(f, i)
    end
end
output2 = "$(prefix)$(N)_d_chunk.dat"
open(output2, "w") do f
    for i in d_chunk
        println(f, i)
    end
end 

#confronto i due risultati
# ricorda che isapprox qua è in broadcasting e dà boleani che di volta in volta vengono valutati da all()
# non si crea in memoria un array di booleani, ma si valuta uno per uno, quindi alla fine all restituisce true se son tutti true
if differenceall(isapprox.(d, d_chunk; rtol=0.0, atol=0.0)) == true # ho posto tolleranze per fare più prove (ma in questo caso il risultato è sempre esatto)
    println("the two vectors are equal")
else
    println("the two vectors are not equal")
end 
```
