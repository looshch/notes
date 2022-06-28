# Notes On Go
* [Types](#types)
  * [Strings](#strings)
  * [Arrays](#arrays)
  * [Slices](#slices)
  * [Maps](#maps)
  * [Interfaces](#interfaces)
  * [Structs](#structs)
* [Data](#data)
  * [Variables](#variables)
  * [Constants](#constants)
* [Functions](#functions)
* [Methods](#methods)
* [Concurrency](#concurrency)
  * [Mutexes](#mutexes)
  * [Channels](#channels)
* [Testing](#testing)
* [Standard Library](#standard-library)
  * [Formatting Functions](#formatting-functions)
  * [`Context`](#context)
* [Miscellaneous](#miscellaneous)
  * [Code Flow](#code-flow)
  * [Errors](#errors)
  * [Environment](#environment)
  * [Naming](#naming)
  * [Operators](#operators)
* [Resources](#resources)
  * [Learning](#learning)
  * [Blogs](#blogs)
    * [Posts](#posts)
* [Communities](#communities)

## Types
* for every type `T`, there is a corresponding conversion operation `T(x)` that
  converts the value `x` to type `T` if both have the same underlying type, or
  if both are unnamed pointer types that point to variables of the same
  underlying type; these conversions change the type but not the representation
  of the value. If `x` is assignable to `T`, a conversion is usually redundant.
  Conversions are also allowed between numeric types, and between string and
  some slice types. These conversions may change the representation of the
  value. For instance, converting a floating-point number to an integer discards
  any fractional part, and converting a string to a `[]byte` slice allocates a
  copy of the string data. A conversion never fails at run time. It‘s an idiom
  in Go programs to convert the type of an expression to access a different set
  of methods

* type aliases are not meant for everyday use

* the value of a constant must be of a basic type and is known to the compiler
  and whose evaluation is guaranteed to occur at compile time, not at run time.
  Only constants can be untyped. Although a constant can have any of the basic
  data types, many constants are not committed to a particular type. The
  compiler represents these uncommitted constants with much greater numeric
  precision than values of basic types, and arithmetic on them is more precise
  than machine arithmetic; at least 256 bits of precision may be assumed. There
  are six flavors of these uncommitted constants, called untyped boolean,
  integer, rune, floating-point, complex, and string. By deferring this
  commitment they can participate in many more expressions than committed
  constants without requiring conversions

* `int` is a signed integer type that is at least 32 bits in size. It is a
  distinct type, however, and not an alias for, say, `int32`, like `rune`,
  or for `uint8`, like `byte`

* aggregate types—arrays and structs—form fixed-size data types by concatenating
  of other simpler values in memory. They cannot contain itself but allow
  recursion via pointers. Reference types: pointers, slices, maps, functions,
  and channels

* ### Strings
  * strings are immutable byte slices represented in code by double quotes or
    backticks (will not escape any characters) and in memory as a 2-word
    structure containing a pointer and a length. Slicing can be done without
    allocation or copying, making string slices as efficient as passing around
    explicit indexes They can be concatenated with plus operator; a more
    efficient solution would be to use the `strings.Join` or the `bytes.Buffer`
    type. A byte and a rune are represented by single quotes

  * Go have a couple of optimizations for `[]byte` to string and string to
    `[]byte` conversions to avoid extra allocations. The first optimization
    avoids extra allocations when `[]byte` keys are used to lookup entries in
    `map[string]` collections: `m[string(key)]`. The second optimization
    avoids extra allocations in `range` clauses where strings are converted
    to `[]byte`: `for i,v := range []byte(str) {...}`. An optimizing compiler
    may be able to avoid the allocation and copying in some cases, but in
    general copying is required to ensure that the bytes of a string remain
    unchanged even if those of other string are subsequently modified. The
    conversion from byte slice back to string with `string(b)` also makes a
    copy, to ensure immutability of the resulting string. To avoid conversions
    and unnecessary memory allocation, many of the utility functions in the
    bytes package directly parallel their counterparts in the strings package

  * the `bytes` package provides the `Buffer` type for efficient manipulation of
    byte slices. A `Buffer` starts out empty but grows as data of types like
    `string`, `byte`, and `[]byte` are written to it. When appending the UTF-8
    encoding of an arbitrary rune to a `bytes.Buffer`, it’s best to use
    `bytes.Buffer`’s `WriteRune` method

* ### Arrays
  * an array variable denotes the entire array; it is not a pointer to the first
    array element

  * the size of an array must be a constant expression

  * array occupies a multiple of its element storage occupation

  * in an array literal, if an ellipsis appears in place of the length, the
    array length is determined by the number of initializers
    ```
    var q [3]int = [3]int{1, 2, 3}
    var q = [...]int{1, 2, 3}
    ```

  * it is also possible to specify a list of index and value pairs, like this
    ```
    type Currency int
    const (
        USD Currency = iota
        EUR
        GBP
        RMB
    )
    symbol := [...]string{USD: "$", EUR: "€", GBP: "£", RMB: "¥"}
    fmt.Println(RMB, symbol[RMB]) // "3 ¥"
    ```
    in this form, indices can appear in any order and some may be omitted.
	Unspecified values take on the zero value for the element type

  * each access to the Nth element requires the compiler to insert a bounds
    checks so it’s better to avoid accessing the same element

* ### Slices
  * slice is a 3-word run-time data structure holding the pointer, length, and
    capacity and is called slice header. The pointer points to the first element
    of the array that is reachable through the slice

  * re-slicing a slice doesn’t make a copy of the underlying array. The full
    array will be kept in memory until it is no longer referenced. This can
	cause the program to hold all the data in memory when only a small piece of
	it is needed

  * `a[low:high:max]` constructs a slice of the same type, and with the same
    length and elements as the simple slice expression `a[low:high]`.
    Additionally, it controls the resulting slice’s capacity by setting it to
    max-low. Only the first index may be omitted; it defaults to 0

  * slices are not comparable. `bytes.Equal` is highly optimized function for
    comparison, but for other types, the comparison must be done manually. Deep
    equivalence is problematic because elements of a slice are indirect, making
    it possible for a slice to contain itself and/or different elements at
    different times

  * to test whether a slice is empty, `len(s) == 0` is used, not `s == nil`

  * unless clearly documented to the contrary, Go functions should treat all
    zero-length slices the same way, whether nil or non-nil

  * when declaring an empty slice, prefer `var t []string` over
    `t := []string{}`

  * `copy` will copy bytes from a string to a slice of bytes and it is legal to
    append a string to a byte slice

* ### Maps
  * the map capacity can be specified on creation, but `cap()` can‘t be used on
    maps

  * a key in a map can be of any type for which the equality operator is
    defined. Though floating-point numbers are comparable, it’s a bad idea to
    compare floats for equality and especially bad if `NaN` is a possible value

  * a map element is not a variable, and its address cannot be taken because
    rehashing can invalidate the address

  * the order of map iteration is unspecified what helps force programs to be
    robust across implementations

  * individual struct value fields can’t be updated in a map because map
    elements are not addressable but slice elements are addressable. One can use
    a temporary variable or a map of pointers

  * maps are not thread safe and require mutexes. All operations upon maps
    should be wrapped in setters, getters, and deleters

* ### Interfaces
  * the bigger the interface, the weaker the abstraction

  * pair inside an interface always has the form (value, concrete type) and
    cannot have the form (value, interface type). Interfaces do not hold
    interface values

  * an interface value is `nil` only if the `V` and `T` are both unset,
    (`T=nil`, `V` is not set). If there is a nil pointer of type `*int` inside
    an interface value, the inner type will be `*int`

  * when embedded methods are invoked the receiver of the method is the inner
    type

  * to guarantee that type satisfies an interface the compiler can be asked to
    check that the type `T` implements the interface `I` by attempting an
    assignment using the zero value for `T` or pointer to `T`, as appropriate
    ```
    type T struct{}
    var _ I = T{}       // verify that T implements I
    var _ I = (*T)(nil) // verify that *T implements I
    ```
    if there is a need to explicitly declare that an interface users implement
    it, a method can be added with a descriptive name to the interface’s method
    set
    ```
    type Fooer interface {
        Foo()
        ImplementsFooer()
    }
    ```
    sometimes they’re necessary to resolve ambiguities among similar interfaces

  * there is no point in creating interfaces with only one implementation: it’s
    unnecessary abstractions; they also have a run-time cost. There can be an
    exception to this rule when an interface is satisfied by a single concrete
    type but that type cannot live in the same package as the interface because
    of its dependencies

  * `any` is an alias for `interface{}`

* ### Structs
  *  ```
     employeeOfTheMonth.Position += " (proactive team player)"
     ```
     is equivalent to
     ```
     (*employeeOfTheMonth).Position += " (proactive team player)"
      ```

  * there should be not more than one tag per field

  * tags shouldn’t be ignored if a library uses them

  * the struct fields starting with lowercase letters will not be encoded, so
    when you decode the structure you’ll end up with zero values in those
    unexported fields

  * if there is a need to refer to an embedded field directly, the type name of
    the field, ignoring the package qualifier, serves as a field name
    ```
    type Job struct {
        Command string
        *log.Logger
    }
    func (job *Job) Printf(format string, args ...interface{}) {
        job.Logger.Printf("%q: %s", job.Command, fmt.Sprintf(format, args...))
    }
    ```

  * embedded types should be at the top of the field list of a struct, and there
    must be an empty line separating embedded fields from regular fields

  * empty structs don’t occupy storage and can be used for signaling between
    channels

  * named fields should be always used in struct literals
    ```
      type s struct {
          a int
    +     b string
      }

      func main() {
          t := T{123}                  // doesn't compile
	  t := T{a: 123, b: "example"} // compile
      }
    ```

## Data
* all indexing in Go include the first index but exclude the last one

* `new(T)` creates an unnamed variable of type `T`, allocates zeroed storage
  and returns `*T`. There are almost always easier or cleaner ways to write a
  program without it

* `make()` creates slices, maps and channels only, and it returns an initialized
  (not zeroed) value of type `T`\
  *how to remember*: `make` creates Go-specific structures (slices and channels)
  and **ma**ps

* capacity should be specified where possible

* `cap` returns the channel buffer capacity

* `len` returns the length of `v`, according to its type:
  * string: the number of bytes in `v`
  * channel: the number of elements queued (unread) in the channel buffer

* if the result of an arithmetic operation, whether signed or unsigned, has more
  bits than can be represented in the result type, it is said to overflow. The
  high-order bits that do not fit are silently discarded

* `float64` should be preferred for most purposes because `float32` computations
  accumulate error rapidly unless one is quite careful. Digits may be omitted
  before the decimal point (`.707`) or after it (`1.`). Very small or very large
  numbers are better written in scientific notation

* taking a pointer to a struct or an array places it in heap memory rather than
  stack where it would normally be

* ### Variables
  * if a variable is not explicitly initialized, it is implicitly initialized to
    the zero value for its type: `0` for numbers, `false` for booleans, `""` for
    strings, and `nil` for other types

  * a short variable declaration acts like an assignment only to variables that
    were already declared in the same lexical block; declarations in an outer
    block are ignored. This form can’t be used in declarations to set field
    values and a variable can’t be redeclared in a standalone statement, but it
    is allowed in multi-variable declarations where at least one new variable is
    also declared

  * it’s perfectly OK to return the address of a local variable

  * not every value has an address, but every variable does

  * ```
    var global *int
    func f() {
        var x int
        x = 1
        global = &x
    }
    func g() {
        y := new(int)
        *y = 1
    }
    ```
    `x` must be heap-allocated because it escapes from `f`. Conversely, when `g`
    returns, the variable `*y` becomes unreachable and can be recycled, so it’s
    safe for the compiler to allocate `*y` on the stack

* ### Constants
  * when a sequence of constants is declared as a group, the right-hand side
    expression may be omitted for all but the first of the group, implying that
    the previous expression and its type should be used again. For example
    ```
    const (
        a = 1
        b
        c = 1
        d
    )
    fmt.Println(a, b, c, d) // 1 1 2 2
    ```

  * enumerated constants are created using the `iota` enumerator. Values of
    `iota` are same in one line. Since `iota` can be part of an expression and
    expressions can be implicitly repeated, one can do something like this
    ```
    type ByteSize float64
    const (
        _           = iota
        KB ByteSize = 1 << (10 * iota)
        MB
        ...
    ```
    because variables have a 0 default value

  * `String` method may provide a meaningful name instead of an integer
    ```
    type A int
    const (
        B A = iota
	...
    func (i A) String() {
        switch i {
        case B:
	    return "B"
	...
    ```

  * adding (in this case) lowercase constants `..._beg` and `..._end` can help
    to determine whether a given constant belongs to a subgroup
    ```
    const (
        literal_beg
        A
        B
        literal_end

        keyword_beg
        C
        keyword_end
    )
    func (tok Token) IsLiteral() bool {
        return literal_beg < tok && tok < literal_end
    }
    ```

## Functions
* function receives copies of arguments

* when an anonymous function requires recursion, we must first declare a
  variable, and then assign the anonymous function to that variable. Had these
  two steps been combined in the declaration, the function literal would not be
  within the scope of the variable holding the value of itself so it would have
  no way to call itself recursively

* named functions can be declared only at package level

* when function return variables are named, they are initialized to the zero
  values for their types when the function begins

* the arguments to the deferred function are evaluated when the defer executes

* deferred functions can modify named return values

* consider what it will look like in `godoc`. Named result parameters like
  ```
  func (n *Node) Parent1() (node *Node) {}
  func (n *Node) Parent2() (node *Node, err error) {}
  ```
  will be repetitive in `godoc`; better use
  ```
  func (n *Node) Parent1() *Node {}
  func (n *Node) Parent2() (*Node, error) {}
  ```
  on the other hand, if a function returns two or three parameters of the same
  type, or if the meaning of a result isn’t clear from context, adding names may
  be useful
  ```
  func (f *Foo) Location() (float64, float64, error)
  ```
  is less clear than
  ```
  // Location returns f’s latitude and longitude.
  // Negative values mean south and west, respectively.
  func (f *Foo) Location() (lat, long float64, err error)
  ```
  naked returns are OK if the function is a handful of lines. Once it’s a
  medium sized function, return values must be explicit. Clarity of docs is
  always more important than saving a line or two in your function. In some
  cases you need to name a result parameter in order to change it in a deferred
  closure. That is always OK

* repetitive logic should be extracted to `with...Context` wrappers
  ```
  func foo() {
      mu.Lock()
      defer mu.Unlock()
      ...

  func bar() {
      mu.Lock()
      defer mu.Unlock()
      ...
  ```
  ```
  func withLockContext(fn func()) {
      mu.Lock()
      defer mu.Unlock()

      f()
  }
  ```

## Methods
* by convention, one-method interfaces are named by the method name plus an
  ‘-er’ suffix, even when it’s not a proper English

* name of a method’s receiver should be often a one or two letter abbreviation
  of its type and can be omitted if unused

* if type implements a method with the same meaning as a method on a well-known
  type, the same name and signature should be given to it; string-converter
  method should be called `String` not `ToString`

* methods can be defined for any named type except a pointer or an interface
  because in cases like this one
  ```
  type T int
  func (t *T) Get() T {
      return *t + 1
  }
  type P *T
  func (p P) Get() T {
      return *p + 2
  }
  func F() {
      var v1 T
      var v2 = &v1
      var v3 P = &v1
      fmt.Println(v1.Get(), v2.Get(), v3.Get())
  }
  ```
  it’s unclear which `Get()` method should be called

* if any method of `T` has a pointer receiver, then all methods of `T` should
  have a pointer receiver, even ones that don’t strictly need it

* if the receiver argument is a variable of type `T` and the receiver parameter
  has type `*T`, the compiler implicitly takes the address of the variable. If
  the receiver argument has type `*T` and the receiver parameter has type `T`,
  the compiler implicitly dereferences the receiver

* `nil` is a valid receiver value. When you define a type whose methods allow
  `nil` as a receiver value, it’s worth pointing this out explicitly in its
  documentation comment

* ```
  func (f foo) method() // -> func (foo) method() if unused
  ```

## Concurrency
* pointers should be used to return results instead of channels to reduce the
  number of channels to manage

* errors should be emitted through a `<-chan error`, so blocking operations
  could be waited to complete before returning responses

* when there is a need to break out of from a `for`-`select` idiom, one needs to
  use labels
  ```
  F:
      for {
          select {
          case ...
          default:
              break F
          }
      }
  ```
  wrapping it into a function eliminates this need and provides an ability to
  return an error
  ```
  func foo() {
      for {
          select {
          case ...
          default:
              return
          }
      }
  }
  ...
  if err := foo(); err != nil ...
  ```

* ### Mutexes
  * a set of exported functions encapsulates one or more variables so that the
    only way to access the variables is through these functions (or methods, for
    the variables of an object). Each function acquires a mutex lock at the
    beginning and releases it at the end, thereby ensuring that the shared
    variables are not accessed concurrently. This arrangement of functions,
    mutex lock, and variables is called a monitor. This older use of the word
    ‘monitor’ inspired the term ‘monitor goroutine’. Both uses share the meaning
    of a broker that ensures variables are accessed sequentially

  * by convention, the variables guarded by a mutex are declared immediately
    after the declaration of the mutex itself. Deviating from this needs to be
    documented

  * * here, `rateMu` is a mutex hat. It sits, like a hat, on top of the
      variables that it protects, without needing to write a comment
      ```
      struct {
          ...

          rateMu     sync.Mutex
          rateLimits [categories]Rate
          mostRecent rateLimitCategory
      }
      ```

    * when adding a new, unrelated field that isn’t protected by `rateMu`, add
      an empty line like this
      ```
      struct {
          ...

          rateMu     sync.Mutex
          rateLimits [categories]Rate
          mostRecent rateLimitCategory
        +
        + common service
      }
      ```

    * a mutex shouldn’t be embedded on a struct so its methods are not part
      of the struct

* ### Channels
  * sending to a closed channel causes a panic but receiving from it returns
    zero value and sending and receiving from a nil channel blocks forever

  * channels should be always closed in producers and not in consumers

  * channels should usually have a size of one or be unbuffered. The choice
    between unbuffered and buffered channels, and the choice of a buffered
    channel’s capacity, may both affect the correctness of a program. Unbuffered
    channels give stronger synchronization guarantees because every send
    operation is synchronized with its corresponding receive; with buffered
    channels, these operations are decoupled. Also, when an upper bound on the
    number of values that will be sent on a channel is known, it’s not unusual
    to create a buffered channel of that size and perform all the sends before
    the first value is received. Failure to allocate sufficient buffer capacity
    would cause the program to deadlock

  * synchronization:
    * when a function executes `<–c`, it will wait for a value to be sent
    * similarly, when a function executes `c <– value`, it waits for a receiver
      to be ready
    * a sender and receiver must both be ready to play their part in the
      communication. Otherwise we wait until they are
    thus channels both communicate and synchronize

## Testing
* within `*_test.go` files, three kinds of functions are treated specially:
  tests, benchmarks, and examples. A test function, which is a function whose
  name begins with `Test`, exercises some program logic for correct behavior;
  go test calls the test function and reports the result, which is either `PASS`
  or `FAIL`. A benchmark function has a name beginning with `Benchmark` and
  measures the performance of some operation; `go test` reports the mean
  execution time of the operation. And an example function, whose name starts
  with `Example`, provides machine-checked documentation

* each test file must import the testing package.
  `TestFunctionName(t *testing.T)` should be called for each tested function.
   The `t` parameter provides methods for reporting test failures and logging
   additional information

* if the example function contains a final `// Output:` comment like the one
  above, the test driver will execute the function and check that what it
  printed to its standard output matches the text within the comment

* tests can be run in parallel with `t.Parallel()` inside the second argument
  of `t.Run`

* test outputs should output the actual value that the function returned before
  printing the value that was expected. A usual format for printing test outputs
  is `Func(%v) = %v, want %v`

* even after your test cases encounter a failure, they should keep going for as
  long as possible in order to print out all of the failed checks in a single
  run. On a practical level, prefer calling `t.Error` over `t.Fatal`. When
  comparing several different properties of a function's output, use `t.Error`
  for each of those comparisions

* failures that affect a single entry in the test table, which make it
  impossible to continue with that entry, should be reported as follows:
  * if you’re not using `t.Run` subtests, you should use `t.Error` followed by
    a continue statement to move on to the next table entry
  * if you’re using subtests (and you’re inside a call to `t.Run`), then
    `t.Fatal` ends the current subtest and allows your test case to progress to
    the next subtest, so use `t.Fatal`

* a test helper is a function that performs a setup or teardown task, such as
  constructing an input message, that does not depend on the code under test.
  If you pass a `*testing.T`, call `t.Helper` to attribute failures in the test
  helper to the line where the helper is called

* use `cmp.Equal` for equality comparison and `cmp.Diff` to obtain a
  human-readable diff between objects

## Standard Library
* the `golang.org/x/...` repositories are part of the Go Project but outside
  the main Go tree. They are developed under looser compatibility requirements
  than the Go core

* Go programs use `os.Exit` or `log.Fatal*` to exit immediately. `os.Exit` or
  `log.Fatal*` should be called in `main()` only and at most once in a `main()`,
  if possible

* `Must` denotes that it doesn’t return an error because it mustn’t

* `sync.Once` for executing code only once

* ```
  type authHandler struct {
      next http.Handler
  }
  func (h *authHandler) ServeHTTP(w http.ResponseWriter, r  *http.Request) {
     ...
     h.next.ServeHTTP(w, r)
  }
  func MustAuth(handler http.Handler) http.Handler {
      return &authHandler{next: handler}
  }
  ```

* ### Formatting Functions
  * by convention, formatting functions whose names end in ‘-f’, such as
    `log.Printf` and `fmt.Errorf`, use the formatting rules of `fmt.Errorf`,
    whereas those whose names end in ‘-ln’ follow `Println`, formatting their
    arguments as if by `%v`, followed by a newline

  * usually a `Printf` format string containing multiple `%` verbs would require
    the same number of extra operands, but the `[1]` ‘adverbs’ after `%` tell
    `Printf` to use the first operand over and over again. Second, the `#`
    adverb for `%o` or `%x` or `%X` tells `Printf` to emit a `0` or 0x or `0X`
    prefix respectively. Runes are printed with `%c`, or with `%q` if quoting is
    desired

  * when declaring format strings for `Printf`-style functions outside a string
    literal, they should be const values. This helps perform static analysis of
    the format string

* ### `Context`
  * context package serves two primary purposes
    * to provide an API for canceling branches of your call-graph
    * to provide a data-bag for transporting request-scoped data through your
      call-graph

  * most functions that use a `Context` should accept it as their first
    parameter

  * values of the `context.Context` type carry security credentials, tracing
    information, deadlines, and cancellation signals across API and process
    boundaries. Go programs pass Contexts explicitly along the entire function
    call chain from incoming RPCs and HTTP requests to outgoing requests

  * `context.Context` values should only live in function arguments, never
    stored in a field or global. A `Context` member shouldn’t be added to a
    struct type; instead a `ctx` parameter should be used in each method on that
    type that needs to pass it along. The one exception is for methods whose
    signature must match an interface in the standard library or in a third
    party library

  * contexts are used to pass information into deeper layers of the code, not
    otherwise

## Miscellaneous
* standard library packages should be separated from third-party ones by an
  empty line in imports

* the iteration variables in `for` statements are reused in each iteration. This
  means that each closure created in your for loop will reference the same
  variable (and they’ll get that variable’s value at the time those goroutines
  start executing). The easiest solution is to save the current iteration
  variable value in a local variable inside the loop block. Also a variable can
  be passed as a parameter to an anonymous goroutine

* the zero value should be useful

* running `go vet` for all of sub-packages of code by `go vet ./...` uses
  heuristics that do not guarantee all reports are genuine problems, but it can
  find errors not caught by the compilers

* Go reuses the XOR operator (`^`) for unary NOT. Go also has a special
  ‘AND NOT’ bitwise operator (`&^`)

* errors and too small objects or similar structures should be eliminated,
  common case should be effortless

* if need be, the Pacer slows down allocation while speeding up marking. At a
  high level the Pacer stops the Goroutine, which is doing a lot of the
  allocation, and puts it to work doing marking. The amount of work is
  proportional to the Goroutine’s allocation. This speeds up the garbage
  collector while slowing down the mutator

* to limit the rate of operations per unit time, use a `time.Ticker`. This works
  well for rates up to tens of operations per second. For higher rates, prefer a
  token bucket rate limiter such as golang.org/x/time/rate.Limiter (also search
  `godoc` for rate limit)

* a great rule of thumb is accept interfaces, return structs

* for configuration, instead of accepting arguments for all possible settings,
  or a configuration object, or splitting a function in different ones for
  different use cases, accept a variadic argument of configuration functions
  which are functions operating on a subject of configuration and exported from
  the API provider package. They can be wrappers around internal methods, or
  just return another functions, or anything else. This provides:
  * sensible defaults
  * high configurability
  * ability for API to grow over time

*  ```
   // The loop condition is < instead of <= so that the last byte does not
   // have a zero distance to itself. Finding this byte out of place implies
   // that it is not in the last position.

   // ... The zero value is ready to use.
   // Do not copy a non-zero Builder.

   // Compare is included only for symmetry with package bytes.
   // It is usually clearer and always faster to use the built-in
   // string comparison operators ==, <, >, and so on.
   func Compare(a, b string) int {
       // NOTE(rsc): This function does NOT call the runtime cmpstring function,
       // because we do not want to provide any performance justification for
       // using strings.Compare. Basically no one should use strings.Compare.
       // As the comment above says, it is here only for symmetry with package
       // bytes. If performance is important, the compiler should be changed to
       // recognize the pattern so that all code doing three-way comparisons,
       // not just code using strings.Compare, can benefit.

   func CopyBuffer(dst Writer, src Reader, buf []byte) (written int64, err error) {
       if buf != nil && len(buf) == 0 {
           panic("empty buffer in CopyBuffer")
       }
       return copyBuffer(dst, src, buf)
   }
   // copyBuffer is the actual implementation ...
   func copyBuffer(dst Writer, src Reader, buf []byte) (written int64, err error) {

   nr, er := src.Read(buf)
   nw, ew := dst.Write(buf[0:nr])

   type LimitedReader struct {
       R        Reader // underlying reader
       N        int64  // max bytes remaining
       lastByte int    // last byte read for UnreadByte; -1 means invalid

   func NewReader(rd io.Reader) *Reader {
   // NewReaderSize returns a new Reader whose buffer has at least the specified
   func NewReaderSize(rd io.Reader, size int) *Reader {

   // discard implements ReaderFrom as an optimization so Copy to
   // io.Discard can avoid doing unnecessary work.
   var _ ReaderFrom = discard{}

   // Reader is the interface that ...

   // ErrShortWrite means that ...
   // EOF is the error returned by ... when ...

   tmpl   map[string]*Template // Map from name to defined templates.
   muTmpl sync.RWMutex         // protects tmpl
   ```

* ### Code Flow
  * at the beginning of a file: errors, consts

  * define var/consts as close to usage points as possible

  * in var/const definition, exported first separated by an empty line

* ### Errors
  * error types should be of the form `FooError` and error values should be of
    the form `ErrFoo`

  * * `errors.New` for static error messages
    * top-level `var` with `errors.New` for static ones when one must be matched
    * `fmt.Errorf` for dynamic ones
    * custom `error` type for dynamic ones when one must be matched
    * error matching means using `errors.Is` or `errors.As`

  * for error values stored as global variables, use the prefix `err`.
    For custom error types, use the suffix `error` instead

* ### Environment
  * Go works in its own workspace. It should contain `src` directory for sources
    and `bin` for binaries. `src` may contain multiple repositories. If repo
    like github.com is used, code must be putted under directory called
    `github.com`. Here is what a workspace may look like
    ```
    ├── bin             executables
    ├── src/github.com  repo
    │        └── a      user
    │            └── a  package
    └── pkg             compiled files
    ```
    if project uses modules, it can be placed outside of `src`

  * * `/cmd` — public applications (binaries)
    * `/app` — main application
    * `/internal` — private applications and library code which aren’t intented
      to be imported in another projects
    * `/pkg` — library code which could be imported in another projects
    * `/vendor` — dependencies; needed if project is not a library
    * `/api` — API specs and schemas, protocol definitions
    * `/web` — assets, templates, SPAs
    * `/configs` — configs, examples and templates for them
    * `/scripts` — scripts to build, install, run, etc.
    * `/build`
      * `/package` — configs for containers, clouds, package managers, etc.
      * `/ci` — configs for CI/CD
    * `/deployments` — configs for *aaS (docker-compose, k8s, helm, terraform,
       etc.)
    * `/test` — external tests and test data
    * `/doc` — design and user docs
    * `/tools` — supporting tools
    * `/examples` — usage examples
    * `/third_party` — external supporting tools, forks
    * `/assets` — images, logos, etc.
    * `/website` — project website

  * any comment immediately preceding a top-level declaration serves as a doc
    comment for that declaration. Every exported entity should have one. The
    first sentence should be a summary that starts with the name being declared.
    Line comments are the norm; block comments appear mostly as package
    comments. A single doc comment can introduce a group of related constants or
    variables

  * `godoc` displays indented text in a fixed-width font

  * variables:
    * GOGC controls the aggressiveness of the garbage collector. Equals 100 by
      default
    * GOMAXPROCS controls the number of operating system threads allocated to
      goroutines

* ### Naming
  * if there is a field called `owner`, the getter method should be called
    `Owner`, not `GetOwner`. A setter function, if needed, will likely be called
    `SetOwner`

  * the letters of acronyms and initialisms like ASCII and HTML are always
    rendered in the same case, so a function might be called `htmlEscape`,
    `HTMLEscape`, or `escapeHTML`, but not `escapeHtml`

  * * `explode` splits `s` into a slice of UTF-8 strings
    * `Count` counts the number of ...

  * for brands or words with more than 1 capital letter, lowercase all letters
    if not exported
    ```
    var oauthEnabled

    var OAuthEnabled
    ```

* ### Operators
  * the operator precedence hierarchy is reflected by the spacing

  * the sign of the remainder is always the same as the sign of the dividend, so
    `-5%3` and `-5%-3` are both `-2`

  * `%` operator applies only to integers

  * the behavious of `/` depends on whether its operands are integers, so
    `5.0/4.0` is `1.25`, but `5/4` is `1` because integer devision truncates the
    result toward zero

  * left shifts fill the vacated bits with zeros, as do right shifts of unsigned
    numbers, but right shifts of signed numbers fill the vacated bits with
    copies of the sign bit. For this reason, it is important to use unsigned
    arithmetic when you’re treating an integer as a bit pattern

* ### Garbage Collector
  * garbage collector is based on the tricolor mark-and-sweep algorithm. It can
    work concurrently with the program and uses a write barrier. It divides the
    objects of the heap into three different sets according to their color
    * the objects in the white set can have pointers to the objects of the black
      set and are candidates for garbage collection
    * the objects of the grey set might have pointers to some objects of the
      white set. The roots are the objects that can be directly accessed by the
      application, which includes global variables and other things on the stack
    * the objects of the black set are guaranteed to have no direct pointers to
      any object of the white set. No object can go directly from the black set
      to the white set, which allows the algorithm to be able to clear the
      objects in the white set
    when the garbage collection begins, all objects are white and the garbage
    collector visits all of the root objects and colors them grey. After this,
    the garbage collector picks a grey object, makes it black until the end of
    the search, and starts searching to determine if that object has pointers
    to other objects of the white set. If that scan discovers that this
    particular object has one or more pointers to a white object, it puts that
    white object in the grey set. This process keeps going for as long as
    objects exist in the grey set. After that, the objects in the white set are
    unreachable and their memory space can be reused. If an object of the grey
    set becomes unreachable at some point in a garbage collection cycle, it will
    not be collected in that garbage collection cycle but rather in the next one
    (not efficient). During this process, the running application is called the
    mutator. The mutator runs a small function named write barrier that is
    executed each time a pointer in the heap is modified. If the pointer of an
    object in the heap is modified, which means that this object is now
    reachable, the write barrier colors it grey and puts it in the grey set. The
    mutator is responsible for the invariant that no element of the black set
    has a pointer to an element of the white set. This is accomplished with the
    help of the write barrier function. When the garbage collector finds out
    that a channel is unreachable and that the channel variable can no longer be
    accessed, it will free its resources even if the channel has not been
    closed. Go allows to initiate a garbage collection manually by putting a
    `runtime.GC()` statement what will block the caller. The Go scheduler is
    responsible for the scheduling of the application and the garbage collector

## Resources
* [Standard library](https://pkg.go.dev/std)
* [Specification](https://go.dev/ref/spec)
* [Effective Go](https://go.dev/doc/effective_go)
* [Wiki](https://github.com/golang/go/wiki)
* [Gopher Reading List](https://github.com/enocom/gopher-reading-list)
* [Go Proverbs](https://go-proverbs.github.io)

* ### Learning
  * [Tour of Go](https://go.dev/tour)
  * [The Go Programming Language](https://www.gopl.io)
  * [Go by Example](https://gobyexample.com)
  * [Learn Go with Tests](https://quii.gitbook.io/learn-go-with-tests/)

* ### Blogs
  * [The Go Blog](https://go.dev/blog)
  * [Russ Cox](https://research.swtch.com)
  * [Dave Cheney (The acme of foolishness)](https://dave.cheney.net/category/golang)
  * [Three Dots Labs](https://threedots.tech)
  * [Go go-to guide](https://yourbasic.org/golang)
  * [Peter Bourgon](https://peter.bourgon.org/blog)
  * [Mat Ryer (Pace)](https://pace.dev/blog)
  * [Ardan labs](https://www.ardanlabs.com/blog/)
  * [Eli Bendersky](https://eli.thegreenplace.net/tag/go)
  * [Ilija Eftimov](https://ieftimov.com/posts)
  * [Golang By Example](https://golangbyexample.com)
  * #### Posts
    * [Parallelize your table-driven tests](https://rakyll.org/parallelize-test-tables/)
    * [50 Shades of Go](http://devs.cloudimmunity.com/gotchas-and-common-mistakes-in-go-golang/)

## Communities
  * [golang-nuts (mailing list)](https://groups.google.com/g/golang-nuts)
  * [Golang Weekly (newsletter)](https://golangweekly.com)
