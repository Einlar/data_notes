# Floating Point Representation
The range of representable numbers is limited by the number of bits available. Floating point representation trades accuracy with an extended range: a float can go from $1.4 \times 10^{-45}$, to $3.4 \times 10^{38}$.

A single precision floating point number is defined, according to the IEEE 754 standard, as a chunk of $32$ ($4$ bytes) divided in the following parts:
- **Sign** (most significant bit)
- **Exponent** (bits [2,9])
- **Significand** (the old *mantissa*, last 23 bits)

For non-zero non-maximum exponents, the following formula returns the represented value:
$$ n = (-1)^\mathrm{signbit} \times 2^{\mathrm{exponentbits}-127} \times 1.\mathrm{significandbits} $$
Otherwise, the significand can have different meanings: a subnormal number, infinity or NaN (Not A Number, returned after an impossible operation like 1/0).

Precision:
- Half: `__fp16`
- Single: `float`
- Double: `double`
- Extended precision: `long double`
- Quadruple: `__float128` (implemented in software, so it is much slower)

Real numbers are infinite and uncountable, so there is no way to define the number "immediately following" a real number. However, floating point numbers are obviously finite, and so there is a way to navigate them "in steps". So, the standard library contains a function to do this: `std::nextafter(x,-inf)` (previous number), and `std::nextafter(x,inf)` (next number). So, in reality, floating point numbers represent *intervals* on $\mathbb{R}$. This is especially important for `M_PI`, which (in double precision) sits very near to the lower limit of the range of the respective double.

This leads to very strange situations: floating point computations are **not** commutative! After every operation, the result is rounded to the nearest float. If the number sits exactly in the middle of a range, it is rounded towards the nearest *even* float (corresponding to an interval closed to the left, and open to the right).

Search for: Vancouver stock market failure, Ariane 5 failure, Patriot failure.

<!-- https://www.heidelberg-laureate-forum.org/video/lecture-desperately-needed-remedies-for-the-undebuggability-of-large-scale-floating-point-computations-in-science-and-engineering.html -->

Fun fact: the number of different floating points is the same in [1,2] and [2,4].

Search for Kahan summation.