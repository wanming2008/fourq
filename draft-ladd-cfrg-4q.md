---
title: Curve4Q
abbrev:
docname: draft-ladd-cfrg-4q-latest
category: std

ipr: trust200902
area: Security
workgroup:
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
 -
       ins: W. Ladd
       name: Watson Ladd
       organization: UC Berkeley
       email: watsonbladd@gmail.com
 -
       ins: P. Longa
       name: Patrick Longa
       organization: Microsoft Research
       email: plonga@microsoft.com
 -
       ins: R. Barnes
       name: Richard Barnes
       organization: Mozilla
       email: rlb@ipv.sx

informative:

    Curve4Q:
      target: "https://eprint.iacr.org/2015/565.pdf"
      title: "FourQ: four-dimensional decompositions on a Q-curve over the Mersenne prime"
      date: 2016
      author:
         -
              ins: C. Costello
         -
              ins: P. Longa

    Qcurve:
      target: "http://eprint.iacr.org/2014/727.pdf"
      title: "The Q-curve Construction for Endomorphism-Accelerated Elliptic Curves"
      date: 2016
      author:
         -
              ins: B. Smith

    GLV4:
      target: "http://eprint.iacr.org/2013/311.pdf"
      title: "Four-dimensional GLV via the Weil restriction"
      date: 2013
      author:
         -
              ins: A. Guillevic
         -
              ins: S. Ionica

    Distinguished:
       target: "http://people.scs.carleton.ca/~paulv/papers/JoC97.pdf"
       title: "Parallel Collision Search with Cryptanalytic Applications"
       date: 1996
       author:
          -
             ins: P.C. van Oorschot
          -
             ins: M.J. Wiener

    Exceptional:
       target: "https://www.iacr.org/archive/pkc2003/25670224/25670224.pdf"
       title: "Exceptional procedure attack on elliptic curve cryptosystems"
       date: 2003
       author:
          -
              ins: T. Izu
          -
              ins: T. Takagi

    Invsqr:
        target: "http://eprint.iacr.org/2012/309.pdf"
        title: "Fast and compact elliptic-curve cryptography"
        date: 2012
        author:
           -
              ins: M. Hamburg

    FourQlib:
      target: "https://www.microsoft.com/en-us/research/project/fourqlib/"
      title: "FourQlib"
      date: 2016
      author:
         -
              ins: C. Costello
         -
              ins: P. Longa

    GLV:
       target: "https://www.iacr.org/archive/crypto2001/21390189.pdf"
       title: "Faster Point Multiplication on Elliptic Curves with Efficient Endomorphisms"
       date: 2001
       author:
          -
             ins: R. P. Gallant
          -
             ins: R. J. Lambert
          -
             ins: S. A. Vanstone

    GLS:
        target: "https://www.iacr.org/archive/eurocrypt2009/54790519/54790519.pdf"
        title: "Endomorphisms for Faster Elliptic Curve Cryptography on a Large Class of Curves"
        date: 2009
        author:
           -
             ins: S. D. Galbraith
           -
             ins: X. Lin
           -
             ins: M. Scott

    SchnorrQ:
       target: "https://www.microsoft.com/en-us/research/wp-content/uploads/2016/07/SchnorrQ.pdf"
       title: "SchnorrQ: Schnorr Signatures on FourQ"
       date: 2016
       author:
          -
             ins: C. Costello
          -
             ins: P. Longa

    TwistedRevisited:
       target: "http://iacr.org/archive/asiacrypt2008/53500329/53500329.pdf"
       title: "Twisted Edwards Curves Revisited"
       date: 2008
       author:
          -
              ins: H. Hisil
          -
              ins: K-H. Wong
          -
              ins: G. Carter
          -
              ins: E. Dawson

    Twisted:
       target: "http://eprint.iacr.org/2008/013.pdf"
       title: "Twisted Edwards Curves"
       date: 2008
       author:
          -
              ins: D. J. Bernstein
          -
              ins: P. Birkner
          -
              ins: M. Joye
          -
              ins: T. Lange
          -
              ins: C. Peters

--- abstract

This document specifies Curve4Q, a twisted Edwards curve proposed in {{Curve4Q}}
that takes advantage of arithmetic over the field GF(2^127-1) and two endomorphisms
to achieve the speediest Diffie-Hellman key agreements over a group of order
approximately 2^246, which provides around 128 bits of security.
Curve4Q implementations are more than two times faster than those of Curve25519
and, when not using endomorphisms, are between 1.2 and 1.6 times faster.

