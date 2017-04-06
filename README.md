# Param.jl

# USE Parameters.jl INSTEAD
This package is just here now for my own amusement. Paramters.jl is much better.

# Install
From the julia command prompt, run:
```julia
Pkg.clone("https://github.com/gabrielgellner/Param.jl")
```

# Movtivation
This package is motivated by the common need to pass a collection of parameters to functions
such as for ODE's or optimization routines. There are two obvious ways of doing this: 1)
defining a special type

```julia
type ODEParam
    a::Float64
    b::Float64
end

ODEParam(1.0, 2.0)
```

or as a `Dict`

```julia
Dict(a => 1.0, b => 2.0)
```

From these examples we see clearly that the `Dict` option is easier to read what the
values of a and b are. The downside is that we likely do not need the flexibility of
a `Dict` for this issue, and potentially pay a performance penalty for this unwanted
flexibility. For the custom type we have the issue that the default constructor is
opaque, requiring the user to remember the order of the fields in the defined type. This
can be fixed with making a constructor like:

```julia
ODEParam(;a = 0.0, b = 0.0) = Param(a, b)
# with usage
ODEParam(b = 2.0, a = 3)
# etc
```

This is not bad, but we have a lot of boilerplate to generate this every time this is
wanted. So in this package we make some simple macros that do this boilerplate
automatically. So instead you can do:

```julia
@with_kw type ODEParam
    a::Float64 = 0.0
    b::Float64 = 0.0
end
```

The second convenience feature is for using the generated type in functions. Normally
we need to do something like:

```julia
# using a type, like the one generated by `@with_kw`
function ydot(t, y, par)
    dydt = zeros(2)
    dydt[1] = par.a*y[1] + par.b
    dydt[2] = par.b*y[1] - par.b*y[1]*y[2]
    return dydt
end

# or using Dict
function ydot(t, y, par)
    dydt = zeros(2)
    dydt[1] = par[:a]*y[1] + par[:b]
    dydt[2] = par[:b]*y[1] - par[:b]*y[1]*y[2]
    return dydt
end

```  

So `Param.jl` defines the macro `@withparam <param name> <parameter argument name>` that
can be used as follows:

```julia
@withparam ODEParam par function ydot(t, y, par)
    dydt = zeros(2)
    dydt[1] = a*y[1] + b
    dydt[2] = b*y[1] - b*y[1]*y[2]
    return dydt
end
```
This macro goes over the definition of the function and replaces all parameter names with
the prefixed names. For example in the above would be translated into:

```julia
function ydot(t, y, par)
    dydt = zeros(2)
    dydt[1] = par.a*y[1] + par.b
    dydt[2] = par.b*y[1] - par.b*y[1]*y[2]
    return dydt
end
```
