---
layout: post
title: "Go Lang Notes"
date: 2012-04-08 14:50
comments: true
categories: Lang
---
## TL;DR ##
## 0. Resources ##
- [Go Dashboard](http://godashboard.appspot.com/project)
- [Go Lang SRC](http://go-lang.cat-v.org)
- [tour.golang](http://tour.golang.org), [solutions](https://gist.github.com/2317744)
- [My first Go program](https://gist.github.com/2317067)
- PaaS: GAE, Heroku

## I. Interaction with other language ##

### C ###

- [cgo](http://golang.org/cmd/cgo). Include C-libs or even valid C-code by placing these statements as comments _immediately_ above the import "C" line:

``` go
//#include <stdio.h>
//#include <stdlib.h> 
import "C"
import "unsafe"
```

- `C.uint`, `C.long`, `C.random()`:

``` go
var i int
C.uint(i)
int(C.random())
```

- Strings convertion: `C.CString(s)`, `C.GoString(cs)`:

``` go
    cstring := C.CString(gostring)
    defer C.free(unsafe.Pointer(cstring))
```

- Memory allocations made by C code are not known to Go's memory manager:

``` go
defer C.free(unsafe.Pointer(Cvariable))
```

- Pseudo **#cgo** directives:

``` go hdfs.go
// #cgo linux CFLAGS: -I/opt/jdk/include -I/opt/jdk/include/linux
// #cgo linux LDFLAGS: -Llib -lhdfs -L/opt/jdk/jre/lib/amd64/server -ljvm
// #cgo darwin LDFLAGS: -L/usr/lib/java -lhdfs -framework JavaVM
// #include "hdfs.h"
/*
int getlen(char*** ptr) {
    int i = 0;
    while (ptr[i] != NULL) ++i;
    return i;
}
int getl(char*** ptr) {
    int i = 0;
    while (ptr[0][i] != NULL) ++i;
    return i;
}
char* getstring(char*** ptr, int i, int j) {
    return ptr[i][j];
}
*/
import "C"
```
- Pointer arithmetic:

``` go
var p *C.hdfsFileInfo
p = (*C.hdfsFileInfo)(unsafe.Pointer(uintptr(unsafe.Pointer(info)) + uintptr(i)*unsafe.Sizeof(C.hdfsFileInfo{})))
```

### C++ ###

- SWIG (Simplified Wrapper and Interface Generator) supports for calling C++/C code from Go on Linux.
    - Write the SWIG interface file for the lib to be wrapped
    - SWIG generates the C stub functions
    - called using the `cgo` machinery

- SWIG does _NOT_ understand all of C++.


## II. Basic constructs and elementary data types ##

### Filenames, Keywords, Identifiers ###

- Filenames may not contain spaces or any other special chars.
- Identifiers: valid if begin with a letter (even Unicode) and followed by 0 or more letters or Unicode digits.
- `_` is _blank identifier_.
- _anonymous_
- 25 _keywords_ or reserved words:
    `break`, `case`, `chan`, `const`, `continue`, `default`, `defer`, `else`, `fallthrough`, `for`, `func`, `go`, `goto`, `if`, `import`, `interface`, `map`, `package`, `range`, `return`, `select`, `struct`, `switch`, `type`, `var`
- 36 _predeclared identifiers_:
    `append`, `bool`, `byte`, `cap`, `close`, `complex`, `complex64`, `complex128`, `copy`, `false`, `float32`, `float64`, `imag`, `int`, `int8`, `int16`, `int32`, `int64`, `iota`, `len`, `make`, `new`, `nil`, `panic`, `print`, `println`, `real`, `recover`, `string`, `true`, `uint`, `uint8`, `uint16`, `uint32`, `uint64`, `uintptr`
- Used punctuation chars: `.`,`,`,`;`,`:`,`...`
- Used delimiters: `()`, `[]`, `{}`
- Automatic semicolon insertion. However, multiple statements must be seperated by `;` on one line.

### Basic structure and components ###

#### Packages, import and visibility ####

- Every go-file belongs to one (and only one) _package_. Many different `.go` files can belong to one _package_.
- Package must be indicated on the first line. A standalone exec belongs to package **main**; each go app contains one package _main_; a package name is written in __lowercase__ letters.
- Package is compiled as a unit; each dir contains one package by convetion.
- _Every piece of code is compiled only once_.
- Apart from `_`, Ids of code-objs have to be unique in a package.
- **Visibility Rule**:

    - Id start with an uppercase, then the 'obj' with this identifier is _visible_ in code outside the package: _exported_, like _public_ in OO.
    - Id start with a lowercase are not visible outside the package: like _private_.

- alias package: `import fm "fmt"`

<!-- more -->
#### Functions ####

- `func main` must have no arguments and no return values results.
- the first `{` must be on the same line as the func-declaration.
- the last `}` is positioned after the function-code in the column beneath **f**unction.
- Every package _should_ have a _package comment_, a block comment immediately preceding the package statement.
- Nearly every top-level type, const, var and func, and certainly every exported name in a program should have a comment: _doc comment_, starts with the name.

#### Types ####

- Types can be _elementary_, _structured_, and _interfaces_.
- `nil`: value for structured type which has no real value; also the default value.
- There is no type-hierarchy.
- A function can _return more than one variable_.
- **GO is a _statically typed_ language**.
- _order of execution_

    - all packages in package main are imported in the order as indicated, in every package;
    - recursively imports packages, but a certain package is imported only once;
    - for every package (in reverse order) all constants and variables are evaluated, and `init()` if exists;
    - at last the package main; then `main()` starts executing.


#### About naming ####

- Names of things in Go should be short, concise, evocative;
- Names should not contain an indication of the package;
- Use MixedCaps or mixedCaps rather than underscores;
- No `Get...()`; but `SetName()`

#### Constants ####

- Can only be of type _boolean_, _number_, or _string_.

``` go
const identifier [type] = value // type is optinal
```

- Constants must be evaluated at compile time.
- Constants can be used for _enumerations_:

``` go
const (
    Unkonwn = 0
    Female  = 1
    Male    = 2
)
```

- So does the value `iota`:

``` go
const (
    a = iota
    b //= iota
    c //= iota
)
```
- A new const block or declaration initializes `iota` back to 0.

### Variables ###

- _all memory in Go is initialized_.
- if the variable has to be exported, it must start with a capital letter.
- variable _scope_: global/package scope; local scope.
- declaration: `var a int`, `a := 1`
- all vars of elementary types (int, float, bool, string, ...) are _value_ types, in _stack_ memory.
- more complex data which usually needs several words are treated as _reference types_.
- _pointers_ are _reference types_, as well as _slices_, _maps_, and _channels_.
- vars that are referenced are stored in the _heap_, where gc works.
- Printing:

    - `fmt.Printf()`: `%s`, `%v`
    - `fmt.Sprintf()`: return formatted string
    - `fmt.Print()`, `fmt.Println()`: automatic formatting

- `:=` can only be used inside functions, not in package scope.
- blank identifier `_` can also be used to throw away values: `_, b = 5, 7`
- `init()`:

    - every source file can contain only **1** init() function
    - initialization is always single-threaded and package dependency guarantees correct exec order
    - used to verify or repair correctness of the program state before real exec begions

### Elementary types and operators ###

- must be explicit conversion: Go is _strongly typed_
- no operator overloading
- an expression is by default evaluated from **left to right**
- format specifier:
    
    - `%t` -> booleans
    - `%d`, `%x`, `%X` -> integers
    - `%g` -> float; `%f` -> floating point, `%e` -> scientific notation; also `%n.mg`, `%n.me`, `%n.mf`
    - `%b` -> bit-representations
    - `%U` -> UTF-8 code point: _U+hhhh_ notation
    - `%p` -> pointer representation
    - `%T` -> complete type specification
    - `%#v` -> complete output of the instance with its fields

- `int`, `uint`, `uintptr` are _architecture dependent types_; but no <code><del>float</del></code> type
- **comlex number**: `real(c)`, `imag(c)`

``` go
var c1 complex64 = 5 + 10i
fmt.Printf("%v", c1)
```

- `&^` -> force a specified bit to 0
- no error is generated when an _overflow_ occurs during an arithmetic operation
- pseudo random numbers: `rand.Int()`, `rand.Intn(n)`, `rand.Float32()`, `rand.Float64()`, `rand.Seed(sd)`
- **operator precedence**:

        ^ !
        * / % << >> & &^
        + - | ^
        == != < <= >= >
        <-
        &&
        ||

- **character**: `var ch byte = '\x41'`, `var ch int = '\u0041'`, `var ch int = '\U00101234'`
- type `rune` an alias for type `int32`, for Unicode
- **strings** are a sequence of UTF-8 characters, _value_ types and _immutable_ arrays of bytes.

    - interpreted strings: surrounded by `"intr string"`
    - raw strings surrounded by `` `raw string` ``
    - length-delimited and do not terminate by a special char as in C/C++; `len(str)`
    - default value is _empty_: `""`
    - `&str[i]` is illegal
    - concatenating strings: `+`, `strings.Join()`

### `strings` and `strconv` package ###

- `strings.IndexRune(s string, ch int)` for non-ASCII character
- `strings.HasPrefix(s, prefix string)`, `strings.HasSuffix(s, suffix string)`
- `strings.Contains(s, substr string)`
- `strings.Index(s, str string)`, `strings.LastIndex(s, str string)`
- `strings.Replace(str, old, new, n)`, if `n = -1` all occurrences are replaced
- `strings.Repeat(s string, count int)`
- `strings.ToLower()`, `strings.ToUpper()`
- `strings.TrimSpace(s)`, `strings.Trim(s, "\r\n")`, `strings.TrimLeft()`, `strings.TrimRight()`
- `strings.Fields(s)`, `strings.Split(s, sep)`
- `strings.Join(s1 []string, sep string)`
- `strings.NewReader(str)` -> `r.Read()` read a []byte; `r.ReadByte()` `r.ReadRune()` read next byte or rune
- to convert a variable of a certain type `T` to a string will always succeed: `strconv.Itoa()`, `strconv.FormatFloat()`

### times and dates ###

- package **time**, datatype `time.Time`, type `Duration`, type `Location`
- `time.Now()`, `t.Day()`, `t.Add()`, etc.
- `time.After`, `time.Ticker`, `time.Sleep(Duration d)`
- `time.Unix()`

### Pointers ###

- `&` _address of_
- `*` _type modifier_, _dereference_
- cannot take the address of a literal or a constant
- _pointer arithmetic_ is **NOT** allowed
- `nil` pointer dereference is illegal


## III. Control structures and Functions ##

- `if else`

    - the `else if` and `else` keywords must be on the same line as the closing `}`
    - idiom: omit the else-clause when the if ends in a `break`, `continue`, `goto` or `return` statement
    - do not use if/else with a return in both branches
    - `if val := xxx; val > yyy { ... }`

- `switch case`

    - `break` is implicit
    - keyword `fallthrough`
    - `case val1, val2, val3:`
    - when the case-statements ends with a `return` statement, there also has to be a `return` statement after the `}` of the switch.
    - _type-switch_: [see details in interface section]

- `for (range)`

    - `for ix, val := range coll {}`
    - `val` here is a copy of the value at that index in the collection, so it can only be used in read purpose;

- Infinite loops: `for {}`

- Condition controlled iteration: `for condition {}`

- `break`/`continue`

    - a break statement always breaks out of the innermost structure in which it occurs; execution continues after the ending `}` of that structure;
    - `continue` skips the remaining part of the loop, but the next iteration.

- Label/`goto`

    - discouraged.
    - do not declare any new variables between the goto and the label.

- `select` for switching between channels

- Starting an external command or program: `os` and `os/exec` package

## IV. Arrays and Slices ##

### Arrays ###

- An array is a numbered and fixed-length sequence of data items of the same single-type.
- Length must be a constant expression, non-negative integer value; it is part of the type of the array, e.g. `[2]int`.
- Arrays are mutable.
- An array is a value type, (not a pointer to the first element like in C/C++), and can be created with `new()`.
- Array literals:

    1. `var arrAge = [5]int{1, 2, 3, 4, 5}`
    2. `var arrAge = [...]int{1, 2, 3, 4, 5}`
    3. `var arrKV = [5]string{3: "Andy", 4: "Ken"}`

- Arrays are always 1-dimensional, but may be composed to form multidimensional arrays: `[3][5]int`;
- The inner arrays have always the same length; Multidimensional arrays are _rectangular_, except _arrays of slices_.

### Slices ###

- A slice is a reference to a contiguous segment of an array (the underlying array).
- Slice is a reference type.
- Slice provide a dynamic window onto the underlying array: `slice[a:b]`, b is not included in the slice.
- A slice is a variable-length array.
- `len()`, `cap()`
- A slice in memory is in fact a struct with 3 fields: a pointer to the underlying array, the length of the slice, and the capacity of the slice.
- Create a slice with `make()`
- Slices of slices: the lengths may be vary dynamically, _jagged_.
- Inner slices must be allocated individually, with `make()`
- `func append(s []T, x ...T) []T`: allocate a new, sufficiently large slice.
- `func copy(dst, src []T) int`
- make a slice of bytes from string: `c := []byte(s)`

## V. Maps ##

- A map is a reference type: `var map1 map[keytype]valuetyep`
- The length does not have to be known at declaration, but can grow dynamically.
- Key type can be any type for which the operations `==` and `!=` are defined.
- Value type can be any type, even a func() type:

``` go
mf := map[int]func() int {
    1: func() int { return 10 },
    2: func() int { return 20 },
    5: func() int { return 50 },
}
mf[2]()
```

- Arrays, slices and structs cannot be used as key type, but pointers and interface type can.
- One way to use structs as a key is to provide them with a Key() or Hash() method.
- Map indexing is much faster than a linear search; but still 100x slower than direct array or slice indexing.
- Create maps by `make()`, never `new()`.
- Capacity: `make(map[key]value, cap)`
- Test if a key-value exists in a map: `_, exists := map1[key1]`
- Remove: `delete(map1, key1)`
- for-range to get all keys or values.
- If the value type of a map is acceptable as a key type, and the map values are unique, inverting can be done easily, via for-range.

## VI. Packages ##

- import "path or url to the package"
- By convention, each package (all the go files belonging to it) resides in its own subdirectory, which has the same name as the package.
- `import . "packa"`: use items from the package 'packa' without qualifying with the package name;
- `import _ "packa"`: imported for side-effects only, the init functions are executed and global variables initialized;
- import external packages: `go get URL`, installed at `$GOPATH/src/` or `$GOROOT/src/`, then call `go install URL`
- `import goex "codesite.ext/author/GoExample/goex"`
- Use `go doc` for custom packages

## VII. Methods ##

- Method is a function that acts an variable of a certain type, called _receiver_.
- The receiver can be almost anything: even a function type or alias type for int, bool, string or array.
- The receiver cannot be an interface type.
- The receiver cannot be a pointer type, but it can be a pointer to any of the allowed types.
- An alias of a certain type **doesn't have* the methods defined on that type.
- If the method does not need to use the value recv, discard it by `_`.
- A method and the type on which it acts must be defined in the same package. Use an _alias_ type, or `struct` instead.
- The receiver must have an explicit name, and this name must be used in the method.
- The receiver type is called the _(receiver) base type_; it must be declared within the same package as all of its methods
- If for a type T a method Meth() exists on *T and t is a variable of type T, then `t.Meth()` is automatically translated to `(&t).Meth()`.
- Pointer and value methods can both be called on pointer or non-pointer values.
- It should not be possible that the fields of an object can be changed by 2 or more different threads at the same time. Use methods of the package _sync_.
- When an anonymous type is embedded in a struct, the **visible** methods of that type are inherited by the outer type:

``` go
type Point struct {
    x, y float64
}

func (p *Point) Abs() float64 {
    return math.Sqrt(p.x*p.x + p.y*p.y)
}

type NamedPoint struct {
    Point
    name string
}

func main() {
    n := &NamedPoint{Point{3,4}, "Pythagoras"}
    fmt.Println(n.Abs())
}
```

- A method in the embedding type with the same name as a method in an embedded type overrides this.

``` go
func (n *NamedPoint) Abs() float64 {
    return n.Point.Abs() * 100
}
```

- Embedding multiple anonymous types -> _multiple inheritance_.
- Structs embedding structs from the same package have all access to one another's fields and methods.
- Embed functionality in a type:

    - _Aggregation/composition_: include a named field of the type of the wanted functionality
    - _Embedding_: anonymously embed the type of the wanted functionality

- `String()` method:
    
    - do not make mistake of defining `String()` in terms of itself -> infinite recursion.

- Summarzed: in Go types are basically classes. Go does not know inheritance like class oriented OO languages. 
- More OO capabilities: [goop](https://github.com/losalamos/goop) provides Go with JavaScript-style objects but supports multiple inheritance and type-dependent dispatch.
- Suppose special action needs to be taken right before an object _obj_ is removed from memory (gc), like writing to a log-file, achieved by:

``` go
runtime.SetFinalizer(obj, func(obj *typeObj))
```

- `SetFinalizer` does not execute when the program comes to an normal end or when an error occurs, before the object was chosen by the gc process to be removed.

## VIII. Interface ##

### Polymorphism ###

- An interface defines _abstract method set_, but cannot contain variables;
- Interface internal:

        +------------+------------+
        |            |   method   | 
        |  receiver  |   table    |
        |            |   pointer  |
        +------------+------------+

    - Thus, pointers to interface values are illegal.
    - Table of method pointers is built through runtime reflection capability.

- Multiple types can implement the same interface;
- A type that implements an interface can also have other methods;
- A type cam implements many interface;
- An interface type can contain a reference to an instance of any of the types that implement the interface: _dynamic type_;
- Interface embedding interface(s): enumerating the methods;
- type assertions:

    - `v := varI.(T) // unchecked type assertion`
    - `if v, ok := varI.(T); ok {...} // checked type assertion`
    - `varI` must be an interface variable.
    - testing if a value implements an interface.


- type switch:

``` go
switch t := var.(type) { // t can be omited
case a:
    //
case b:
    //
default:
    //
}
```

- An interface value can also be assigned to another interface value, as long as the underlying value implements the necessary methods.

- Summarized:
    - Pointer methods can be called with pointers;
    - Value methods can be called with values;
    - Value-receiver methods can be called with pointer values because they can be dereferenced first.
    - Pointer-receiver methods **cannot** be called with values, because the value stored inside an interface has no address.

### Empty Interface ###

- The _empty interface_ has no methods: `type Any interface{}`.
- Any variable, any type implements it.
- It can through assignment receive a variable of **any** type.
- Each `interface{}` takes up 2 words in memory: one word for the type, the other word for either value or pointer to value.
- Can perform templates role in container.
- copying a data-slice in a slice of interface{} [dose not work](http://code.google.com/p/go-wiki/wiki/InterfaceSlice):

``` go
var dataslice []myType = getSlice()
var interfaceSlice []interface{} = dataslice
// must be done explicitly with for-range, e.g.
```

- **Overloading** functions: `func DoSomething(f int, a ...interface{}) (n int, errno error)`

### Dynamic typing ###

- In Go, methods and data are treated _orthogonally_, loosely coupled.
- _duck typing_ in dynamic languages like Python and Ruby.
- However, the implementation requirements is _statically checked by the compiler_;

### Object-orientedness of Go: summary ###

1. Encapsulation: 
    - only 2 access-levels: _package scope_, and _exported_;
    - a type can only have methods defined in its own package;

2. Inheritance: embedding of one or more types;

3. Polymorphism: through interface.

## IX. Reflection ##

### Metaprogramming ###

``` go
func TypeOf(i interface{}) Type
func ValueOf(i interface{}) Value
```

- `Kind()` function and kind constants;
- `Interface()` recovers the (interface) value;
- Modifying a value through reflection:

    - `CanSet()` -> settability;
    - `v := reflect.ValueOf(x)` creates a copy of x;
    - to change value of x, must pass the address of x: `v := reflect.ValueOf(&x)`
    - to make it settable: `v = v.Elem()`


- Reflection on structs:

    - `NumField()` gives the number of fields in the struct;
    - call its methods with `Method(n).Call(nil)`;
    - only exported fields of a struct are settable;

### Reflection with `unsafe` ###

``` go
var byteSliceType = reflect.TypeOf(([]byte)(nil))

func AsByteSlice(x interface{}) []byte {
    v := reflect.New(reflect.TypeOf(x))
    h := *(*reflect.SliceHeader)(unsafe.Pointer(v.UnsafeAddr()))
    size := int(v.Type().Elem().Size())
    h.Len *= size
    h.Cap *= size
    return unsafe.Unreflect(byteSliceType, unsafe.Pointer(&h)).([]byte)
    // but in Go1 Unreflect are gone?
}

// or, if you prefer:

func AsByteSlice(x interface{}) []byte {
    v := reflect.New(reflect.TypeOf(x))
    h0 := (*reflect.SliceHeader)(unsafe.Pointer(v.UnsafeAddr()))
    size := int(v.Type().Elem().Size())
    h := reflect.SliceHeader{h0.Data, h0.Len * size, h0.Cap * size}
    return unsafe.Unreflect(byteSliceType, unsafe.Pointer(&h)).([]byte)
}
```

### [Turning C arrays into Go slices](http://code.google.com/p/go-wiki/wiki/cgo) ###

``` go
func C.GoBytes(cArray unsafe.Pointer, length C.int) []byte
```

To create a Go slice backed by a C array (without copying the original data), one needs to acquire this length at runtime and use `reflect.SliceHeader`.


``` go
import "C"
import "unsafe"
//...
        var theCArray *TheCType := C.getTheArray()
        length := C.getTheArrayLength()
        var theGoSlice []TheCType
        sliceHeader := (*reflect.SliceHeader)((unsafe.Pointer(&theGoSlice)))
        sliceHeader.Cap = length
        sliceHeader.Len = length
        sliceHeader.Data = uintptr(unsafe.Pointer(&theCArray[0]))
        // now theGoSlice is a normal Go slice backed by the C array
```

It is important to keep in mind that the Go garbage collector will not interact with this data, and that if it is freed from the C side of things, the behavior of any Go code using the slice is nondeterministic.

## X. Error-handling and Testing ##

- Always assign an error to a variable within a compound if-statement, making for clearer code.
- Predefined error interface type: `type error interface { Error() string }`.
- Package `errors`; function `fmt.Errorf()`.
- `syscall.Errno`; `os.EPERM`, etc.

### _defer-panic-and-recover_ machanism ###
#### Run-time exceptions and panic ####

- _runtime panic_: `runtime.Error`, `RuntimeError()`;
- _panicking_: `panic(...)`; all `defer` statements are guaranteed to execute and then control is given to the function caller, which receivers this call to panic;
- std library **Must** functions, e.g. `regexp.MustCompile`, `template.Must`, would `panic()` when encounter an error;

#### Recover ####

- `recover` is only useful when called inside a _deferred_ function: it retrieves the error value passed through the call of panic; when used in normal execution a call to recover will return nil and have no other effect.
- panic causes the stack to unwind until a deferred `recover()` is found or the program terminates.

``` go
func proect(g func()) {
    defer func() {
        log.Println("done")   
        if err := recover(); err != nil {
            log.Printf("runtime panic: %v", err)
        }
    }()
    log.Println("start")
    g()
}
```

- custom packages:

    1. _always recover from panic in your package_: no explicit panic() should be allowed to cross a package boundary;
    2. return errors as error values to the _callers of your package_.


- error-handling scheme with closures: (use a slice of empty interface as parameter and return type, to be more general)

``` go
func check(err error) { if err != nil { panic(err) } }
func errorHandler(fn fType1) fType1 {
    return func(a type1, b type2) {
        defer func() {
            if e, ok := recover().(error); ok {
                log.Printf("runtime panic: %v", err)
            }
        }()
        fn(a, b)
    }
}

func f1(a type1, b type2) {
    //...
    f, _, err := // call
    check(err)
    t, err1 := // call
    check(err1)
    //...
}

func main() {
    errorHandler(f1)
    errorHandler(f2)
    //...
}
```

### Testing and Benchmarking in Go ###

- Test-programs must be within the same package, and the files must have names of the form `*_test.go`;
- `_test` programs are **NOT** compiled with the normal Go-compilers, and not deployed in production;
- Must `import "testing"`, with global functions with names starting with `TestZzz`: `func TestAbc(t *testing.T)`
- To signal a failure:

    1. `func (t *T) Fail()`: marks the test function as having failed, but continues its execution;
    2. `func (t *T) FailNow()`: marks failed and stops its execution; execution continues with the next text **file**;
    3. `func (t *T) Log(args ...interface{})`: formatted log in the error-log;
    4. `func (t *T) Fatal(args ...interface{})`: 2+3.


- `go test` parameters: `-v` for verbose;

- Benchmarking-programs must contains functions starting with `BenchmarkZzz`: `func BenchmarkZyx(b *testing.B)`
- Table-driven tests: use an array to collect the test-inputs and the expected results together;

### Tuning and profiling ###

- `go test -x -v -cpuprofile=prof.out -file x_test.go`
- with `runtime/pprof`: 

``` go CPUProfile.go
f, err := os.Create(*cpuprofile)
pprof.StartCPUProfile(f)
defer pprof.StopCPUProfile()
f.Close()
```

``` go MemoryProfile.go
f, err := os.Create(*memprofile)
pprof.WriteHeapProfile(f)
f.Close()
```

- [gopprof](http://code.google.com/p/google-perftools/)
- [Profiling Go Programs](http://blog.golang.org/2011/06/profiling-go-programs.html)

## XI. Multithreading ##

### Goroutine ###

- GOMAXPROCS is equal to the number of (concurrent) threads, on a machine with more than 1 core, as many threads as there are cores can run in parallel.
- When the function `main()` returns, the program exits: it does not wait for other goroutines to complete.
- The logic of code must be independent of the order in which goroutines are invoked.

#### goroutines vs. coroutines ####

1. goroutines imply parallelism (or can deployed in parallel), coroutines in general do not
2. goroutines communicate via channels; coroutines communicate via yield and resume operations

### Channel: communication & synchronization ###

- Only one goroutine has access to a data item at any given time: so data races cannot occur, by design;
- `var identifier chan datatype`; reference type;
- a channel of channels of int: `chanOfChans := make(chan chan int)`
- a channel of functions: `funcChan := chan func()`
- communication operator: `<-`
- The channel _send_ and _receive_ operations are atomic: they always complete without interruption.
- *Don’t use print statements to indicate the order of sending to and receiving from a channel: this could be out of order with what actually happens due to the time lag between the print statement and the actual channel sending and receiving*.
- By default, communication is synchronous and unbuffered: sends do not complete until there is a receiver to accept the value.
- Buffered channel: `ch1 := make(chan string, buf)`
- Sending to a buffered channel will not block unless the buffer is full; reading from a buffered channel will not block unless the buffer is empty.


## XII. Network ##

- `net` [package](http://golang.org/pkg/net/).
- For web applications: `text/template`, `net/http` packages.
- `net/rpc`.

## XIII. Common Go pitfalls and mistakes ##

1. Never use the value in a for-range loop to change the value itself;
2. Never use `goto` with a preceding label;
3. Never forget `()` after a function-name, specifically when calling on a receiver or invoking a lambda function;
4. Never use `new()` with maps, always `make`;
5. Never forget to use `Flush()` when terminating buffered writing;
6. Do not use global variables or shared memory for concurrent programs;
7. Use println only for debugging purpose;

- Initialize a slice of maps the right way;
- Always use the comma, ok form for type assertions;
- Make and initialize your types with a factory;
- Use a pointer as a receiver for a method on a struct only when the method modifies the structure, otherwise use a value;

1. One should use a `bytes.Buffer` to accumulate string content, when concatenating strings.
2. Due to compiler-optimizations and depending on the size of the strings using a Buffer only starts to become more efficient when the number of iterations is > 15;
3. `defer` is only executed at the `return` of a function, not at the end of a loop or some other limited scope;
4. No need to pass a pointer to a slice to a function;
5. Never use a pointer to an interface type, this is already a pointer;
6. Use goroutines and channels only where concurrency is important;
7. Goroutines will probably not begin executing until after the loop;
8. For error handling, do not use booleans.


_To be continued_



