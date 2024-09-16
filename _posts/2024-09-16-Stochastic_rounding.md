---
title: The nice surprise with stochastic rounding
date: 2024-09-16 3:52:01 +0600
categories: [Math,Numerical Analysis]
tags: [math, programming]     # TAG names should always be lowercase
math: true
---

For readers unfamiliar with stochastic rounding, I recommend the brief but highly informative [late Nick Higham's article here](https://nhigham.com/2020/07/07/what-is-stochastic-rounding/).

What you should take away from that article is that:

1. Regardless of the order in which you perform addition using stochastic floats, the *expectation* in the result is invariant.

1. The above means that the *expectation* of stochastic float addition is associative!

NB: I emphasize the fact that the expectation of result is invariant with respect to the order in which floating point operations are performed and not stochasting-floating-point arithmetic being associative is because it's not, this is due to the inherent indeterminism.  

I repeat, merely the expectation of stochastastic-floating-point arithmetic is associative, not stochastic-rounding itself, this is an important distinction.

It is known that IEEE 754 floating point arithmetic is not associative, this leads to some constraints such as the order in which you perform operations matters and inhibits some optimizations which can be done.  

The following may be a surprise to some readers:

$$
\text{Let } F \text{ be the set of floating point numbers} \\

\text{Let } a,b,c \in F \\

\begin{equation*}
    a + (b + c) \ne (a + b) + c
\end{equation*}
$$

As defined by IEEE 754, floating-point addition associates to the left.

```cpp
float result = a + b + c + d;
// This is interpretted like so by the computer, since floating point arithmetic is left-associative.
float result = ((a + b) + c) + d;
```

Notice we are at most capable of operating on two floats at once.  If floating point arithmetic were associative we could utilize SIMD and instead do the following:

```cpp
float result = (a + b) + (c + d);
```

The above permits us to operate on four floating point numbers simultaneously with a single vector instruction leading to two instructions executed in total (because we can replace two floating point operations using a single vector instruction).  

In contrast, the former example requires three floating point instructions to compute.  This performance desparity is only further exacerbated by use of even wider vector instructions and larger floating point sums.

SIMD is highly important when writing performant code, so floating point sums can pose a bottle neck as it's non-associative property means that SIMD cannot be fully utilized (if at all).

Since the expected value of stochastic-floating-point arithmetic is associative, we can leverage SIMD.  The expected value in error-accumulation is zero.

This seemingly insignificant property of the associativity of stochastic-rounding seems to have been forgone by many authors as it seems ungoogleable thus far.  Perhaps it is due to the fact that the practical implementation of such algorithms is not yet possible.  Maybe because multiple accumulators are just better in every way?    

I've asked Bing's search engine and as of the publication of this article [no one has seem to mentioned this seemingly innocuous fact](https://www.bing.com/chat?sendquery=1&q=Articles%20that%20mention%20stochastic%20rounding%20being%20associative&form=HECODX).