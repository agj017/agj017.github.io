---
layout: post
title: proof-binary-search-mathematically
date: 2024-02-15
category: mathematic
---

# proof of time complexity of binary search

## Initial Setup

- n: Total number of elements in the array.

- k: At each step, the array is divided into half.

## Steps Calculation

1. After the first division, the number of elements is `n / 2`

2. After the second division, the number of elements is `n / (2^2)`

3. After the third division, the number of elements is `n / (2^3)`

4. After the third division, the number of elements is `n / (2^k)`

## Mathematical Formulation

```
n / (2^k) = 1

2^k = n

k = log n
```

# proof of midpoint of array

## Initial Setup

- s: Start index in the range of array.

- e: End index in the range of array.

- m: Middle index in the range of array.

## Steps Calculation

1. Calculate the distance from the first index to the last index: `l - f`

2. Calculate the distance from the first index to the middle index: `(l - f) / 2`

3. Determine the middle index by adding the distance calculated in step 2 to the first index: `f + ((l - f) / 2)`

## Mathematical Formulation

```
m = f + ((l - f) / 2)

2m = 2f + l - f

2m = f + l

m = (f + l) / 2
```
