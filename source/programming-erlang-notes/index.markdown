---
layout: page
title: "programming erlang notes"
date: 2012-05-23 14:39
comments: true
sharing: true
footer: true
---

## 0. Building ##
``` bash configure on os x
./configure --prefix=/opt --disable-debug --enable-kernel-poll --enable-threads --enable-dynamic-ssl-lib --enable-shared-zlib --enable-smp-support --enable-darwin-64bit --with-dynamic-trace=dtrace --enable-hipe
```

## Basics ##
### Variables ###
- All variable names _must_ start with an uppercase letter.
- In Erlang, `=` denotes a _pattern maching_ operation.

### Atoms ###
- atoms are used to represent different non-numerical constant values.
- atoms starts with lowercase letters, followed by a sequence of alphanumeric characters or `_`, or `@`.
- atmos can also be quoted with a single quotation mark.

### Tuples ###
- similar to structs in C.
- the fields of a tuple have no names.
- `Person = {person, {name, joe}, {height, 1.82}}`
- extraction: `{person, X, Y} = Person`, `{_,{_,Who},_} = Person`

### Lists ###
- created by enclosing the list elements in square brackets and seperating them with commas.
- if `T` is a list, then `[H|T]` is also a list, with head `H` and tail `T`. `[]` is the empty list.
- `[H1, H2, ..., Hn | T]`
- extraction: `[What|Left] = TheList`

### Strings ###
- _Strings_ are really just lists of integers.
- Strings are enclosed in double quotation marks.
- dollar syntax: `[I-32,$u,$r,$p,$r,$i,$s,$e]`

### Pattern Matching ###
- `f()` tells the shell to forget any bindings it has.


## Sequential Programming ##
_chapter 3, 5_

## Exceptions ##
_chapter 4_

## Comipling and Running ##
_chapter 6_

## Concurrenct Programming ##
_chapter 7, 8, 9_

## Distributed Programming ##
_chapter 10_

## Interfacing Techniques ##
_chapter 12_

## Programming with Files/Sockets ##
_chapter 13, 14_

## OTP ##
_chapter 16, 18_

## Multicore Programming ##
_chapter 19, 20_