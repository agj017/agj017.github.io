---
layout: post
title: proof-euclidean-method-of-mutual-division
date: 2024-05-13
category: mathematic
---

# proof of euclidean method of mutual division

## Initial Setup

- a: an interger number

- m: divisor of a

- b: an interger number

- n: divisor of b

- d: greatest common divisor of a and b

- r: a remainder of a and b

- gcd(): a function returning greatest common divisor

## Steps Calculation

1. 22 % 8 = 6.

2. 8 % 6 = 2.

3. 6 % 2 = 0.

4. The greatest common divisor is 2.

## Mathematical Formulation

```
If r = a % b, gcd(a, b) = gcd(b, r)

a = d*m and b = d*n

r = a - q*b

r = d*m - q*d*n

r = d*(m - q*n)
```
