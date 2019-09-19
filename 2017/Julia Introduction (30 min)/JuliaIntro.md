# Julia for Scientific Computing
## Josh Day
###### https://github.com/joshday 
###### emailjoshday@gmail.com


---
# Don't have Julia downloaded?
- https://juliacomputing.com/products/juliapro.html (IDE)
- https://julialang.org/downloads/

---
# Packages used in Tutorial
- This will download and precompile everything you need to reproduce what I'm using
```julia
Pkg.clone("https://github.com/joshday/TalkSetup.jl")
using TalkSetup
```


---
# Before We Get To Julia
## Sapir-Worf Hypothesis
- Your language determines/influences how you think


---
# Do we need another language?
- How does R/Matlab/Python influence how you solve problems?
  - Don't use for loops
  - Write performance-critical code in C, Fortran, etc.
  - Put everything in a data frame (R)
  - etc.
- Given modern data sizes, we've reached the limits of what R can do by itself

---
# My ideal scientific programming language:
- looks nice
- is fast
- is open source
- doesn't restrict how I think


---
# What is Julia?
Julia is a high-level, high-performance dynamic programming language for technical computing, with syntax that is familiar to users of other technical computing environments.
- http://julialang.org

---
# Why Julia?
## Julia solves the "Two-Language Problem"
- **Prototype** code goes into high-level language like R/Python
- **Production** code goes into low-level language like C/C++

---
# Why Julia?
#### Write high-level code that resembles mathematical formulas
- Yet produces fast, low-level machine code
- Traditionally, this has only been generated by static languages

---
# Why Julia?
- Most of Julia is written in Julia
- Trying to figure out what someone else's code is doing?  Go right to the source:  `@edit`

---
# R is Great, but...
#### Not meant/designed for high performance
- http://adv-r.had.co.nz/Performance.html
#### Deficiencies in core language
- Some fixed with packages (`devtools`, `roxygen2`, `Matrix`)
- Some harder to fix (R uses old version of BLAS)
- Some impossible to fix (clunky syntax, poor design choices)

---
# R is Great, but...
#### Only 6 active developers left (out of 20 R-Core members)
- JuliaLang organization has 73 members, with 622 contributors
#### Doug Bates (`lme4`, `Matrix`) quote:
> As some of you may know, I have had a (rather late) mid-life crisis and run off with another language called Julia
- Doug Bates' work in Julia is an example of how Julia gets to learn from the mistakes of other languages
- https://github.com/dmbates/MixedModels.jl

---
![inline](features.png)


---
# Benchmarks (time relative to C)
![inline](benchmarks.png)

---
# Not just "Fast R/Python/Matlab"
- Julia is fast because of features which work well together
- You can't just take the magic dust that makes Julia fast and apply it to your favorite language

---
# Julia Language Design
- Type system and type inference
- Multiple dispatch
- Just-in-time (JIT) compilation
- Metaprogramming (macros)
- pass by reference (mutate objects in place)

