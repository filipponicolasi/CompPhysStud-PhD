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
# 2) $C = AB \Rightarrow c_{ij} = \sum_{k=1}^{N} a_{ik} \, b_{kj}$&nbsp;&nbsp;&nbsp;&nbsp;with Julia
```julia
n_1 = 10
n_2 = 100
n_3 = 10000
A_1 = fill(3.0, n_1, n_1)
#fill(3, n, n) crea Matrix{Int}; moltiplicando con Matrix{Float64}, ovvero B, Julia deve convertire, creando copie temporanee. Meglio tipizzare da subito!
#quindi fill(3.0, n, n)
B_1 = fill(7.1, n_1, n_1)
C_1 = A_1 * B_1
A_2 = fill(3.0, n_2, n_2)
B_2 = fill(7.1, n_2, n_2)
C_2 = A_2 * B_2
A_3 = fill(3.0, n_3, n_3)
B_3 = fill(7.1, n_3, n_3)
C_3 = A_3 * B_3
# Nota: per N=10000 ogni matrice Float64 ≈ 0.8 GB; A,B,C ~ 2.4+ GB totali.
# Calcolo O(N^3): molto pesante; farlo solo se la macchina lo consente.
#credo ci sia un piccolo refuso nella consegna, gli elementi di C = N*21.3
#println("Tutti gli elementi di C per N = 10 sono corretti: ",(all(isapprox.(C_1, 21.3; rtol=1e-12)))) -> false
#println("Tutti gli elementi di C per N = 100 sono corretti: ",(all(isapprox.(C_2, 21.3; rtol=1e-12)))) -> false
#println("Tutti gli elementi di C per N = 10000 sono corretti: ",(all(isapprox.(C_3, 21.3; rtol=1e-12)))) -> false
#uso rtol e non atol, perché moltiplico un Float64 che ha errore sulla 15'esima ad N, e quindi dovrei variare di volta in volta il paragone con atol
# isapprox usa: |x - y| ≤ atol + rtol*max(|x|,|y|)
# qui mettiamo rtol=1e-12 (atol implicito = 0), così la tolleranza scala con N
println("Tutti gli elementi di C per N = 10 sono corretti: ",(all(isapprox.(C_1, 21.3*n_1; rtol=1e-12))))
println("Tutti gli elementi di C per N = 100 sono corretti: ",(all(isapprox.(C_2, 21.3*n_2; rtol=1e-12))))
println("Tutti gli elementi di C per N = 10000 sono corretti: ",(all(isapprox.(C_3, 21.3*n_3; rtol=1e-12))))
```