--- middle

# Introduction

Public key cryptography continues to be computationally expensive, particularly
on less powerful devices. While recent advances in efficient formulas for
addition and doubling have substantially reduced the cost of elliptic curve
operations in terms of field operations, the number of group operations involved
in scalar multiplication has not been reduced in the curves considered for IETF
use. Using curves with efficiently computable endomorphisms can reduce the
number of group operations by turning one long scalar multiplication into the
sum of several multiplications by smaller scalars, which can be evaluated more
efficiently.

For curves over quadratic extension fieldss, there are more endomorphism
families to choose from, and the field operations are often more efficient
compared to prime fields of the same size.  The ideal case is given by curves
equipped with two distinct endomorphisms, so that it becomes possible to divide
scalars into four parts.  We focus on curves defined over the field GF(p^2)
for the Mersenne prime p = 2^127 - 1, which offers extremely efficient
arithmetic.  Together, these improvements substantially reduce computation
time compared to other proposed Diffie-Hellman key exchange and digital
signature schemes.  However, the combined availability of these features
severely restricts the curves that can be used for cryptographic applications.

As described in {{Curve4Q}}, the elliptic curve "Curve4Q" defined in this
document is a special instance of the recent endomorphism-based constructions
from {{Qcurve}} and {{GLV4}}, and is the only known elliptic curve that
(1) permits a four dimensional decomposition (using two endomorphisms) over
GF(p^2) and (2) has a large prime order subgroup.  The order of this subgroup
is approximately 2^246, which provides around 128 bits of security.  No other
known elliptic curve with such a decomposition has a larger prime order subgroup
over this field.  This "uniqueness" allays concerns about selecting curves
vulnerable to undisclosed attacks.

Curve4Q can be used to implement Diffie-Hellman key exchange, as described
below.  It is also possible to use Curve4Q as the basis for digital signature
scheme (e.g., {{SchnorrQ}}).

# Mathematical Prerequisites

Curve4Q is defined over the finite field GF(p^2), where p is the Mersenne prime
2^127 - 1.  Elements of this finite field have the form (a + b * i), where a and
b are elements of the finite field GF(p) (i.e., integers mod p) and i^2 = -1.

Let A = a0 + a1\*i and B = b0 + b1\*i be two elements of GF(p^2). Below we
present formulas for computing addition, subtraction, multiplication, squaring,
conjugation and inversion.

~~~~
A + B = (a0 + b0) + (a1 + b1)*i

A - B = (a0 - b0) + (a1 - b1)*i

A * B = (a0*b0 - a1*b1) + ((a0 + a1)*(b0 + b1)-(a0*b0 + a1*b1))*i
      = (a0*b0 - a1*b1) + (a0*b1 + a1*b0)*i

A * A = (a0 + a1)*(a0 - a1) + 2*a0*a1*i

conj(A) = a0 - a1*i

1/A = conj(A) / (a0^2 + a1^2)
~~~~

The GF(p) division in the formula for 1/A can be computed using an exponentiation
via Fermat's little theorem: 1/a = a^(p - 2) = a^(2^127 - 3) for any element a of GF(p).
One can use a fixed addition chain to compute a^(2^127 - 3) (e.g., see {{FourQlib}}).

Curve4Q is the twisted Edwards curve E over GF(p^2) defined by the
following curve equation:

~~~~~
E: -x^2 + y^2 = 1 + d * x^2 * y^2, with

d = 0x00000000000000e40000000000000142 +
    0x5e472f846657e0fcb3821488f1fc0c8d * i
~~~~~

Let E(GF(p^2)) be the set of pairs (x, y) of elements of GF(p^2) satisfying this
equation. This set forms a group with the addition operation (x1, y1) + (x2, y2)
= (x3, y3), where:

~~~~
          x1 * y2 + y1 * x2                y1 * y2 + x1 * x2
x3 = ---------------------------, y3 = ---------------------------
      1 + d * x1 * y1 * x2 * y2         1 - d * x1 * y1 * x2 * y2
~~~~

