# a) Vector sum with julia
## 1- Using for loop
```julia
v = [1.0, 1.0e16, -1.0e16, -0.5]
somma = 0.0
for i in v
    somma += i
end
println(somma)
# ma sta somma dà -0.5 come risultato!

# il motivo è semplice, julia passa quei numeri come Float64, quindi tenta la somma col ciclo for
# Float64 ha 53 bit di precisione (52 di frazione memorizzati + 1 implicito) ≈ 15,95 cifre decimali di precisione relativa; 
# i numeri interi fino a 2^53 (±9007199254740992) sono rappresentabili esattamente.
# a grandezze ~10^16 lo spacing (ULP) è 2: 1.0 + 10^16 == 10^16; quindi 10^16 + 1 viene arrotondato.

# ricorda che la macchina riesce a distinguere solo i 53 bit più significativi di un numero (52 + 1 implicito)
# c'è però un trucco per alcuni numeri: il primo a usarlo è 2^53 stesso, che in binario è 1 seguito da 53 zeri
# ammettendo di normalizzare portando il leading 1 (implicito), NON c'è il problema "53 cifre contro 52":
# la precisione effettiva è di 53 bit grazie al bit implicito.
# non c'è problema alcuno perché abbiamo anche 11 bit di esponente da sfruttare
# in pratica gli zeri finali si "trasferiscono" nell'esponente: 2^53 si scrive semplicemente come 1.0 * 2^53
# tradotto in campi: segno=0, esponente (bias 1023)=10000110100₂, frazione=52 zeri

# per cui alcuni numeri interi, superiori a 2^53, possono comunque essere rappresentati esattamente con questo meccanismo.
# l'importante è che abbiano abbastanza zeri finali (in binario) da far rientrare la parte rimanente nei 53 bit significativi
# non sarà sicuramente il caso di 2^53 + 1 = 10000....00001 con l'1 nella 54ª cifra finale: cade fuori dai 53 bit significativi.
# in questo caso il numero va arrotondato seguendo il formato IEEE 754: round to nearest, ties to even!
# Quindi dobbiamo trovare i due numeri più vicini rappresentabili che saranno -> 1.0000...0000 * 2^53  e  1.0000...0010 * 2^53
# Sono entrambi vicini "uguali" a 2^53+1, quindi ties to even ci dice di scegliere il "pari"
# In binario tra un numero che termina con LSB=0 (pari) e uno che termina con LSB=1 (dispari) (LSB = least significant bit)
# Quindi scegliamo quello con LSB=0 ovvero 2^53

# Per vedere se un numero n > 2^53 è rappresentabile senza arrotondamento in Float64 basterà applicare la regola:
# log2(n) + 1 - v(n) ≤ 53, dove v(n) è il numero di fattori 2 in n  (equivale a usare bit_length(n)=⌊log2 n⌋+1)

# Ora con 10^16 è esattamente la stessa cosa.
# log2(10^16) + 1 - 16 ≤ 53, dove 16 = v(10^16), dato che 10^16 = 2^16 * 5^16 (quindi 16 fattori di 2)
# ≈ 53.15 + 1 - 16 = 38.15 ≤ 53, quindi 10^16 è rappresentabile esattamente in Float64
# ma 10^16 + 1 non lo è, dato che v(10^16 + 1) = 0 (è dispari)
# quindi 10^16 + 1 va arrotondato a 10^16, seguendo la regola vista prima.
# ECCO PERCHÉ DÀ -0.5 COME RISULTATO!
# 10^16 + 1 == 10^16   (in Float64)
# 10^16 - 10^16 = 0
# 0 - 0.5 = -0.5
```
## 2- Using Xsum package
Xsum does not make all computations “perfect”; it specifically computes the sum correctly rounded to Float64—that is, the value you would obtain by performing the summation in infinite precision and then rounding to the nearest Float64. Naturally, the library does not actually use infinite-precision arithmetic. Instead, it implements an algorithm known as exact floating-point summation (developed by Radford Neal). The key idea is that, rather than discarding the bits that cannot fit into a Float64, these bits are accumulated in auxiliary structures so that no contribution is lost. In the end, the algorithm reconstructs the exact base-2 sum—as if all additions had been done with arbitrarily large integers—and then rounds it once to the nearest Float64.
```julia
using Xsum
vec = [1.0, 1.0e16, -1.0e16, -0.5]
somma = xsum(vec)   
println(somma)
```
## 3- Using Kahan summation algorithm
In Julia there is already the algorithm in the package 'KahanSummation'. The Kahan summation algorithm is compensated with the strongest Kahan–Babuška–Neumaier algorithm:
```julia
using KahanSummation
vec = [1.0, 1.0e16, -1.0e16, -0.5]
somma = sum_kbn(vec)           
println(somma)                
```
without the package, only Kahan summation does not work well (it returns -0.5). It has to be implemented with the compensated summ KBN (Kahan–Babuška–Neumaier):
```julia
vec = [1.0, 1.0e16, -1.0e16, -0.5]
function KBN_sum(a)
    somma = 0.0
    c = 0.0
    for x in a
        t = somma + x
        if abs(somma) >= abs(x)
            c += (somma - t) + x
        else
            c += (x - t) + somma
        end
        somma = t
    end
    println(somma + c)
end

KBN_sum(vec)  
```
## Consideration
No, they are not the same. The simplest for loop does not take into account that there is a loss in information for integers higher than $2^{53}$. So the extrabit in the mantissa for non perfectly definite number (like $10^16 + 1$) are lost and the number is round-off using the round to nearest, ties to even rule (in this case $10^{16} + 1 = 10^{16}$).

# b) Code for daxpy mean = 0 std = 1
```julia
# randn(N) restituisce un array di N numeri pseudocasuali indipendenti, ciascuno distribuito secondo una gaussiana standard (µ=0, σ=1).
# L’array risultante in media avrà proprietà vicine a quelle teoriche, ma non perfettamente esatte secondo normale fluttuazione campionaria.
# Più grande è N, più i valori osservati si avvicineranno a quelli teorici.


using Statistics
using Random # quello che ci serve per randn(N)

if length(ARGS) == 2 
    N = parse(Int64, ARGS[1])
    name_x = "$(ARGS[2])N$(ARGS[1])_x.dat"
    name_y = "$(ARGS[2])N$(ARGS[1])_y.dat"
    # ora creo due bei array:
    x = randn(N) 
    y = randn(N)
    open(name_x , "w") do f
        for i in x    
            write(f, string(i, "\n")) 
        end
    end
    open(name_y , "w") do f
        for i in y
            write(f, string(i, "\n"))
        end
    end
    println("vector_N$N_x.dat created with mean $(mean(x)) and std $(std(x))")
    println("vector_N$N_y.dat created with mean $(mean(y)) and std $(std(y))")
else
    println("Error: invalid number of arguments. Compile as: julia task3_1.jl <N> </path/to/my/outputdir/vector_>")
    exit(1)
end
```
