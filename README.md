# Class 5

## (Tail) Recursion

> In order to understand recursion, you must first understand recursion.

*Recursion* is when a subroutine is called from within itself.

Example:

```scala
def fac(n: Int): Int =
  if (n == 0) 1 else n * fac(n - 1)
```

Note that recursion requires a stack-based subroutine calling protocol.

What are some advantages and disadvantages of using recursion?

* *Advantages*: often conceptually easier; yields easier to understand
  code (compared to using loops).

* *Disadvantages*: usually slower due to overhead of executing call
  sequences; can lead to stack overflow.

There is one case when recursion can be implemented without using a
new activation record for every recursive call:

A *tail-recursive* subroutine is one in which no additional
computation ever follows a recursive call.

For tail-recursive subroutines, the compiler can *reuse* the current
activation record at the time of the recursive call, eliminating the
need to allocate a new one. This optimization is called *tail call
elimination*. 

Compilers for many modern programming languages perform tail call
elimination. This feature is particularly prevalent in compilers for
functional languages because functional programs tend to use recursion
in favor of loops.

Example:

```scala
def fac(n: Int): Int =
  if (n == 0) 1 else n * fac(n - 1)
```

This function is not tail-recursive, because the recursive call to
`fac` appears below the multiplication with `n`. Hence, there is still
work to be done when the recursive call returns. In particular, we
need to remember the current value of `n` until after the recursive
call. Hence, the current activation record must remain on the stack
and cannot be reused for the recursive call.

However, we can rewrite this function so that it becomes
tail-recursive:

```scala
def facTail(acc: Int, n: Int): Int =
  if (n == 0) acc else facTail(n * acc, n - 1)

def fac(n: Int): Int = facTail(1, n)
```
  
The Scala compiler will detect the tail recursion and performs
tail-call elimination, ensuring that `facLoop` and hence `fac` run in
constant stack space. Effectively, tail call elimination yields an
implementation that is equivalent to one that uses a loop:

```scala
def facLoop(acc0: Int, n0: Int): Int = {
  var acc = acc0
  var n = n0
  
  while (n != 0) {
    acc = n * acc
    n = n - 1
  }

  return acc
}

def fac(n: Int): Int = facLoop(1, n)
```

except that the tail-recursive code is simpler and does not use side
effects.

## Functional Programming

*Functional Programming* refers to a programming style in which every
procudure is *functional*, i.e. it computes a function of its
inputs with no side effects.

Functional programming languages are based on this idea, but they also
provide a number of interesting features that are often missing in
imperative languages.

One of the most important and interesting features of functional
programming languages is that functions are *first-class* values.  The
means that programs can create new functions at run-time. This can be
leveraged to build powerful *higher-order* functions: a higher-order
function either takes a function as an argument or returns a function
as a result (or both). Many modern programming languages and
frameworks draw heavily from these ideas, including Google's
Map/Reduce and TensorFlow frameworks as well as Apache Spark.

Functional languages in turn draw heavily on the *λ-calculus* (lambda
calculus) for inspiration. To understand how functional programming
works in general, it is therefore helpful to first study this very
basic calculus.

### Lambda Calculus

The lambda calculus was invented by Alonzo Church in 1932 as a model of
computation. It is the basis for functional languages including
e.g. Lisp, Scheme, SML, OCaml, and Haskell. It has typed and untyped
variants with different syntax and reduction rules.

We will discuss the *pure*, *untyped* variant of the calculus.

#### Syntax

The syntax is simple: a *term* *t* in the lambda calculus is either

* a function with formal parameter *x* and body *t'*: *λ x. t'*
* an application of a function *t₁* to an argument *t₂*: *t₁ t₂* 
* a variable: *x*

We can use parentheses to indicate grouping but omit them when intend
is clear. In particular 

* *λ x y. t* is a shorthand for *λ x. (λ y. t)* 
* *t₁ t₂ t₃* is a shorthand for *(t₁ t₂) t₃*