As d is not a square in GF(p^2), and -1 is, this formula never involves a
division by zero when applied to points on the curve. That is, the formula is
complete and works without exceptions for any input in E(GF(p^2)).  The
identity element is (0, 1), and the inverse of (x, y) is (-x, y).  The order of
this group is \#E = 2^3 · 7^2 · N, where N is the following 246-bit prime:

~~~~~
N = 0x29cbc14e5e0a72f05397829cbc14e5dfbd004dfe0f79992fb2540ec7768ce7
~~~~~

Points P on E such that [N]\*P = (0, 1) are N-torsion points. Given a point P and
Q which are both N-torsion points, it is difficult to find m such that Q = [m]\*P.
This is the elliptic curve discrete logarithm problem, which is closely related
to the security of Diffie-Hellman key exchanges as the best known attacks on the
Diffie-Hellman problem involve solving the discrete logarithm problem. The best
known algorithms take approximately 2^123 group operations.

This group has two different efficiently computable endomorphisms, as described
in {{Curve4Q}}. As discussed in {{GLV}} and {{GLS}}, these endomorphisms
allow a multiplication by a large scalar to be computed using multiple multiplications
by smaller scalars, which can be evaluated in much less time overall.

# Representation of Curve Points

Elements a in GF(p) are represented as 16 byte little endian integers which are
the numbers in the range [0, p). The 16 bytes b[0], b[1], ... b[15] represent
b[0] + 256\*b[1] + 256^2\*b[2] + ... + 256^15\*b[15].  Since we are representing numbers
in the range [0, 2^127-1), the top bit of b[15] is always zero.

An element x0 + x1\*i of GF(p^2) is represented on the wire by the concatenation
of the encodings for x0 and x1. A point (x, y) on Curve4Q is serialized in a
compressed form as the representation of y with a modified top bit. This top bit
is used to disambiguate between x and -x during decoding.

To carry out this disambiguation we use the lexicographic order of elements
in GF(p^2): define two elements a = a0 + a1\*i and b = b0 + b1\*i with all their
coordinates in [0, p); a is greater than b if a0 is greater than b0. If a0 and
b0 are equal, a is greater than b if a1 is greater than b1.

Set the coordinate value x and its negative -x. The top bit of a compressed
point is 0 if x is smaller than -x. Otherwise, the top bit is 1.

~~~~~
|--------------- y ---------------|
|       y0     |0|       y1     |s|
|..............|.|..............|.|
~~~~~

To decode an encoded point from a 32-byte sequence B:

* Parse out the encoded values y = y0 + y1 * i and s
* Check that y0 and y1 are both less than p
* Solve x^2 = (y^2 - 1) * (d * y^2 + 1) for x
* If s is 0, return the smaller of x and -x (in the lexicographic ordering)
* If s is 1, return the larger of x and -x
* Check that (x,y) is a valid point on the curve

The appendix {{point-decompression}} details an algorithm for decoding a point
following the steps above. 

We call the operation of compressing a point P into 32 bytes Compress(P),
and decompression Expand(S). Expand(Compress(P))=P for all the points P on the curve,
and Compress(Expand(S))=S if and only if S is a valid representation of a point.

Not all 32 byte strings represent valid points. Implementations MUST reject
invalid strings and check that decompression is successful. Strings are invalid
if they are not possible outputs of the compression operator.  In particular the
values of y0 and y1 MUST be less then p.

# Scalar multiplication

Below, we present two algorithms for scalar multiplication on Curve4Q.  Each
algorithm takes as input a 256-bit unsigned integer m and an N-torsion point P
and computes the product [m]\*P.

The first algorithm uses a simple fixed-window exponentiation without exploiting
endomorphisms.  The second algorithm uses endomorphisms to accelerate
computation.  The execution of operations in both algorithms has a regular
pattern in order to enable constant-time implementations and protect against
timing and simple side channel attacks.  Both algorithms use the same addition
and doubling formulas.

First, we discuss explicit formulas and efficient projective coordinate
representations.

## Alternative Point Representations and Addition Laws

