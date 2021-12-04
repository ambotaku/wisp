# wisp

## A light lisp written in C++
![Wisp](./assets/wisp.png)

## Why write a lisp for microcontrollers ?

I ported Adam McDaniel's Wisp to get a small interactive and embeddable language for modern popular microcontrollers 
like the Raspberry Pi Pico or ESP32. I know there are MicroPython, CircuitPython and Lua available for those, 
but creating native libraries (C/C++) code is is not only elaborate, but debugging possibilities are awful.

I tried several small Scheme and Lisp interpreters written in C, but due to missing support for dynamic data
structures in plain C such code gets difficult to understand especially when creating a LISP-like language, which was created for data abstraction on a high level. 

Adam's code is using STL libraries like <string>, <vector>, <map> and <sstream> 
which near LISP conditions and make the source code easy to understand and maintain.

Fortunately Adam isolated STL libraries like <iostream> and <fstream> which depend on an operating system in his code 
by a define USE_STD, also the <math> functions are optionals by a define NO_LIBM_SUPPORT. That is very useful for porting thw code to microcontrollers which have neither an operating system nor float arithmetics.

So currently this project has two branches:
  - the "main" branch allows easy extending, running and testing on a linux system
  - the "Pico" branch runs on a Raspberry Pi Pico microcontroller with an attached terminal emulator

  Comfortable C++ Source code debugging is possible with a second Pi Pico as "probe" or a Raspberry Pi4 desktop,
  but uploading takes sume time, since the STL- libraries require more memory as plain C code.

Currently I just ported Adam's code without extra functionality, so I just attached Adam's language descriptions.
Due to the lack of an OS and mass storage for the controller, currently only the REPL can be used.
## Syntax and Special Forms

Like every other lisp, this language uses s-expressions for code syntax and data syntax. So, for example, the s-expression `(print 5)` is both a valid code snippet, and a valid list containing the items `print` and `5`.

When the data `(print 5)` is evaluated by the interpreter, it evaluates `printli` and `5`, and then applies `print` to `5`. 

Here's the result.

```lisp
>>> (print 5)
5
 => 5
```

That's super cool! But what if we want to define our own functions? _We can use the builtin function `defun`!_

```lisp
; define a function `fact` that takes an argument `n`
(defun fact (n)
  (if (<= n 1)
     1
     (* n (fact (- n 1)))
   ))
```

Thats awesome! But did you notice anything different about the `defun` function? _It doesn't evaluate its arguments._ If the atom `fact` were evaluated, it would throw an error like so:

```lisp
>>> fact
error: the expression `fact` failed in scope { } with message "atom not defined"
```

This is known as a special form, where certain functions "quote" their arguments. We can quote things ourselves too, but the language _automatically_ quotes arguments to special forms itself.

If you want to "quote" a value yourself, you can do it like this.

```lisp
; quote the s-expression (1 2 3) so it's not evaluated
>>> (print '(1 2 3))
(1 2 3)
 => (1 2 3)
```

As you can see, quote negates an evaluation. For example, whenever the expression `''a` is evaluated, it becomes `'a`. This can be useful for when you want to write long lists of data or variable names without wanting to evaluate them as code.

|Special Form|Argument Evaluations|Purpose|
|:-|-|-|
|`(if cond a b)`|`if` only evaluates its `cond` argument. If `cond` is truthy (non-zero), then `a` is evaluated. Otherwise, `b` is evaluated.|This special form is the main method of control flow.|
|`(do a b c ...)`|`do` takes a list of s-expressions and evaluates them in the order they were given (in the current scope), and then returns the result of the last s-expression.|This special form allows lambda functions to have multi-step bodies.|
|`(scope a b c ...)`|`scope` takes a list of s-expressions and evaluates them in the order they were given _in a new scope_, and then returns the result of the last s-expression.|This special form allows the user to evaluate blocks of code in new scopes.|
|`(defun name params body)`|`defun` evaluates none of its arguments.|This special form allows the user to conveniently define functions.|
|`(define name value)`|`define` evaluates the `value` argument, which is then assigned to `name` in the current scope.|This special form allows the user to bind atoms to values in a scope.|
|`(lambda params body)`|`lambda` evaluates none of its arguments.|This special form allows the user to define anonymous functions.|
|`(quote x)`|`quote` evaluates none of its arguments.|This is equivalent to the `'expr` syntactic sugar.|
|`(for x list ...)`|`for` evaluates only its list argument.|`for` iterates through the list storing each element in `x`, and then evaluating all of the rest of the values in the `for` body. It then returns the last value evaluated.|
|`(while cond ...)`|`while` evaluates only its cond argument.|`while` evaluates its condition expression every iteration before running. If it is true, it continues to evaluate every expression in the `while` body. It then returns the last value evaluated.|


## Examples

Here are some example math-y functions to wrap your head around.

```lisp
; quicksort
(defun qs (l)
    (if (<= (len l) 1)
        l
        (do
            (define pivot (first l))
            (+
                (qs (filter (lambda (n) (> pivot n)) l))
                (list pivot)
                (qs (tail (filter (lambda (n) (<= pivot n)) l)))
            ))
    ))

; decrement a number
(defun dec (n) (- n 1))
; increment a number
(defun inc (n) (+ n 1))
; not a bool
(defun not (x) (if x 0 1))

; negate a number
(defun neg (n) (- 0 n))

; is a number positive?
(defun is-pos? (n) (> n 0))
; is a number negative?
(defun is-neg? (n) (< n 0))
```

## Usage

Using and compiling wisp

#### Dependencies

Compile with your C++ compiler of choice. This is compatible with all standard versions of C++ since ANSI C++.
To build wisp for microcontrollers like the Raspberry Pi Pico you need a matching toolchain / SDK.

Learn more with [Getting started with RaspberryPi Pico](https://datasheets.raspberrypi.com/pico/getting-started-with-pico.pdf)

To support different target builds **cmake** is used. For a complete toolchain Microsoft Visual Code (VSCode)
is supported by settings in folder **.vscode**.

```bash
$ git clone https://github.com/ambotaku/wisp.git  # to get the main branch (Linux)
$ cd wisp
$ git checkout Pico # for getting the branch running on Raspberry Pi Pico (RP2040)
$ mkdir build # create a build directory
$ cd build # enter build directory
$ cmake .. # configure build target
$ make     # build the target
```
For the Linux branch just move the **wisp** command elsewhere or start it from the *build* folder.
For Raspberry Pi Pico copy the **wisp.uf2** installer onto the device or run the device with a probe
attached to its debugging port.

By using VSCode (enter //code .// in the project workspace) all operations can be automated by that IDE.

#### Debugging
For the Linux branch just use GDB (via  VSCode), the Raspberry Pi Pico needs a "probe" connected to its debug connector. Such a probe can be a Raspberry Pi or another Raspberry Pi Pico programmed with **PicoProbe** software.
See Appendix A in [Getting started with RaspberryPi Pico](https://datasheets.raspberrypi.com/pico/getting-started-with-pico.pdf)

#### Using the binary

By uncommenting the line
```#add_definitions(-DUSE_STD -DHAS_LIBM)```
in file **CMakeLists.txt** you can build a *Wisp* executable that has nearly all features of Adam's original release when using it on a Linux system.

That line is commented to remove any operating system dependencies (as needed for microcontrollers). 
Under Linux such a build should behave like being run on a controller and only the REPL mode is available.