That is, we encode multi-argument functions using nested lambda terms
(*currying*) and function application is left-associative.

In a term *λ x. t*, the scope of *x* is *t*.  We also say that the
outer *λ x* *binds* *x* in *t*, respectively, that *x* is bound in *λ
x. t*. A variable that occurs in a term *t* without being bound is
called *free*. A term that has no free variables is called *closed*.

Example:

*(λ x. (λ y. x (z y))) y*

* *z* is free
* the last occurrence of *y* is free
* *x* and the remaining occurrence of *y* are bound

Here is a simple way to determine whether a particular using
occurrence of a variable in a lambda term is free or bound and if it
is bound, where it is bound. Take the lambda term from the example
above. It helps a lot to look at the term in its abstract syntax tree
representation (i.e. answering these questions is similar to answering
static scoping questions in general for programming languages). Here
is the abstract syntax tree for the above term:

```
    app
   /   \
  λ x   y
   |
  λ y
   |
  app
 /   \
x    app
    /   \
   z     y
```

A subtree rooted in a node labeled `app` represents a term that starts
with a function application (with the subtree rooted at the left child
of that node representing the function being applied and the right
child subtree the argument term). A subtree whose root node is labeled
with `λx` is a lambda abstraction term (with the subtree rooted in the
child of that node representing the body of the function).

Now, note that all the using occurrences of variables in the term are
the ones that label the leaves of the tree. To determine whether a
particular occurrence is free, we start from the corresponding leaf
node and walk the tree upwards towards the root. If we come through a
node labeled by λ that binds the variable labeling the leaf node we
started from, then that occurrence is bound by that λ. Note that the
first λ that binds the variable starting from the leaf is the one that
the leaf refers to. So, in the example, if we start from the leaf
labeled by `y` that is below the nested `app` nodes, then the first λ
that we encounter on the way to the root binds that `y`. So this
occurrence of `y` is bound. On the other hand, if we start from the
right-most leaf that is labeled by `y`, then there is no λ at all on
the path to the root, so this occurrence is free. Similarly, the
left-most leaf note, which is labeled by `x` is a bound occurrence. It
is bound by the second λ on the path to the root. The leaf note
labeled by `z` is again a free occurrence, because there is no `λz` on
the path to the root.

#### Alpha-renaming

We can perform consistent renaming of bound variables at will. This is
referred to as α-conversion or α-renaming:

*(λ x. x)* =α= *(λ y. y)*

*(λ y. x y)* =α= *(λ z. x z)*

However, *(λ x. x x)* is not a valid α-renaming of *(λ y. x y)*
because *x* occurs free in the latter but not in the former.

More generally, α-renaming must obey the following rules:

* Only bound variable occurrences can be renamed, not free
  occurrences.
  
* Renaming must be consistent: if we rename *x* in a subterm *(λ
  x. t)* to *y*, all free occurrences of *x* in *t* must be replaced
  by *y*.
  
* Renaming must be *capture-avoiding*: if we rename *x* to *y* in *(λ
  x. t)* then for every subterm *t'* of *t*, if *t'* has a free
  occurrence of *x* that is bound by the outer λ in *(λ x. t)* before
  the renaming, then *y* must be free in the term obtained from *t'*
  after the renaming. Moreover, if *y* already occurs free in *t*
  before the renaming, then renaming *x* to *y* is disallowed.
  
  So, e.g. renaming *x* to *y* in *(λ x. (λ y. y x))* to obtain *(λ
  x. (λ y. y y))* is disallowed because *x* occurs free in the subterm
  *(λ y. y x)* but *y* does not occur free in *(λ y. y y)*. We say
  that *y* has been *captured* by the *binder* *λ y*. Capturing can be
  avoided by first α-renaming all conflicting bound variables in
  subterms. For instance, in the example, we can first α-rename the
  nested bound variable *y* to *z*, obtaining the term *(λ x. (λ z. z
  x))* and then α-rename *x* to *y*, obtaining *(λ y. (λ z. z y))*.
  

