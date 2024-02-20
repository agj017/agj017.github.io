---
layout: post
title: thinking about static type and dynamic type
date: 2024-02-15
category: thinking
---

# Function of memory

cpu의 명령에 따라 특정 주소에 데이터를 저장한다.

## Cpu and memory

64bit cpu은 0 ~ (2^64-1) 까지의 수를 한번에 처리 할 수 있다. 그러므로 64bit cpu는 16EB의 memory까지 처리 가능하다. 왜냐하면 16EB의 memory의 가장 큰 memory address가 (2^64-1)이기 때문이다.

현재 나의 컴퓨터를 확인해보면 memory 사이즈가 8GB이므로 (2^33 - 1)까지의 memory address를 처리할 수 있다.

```shell
uname -m

x86_64
```

```shell
sysctl hw.memsize

hw.memsize: 8589934592
```

## Check address of variable

c언어를 통해서 memory에 할당 된 변수의 address를 확인 할 수 있다. 출력 된 address는 (2^33 - 1)보다 클수 없다.

- C code

```c
#include <stdio.h>

int main() {
    int num = 42;

    printf("The address of the variable 'num' is: %p\n", (void *)&num);

    return 0;
}
```

- 실행결과

```shell
The address of the variable 'num' is: 0x16fc3eed8
```

# What is type

타입이란 memory에 data를 저장하기 위한 정보이다. data를 저장하기 위해서 3가지 정보가 필요하다.

- address

- size

- method

c 코드를 assembly로 변환하여 위 3가지 정보를 확인 할 수 있다.

## Int type

- Command

```shell
gcc int.c -S -o int.s
```

- C code (input: int.c)

```c
int a = -1;
```

- Assembly code (output: int.s)

```c
	.section	__TEXT,__text,regular,pure_instructions
	.build_version macos, 13, 0	sdk_version 14, 0
	.section	__DATA,__data
	.globl	_a                              ; @a
	.p2align	2, 0x0
_a:
	.long	4294967295                      ; 0xffffffff

.subsections_via_symbols
```

- `.globl _a ; @a` -> address 저장

- `.p2align 2, 0x0` -> 2^2 byte의 공간에 0 저장

- `_a:` -> address로 접근

- `.long 4294967295 ; 0xffffffff` -> integer로 데이터 저장

## Double type

- Command

```shell
gcc double.c -S -o double.s
```

- C code (input: double.c)

```c
double a = 1;
```

- Assembly code (output: double.s)

```c
	.section	__TEXT,__text,regular,pure_instructions
	.build_version macos, 13, 0	sdk_version 14, 0
	.section	__DATA,__data
	.globl	_a                              ; @a
	.p2align	3, 0x0
_a:
	.quad	0x3ff0000000000000              ; double 1

.subsections_via_symbols
```

- `.globl _a ; @a` -> address 저장

- `.p2align 3, 0x0` -> 2^3 byte의 공간에 0 저장

- `_a:` -> address로 접근

- `.quad 0x3ff0000000000000 ; double 1` -> double 데이터 저장

# Different between static type and dynamic type

정적타입 언어와 동적타입 언어는 Type을 알아내는 시점과 방법이 다르다.

## Time

정적타입 언어는 compiler가 compile time에 type을 알아낸다. 동적타입 언어는 interpreter가 run time에 type을 알아낸다.

## Method

정적타입 언어는 아래와 같이 code에 직접 type을 명시하여 compiler에게 type 정보를 알린다. 즉, 아래의 코드의 'int'로 인해서 compiler는 해당 변수의 type을 알아낼수 있다.

```c
int a = -1;
```

동적타입 언어는 literal를 통해서 interpreter에게 type 정보를 알린다. 즉, 아래의 코드에 '-1'로 인해서 interpreter는 해당 변수의 type을 알아낼수 있다.

```javascript
const a = -1;
```
