# powerresiduesymbol #

## Introduction
The algorithm in this repository is written in Magma [[BCP97](#BCP97)]  and is able to compute power residue symbols and hilbert symbols effectively; it is also able to compute a local-field invariant, called Ibeta. 

The algorithm that computes Hilbert symbols is invented by Jan Bouw from Universiteit Leiden [[Bou17](#Bou17)], the Ibeta-algorithm is invented by Carlo Pagano from the Universiteit Leiden [[Pag17](#Pag17)] and the power residue symbol algorithm is invented and implemented by Koen de Boer @kodebro [[Boe16](#Boe16)].

Very soon, a paper about the power residue symbol and ibeta will be published.

## Installing
Put the files in a folder on a computer that has the latest version of Magma installed. Run Magma in this folder, and type
`load main;`
in the console. This constructs so-called `.sig`-files; this is normal. After this, you are ready to use the algorithms from this repository.

## Using the algorithms
To use the algorithm, you might first want to get familiar with the Magma language: http://magma.maths.usyd.edu.au/magma/

In the file `examples`, a few small examples are examined. I will go through these examples here.

### Using the power residue symbol algorithm
Suppose we want to compute the power residue symbol of two elements in some field. We first have to tell Magma what the field is, and what the elements are.
```
m := 7^2;
L<z> := CyclotomicField(m);
R := Integers(L);
```
Here, `R` is the ring of integers of the field `L`. In the algorithm, `R` can be any order, so it doesn't need to be the maximal order.
```
a := 4323* z^9 + 532*z^2 + 2;
b := 432*z^3 + 324*z^2 + z + 4;
```
Now, the elements are defined. Beware that the elements that you put in the power residue symbol must be coprime with each other, and both coprime with `m`. 
```
s1 := powerResidueSymbol(a,b,m,z,R);
s2 := powerResidueSymbol(b,a,m,z,R);
s3 := umkehrFaktor(a,b,ideal<R1|m1>,z1); ```
```
The output of `powerResidueSymbol` consists of an integer `a` modulo `m` and the element `z`. This means that the power residue symbol of the input equals `z^a`.

We took this example to show how reciprocity works; if the reciprocity law is right, we must have `s1 = s2 + s3` modulo `m`.
```
printf "The symbol %o should be %o + %o mod %o \n",s1,s2,s3,m;
```

In the subsequent part the function `getTwoCoPrimeElements` is used, to generate two elements that are coprime to each other and both coprime to `m`. This function is handy for testing the power residue symbol:
```
    a,b := getTwoCoPrimeElements(R,m,4); // from "getRandomElement"
    a := L!a; b := L!b;
    s1 := powerResidueSymbol(a,b,m,z,R);
    s2 := powerResidueSymbol(b,a,m,z,R);
    s3 := umkehrFaktor(a,b,ideal<R1|m1>,z1); 
```
Remark the code snippet `a := L!a; b := L!b;`. This shows that the power residue symbol algorithm only accepts `a` and `b` as input when they are field elements -- that is why we coerce them first to their parent field.

There are also more "user friendly" power residue symbol functions, that are of the form `powerresiduesymbol(a,b,m)`. Those are probably slower, because then the ring of integers of the parent field need to be computed, which is slow.
### Using the Hilbert symbol algorithm
We distinguish the "global" hilbert symbol and "local" hilbert symbol. As an algorithm, they are the same. As input the "global" hilbert symbol only accepts inputs from number fields (also called 'global arithmetic fields'). The local hilbert symbol accepts inputs from local number fields only. 

The global hilbert symbol needs an extra parameter, a prime ideal. With this prime ideal, the global field is localized into a local arithmetic field. The global inputs are then mapped to the local field, so that the hilbert symbol is computable.
#### "Global" hilbert symbol
Again, we first need to construct a field and elements in it.
```
m := 5^2*3^2;
L<z> := CyclotomicField(m);
R := Integers(L);
a := 4323* z^9 + 532*z^2 + 2;
b := 432*z^3 + 324*z^2 + z+ 4;
```
For the "global" hilbert symbol, one needs a prime that is used to localize the field.
```
pi := Decomposition(R,5)[1][1]; // Prime ideal above (5)
```
Then, the hilbert symbol can be computed as follows:
```
print hilbertSymbol(a,b,pi,m,z);
```
#### Local hilbert symbol
In order to compute the local hilbert symbol, one needs to construct a local field in the so-called ramified representation. That is a local field represented as a totally ramified extension over a unramified extension. Luckily, Magma can compute this for us.
```
m := 3^2*7;
F3 := pAdicField(3,400);
F<z> := LocalField(F3,CyclotomicPolynomial(m));
F,map := RamifiedRepresentation(F);
```
We also need the root of unity `z` coerced to `F`, and we need an uniformizer of `F`:
```
z := map(z);
pi := UniformizingElement(F);
```
Now, construct the elements, and compute the symbol.
```   
x := 1 + 2*z*pi + 3*z^2*pi^2 + (1-z^3)*pi^3;
y := 1 + 1*z*pi + 2*z^3*pi^2 + (1-z^2)*pi^3;
symbol,zeta := hilbertSymbol(x,y,m,z);
print symbol;
```
Again, the symbol is represented as an integer `a` modulo `m` and a root of unity `z`; the symbol then equals `z^a`. Since `z` can have quite a lengthy representation, we don't print it.
### Using the Ibeta algorithm
#### Ibeta for arbitrary cases
```
K<z3> := CyclotomicField(3);
PK<y> := PolynomialRing(K);
f := y^(3^3) - 3*y^(3) - 3*(1-z3)^2*y^1 + (1-z3); // saturated eisenstein polynomial
L := ext<K|f>;
I, beta := ibeta(L,3);
printf "I = %o, beta = %o\n",I,beta;
```
#### Ibeta when the defining polynomial is unsaturated-Eisenstein
```
K<z3> := CyclotomicField(3);
PK<y> := PolynomialRing(K);
f := y^(3^4) - (1-z3)*y^(3^2) - (1-z3)^2*y^1 + (1-z3); // unsaturated eisenstein polynomial

I, beta := ibeta_special(f,3); // special ibeta, for unsaturated eisenstein
L := ext<K|f>;
I2, beta2 := ibeta(L,3); // normal ibeta
printf "(%o,%o) and (%o,%o) are the same.\n",I,beta,I2,beta2;
```


# Bibliography #
<a name="Bou17">[Bou17]</a> Jan Bouw, 2017, On the computation of Hilbert norm residue symbols. Ph.D. dissertation. Universiteit Leiden, forthcoming

<a name="BCP97">[BCP97]<a/> Wieb Bosma, John Cannon, and Catherine Playoust, The Magma algebra system. I. The user language, J. Symbolic Comput., 24 (1997), 235–265. 

<a name="Boe16">[Boe16]</a> Koen de Boer, Computing the Power Residue Symbol. Master's thesis. Nijmegen, Radboud University. www.koendeboer.com/masterthesis_deBoer.pdf

<a name="Pag17">[Pag17]</a> Carlo Pagano, 2017. Jump sets for local fields, forthcoming


