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
* [Marking the API experimental](#marking-the-api-experimental)
* [Compiling the library](#compiling-the-library)
* [Using the experimental API](#using-the-experimental-api)
  * [Propagating opt-in](#propagating-opt-in)
  * [Non-propagating opt-in](#non-propagating-opt-in)
  * [Opt-in for the whole module](#opt-in-for-the-whole-module)

<!--- END_TOC -->

## Creating a library

To create a library, go to your root directory, create a ``library.kt`` file, open it in your favorite text editor and insert the following code:

```kotlin
package combinatorics

fun factorial(num:Long):Long{
    if (num <= 1) return 1 else return factorial(num - 1) * num
}
```
This is declaration of a package containing one function.

## Creating an experimental API marker

To create an experimental API marker, create a new class annotated with
[``@Experimental``](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-experimental/index.html):

```kotlin
@Experimental
annotation class Underdog
```
For further notice, keep in mind that there are certain limitations for marker annotations:

* The EXPRESSION and FILE targets are not allowed.
* The retention must be BINARY.
* Annotations must have no parameters.

## Marking the API experimental

To mark the ``factorial()`` function experimental, annotate it with ``@Underdog`` on declaration:

```kotlin
package combinatorics

@Underdog
fun factorial(num:Long):Long{
    if (num <= 1) return 1 else return factorial(num - 1) * num
}
```
The ``Underdog`` marker will be used for opting-in to the function.
## Compiling the library

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
To compile the library, open console and run:

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

// The function uses the experimental factorial() function.
// The propagating opt-in makes the function experimental.
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
The propagating opt-in is used for both functions because they are marked with ``@Underdog`` on declaration.

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

// The function uses the experimental factorial() function.
// The propagating opt-in makes the function experimental.
@Underdog
fun permutations(n:Long, k:Long):Long{
    return factorial(n)/factorial(n-k)
}

// The function uses the experimental permutations() and factorial() functions.
// The non-propagating opt-in doesn't make the function experimental.
@UseExperimental(Underdog::class)
fun combinations(n:Long, k:Long):Long{
    return permutations(n,k)/factorial(k)
}

// The functions does not require opt-in because the combinations() function hasn't become experimental.
fun main(args: Array<String>) {
    println("The number of combinations(10,2) is ${combinations(10,2)}.")
    println("The number of combinations(20,6) is ${combinations(20,6)}.\n")
}
```
Compile and run the application to get:

```console
The number of combinations(10,2) is 45.
The number of combinations(20,6) is 38760.
```

### Opt-in for the whole module

Experimental API can be enabled for a whole module rather than opting-in each time on declarations.

Remove all the markers from your application to opt-in all functions at once:

```kotlin
import combinatorics.*

fun permutations(n:Long, k:Long):Long{
    return factorial(n)/factorial(n-k)
}

fun combinations(n:Long, k:Long):Long{
    return permutations(n,k)/factorial(k)
}

fun main(args: Array<String>) {
    println("The number of combinations(10,2) is ${combinations(10,2)}.")
    println("The number of combinations(20,6) is ${combinations(20,6)}.\n")
}
```
For non-propagating opt-in of the whole module include the name of the marker in the ``-Xuse-experimental`` option:

```console
$ kotlinc application.kt library.kt -include-runtime -d application.jar -Xuse-experimental=kotlin.Experimental,combinatorics.Underdog && java -jar application.jar
```
For propagating opt-in include the name of the marker in the ``-Xexperimental`` option:

```console
$ kotlinc application.kt library.kt -include-runtime -d application.jar -Xuse-experimental=kotlin.Experimental -Xexperimental=combinatorics.Underdog && java -jar application.jar
```
The application should run without errors.

Such an approach works in the same way as if you have added the marker directly in your code.
