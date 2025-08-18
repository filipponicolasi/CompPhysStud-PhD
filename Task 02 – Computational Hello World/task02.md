# 1) $\vec{d}=a\vec{x} + \vec{y}$&nbsp;&nbsp;&nbsp;&nbsp;with Julia


```julia
a = 3
n_1 = 10
n_2 = 10^6
n_3 = 10^8
x_1 = fill(0.1, n_1)
x_2 = fill(0.1, n_2)
x_3 = fill(0.1, n_3)
y_1 = fill(7.1, n_1)
y_2 = fill(7.1, n_2)
y_3 = fill(7.1, n_3)
d_1 = a*x_1 + y_1
d_2 = a*x_2 + y_2
d_3 = a*x_3 + y_3
#println("Tutti gli elementi per N = 10 sono corretti: $(all(d_1 .== 7.4))")    
    # .== compara ogni elemento
    # all verifica che tutte le comparazioni siano true, se così sputa fuori "true" altrimenti "false"
#println("Tutti gli elementi per N = 10^6 sono corretti: $(all(d_2 .== 7.4))")
#println("Tutti gli elementi per N = 10^8 sono corretti: $(all(d_3 .== 7.4))")
#facendo girare il codice il test sulle componenti definiva: "false".
#questo perché 0.1 e 7.1 in Float64, che qui Julia utilizza di default, non hanno rappresentazione finita in binario
#soluzione è usare all(isapprox.(d, 7.4; atol=1e-12)) con appunto tolleranza più larga paragonata alla precisione di un Float64.
#altra soluzione è usare BigFloat come tipo per i decimali. Es: x = fill(big(0.1), n). Alta precisione, ma lento e pesante per N grandi!!!
# oppure razionali (1//10, 71//10, 74//10) per una verifica “esatta” ma poco performante. Ogni elemento non è più un singolo numero in hardware, ma una coppia “numeratore/denominatore” arbitrariamente grandi. Anche qui, super preciso, ma costosissimo in termini di tempo e memoria.
println("Tutti gli elementi per N = 10 sono corretti: $(all(isapprox.(d_1, 7.4; atol=1e-12)))")
println("Tutti gli elementi per N = 10^6 sono corretti: $(all(isapprox.(d_2, 7.4; atol=1e-12)))")
println("Tutti gli elementi per N = 10^8 sono corretti: $(all(isapprox.(d_3, 7.4; atol=1e-12)))")
```
