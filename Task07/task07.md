# Task07: unit test for daxpy code (task3_2.jl in task03 dir)
```julia
#julia test_task07.jl

# Il testing è l’esecuzione controllata di un programma con casi di prova per rilevare difetti e verificare la conformità ai requisiti.
# Un caso di test = (precondizioni, input, oracolo). 
# L’oracolo è la regola che definisce l’esito atteso; il test passa se l’output osservato coincide con l’atteso (entro tolleranze, se necessario). 
# Limite: il testing può mostrare la presenza di bug, non dimostrarne l’assenza.
#
# Obiettivi: verificare correttezza, prevenire regressioni, documentare il
# comportamento atteso, .....
#
# Per questa Task07: unit test mirati su daxpy (d = a·x + y) e su I/O .dat;
#
# Unit test, sono un tipo di testing 
# Un unit test verifica in isolamento l’unità minima di software (tipicamente una funzione/metodo):
# date precondizioni e input, controlla che l’unità produca l’output atteso (oracolo).
# Deve essere rapido, deterministico e indipendente da risorse esterne (deve dipendere solo dagli input, non da file, rete, clock ecc.).
#
# Quindi andiamo ad inserire unità di test nel daxpy code generato per la task03
#
# Nel nostro caso andiamo a creare un piccolo file di test che chiameremo daxpy_lib.jl (ovvero questo stesso file!)
# dove definiamo solo la funzione daxpy con solo la funzione matematica pura:


# piccola sottigliezza per la funzione qui sotto: a è indicizzato come tipo Real, quindi a::Real ci dice che a deve essere qualsiasi sottotipo di Real (Int64, Float64.....)
# ma in x::AbstractVector{<:Real} non utilizziamo però la sintassi AbstractVector{Real}
# I tipi singoli (come Int64) sono sottotipi → quindi Int64 <: Real.
# I tipi parametrici (come Vector{T}) sono invarianti → quindi Vector{Int64} <: Vector{Real} è falso!
# Se hai un tipo parametrico Container{T}, il fatto che S <: T non implica che Container{S} <: Container{T}.
# Quindi: anche se Int64 <: Real, non vale Vector{Int64} <: Vector{Real}.
# Questo è il senso di invarianza: la relazione di sottotipo non “propaga” automaticamente ai tipi parametrici.
# Questo è necessario per prevenire corruzione di memoria.


function daxpy(a::Real, x::AbstractVector{<:Real}, y::AbstractVector{<:Real}) 
    
    length(x) == length(y) || throw(DimensionMismatch("x e y have different lengths!!!")) #scorciatoia sintattica! (paragono x ad y -> restituisce true o false)
    # Se la condizione a sinistra è true, Julia non valuta la parte a destra (quindi non lancia niente).
    # Se è false, allora esegue la parte a destra → cioè throw(DimensionMismatch(...)
    # Se no dovrei scrivere:
        # if length(x) != length(y)
            # throw(DimensionMismatch(...)
        #end
    return a .* Float64.(x) .+ Float64.(y)
end



#########################
# TEST per task3_2.jl  #
#########################
using Test   #Importa il framework di test standard di Julia (modulo Test), da cui arrivano le macro @test, @testset, @test_throws, ecc.

# utilità: scrive un vettore .dat (un numero per riga)
function write_vector(path::AbstractString, v::AbstractVector{<:Real})
    open(path, "w") do f
        for el in v
            println(f, el)
        end
    end
end

# --- Caso OK: N coerente, vettori coerenti ---
# @testset: raggruppa i test con un titolo.
@testset "task3_2.jl - caso OK" begin
    mktempdir() do dir
        N = 4
        prefix = joinpath(dir, "vector_")
        xp = "$(prefix)N$(N)_x.dat"
        yp = "$(prefix)N$(N)_y.dat"

        write_vector(xp, fill(0.1, N))
        write_vector(yp, fill(7.1, N))

        cfg = joinpath(dir, "config.yaml")
        open(cfg, "w") do f
            println(f, "N: $N")
            println(f, "scalar_a: 3")
            println(f, "vector_x_path: \"$xp\"")
            println(f, "vector_y_path: \"$yp\"")
            println(f, "prefix_output: \"$prefix\"")
        end

        # eseguo il programma originale come processo esterno
        script = abspath(joinpath(@__DIR__, "task3_2.jl"))
        cmd = `julia $script $cfg`

        # deve uscire con exit code 0
        run(pipeline(cmd, stdout=devnull, stderr=devnull))

        # verifica file di output e valori
        dpath = "$(prefix)N$(N)_d.dat"
        @test isfile(dpath)
        dvals = parse.(Float64, readlines(dpath))
        @test length(dvals) == N
        @test all(isapprox.(dvals, 7.4; rtol=1e-12))
    end
end

# --- Caso ERRORE: N del config NON coerente con i file ---
@testset "task3_2.jl - caso errore (exit != 0)" begin
    mktempdir() do dir
        Nfiles = 2          # vettori con 2 elementi
        Ncfg   = 3          # nel config metto 3 -> DEVE fallire
        prefix = joinpath(dir, "vector_")
        xp = "$(prefix)N$(Nfiles)_x.dat"
        yp = "$(prefix)N$(Nfiles)_y.dat"

        write_vector(xp, fill(0.1, Nfiles))
        write_vector(yp, fill(7.1, Nfiles))

        cfg = joinpath(dir, "config_bad.yaml")
        open(cfg, "w") do f
            println(f, "N: $Ncfg")
            println(f, "scalar_a: 3")
            println(f, "vector_x_path: \"$xp\"")
            println(f, "vector_y_path: \"$yp\"")
            println(f, "prefix_output: \"$prefix\"")
        end

        script = abspath(joinpath(@__DIR__, "task3_2.jl"))
        cmd = `julia $script $cfg`

        # task3_2.jl fa exit(1) -> run deve lanciare ProcessFailedException
        #@test_throws ProcessFailedException ... controlla proprio che run(...) sollevi quell’eccezione (exit code diverso da 0).
        @test_throws ProcessFailedException run(pipeline(cmd, stdout=devnull, stderr=devnull))
    end
end
```