We use coordinates based on extended twisted Edwards coordinates introduced in
{{TwistedRevisited}}: the tuple (X, Y, Z, T) with Z nonzero and Z * T = X * Y
corresponds to a point (x, y) satisfying x = X/Z and y = Y/Z. The neutral point
in this representation is (0, 1, 1, 0). The following slight variants are used in
the optimized scalar multiplication algorithm in order to save computations:
point representation R1 is given by (X, Y, Z, Ta, Tb), where T=Ta * Tb;
representation R2 is (N, D, E, F) = (X + Y, Y- X, 2Z, 2dT); representation R3 is (N,
D, Z, T) = (X + Y, Y - X, Z, T); and representation R4 is (X, Y, Z). Similar "caching"
techniques were discussed in {{TwistedRevisited}} to accelerate repeated
additions of the same point. Converting between these representations is
straightforward.

* R1: (X, Y, Z, Ta, Tb), Ta * Tb = T, Z * T = X * Y
* R2: (N, D, E, F) = (X + Y, Y - X, 2 * Z, 2 * d * T)
* R3: (N, D, Z, T) = (X + Y, Y - X, Z, T)
* R4: (X, Y, Z)

A point doubling (DBL) takes an R4 point and produces an R1 point. For addition,
we first define an operation ADD\_core that takes an R2 and an R3 point and
produces an R1 point. This can be used to implement an operation ADD which takes
an R1 and an R2 point as inputs (and produces an R1 point) by first converting the
R1 point to R3, and then executing ADD\_core.  Exposing these operations and the
multiple representations helps save time by avoiding redundant computations: the
conversion of the first argument to ADD can be done once if the argument will be
used in multiple additions.

Below, we list the explicit formulas for the required point operations. These
formulas, which are adapted from {{Twisted}} and {{TwistedRevisited}}, are
complete: they have no exceptional cases, and therefore can be used in any
algorithm for computing scalar multiples without worrying about exceptional
procedure attacks {{Exceptional}}. Note that we do not explicitly note the point
format every time an addition or doubling is used, and assume that conversions
are done when required.

DBL and ADD\_core are computed as follows:

~~~~
DBL(X1, Y1, Z1):
  A = X1^2
  B = Y1^2
  C = 2 * Z1^2
  D = A + B
  E = (X1 + Y1)^2 - D
  F = B - A
  G = C - F
  X3 = E * G
  Y3 = D * F
  Z3 = F * G
  Ta3 = E
  Tb3 = D
return(X3, Y3, Z3, Ta3, Tb3)

ADD\_core(N1, D1, E1, F1, N2, D2, Z2, T2):
   A = D1 * D2
   B = N1 * N2
   C = T2 * F1
   D = Z2 * E1
   E = B - A
   F = D - C
   G = D + C
   H = B + A
   X3 = E * F
   Y3 = G * H
   Z3 = F * G
   Ta3 = E
   Tb3 = H
return (X3, Y3, Z3, Ta3, Tb3)
~~~~

## Multiplication without Endomorphisms

We begin by taking our input point P, and computing a table of points containing
T[0] = [1]P, T[1] = [3]P, ... , T[7] = [15]P as follows:

~~~~
Q = DBL(P)
Convert Q to R2 form
T[0] = P
Convert T[0] to R2 form
for i=1 to 7:
    T[i] = ADD_core(Q, T[i-1])
    Convert T[i] to R2 form
~~~~

Next, take m and reduce it modulo N.  Then, add N if necessary to ensure that m
is odd. At this point, we recode m into a signed digit representation consisting
of 63 signed, odd digits d[i] in base 16. The following algorithm accomplishes
this task.

~~~~
for i=0 to 61:
    d[i] = (m mod 32) - 16
    m = (m - d[i]) / 16
d[62] = m
~~~~

Finally, the computation of the multiplication is as follows.

~~~~
Let ind = (abs(d[62]) - 1) / 2
Let sign = sgn(d[62])
Q = sign * T[ind]
Convert Q into R4 form
for i from 61 to 0:
    Q = DBL(Q)
    Q = DBL(Q)
    Q = DBL(Q)
    Q = DBL(Q)
    ind = (abs(d[i]) - 1) / 2
    sign = sgn(d[i])
    S = sign * T[ind]
    Q = ADD(Q, S)
return Q
~~~~

As sign is either -1 or 1, the multiplication sign * T[ind] is simply a
conditional negation.  To negate a point (N, D, E, F) in R2 form one computes
(D, N, E, -F). The table lookups and conditional negations must be carefully
implemented as described in ``Security Considerations'' to avoid side-channel
attacks.  This algorithm MUST NOT be applied to points which are not N-torsion
points; it will produce the wrong answer.

