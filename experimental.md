# Working with experimental API

Kotlin 1.3 introduced annotations for marking an experimental API and opting-in for it. When creating an API that is unstable or may break compatibility, developers can mark the API experimental. Users have to explicitly choose an opt-in flag when referring to this API. Without an opt-in, usage of the API will cause warnings or errors.

An API is considered experimental if it is annotated with an **experimental API marker**—a class which is in turn annotated with [``@Experimental``](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-experimental/index.html).

There are two mechanisms to opt-in to the experimental API:

* On the **propagating opt-in** the API becomes experimental itself, so it requires opt-in in the further usages.
* On the **non-propagating opt-in** the API does not become experimental, so it does not require opt-in in the further usages.

In either case, after opt-in the API can use other experimental APIs annotated with the same marker.

This guide provides an example of creating, compiling and working with an experimental library using the Command Line Compiler.
It also illustrates the difference between propagating and non-propagating opting-in.

## Table of contents

<!--- TOC -->

* [Creating a library](#creating-a-library)
* [Creating an experimental API marker](#creating-an-experimental-api-marker)
* [Compiling a library](#compiling-a-library)
* [Using the experimental API](#using-the-experimental-api)
  * [Propagating opt-in](#propagating-opt-in)
  * [Non-propagating opt-in](#non-propagating-opt-in)
* [Opt-in for a whole module](#opt-in-for-a-whole-module)

<!--- END_TOC -->

## Creating a library

To create a library, go to your root directory, create a ``library.kt`` file, open it in your favorite text editor and insert the following code:

```kotlin
package combinatorics

@Underdog
fun factorial(num:Long):Long{
    if (num <= 1) return 1 else return factorial(num - 1) * num
}
```
This is a simple library containing one function. Annotating the declared function with ``@Underdog`` marks it experimental.

## Creating an experimental API marker

To create an experimental API marker, create a new class annotated with
[``@Experimental``](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-experimental/index.html):

```kotlin
@Experimental
annotation class Underdog
```

The ``Underdog`` marker will be used for opting-in to the experimental API.

For further notice, keep in mind that there are certain limitations for marker annotations:

* The EXPRESSION and FILE targets are not allowed.
* The retention must be BINARY.
* Annotations must have no parameters.

## Compiling a library

Considering two previous steps, the library looks like:

```kotlin
package combinatorics

@Experimental
annotation class Underdog

@Underdog
fun factorial(num:Long):Long{
    if (num <= 1) return 1 else return factorial(num - 1) * num
}
```

To compile a library, open console and run:

```console
$ kotlinc library.kt -d library.jar -Xuse-experimental=kotlin.Experimental
```

The ``-Xuse-experimental=kotlin.Experimental`` option is required since the experimental API feature is
experimental itself. For information on the other CLI options run:

```console
$ kotlinc -help
```

## Using the experimental API

To create an application that uses the experimental ``factorial()`` function, create an ``application.kt`` file and open it in your favorite text editor.

### Propagating opt-in

To use the experimental ``factorial()`` function with the propagating opt-in, insert the following code to your application:

```kotlin
import combinatorics.*

// The function uses the factorial() function from the library.
// The propagating opt-in makes the function experimental itself.
@Underdog
fun permutations(n:Long, k:Long):Long{
    return factorial(n)/factorial(n-k)
}

// The function uses the factorial() function from the library and the permutations() function that
// became experimental.
// As the function is not used anywhere else, a non-propagating opt-in could be used as well.
@Underdog
fun main(args: Array<String>) {
    println("0 factorial is ${factorial(0)}.")
    println("10 factorial is ${factorial(10)}.\n")

    println("The number of permutations(9,4) is ${permutations(9,4)}.")
    println("The number of permutations(12,42) is ${permutations(12,42)}.\n")

}
```

The propagating opt-in is used for both functions by marking them with ``@Underdog`` on declaration.

To compile and run the application, try:

```console
$ kotlinc application.kt library.kt -include-runtime -d application.jar -Xuse-experimental=kotlin.Experimental && java -jar application.jar

0 factorial is 1.
10 factorial is 3628800.

The number of permutations(9,4) is 3024.
The number of permutations(12,42) is 479001600.
```
Without opt-in the compiler would report ``this declaration is experimental and its usage must be marked with '@combinatorics.Underdog' or '@UseExperimental(combinatorics.Underdog::class)`` error.

### Non-propagating opt-in

To use the experimental ``factorial()`` function with the non-propagating opt-in, insert the following code to your application:

```kotlin
import combinatorics.*

// The function uses the factorial() function from the library.
// The non-propagating opt-in doesn't make the function experimental.
@UseExperimental(Underdog::class)
fun combinations(n:Long, k:Long):Long{
    return permutations(n,k)/factorial(k)
}

// The functions does not require opt-in because the combinations() function hasn't become experimental.
fun main(args: Array<String>) {
    println("The number of combinations(10,2) is ${combinations(10,2)}.")
    println("The number of combinations(22,6) is ${combinations(22,6)}.\n")
}
```

Compile and run the application to get:

```console
The number of combinations(10,2) is 45.
The number of combinations(22,6) is 38760.
```

## Opt-in for a whole module

Experimental API can be enabled for a whole module rather than opting-in each time on declarations.

For propagating opt-in of the whole ``library.kt`` module, include a name of the marker in the ``-Xexperimental`` option:

```console
$ kotlinc application.kt library.kt -include-runtime -d application.jar -Xexperimental=kotlin.Experimental,combinatorics.Underdog
```
For non-propagating opt-in include a name of the marker in the ``-Xuse-experimental`` option:

```console
$ kotlinc application.kt library.kt -include-runtime -d application.jar -Xuse-experimental=kotlin.Experimental,combinatorics.Underdog
```

Such an approach works in the same way as if you have added the marker directly in your code.