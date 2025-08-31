# Task06 — Julia (Answers)

2) **Inverse c2c** — errors (√MSE and √MedSE)  
- **Absolute**: RMSE = **4.91e-16**; RMedSE = **3.33e-16**  
- **Relative**: RMSE = **2.49e-14**; RMedSE = **3.41e-16**

4) **Inverse c2r** — errors (√MSE and √MedSE)  
- **Absolute**: RMSE = **3.62e-16**; RMedSE = **2.22e-16**  
- **Relative**: RMSE = **7.04e-15**; RMedSE = **1.98e-16**

5) **Machine precision**
- **Yes** (practically): absolute errors are ~**1e-16**, consistent with double precision machine epsilon, `eps(Float64) ≈ 2.22e-16`.  
- Relative errors can be slightly larger because they’re computed element-wise and get amplified when `|A[i,j]|` is small.

6) **Numeric value and meaning of `C[0,0]` / `R[0,0]`**  
*(Julia is 1-based: use `C[1,1]` and `R[1,1]`.)*
- `C[1,1] = 1.0029501900728947e6 + 0.0im`  
  `sum(A) = 1.0029501900728948e6`  
  Difference: `C[1,1] − sum(A) = −1.1641532182693481e-10 + 0.0im` (≈ **1.16e-16** relative).  
  **Meaning:** the **DC (zero-frequency) component** of the 2D DFT → the **sum of all entries of `A`** (FFTW forward is unnormalized).
- With `R = rfft(A, 2)` (r2c along columns):  
  `R[1,1] = 910.7399531409385 + 0.0im`  
  `sum(A[1,:]) = 910.739953140939`  
  Difference: `R[1,1] − sum(A[1,:]) = −4.547473508864641e-13 + 0.0im` (≈ **5.0e-16** relative).  
  **Meaning:** the **row-wise DC along columns** for row 1 → the **sum of the first row**.


```julia
#Generazione della matrice di rango 2 1000x1000 con N(1,1)

using Random, LinearAlgebra, Statistics, FFTW

#norm11(v) = (v .- mean(v)) ./ std(v; corrected=false) .+ 1 #funzione se volessi normalizzare a N(1,1), qui forzo la normalizzazione e perdo gaussianità
#Dato che è rango due generiamo due vettori colonna con la distribuzione richiesta N(1,1)
c1 = (randn(1000) .+ 1) #norm11() se volessi normalizzare
c2 = (randn(1000) .+ 1) 
#il +1 è in broadcasting quindi fa la trasformazione affine, ora mean=1

#Nel malaugurato caso in cui col_1 ≈ a * col_2:
tol = 1e-10
while true
    alpha = dot(c1, c2) / dot(c1, c1)   # proiezione di c2 su span(c1)
    r = c2 .- alpha .* c1               # componente ortogonale
    if norm(r) > tol * max(norm(c1), norm(c2))
        break                           # ok: NON collineari
    end
    c2 = (randn(1000) .+ 1)                # rigenera e riprova
end

#Creo vettore con metà 1 e metà 2, sarà un selettore, che mi permetterà di avere metà colonne c1 e meta c2
#perché metà e metà e non 1/3 e 2/3 o 1 c1 e il resto tutte c2? Scelta arbitraria, così
sel = vcat(fill(1, 500), fill(2, 500))
#mescolo casualmente gli 1 e i 2 nell'array monodimensionale, così da avere una distribuzione casuale delle colonne nell'inserimento
#non cambia comunque nulla alla sostanza, il rango rimane 2 e la statistica pure, anche questa scelta arbitraria
shuffle!(sel)

# 3) Costruzione della matrice A (1000x1000, rank 2)
A = Matrix{Float64}(undef, 1000, 1000)
for j in 1:1000
    A[:, j] = (sel[j] == 1 ? c1 : c2) #ciclo if più veloce
end
#ora la matrice ha solo due colonne linearmente indipendenti, perciò rango 2
#ogni colonna ha elementi distribuiti normalmente con media 1 e varianza 1, non è lo stesso per ogni riga ovviamente.
#comunque sia presi tutti gli elementi della matrice A,
#ci aspettiamo di avere una distribuzione normale con media 1 e varianza 1, e ovviamente rango 2 che andiamo a controllare:

# 4) Verifiche (campionarie, ~)
if rank(A) == 2
    println("Matrix has rank 2.")
else 
    println("ERROR: Matrix does NOT have rank 2.")
    exit(1)
end
if isapprox(mean(A), 1; atol=1e-1) == true && isapprox(std(A), 1; atol=1e-1) == true
    println("Matrix elements have mean = $(mean(A)) and std = $(std(A)).")
else 
    println("ERROR: Matrix elements do NOT have mean 1 and std 1.")
    exit(1)
end
 
#faccio la FFT c2c
C = fft(A)

#Inversa c2c per ricostruire A
A_c2c = real(ifft(C))

#faccio la FFT r2c
R = rfft(A, 2)

#Inversa complex-to-real (c2r) per ricostruire A
A_c2r = irfft(R, size(A, 2), 2)

#statistica

#metto i residui in due matrici:
# Residui (ricostruzione - originale)
E_c2c = A_c2c .- A
E_r2c = A_c2r .- A

# Residui relativi:
ϵ   = eps(eltype(A))
den = max.(abs.(A), ϵ)
Er_c2c = E_c2c ./ den
Er_r2c = E_r2c ./ den


# definisco sqrt(mean square) e sqrt(median square)
rms(E)   = sqrt(mean(abs2, vec(E)))        # √( mean(|E|^2) )
rmeds(E) = sqrt(median(abs2.(vec(E))))     # √( median(|E|^2) )       
#vec(E) appiattisce la matrice in un vettore monodimensionale per comodità

# --- c2c ---
abs_mean_c2c   = rms(E_c2c)
abs_median_c2c = rmeds(E_c2c)
rel_mean_c2c   = rms(Er_c2c)
rel_median_c2c = rmeds(Er_c2c)

# --- r2c → c2r ---
abs_mean_r2c   = rms(E_r2c)
abs_median_r2c = rmeds(E_r2c)
rel_mean_r2c   = rms(Er_r2c)
rel_median_r2c = rmeds(Er_r2c)

println("=== c2c reconstruction ===")
println("abs mean (RMSE)   = ", abs_mean_c2c)
println("abs median (RMedSE)= ", abs_median_c2c)
println("rel mean (RMSE)   = ", rel_mean_c2c)
println("rel median (RMedSE)= ", rel_median_c2c)

println("\n=== r2c → c2r reconstruction ===")
println("abs mean (RMSE)   = ", abs_mean_r2c)
println("abs median (RMedSE)= ", abs_median_r2c)
println("rel mean (RMSE)   = ", rel_mean_r2c)
println("rel median (RMedSE)= ", rel_median_r2c)
```