### Evaluation via Reduction

The main reduction rule in the lambda calculus is *β-reduction*,
which formalizes the idea of function application:

*(λ x. t) s → t[s/x]*

The notation *t[s/x]* stands for the term *t* (i.e. the body of the
function being applied) where all **free** occurrences of *x* are
substituted by the term *s*. Note that the free occurrences of *x* in
*t* are exactly those occurrences that refer to the parameter of the
function.

Restriction: *t* should not have any free variables that are
*captured* (i.e. bound by λ's) when we do the substitution in *t*. We
can always use α-renaming of bound variables in *t* to comply with
this restriction.

An expression that cannot be β-reduced any further is in *normal form*.

Examples:

*(λ x. x y) (λ y. y)*
*→ (λ y. y) y*

In the above example, we have *t = (x y)*, *s=(λ y. y)* and *t[s/x]=(λ y. y) y*.

The following example needs α-renaming before performing the β-reduction step:

*(λ x y. x y) (λ x. y)*

*= (λ x. (λ y. x y)) (λ x. y)*

*=α= (λ x. (λ z. x z)) (λ x. y)*  alpha-renaming of *y* to *z*

*→ λ z. (λ x. y) z*

The α-renaming step is needed to obtain deep binding semantics. To see
this more easily, we can translate the lambda term of the last example
into the syntax of a more conventional programming language where we
introduce a named function for every lambda term. We here use
JavaScript because we are in an untyped setting.

```javascript
function f(x) {
  function g(y) { return x(y); }
  return g;
}
function h(x) { return y; }
f(h)
```

The β-reduction step from the example corresponds to the execution of
the function call `f(h)` in the JavaScript code. Note that the body of
`h` refers to the variable `y` (which we assume is defined somewhere
else in the scope where this code appears). Without the α-renaming
step, we would essentially replace `x` in the body of the function `g`
by `h` where we rebind the variable `y` in the body of `h` to the
parameter `y` of `g`. This would give us shallow binding semantics. If
we want to have deep binding semantics, we first need to rename the
formal parameter `y` of `g` to something else, say `z`, so that the
free variable of `h` is not "captured" by that formal parameter.


#### Evaluation strategies

We have the β-reduction rule, but if we have a complex expression, where
should we apply it first?

Consider the following example:

*(λ x. (λ y. y x x)) ((λ x. x) (λ y. z))*

There are two possible places (*redexes*) where β-reduction can be
applied in this term:

1. The function is *(λ x. (λ y. y x x))* and the argument is *((λ
   x. x) (λ y. z))* (i.e. *t = (λ y. y x x)* in and *s=((λ x. x) (λ y. z))*).

2. The function is *(λ x. x)* and the argument is *(λ y. z)* (i.e. *t = x* and *s=(λ y. z)*).

Two popular strategies:

* *normal-order*: reduce the outermost *redex* first (case 1. above):

  *(λ x. (λ y. y x x)) ((λ x. x) (λ y. z)) →
    (λ y. y ((λ x. x) (λ y. z)) ((λ x. x) (λ y. z))*

* *applicative-order*: arguments to a function application are
  evaluated first, from left to right before the function application
  itself is evaluated (case 2. above):

  *(λ x.(λ y. y x x)) ((λ x. x) (λ y. z)) → (λ x (λ y. y x x)) (λ y. z)*

Does the evaluation order actually matter?

**Church-Rosser Theorem**

If a term *t* can be reduced (in 0 or more steps) to terms *t₁* and
*t₂*, then there exists a term *t'* such that both *t₁* and *t₂* can
be reduced to *t'*.

The property expressed in the Church-Rosser theorem is called
*confluence*.  We say that β-reduction is *confluent*.

An important corollary of the theorem is that every term has at most
one normal form.

Why at most one?

There are some terms with no normal form!

Example:

*(λ x. x x) (λ x. x x)*

If we apply β-reduction to this term, we obtain exactly the same
term. So we can keep on applying β-reduction again and again ad
infinitum:

*(λ x. x x) (λ x. x x)*
*→ (λ x. x x) (λ x. x x)*
*→ (λ x. x x) (λ x. x x)*
...

Note that most functional programming languages are not purely
functional (i.e. they allow computations that have side effects such
as variable assignment). In those cases, evaluation order **does
matter**. Also, in general the evaluation order matters for
termination, i.e., whether a normal form is actually computed if it
exists.

For instance, consider the following expression:

*(λ x. (λ x. x)) ((λ x. x x) (λ x. x x))*

The normal form of this expression is *λ x. x* which can be computed
by normal-order. However, applicative-order will keep on reducing the
expression *(λ x. x x) (λ x. x x)* without ever making progress.

Applicative-order closely corresponds to call-by-value parameter
passing and normal-order with call-by-name parameter passing.

#### Computability

Computability theory studies the question "What is computable?". The
notion of *computability* relies on formal models of computation.

Many formal models have been proposed:

* Functions computable by Turing machines (Turing)
* General recursive functions defined by means of an equation calculus (Goedel-Herbrand-Kleene)
* μ-recursive functions and partial recursive functions (Goedel-Kleene)
* Functions defined from canonical deduction systems (Post)
* Functions given by certain algorithms over finite alphabets (Markov)
* Universal Register Machine-computable functions (Shepherdson-Sturgis)
* ...
* Any function you can write in your favorite programming language

**Fundamental Result:** all of these (and many other) models of
computation are equivalent. That is, they give rise to the same class
of computable functions.

Any such model of computation is said to be *Turing complete*.

Alan Turing showed in 1937 that the untyped lambda calculus is Turing complete.
But how can this be?

* There are no built-in types other than "functions" (e.g., no Booleans, integers, etc.).
* There are no imperative features like assignments.
* There are no recursive definitions or looping constructs.

It turns out that the expressive power of the calculus stems from the
fact that we can treat functions as data. It is all just a matter of
finding the right encoding of concepts like numbers and Booleans using
lambda terms.

##### Church Encodings

When we talk about data types such as integers and Booleans in a
programming language, it is important to distinguish the abstract
mathematical concept behind these data types (e.g. integer numbers),
from the representation of these concepts in the language
(e.g. numerals that stand for integer numbers).

Example:

* 15
* fifteen
* XV
* `0F`

These are different numerals that all represent the same number.


To show that concepts such as integers and Booleans and their standard
operations can be expressed in the lambda calculus, we simply have to
agree on a particular representation of these concepts using functions
encoded as lambda terms. The trick is to pick a representation of data
types that lends itself well to implementing the standard operations
on these data types by using the representation itself for performing
the involved computation.

Let us start with Booleans. How can we represent *`true`* and *`false`* in
the lambda calculus? One reasonable definition is as follows:

* *`true`* is a function that takes two values and returns the first, i.e.

  *`true` = (λ x y. x)*

* *`false`* is a function that takes two values and returns the second, i.e.

  *`false` = (λ x y. y)*

The intuition behind this definition is that we can think of *`true`*
and *`false`* as implementing the two branches of a conditional
expression. That is, if we now define

*`ite` = (λ b x y. b x y)*

then *`ite`* is a ternary function that implements a conditional
expression, provided its first argument *b* is always either *`true`* or
*`false`* as defined above. Convince yourself that

*`ite` b x y*

evaluates to *x* if *b* is *`true`* and to *y* if *b* is *`false`*.

Using these definitions, we can now define the standard logical
operations on Booleans:

* *`and` = (λ a b. a b `false`)*

* *`or` = (λ a b. a `true` b)*

* *`not` = (λ b x y. b y x)*

Using a similar idea, one can encode natural numbers using lambda
terms.  The number *n* is represented by a function which maps a
successor function *s* and a zero element *z* to *n* applications of
*s* to *z*: *s (s ... (s z) ...)*

* *`0` = (λ s z. z)*

* *`1` = (λ s z. s z)*

* *`2` = (λ s z. s (s z))*

* *`3` = (λ s z. s (s (s z)))*

* ...

Note that *`0`* and *`false`* are actually the same lambda terms
modulo α-renaming. As is common, we use the same object to mean
different things in different contexts.

Here are the encodings of some operations on natural numbers:

* zero test: *`iszero` = (λ n. n (λ x. `false`) `true`)*
* addition by one (successor):  *`succ` = (λ n s z. s (n s z))*
* addition: *`plus` = (λ m n. m `succ` n)*
* multiplication: *`mult` = (λ m n. m (`plus` n) `0`)*
* exponentiation: *`exp` = (λ m n. n m)*

For example, the definition of *`plus`* essentially says: apply *`succ`*
*m*-times to `n`. In particular, if we take the term *`plus 3 2`*, then
this β-reduces to the term: *`succ` (`succ` (`succ` `2`))*. Since *`succ`*
encodes addition by one, this term thus computes *`5`*. Here is a
complete evaluation of the term *`plus` `1` `2`*, which shows that the
normal form obtained from this term is *`3`*:

*`plus` `1` `2`*

*= (λ m n. m `succ` n)  `1` `2`*  ```; Definition of plus```

*→ (λ n. `1` `succ` n) `2`*

*→ `1` `succ` `2`*

*= (λ s z. s z) `succ` `2`* ```; Definition of 1```

*→ (λ z. `succ` z) `2`*

*→ `succ` `2`*

*= (λ n s z. s (n s z)) `2`* ```; Definition of succ```

*→ (λ s z. s (`2` s z))* 

*= (λ s z. s ((λ s z. s (s z)) s z))* ```; Definition of 2```

*→ (λ s z. s ((λ z. s (s z)) z))*

*→ (λ s z. s (s (s z)))*

*= `3`* ```; Definition of 3```

The other operators work similarly. For instance, definition of *`mult`*
says: apply *(`plus` n)* *m*-times to *`0`*. In particular, the term
*`mult` `3` `2`* reduces to the term

*`plus` `2` (`plus` `2` (`plus` `2` `0`))*

which further reduces to `6`.

Encoding subtraction is a bit more involved. We proceed analogous to
the case for addition and first define a function *`pred`* that
encodes subtraction by one. The idea for *`pred`* is to define a
function that given the number *n*, uses *n* to enumerate all pairs of
numbers *(i, i - 1)* for *i* between 1 and *n*. Then projection on the
second component of the last pair *(n, n - 1)* gives us *n - 1*.

Let's start by defining functions for constructing and decomposing pairs:

* *`pair` = (λ x y b. b x y)*
* *`fst` = (λ p. p `true`)*
* *`snd` = (λ p. p `false`)*

Note that we have *`fst` (`pair` x y) → ... → x* and *`snd` (`pair` x y) → ... → y*.

Then subtraction by one can be defined as

*`pred` = λ n. `snd` (n (λ p. `pair` (`succ` (`fst` p)) (`fst` p)) (`pair` `0` `0`))*

Using `pred` we can now define subtraction

*`minus` = λ m n. n `pred` m*

##### Expressing Recursion

How can we express recursion in the lambda calculus?

As an example, we will implement the factorial function *n!* in the
lambda calculus. Let us start from the following Scala definition of factorial

```scala
def fac(n: Int): Int = 
  if (n == 0) 1 else n * fac(n - 1)
```

First, we do a literal translation of this definition into the
lambda calculus, making use of the operations and constants we have
defined so far:

*`fac` = (λ n. `ite` (`isZero` n) `1` (`mult` n (`fac` (`minus` n `1`))))*

This is still a recursive definition, which we can't express directly
in the lambda calculus. So let's do a simple trick and define a new
function *`Fac`* that takes the function *`fac`* that we need for the
recursive call as an additional input:

*`Fac` = (λ fac n. `ite` (`isZero` n) `1` (`mult` n (fac (`minus` n `1`))))*

Note that this definition is no longer recursive: the function `Fac`
simply delegates the work that needs to be done in the recursive call
to a function *fac* that is provided as an additional parameter. How
does this help?

We can view *`Fac`* as a function that constructs the factorial
function iteratively. That is, *`Fac`* takes *fac* as input and if
*fac* is a function that approximates the factorial function
(i.e. produces the same result as the factorial function on some
inputs), then *`Fac`* returns a new function that is a better
approximation of the factorial function than *fac*. To see this,
consider the following sequence of definitions:

* *`fac0` = (λ x. `0`)*
* *`fac1` = `Fac` `fac0`*
* *`fac2` = `Fac` `fac1`*
* ...

Observe that this sequence of functions approximates the factorial
function with increasing precision: for all natural numbers *`i`* and
*n < `i`*, the term *(`faci` n)* reduces to the numeral representing
*n!*. If we let *`i`* go to infinity, this sequence converges to the
factorial function itself. That is, we can view the factorial function
*`fac`* as the function that satisfies the equation:

*`fac` = `Fac` `fac`*

In other words, we aim to construct the *fixpoint* of `Fac`. What we
thus need is a lambda term that serves as a *fixpoint operator*,
i.e. a function that calculates for us the fixpoint of another
function (here `Fac`). Concretely, we seek a lambda term `fix` such
that β-reduction yields a sequence like this:

*`fix` f → ... → f (`fix` f) → ... → f (f (`fix` f)) → ...*

If we find a term *`fix`* with this property, we can use it to
construct for us automatically on demand the approximation of *`faci`*
that is needed to calculate *n!* for a given input value *n*.

There are various ways to define such a fixpoint operator in the
lambda calculus. The most well-known and simplest one is the so-called
Y combinator due to the logician Haskell Curry:

*`fix` = (λ f. (λ x. f (x x)) (λ x. f (x x)))*

Observe what happens when we apply *`fix`* to *`Fac`*:

   *`fix` `Fac`* 

*→ (λ x. `Fac` (x x)) (λ x. `Fac` (x x))*

*→ `Fac` ((λ x. `Fac` (x x)) (λ x. `Fac` (x x)))*

*→ `Fac` (`Fac` (λ x. `Fac` (x x)) (λ x. `Fac` (x x)))*

*→ `Fac` (`Fac` (`Fac` (λ x. `Fac` (x x)) (λ x. `Fac` (x x))))*

*→ ...*

The sequence of reduction steps computes the approximation sequence
for the factorial function as required! We can thus define the
factorial function using the equation

*`fac` = `fix` `Fac`*

Note that the term *`fix` `Fac`* is indeed a valid term in the lambda
calculus. In particular, this term does not make use of explicit
recursion nor does it rely on any inbuilt data types or operations
for numbers and Booleans.

Let's see how we can put this into practice.

### Scheme

The first programming language based on the lambda calculus was the
language Lisp, developed by John McCarthy in 1958. Instead of Lisp, we
will study the closely related language Scheme, which was first
described in 1975. Scheme shares many of the features of Lisp including:

* support for first-class functions (i.e. lambda terms)
* garbage collection
* homogeneity of programs and data
* continuations
* simple syntax (lots of parentheses)

However, Scheme improves over Lisp in several regards. In particular, it is

* designed to have a clear and simple semantics
* statically scoped
* based on deep binding semantics of lambda terms (implemented via closures)

We will be using [Racket](https://racket-lang.org/) as our Scheme
interpreter. You can download Racket
from [here](https://download.racket-lang.org/).

To start the interpreter, simply type `racket` on the command
line. Alternatively, start the
IDE [DrRacket](http://docs.racket-lang.org/drracket/index.html) that
comes with the Racket distribution. 

Here is a sample Scheme session:

```scheme
> (+ 1 2)
3
> (1 2 3)
; application: not a procedure;
;  expected a procedure that can be applied to arguments
;   given: 1
> a
; a: undefined;
;  cannot reference undefined identifier
> (quote (+ 1 2)) ; a shorthand is '(+ 1 2)
(+ 1 2)
> (car '(1 2 3))
1
> (cdr '(1 2 3))
(2 3)
> (cons 1 '(2 3))
(1 2 3)
```

#### Syntax

Scheme, like Lisp, uses a uniform syntax based on lists. That is, an
expression is either an atom or a list. Atoms are constants (numbers
and Booleans) or symbols (variables and inbuilt functions). Lists can
be nested to form trees. All expressions use *prefix* notation
(i.e. the programmer supplies what would otherwise be the internal
representation of a program):

```scheme
(+ (* 10 12) (* 7 11)) ; means (10 * 12 + 7 * 11)
```

This syntax is also commonly referred to as s-expressions.

A program is simply a closed expression.

Example:

```scheme
(define (fac x) 
        (if (eq? x 0) 
            1 
            (* x (fac (- x 1)))))
            
(display (fac 5))
(newline)
```

#### Semantics

The rules for evaluating Scheme programs are as follows:

* a constant evaluates to itself 
* a symbol evaluates to its current binding 
* a *list* must be
    1. a *form* (e.g., `if`, `lambda`, etc.), or
    2. a function application, i.e.
       * the first element of the list must evaluate to a function
       * the remaining elements are the actual parameters

  Each form has a specific semantics. We describe common forms
  below. Function applications evaluate to the result of applying the
  function to the evaluated actuals. The used evaluation strategy for
  function applications is applicative-order.

#### Quoting Data

Q: If every list is a computation, how do we describe data?

A: Using another primitive: `quote`.

```scheme
> (quote (1 2 3 4)) 
(1 2 3 4) 
> (quote (Baby needs a new pair of shoes))
(Baby needs a new pair of shoes)
> '(this also works)
(this also works)
```

#### Lambda Expressions

Scheme supports *anonymous* functions similar to lambda terms in the
lambda calculus. The following expression defines a function that
takes two parameters `x` and `y` and adds them:

```scheme
(lambda (x y) (+ x y))
```

As in the lambda calculus, the function created by lambda terms are
first-class and can be passed to and returned by other functions.

#### Simple control constructs

Scheme has inbuilt Boolean values:
* `#t` - true
* `#f` - false

However, when evaluating a condition, any value not equal to `#f` is
considered to be true.

Conditional expressions take the form
```scheme
(if condition expr1 expr2)
```

There is also a more general form
```scheme
(cond
  (pred1 expr1)
  (pred2 expr2)
  ...
  (else exprn))
```

This evaluates the `pred`'s in order, until one evaluates to true and
then evaluates the corresponding `expr`, whose result is the value of
the entire `cond` expression.

Note that `if` and `cond` are not regular functions but inbuilt
primitives referred to as *forms*.

#### Global and Local Definitions

Global definitions can be introduced using the `define` form:

```scheme
(define (sqr n) (* n n))
```

The body of the `define` (here `(* n n)`) is not evaluated. Instead, a
binding of a name to an expression is produced. In the example, `sqr`
is bound to the expression:

```scheme
(lambda (n) (* n n))
```

We can also `define` non-functions:

```scheme
> (define x 15)
> (sqr x)
225
```

Note that `define` can only occur at the top level, and creates global
bindings of names to values.

To define local bindings, use one of the `let` forms (`let`, `let*`,
and `letrec`). The basic skeleton of the `let` form is as follows:

```scheme
(let
  ((x1 init1) (x2 init2) ... (xn initn))
  body)
```

The `let` form evaluates all the `init`s in the current environment;
the `x`s are bound to fresh locations holding the results. The scope of these bindings is `body`.

You can thus think of the `let` form as a block that introduces the variables `x1` to `xn` simultaneously and where the scope of the declaration is the remainder of the block (i.e. `body`):

```scala
{
  val x1, ..., xn = init1, ..., initn
  body
}
```

An alternative view is to look at it in terms of the lambda calculus. The above `let` form can be thought of as syntactic sugar for the lambda calculus term:

 
*(λ `x1` ... `xn`. `body`) `init1` ... `initn`*


For example, the following expression evaluates to `3` in an environment where `x` is bound to `2`:

```scheme
(let ((x 1) (y x)) (+ x y))
```

There is also a variant `let*`

```scheme
(let*
  ((x1 init1) (x2 init2) ... (xn initn))
  body)
```

where bindings are performed sequentially from left to right, and each
binding is done in an environment in which the previous bindings are
visible. That is, the `let*` form can be thought of as a block
 
```scala
{
  val x1 = init1
  ...
  val xn = initn
  body
}
``` 

and corresponds to the lambda calculus term

*(λ `x1`. (λ `x2`. ... (λ `xn`. `body`) `initn` ...) `init2`) `init1`*

For instance, the following expression evaluates to `2` regardless of
the environment in which it is evaluated:

```scheme
(let* ((x 1) (y x)) (+ x y))
```

Finally, the `letrec` form can be used to define (mutually) recursive
functions. Here, the `x`s are bound to fresh locations holding
undefined values, the `inits` are evaluated in the resulting
environment (in some unspecified order), each `x` is assigned to the
result of the corresponding `init`.

Example: tail-recursive factorial

```scheme
(define (fac x)
  (letrec
    ((fac-tr (lambda (x acc)
       (if (zero? x) acc
         (fac-tr (- x 1) (* x acc))))))
         (fac-tr x 1)))
```

Many versions of Scheme ensure that tail recursion is implemented
efficiently using tail call elimination.

#### Lists

The inbuilt list data type provides one constant and three primitive operations:

* `nil` or `()`: the empty list
* `cons`: prepend an element to a list
* `car`: get head of list
* `cdr`: get tail of list

Examples of list decomposition:

```scheme
> (car '(this is a list of symbols))
this 

> (cdr '(this is a list of symbols)) 
(is a list of symbols) 

> (cdr '(this that)) 
(that) ; a list 

> (cdr '(singleton)) 
() ; the empty list 

> (car '())
; car: contract violation
;   expected: pair?
;   given: '()
```

Operations like:

```scheme
(car (cdr xs))
(cdr (cdr (cdr ys)))
```
are common. So Scheme provides shorthands:
* `(cadr xs)` is `(car (cdr xs))`
* `(cdddr xs)` is `(cdr (cdr (cdr ys)))`
* ...
Up to 4 (total) `a`s and `d`s can be used.


Examples of list construction:
```scheme
> (cons 'this '(that and the other)) 
(this that and the other) 
> (cons 'a '()) 
(a) 
```

A useful syntactic shorthand is:

```scheme
> (list 'a 'b 'c 'd 'e)
(a b c d e)
```

which is equivalent to:

```scheme
(cons 'a 
      (cons 'b 
            (cons 'c 
                  (cons 'd 
                        (cons 'e '())))))
```

Here is a simple function that checks whether `x` is a member of the
list `xs`:

```scheme
(define (member x xs)
  (cond
    ((null? xs) #f)
    ((eq? x (car xs)) xs)
    (else (member x (cdr xs)))))
```

Note: every non-false value is interpreted as true in a Boolean
context.

Convention: return rest of the list, starting from `xs`, rather than
`#t` to indicate that `x` was found.

#### Type predicates

If variables do not have associated types, we need a way to find out
what a variable is holding:

* `symbol?`
* `number?`
* `pair?`
* `list?`
* `null?`
* `zero?`

Examples:

```scheme
> (number? 1)
#t
> (null? '())
#t
> (number? (+ 1 2))
#t
> (number? '(+ 1 2))
#f
> (list? '(+ 1 2))
#t
> (list? (+ 1 2))
#f
> (pair? '(+ 1 2))
#t
```

Different Scheme dialects use different naming conventions, e.g.
`symbolp`, `numberp`, etc.