## Multiplication with Endomorphisms

This algorithm makes use of the identity [m]\*P = [a_1]\*P + [a_2]\*phi(P) +
[a_3]\*psi(P) + [a_4]\*psi(phi(P)), where a_1, a_2, a_3, and a_4 are 64-bit
scalars that depend on m. The overall product can then can be computed using a
small table of 8 precomputed points and 64 doublings and additions. This is
considerably fewer operations than the number of operations required by the 
algorithm above, at the cost of a more complicated implementation.

We describe each phase of the computation separately: the computation of the
endomorphisms, the scalar decomposition and recoding, the creation of the table
of precomputed points and, lastly, the computation of the final results. Each
section refers to constants listed in an appendix in order of appearance.

### Endomorphisms

The two endomorphisms phi and psi used to accelerate multiplication are computed
as phi(Q) = tau_dual(upsilon(tau(Q)) and psi(Q) = tau_dual(chi(tau(Q))).  Below,
we present procedures for tau, tau_dual, upsilon and chi, adapted from
{{FourQlib}}. Tau_dual produces an R1 point, while the other procedures produce
R4 points.

Note: Tau produces points on a different curve, while upsilon and chi are
endomorphisms of that different curve. Tau and tau_dual are the isogenies
mentioned in the mathematical background above.  As a result the intermediate
results do not satisfy the equations of the curve E.  Implementers who wish to
check the correctness of these intermediate results are referred to {{Curve4Q}}.

~~~~
tau(X1, Y1, Z1):
   A = X1^2
   B = Y1^2
   C = A + B
   D = A - B
   X2 = ctau * X1 * Y1 * D
   Y2 = -(2 * Z1^2 + D) * C
   Z2 = C * D
return(X2, Y2, Z2)
~~~~

~~~~
tau_dual(X1, Y1, Z1):
  A = X1^2
  B = Y1^2
  C = A + B
  Ta2 = B - A
  D = 2 * Z1^2 - Ta2
  Tb2 = ctaudual * X1 * Y1
  X2 = C * Tb2
  Y2 = D * Ta2
  Z2 = C * D
return(X2, Y2, Z2, Ta2, Tb2)
~~~~

~~~~
upsilon(X1, Y1, Z1):
   A = cphi0 * X1 * Y1
   B = Y1 * Z1
   C = Y1^2
   D = Z1^2
   F = D^2
   G = B^2
   H = C^2
   I = cphi1 * B
   J = C + cphi2 * D
   K = cphi8 * G + H + cphi9 * F
   X2 = conj(A * K * (I + J) * (I - J))
   L = C + cphi4 * D
   M = cphi3 * B
   N = (L + M) * (L - M)
   Y2 = conj(cphi5 * D * N * (H + cphi6 * G + cphi7 * F))
   Z2 = conj(B * K * N)
return(X2, Y2, Z2)
~~~~

~~~~
chi(X1, Y1, Z1):
   A = conj(X1)
   B = conj(Y1)
   C = conj(Z1)^2
   D = A^2
   F = B^2
   G = B * (D + cpsi2 * C)
   H = -(D + cpsi4 * C)
   X2 = cpsi1 * A * C * H
   Y2 = G * (D + cpsi3 * C)
   Z2 = G * H
return(X2, Y2, Z2)
~~~~

### Scalar Decomposition and Recoding

This stage has two parts. The first one consists in decomposing the scalar into
four 64-bit integers, and the second one consists in recoding these integers
into a form that can be used to efficiently and securely compute the scalar
multiplication.

The decomposition step uses four fixed vectors called b1, b2, b3, b4, with four
64 bit entries each.  In addition, we have integer constants L1, L2, L3, L4,
which are used to implement rounding.  All these values are listed in
{{constants}}.  In addition, we use two constant vectors derived from these
inputs:

* c  = 5 * b2 - 3 * b3 + 2 * b4
* c' = 5 * b2 - 3 * b3 + 3 * b4 = c + b4

Given m, first compute t[i] = floor(L[i] * m / 2^256) for i between 1 and 4.
Then compute the vector sum a = (m, 0, 0, 0) - t1 b1 - t2 b2 - t3 b3 - t4 b4.
Precisely one of a + c and a + c' has an odd first coordinate: this is the
vector v that is fed into the scalar recoding step. Note that the entries of
this vector are 64 bits, so intermediate values in the calculation above can be
truncated to this width.

The recoding step takes the vector v=(v1, v2, v3, v4) from the previous step and
outputs two arrays m[0]..m[64] and d[0]..d[64]. Each entry of d is between 0 and
7, and each entry in m is -1 or 0. The recoding algorithm is detailed below.
bit(x, n) denotes the nth bit of x, counting from least significant to most,
starting with 0.

~~~~~
m[64] = -1
for i = 0 to 63 do:
    b1 = bit(v1, i+1)
    d[i] = 0
    m[i] = b1

    for j = 2 to 4 do:
        bj = bit(vj, 0)
        d[i] = d[i] + bj * 2^(j-2)
        c = (b1 or bj) xor b1
        vj = vj / 2 + c
d[64] = v2 + 2 * v3 + 4 * v4
~~~~~

### Final Computation

We now describe the last step in the endomorphism based algorithm for computing
scalar multiplication. On inputs m and P, the algorithm first precomputes a
table of images of P under the endomorphisms, then recodes m, then uses these
intermediate artifacts to compute the scalar product.

First, compute a table T of 8 points in representation R2 as shown below.
Computations Q = psi(P), R = phi(P) and S = psi(phi(P)) are carried out using
formulas from {{endomorphisms}}.

~~~~~
Q is phi(P) in R3
R is psi(P) in R3
S is psi(Q) in R2
T[0] is P in R2
T[1] is ADD_core(Q, T[0])  # (P + Q)
Convert T[1] to R2
T[2] is ADD_Core(R, T[0])  # (P + R)
Convert T[2] to R2
T[3] is ADD_Core(R, T[1])  # (P + Q + R)
Convert T[3] to R2
T[4] is ADD_Core(S, T[0])  # (P + S)
Convert T[4] to R2
T[5] is ADD_Core(S, T[1])  # (P + Q + S)
Convert T[5] to R2
T[6] is ADD_Core(S, T[2])  # (P + R + S)
Convert T[6] to R2
T[7] is ADD_Core(S, T[3])  # (P + Q + R + S)
Convert T[7] to R2
~~~~~

Second, apply the scalar decomposition and recoding algorithm from
{{scalar-decomposition-and-recoding}} to m, to produce the two arrays
m[0]..m[64] and d[0]..d[64].

Define s[i] to be 1 if m[i] is 1 and -1 if m[i] is 0. Then the multiplication
is completed as follows:

~~~~~
Q = s[64] * T[d[64]]
Convert Q to R4
for i=63 to 0 do:
    Q = DBL(Q)
    Q = ADD(Q, s[i] * T[di])
return Q = (X/Z, Y/Z)
~~~~~

Multiplication by s[i] is simply a conditional negation. To negate an R2 point
(N, D, E, F) one computes (D, N, E , -F). It is important to do this (as well as
the table lookup) in constant time, i.e., the execution of branches and memory
accesses MUST NOT depend on secret values (see ``Security Considerations'' for
more details).

The optimized multiplication algorithm above only works properly for N-torsion
points. Implementations MUST NOT use this algorithm on anything that is not
known to be an N-torsion point. Otherwise, it will produce the wrong answer,
with extremely negative consequences for security.

# Diffie-Hellman Key Agreement

The above scalar multiplication algorithms can be used to implement
Diffie-Hellman with cofactor.

~~~~
DH(m, P):
      Ensure P on curve and if not return FAILURE

      P1 = DBL(P)                 # [2]P
      P2 = ADD(P1, P)             # [3]P
      P3 = DBL(DBL(DBL(DBL(P2)))) # [48]P
      Q = ADD(P3, P)              # [49]P
      Q = DBL(DBL(DBL(Q))         # [392]P

      Compute [m]*Q

      If Q is the neutral point, return FAILURE
Return [m]\*Q in affine coordinates
~~~~

The role of the multiplication by 392 is to ensure that Q is an N-torsion point
so that the scalar multiplication [m]\*P in the DH function above may be used
safely to produce correct results. In other words, as the cofactor is greater
than one, Diffie-Hellman computations using Curve4Q MUST always clear the cofactor
(i.e., multiply by 392, as explained above).


The base point G for Diffie-Hellman operations has the following affine
coordinates:

~~~~
Gx = 0x1A3472237C2FB305286592AD7B3833AA +
     0x1E1F553F2878AA9C96869FB360AC77F6\*i
Gy = 0x0E3FEE9BA120785AB924A2462BCBB287 +
     0x6E1C4AF8630E024249A7C344844C8B5C\*i
G = (X, Y)
~~~~

The tables used in multiplications of this generator (small multiples of G for
the multiplication without endomorphisms, or endomorphism images for the optimized
multiplication with endomorphisms) can be pre-generated to speed up the first,
fixed-point DH computation.

Two users, Alice and Bob, can carry out the following steps to derive a shared
key: each picks a random string of 32 bytes, mA and mB, respectively. Alice
computes the public key A = Compress([mA]\*G), and Bob computes the public key
B = Compress([mB]\*G). They exchange A and B, and then Alice computes KAB =
DH(mA, Expand(B)) while Bob computes KBA = DH(mB, Expand(A)), which produces the
shared point K = KAB = KBA. The y coordinate of K, represented as a 32 byte
string as detailed in {{representation-of-curve-points}} is the shared
secret.

Before decompressing A and B using the function Expand(), each user SHOULD verify
that the 128th bit of the received public key is zero (i.e., the non-imaginary
part of the corresponding y-coordinate should be < 2^127).

If the received strings are not valid points, the DH function has failed to
compute an answer. Implementations SHOULD return a random 32 byte string as well
as return an error, to prevent bugs when applications ignore return codes. They
MUST signal an error when decompression fails.

Implementations MAY use any method to carry out these calculations, provided
that it agrees with the above function on all inputs and failure cases, and does
not leak information about secret keys. For example, refer to the constant-time
fixed-base scalar multiplication algorithm implemented in {{FourQlib}} to
accelerate the computation of multiplications by the generator G.

# IANA Considerations

[RFC Editor: please remove this section prior to publication]
This document has no IANA actions.

# Security Considerations

The best known algorithms for the computation of discrete logarithms on Curve4Q
are parallel versions of the Pollard rho algorithm in [Distinguished]. On
Curve4Q these attacks take on the order of 2^123 group operations to compute a
single discrete logarithm. The additional endomorphisms have large order, and so
cannot be used to accelerate generic attacks. Quadratic fields are not affected
by any of the index calculus attacks used over larger extension fields.

Implementations MUST check that input points properly decompress to points on
the curve. Removing such checks may result in extremely effective attacks. The
curve is not twist-secure: implementations using single coordinate ladders MUST
validate points before operating on them. In the case of protocols that require
contributory behavior, when the identity is the output of the DH primitive it
MUST be rejected and failure signaled to higher levels. Notoriously {{?RFC5246}}
without {{?RFC7627}} is such a protocol.

Implementations MUST ensure that execution of branches and memory addresses
accessed do not depend on secret data.  The time variability introduced by
secret-dependent operations have been exploited in the past via timing and cache
attacks to break implementations.  Side-channel analysis is a constantly moving
field, and implementers must be extremely careful to ensure that operations do
not leak any secret information. Using ephemeral private scalars for each
operation (ideally, limiting the use of each private scalar to one single
operation) can reduce the impact of side-channel attacks.  However, this might
not be possible for many applications of Diffie-Hellman key agreement.

In the future quantum computers may render the discrete logarithm problem easy
on all abelian groups through Shor's algorithm. Data intended to remain
confidential for significantly extended periods of time SHOULD NOT be protected
with any primitive based on the hardness of factoring or the discrete log
problem (elliptic curve or finite field).

--- back

# Constants

~~~~~
ctau = 0x1964de2c3afad20c74dcd57cebce74c3 +
       0x000000000000000c0000000000000012 * i

ctaudual = 0x4aa740eb230586529ecaa6d9decdf034 +
           0x7ffffffffffffff40000000000000011 * i
~~~~~

~~~~~
cphi0 = 0x0000000000000005fffffffffffffff7 +
        0x2553a0759182c3294f65536cef66f81a * i
cphi1 = 0x00000000000000050000000000000007 +
        0x62c8caa0c50c62cf334d90e9e28296f9 * i
cphi2 = 0x000000000000000f0000000000000015 +
        0x78df262b6c9b5c982c2cb7154f1df391 * i
cphi3 = 0x00000000000000020000000000000003 +
        0x5084c6491d76342a92440457a7962ea4 * i
cphi4 = 0x00000000000000030000000000000003 +
        0x12440457a7962ea4a1098c923aec6855 * i
cphi5 = 0x000000000000000a000000000000000f +
        0x459195418a18c59e669b21d3c5052df3 * i
cphi6 = 0x00000000000000120000000000000018 +
        0x0b232a8314318b3ccd3643a78a0a5be7 * i
cphi7 = 0x00000000000000180000000000000023 +
        0x3963bc1c99e2ea1a66c183035f48781a * i
cphi8 = 0x00000000000000aa00000000000000f0 +
        0x1f529f860316cbe544e251582b5d0ef0 * i
cphi9 = 0x00000000000008700000000000000bef +
        0x0fd52e9cfe00375b014d3e48976e2505 * i
~~~~~

~~~~~
cpsi1 = 0x2af99e9a83d54a02edf07f4767e346ef +
        0x00000000000000de000000000000013a * i
cpsi2 = 0x00000000000000e40000000000000143 +
        0x21b8d07b99a81f034c7deb770e03f372 * i
cpsi3 = 0x00000000000000060000000000000009 +
        0x4cb26f161d7d69063a6e6abe75e73a61 * i
cpsi4 = 0x7ffffffffffffff9fffffffffffffff6 +
        0x334d90e9e28296f9c59195418a18c59e * i
~~~~~

~~~~~
L1 = 0x7fc5bb5c5ea2be5dff75682ace6a6bd66259686e09d1a7d4f
L2 = 0x38fd4b04caa6c0f8a2bd235580f468d8dd1ba1d84dd627afb
L3 = 0x0d038bf8d0bffbaf6c42bd6c965dca9029b291a33678c203c
L4 = 0x31b073877a22d841081cbdc3714983d8212e5666b77e7fdc0
~~~~~

~~~~~
b1 = [ 0x0906ff27e0a0a196, -0x1363e862c22a2da0,
       0x07426031ecc8030f, -0x084f739986b9e651]
b2 = [ 0x1d495bea84fcc2d4, -0x0000000000000001,
       0x0000000000000001,  0x25dbc5bc8dd167d0]
b3 = [ 0x17abad1d231f0302,  0x02c4211ae388da51,
      -0x2e4d21c98927c49f,  0x0a9e6f44c02ecd97]
b4 = [ 0x136e340a9108c83f,  0x3122df2dc3e0ff32,
      -0x068a49f02aa8a9b5, -0x18d5087896de0aea]
~~~~~

# Point Decompression

The following algorithm is an adaptation of the decompression algorithm
from {{SchnorrQ}}. It decodes a 32-byte string B which is formatted as
detailed in {{representation-of-curve-points}}. The result is a valid
point P = (x, y) that satisfies the curve equation, or a message of FAILED
if the decoding had a failure.

~~~~~
Sign(x0 + x1*i):
    s0 = X[0] >> 126
    s1 = X[1] >> 126
    if X[0] != 0:
        return s0
    else:
        return s1

Compress(X, Y):
    B = Y encoded following {{representation-of-curve-points}}
    Set the top bit to Sign(X)
    return B

Expand(B = [y, s]):
    Parse out the encoded values y = y0 + y1 * i and s
    if y0 or y1 >= p:
        return FAILED

    u = y^2 - 1             # Set u = u0 + u1 * i
    v = d*y^2 + 1           # Set v = v0 + v1 * i

    t0 = u0*v0 + u1*v1;
    t1 = u1*v0 - u0*v1;
    t2 = v0^2 + v1^2
    t3 = (t0^2 + t1^2)^(2^125)

    t = 2*(t0 + t3)
    if t = 0:
        t = 2*(t0 - t3)

    a = (t * t2^3)^(2^125-1)
    b = (a * t2) * t
    x0 = b/2
    x1 = (a * t2) * t1
    if t2 * b^2 != t:
        Swap x0 and x1

    x = x0 + x1 * i
    if Sign(x) != s:
      x = -x

    if -x^2+y^2 != 1+d*x^2*y^2:  # Check curve equation with x
        x = conj(x)
    if -x^2+y^2 != 1+d*x^2*y^2:  # ... or its conjugate
        return FAILED
    return P = (x,y)
~~~~~
