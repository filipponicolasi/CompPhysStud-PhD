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

