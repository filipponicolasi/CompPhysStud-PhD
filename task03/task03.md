# 1) Task3.1 with julia
```julia
if length(ARGS) == 2 #lenght si modula in base al tipo, con array ti dà il numero di oggetti nell'array
    N = parse(Int64, ARGS[1])
    name_x = "$(ARGS[2])N$(ARGS[1])_x.dat"
    name_y = "$(ARGS[2])N$(ARGS[1])_y.dat"
    x = fill(0.1 , N)
    y = fill(7.1 , N)
    open(name_x , "w") do f
    #'do-block': f rappresenta il collegamento al file: è lì che stai scrivendo.
    #È una scorciatoia comoda per non dimenticarsi di scrivere close(f).
        for i in x    #l'indice cammina sugli elementi dell'array
            write(f, string(i, "\n")) #converto in stringa (perché stampo stringhe) e aggiungo newline
        end
    end
    open(name_y , "w") do f
        for i in y
            write(f, string(i, "\n"))
        end
    end
else
    println("Error: invalid number of arguments. Compile as: julia task3_1.jl <N> </path/to/my/outputdir/vector_>")
    exit(1)
end
```
# 2) Task3.2 with julia
### config_file
```yaml
N : 10
scalar_a : 3
vector_x_path : "/home/pipett/esercizi_julia/task03/vector_N10_x.dat"
vector_y_path : "/home/pipett/esercizi_julia/task03/vector_N10_y.dat"
prefix_output : "/home/pipett/esercizi_julia/task03/vector_N"
```
```julia
using YAML #pacchetto per leggere file YAML

if length(ARGS) != 1
    println("Usage: julia task3_2.jl <config_file>")
    exit(1)
end

config_file = YAML.load_file(ARGS[1]) #funzione per leggere il file config da YAML
#Quando fai parsing di un file YAML con YAML.load_file, Julia ti restituisce un dizionario (Dict).
#Un dizionario è un oggetto che contiene chiavi (sempre stringhe), a cui sono associati valori.
#Il tipo è Dict{string, any}, dove any è il tipo dei valori (può essere qualsiasi cosa, YAML permette numeri, stringhe, liste, ecc.)
#In questo Dict le chiavi sono: scalar_a per lo scalare, vector_x_path per il path del file contenente il vettore x, vector_y_path per y.
a = config_file["scalar_a"]
#nel file configurazione a è un intero, quindi julia parsa ad Int64. Non crea problemi nella moltiplicazione successiva (qui sotto)
#perché usi operazioni in bradcasting puro (.*, .+) e julia promuove a in Float64, automaticamente, un'unica volta.
xpath = config_file["vector_x_path"]
ypath = config_file["vector_y_path"]
prefix = config_file["prefix_output"]
N = config_file["N"]
#qui ho estratto le chiavi nel config_file, per leggibilità e maggior comprensione del codice
x = Float64[]    # array vuoto d float64
y = Float64[]
open(xpath, "r") do f
#do f è la scorciatoia che ti permette di associare all'apertura del file f e di chiudere il file automaticamente
#ovviamente "r" sta per read
    for i in eachline(f)
#eachline è un iteratore sulle righe del file, qualcosa che, ogni volta che lo interroghi, ti dà la riga successiva del file
        push!(x, parse(Float64, i))
#push!(array, elemento) serve per aggiungere un nuovo elemento in fondo a un array.
#Il ! alla fine del nome significa che la funzione modifica l’oggetto esistente (in questo caso l’array)
#è come dire: 'spingi in fondo ad x, chi? L'elemento della riga-iesima che dev'essere parsato come Float64 (leggo stringhe da un file)
    end
end
open(ypath, "r") do f
    for i in eachline(f)
        push!(y, parse(Float64, i))
    end
end
if length(x) == length(y) == N
    d = a .* x .+ y
    output = "$(prefix)$(N)_d.dat"
    open(output, "w") do f
        for i in d
                println(f, i)
        end
    end
else
    exit(1)
end
```
# 3) makefile
### makefile to create the two vectors and perform the daxpy
```make
# Parametri (sovrascrivibili: make N=20 PREFIX=/tmp/vector_)
N       ?= 10
PREFIX  ?= /home/pipett/esercizi_julia/task03/vector_
CONFIG  ?= config_file.yaml
RESULT  ?= $(PREFIX)N$(N)_d.dat

# Target di default
all: vectors daxpy

# all è un target phony (non è un file vero).
# Non “si ricrea” mai: non ha ricetta né file.
# Significa solo: “quando chiedo make (cioè all), assicurati che vectors e daxpy siano aggiornati”.
# Quindi make non confronta timestamp per all, ma va direttamente a verificare/aggiornare le sue dipendenze.

# 1) Genera i vettori x e y
vectors: $(PREFIX)N$(N)_x.dat $(PREFIX)N$(N)_y.dat
#Anche vectors è un contenitore (phony) senza ricetta.
# Non “si ricrea” lui: fa sì che quei due file (…_x.dat e …_y.dat) siano aggiornati.
# Qui sì, su quei file si applica la logica dei timestamp.

$(PREFIX)N$(N)_x.dat $(PREFIX)N$(N)_y.dat: task3_1.jl
        julia task3_1.jl $(N) $(PREFIX)
# Questa è la vera regola che può essere eseguita.
# Timestamp rule: se uno dei due file non esiste oppure è più vecchio di task3_1.jl, esegui la ricetta (che genera entrambi).

# 2) Esegui daxpy (produce RESULT)
daxpy: $(RESULT)
# daxpy è phony, serve solo a dire: “assicura che $(RESULT) sia aggiornato”.
$(RESULT): task3_2.jl $(CONFIG) $(PREFIX)N$(N)_x.dat $(PREFIX)N$(N)_y.dat
        julia task3_2.jl $(CONFIG)
# Se $(RESULT) non esiste o è più vecchio di una qualsiasi di queste dipendenze, esegui la ricetta.

# Pulizia
.PHONY: all vectors daxpy clean
#.PHONY definisce i target fittizi, .PHONY = dice a make che quei target non sono file veri, ma vanno sempre eseguiti.

clean:
        rm -f $(PREFIX)N$(N)_x.dat $(PREFIX)N$(N)_y.dat $(RESULT)
# Rimuove (senza mostrare errori se i file non esitono (-f))

# per lanciare il makefile basta digitare make da terminale
# per default make cerca i file che si chiamano esattamente makefile, Makefile o GNUmakefile
# se il nome è diverso lancia il makefile con:
#   make -f <nome del makefile>
#
# Esempi:
#   make            : esegue solo il primo target del makefile (all) e quindi i suoi sottotarget.
#   make clean all  : esegue prima clean, poi all
```