---
# Julia's Growth (Number of Packages)
![](http://pkg.julialang.org/img/allver.svg)

---
# Julia's Growth (Stars on GitHub)
![](http://pkg.julialang.org/img/stars.svg)


---
# Julia Basics
## Everything has a type
```julia
1    # Int64

1.0  # Float64

[1.0, 2.0]  # Vector{Float64}
```


---
# Julia Basics
## Use Unicode
- Code that looks like math:
  $$\hat\beta = (X^TX)^{-1}X^Ty$$
```julia
β̂ = inv(x'x)x'y  # Implicit multiplication

β̂ = inv(x' * x) * x' * y  
```
- Compare this to R:
```r
betahat = solve(t(x) %*% x) %*% t(x) %*% y
```


---
# Julia Basics
## Functions
```julia
f(x) = x ^ 2

# Code blocks require an `end`
function f(x)
    x ^ 2
end
```


---
# Julia Basics
## Code optimizations for free

```julia
f(x) = x ^ 2

@code_llvm f(1)

@code_llvm f(1.0)
```

---
# Just-in-time Compilation (JIT)
```julia
x = randn(100)
@time sum(x)
@time sum(x)
```
```
0.029059 seconds (14.51 k allocations: 673.044 KB)
0.000011 seconds (5 allocations: 176 bytes)
```

---
# Julia Basics
## For loops
Input:
```julia
for i in 1:3
    println(i)
end
```
Output:
```
1
2
3
```

---
# Type System
- When thinking about types, think about sets
- An **abstract** type defines a set of other types
- One abstract type in Julia is `Number`

---
# `Number`
![](tree.png)


---
# Type Annotations
- A number should be able to be added to itself

```julia
f(x) = x + x  # too broad?


g(x::Float64) = x + x  # too specific?


h(x::Number) = x + x  # just right
```

---
# Type Annotations
Useful for error handling
<br>
```julia
h(x::Number) = x + x
```
<br>

```julia
function h(x)
    if isa(x, Number)
        return x + x
    else
        throw(ArgumentError("x should be a number"))
    end
end
```


---
# Multiple Dispatch
- **Multiple dispatch** means a function calls different code depending on the types of the arguments.

- Consider the Distributions package:
```julia
using Distributions

mean(Normal(0, 1)) == 0.0

mean(Gamma(10, 6)) == 60.0
```

---
# Multiple dispatch
Because of multiple dispatch, Julia packages are typically based around an interface, or grammar for dealing with a set of types.

- What methods should be part of a Distribution's grammar?
	- mean, var, std, skewkess, kurtosis, mode, pdf, cdf, ...

---
# Quantile Example
- Suppose I want to find quantiles using Newton's method:
$$\theta_{t+1} = \theta_t - \frac{F(\theta_t) - q}{F'(\theta_t)}$$
  where $F$ is the CDF of a distribution
- In R, I would need a different function for every distribution!
  - p-q-r-d family of functions
- In Julia, we can do this in a single function

---
# The Power of Julia: Abstraction
- A `Distribution` has methods `mean`, `cdf`, `pdf`
```julia
using Distributions

function myquantile(d::Distribution, q::Number)
    θ = mean(d)
    tol = Inf
    while tol > 1e-5
        θold = θ
        θ = θ - (cdf(d, θ) - q) / pdf(d, θ)
        tol = abs(θold - θ)
    end
    θ
end
```

---
Input:
```julia
for d in [Normal(), TDist(4)]
    println("For $d")
    println("  > myquantile: $(myquantile(d, .4))")
    println("  > quantile:   $(quantile(d, .4))\n")
end
```
Output:
```julia
For Distributions.Normal{Float64}(μ=0.0, σ=1.0)
  > myquantile: -0.2533471031356957
  > quantile:   -0.2533471031357997

For Distributions.TDist{Float64}(ν=4.0)
  > myquantile: -0.27072229470638115
  > quantile:   -0.27072229470759746
```

---
# The Julian Way
- Keep things abstract.
	- Not extra work **if** the type hierarchy is well designed.



---
# Pass by reference
- Why does this matter?
- Garbage collection (removing temporary storage) can be expensive

```julia
using BenchmarkTools

A = zeros(50_000, 500)
B = randn(50_000, 500);
```

---
# Pass by reference
```
@benchmark copy(B)
```
```
BenchmarkTools.Trial: 
  memory estimate:  190.73 MiB
  allocs estimate:  2
  --------------
  minimum time:     110.258 ms (12.06% GC)
  median time:      122.323 ms (11.79% GC)
  mean time:        126.339 ms (14.04% GC)
  maximum time:     190.705 ms (42.28% GC)
  --------------
  samples:          40
  evals/sample:     1
  time tolerance:   5.00%
  memory tolerance: 1.00%
 ```

---
# Pass by reference
```
@benchmark copy!(A, B)  # copy B into preallocated A
```
```
BenchmarkTools.Trial: 
  memory estimate:  0 bytes
  allocs estimate:  0
  --------------
  minimum time:     25.826 ms (0.00% GC)
  median time:      31.706 ms (0.00% GC)
  mean time:        32.591 ms (0.00% GC)
  maximum time:     44.245 ms (0.00% GC)
  --------------
  samples:          154
  evals/sample:     1
  time tolerance:   5.00%
  memory tolerance: 1.00%
```



---
# My Thoughts
- **Julia is ideal for developing projects with few dependencies**
- The package ecosystem is growing fast but is still lacking much of the functionality you can find in R
- However, there are many interesting things in Julia that R doesn't have:
  - Plots, Convex, JuMP, OnlineStats, Distributions, LossFunctions, Autodiff,...


---
# Thank You