# Foi Language: The Incomplete Guide

This stuff is pretty much all subject to change. Consider everything experimental R&D for the foreseeable future.

If you're looking for a [formal grammar specification](Grammar.md) for **Foi**, it's in progress.

## Table of Contents

* [Primitive Values](#primitive-values)
    - [Empty](#empty)
    - [Numbers](#numbers)
    - [Booleans](#booleans)
    - [Strings](#strings)
* [Identifiers](#identifiers)
* [Code Comments](#code-comments)
* [Imports And Exports](#imports-and-exports)
* [Function Calls](#function-calls)
* [Evaluation-Expression Form](#evaluation-expression-form) (optional lisp-like function call form)
    - [Expression Readability](#expression-readability)
    - [Motivations](#motivations)
    - [Reversing Argument Order](#reversing-argument-order)
    - [Partial Application](#partial-application)
    - [N-Ary Operators](#n-ary-operators)
    - [Apply (aka Spread)](#apply-aka-spread)
* [Defining Variables](#defining-variables)
    - [Block-Definitions Clause](#block-definitions-clause)
    - [Destructured Definitions](#destructured-definitions)
* [Boolean Logic](#boolean-logic)
* [Equality and Comparison](#equality-and-comparison)
* [Pattern Matching](#pattern-matching)
    - [Guard Expressions](#guard-expressions)
* [Records and Tuples](#records-and-tuples)
    - [Equality Comparison](#equality-comparison)
    - [Inspecting](#inspecting)
    - [Generating Sequences (Ranges)](#generating-sequences-ranges)
    - [Deriving Instead Of Mutating](#deriving-instead-of-mutating)
    - [Maps](#maps)
    - [Sets](#sets)
* [Functions](#functions)
    - [Default Parameter Values](#default-parameter-values)
    - [Gathering Arguments](#gathering-arguments)
    - [Negating a Predicate](#negating-a-predicate)
    - [Function Pre-conditions](#function-pre-conditions)
    - [Named Arguments](#named-arguments)
    - [Function Recursion](#function-recursion)
    - [Function Currying](#function-currying)
    - [Function Overs](#function-overs)
    - [Function Composition](#function-composition)
    - [Function Pipelines](#function-pipelines)
* [Base Unit Functions](#base-unit-functions)
    - [Value Identity Function](#value-identity-function)
    - [Null-Application Function](#null-application-function)
* [Loops](#loops)
* [List Comprehensions](#list-comprehensions)
    - [Filter Comprehension (List)](#filter-comprehension-list)
    - [Map Comprehension (List)](#map-comprehension-list)
    - [FlatMap Comprehension (List)](#flatmap-comprehension-list)
    - [Do Comprehension (List)](#do-comprehension-list)
    - [Fold Comprehensions (List)](#fold-comprehensions-list)
* [Monads (And Friends)](#monads-and-friends)
    - [Monadic Map](#monadic-map)
    - [Monadic Bind](#monadic-bind)
    - [The Monad Laws](#the-monad-laws)
    - [Monadic Do Comprehension](#monadic-do-comprehension)
    - [Monadic Function Returns](#monadic-function-returns)
    - [Pattern Matching Monads](#pattern-matching-monads)
    - [List Monad](#list-monad)
    - [Maybe Monad](#maybe-monad)
    - [Foldable / Catamorphism](#foldable--catamorphism)
    - [Either Monad](#either-monad)
* [Broader Category Theory Capabilities](#broader-category-theory-capabilities)
    - [Applicative](#applicative)
    - [Concatable / Semigroup](#concatable--semigroup)
    - [Monoid](#monoid)
* [Concurrency / Asynchrony](#concurrency--asynchrony)
    - [Promise Monad](#promise-monad)
    - [Channel](#channel)
    - [PushStream Monad](#pushstream-monad)
    - [PullStream Monad](#pullstream-monad)
* [IO monad](#io-monad)
    - [Task](#task)
    - [Reader](#reader)
    - [Transforming Over Concurrency](#transforming-over-concurrency)
* [Generator Monad](#generator-monad)
* [Type Annotations](#type-annotations)

## Primitive Values

**Foi** comes with the following built-in primitive value types:

* Empty: `empty`
* Numbers: `42`, `-3.14`
* Booleans: `true`, `false`
* Strings: `"Hello, world!"`
* [Records, Tuples](#records-and-tuples): `< id: 123 >`, `< 1, 2, 3 >`
* [Monadic instances](#monads-and-friends): `Id`, `Left`

### Empty

The `empty` value signifies the absence of any other affirmative value.

### Numbers

Numbers in **Foi** are by default specified in base-10 literals (digits `0-9`), including negative values (with `-`) and decimals (with `.`).

`4.0` is a floating point value, whereas `4` is an integer value. These are sometimes interchangeable, and sometimes not.

There are no special numbers (e.g. `NaN`, `Infinity`) in **Foi**. If a value literal, or operation, produces a number value that does not fit validly into one of the number sub-types (floats, integers, etc), the result is a `Left` (monad instance) holding a message indicating the nature of the failure.

```java
6 / 0;                  // Left{"Divide by zero is infinite"}

6 / "a";                // Left{"Non-numeric operand"}

0.0000000000000001;     // Left{"Number precision underflow"}

9999999999999999.9;     // Left{"Number precision overflow"}
```

**Foi** does however have a "negative zero" value, consistent with IEEE-754, which is distinct from *regular* zero:

```java
0 / 3;                  // 0
0 / -3;                 // -0

0 ?= -0;                // false
```

To specify a *more readable* numeric literal, using `_` as an arbitrary separator (in any position), prefix the number literal with a `\`:

```java
\100_000_003.25;        // 100000003.25
```

Hexadecimal, Octal, and Binary integers can be specified with respective prefixes (lowercase):

```java
\h603A;                 // 24634
\o60072;                // 24634
\b110000000111010;      // 24634
```

You can also specify an arbitrarily large (or small) precision value by prefixing with `\@`:

```java
\@9999999999999999.9;   // Number{9999999999999999.9}
```

Such values are held in a `Number` [monadic instance](#monads-and-friends). To perform mathematical operations with such values, an arbitrary-precision library must be used.

**Note:** One special numeric prefix is `\u`, which actually produces a character/string from the numeric hexadecimal representation of its code-point. So the numeric literal `\u263A` actually produces the single-character string `"☺"`.

### Booleans

There are two boolean values: `true` and `false`; there are no implicitly "truthy" or "falsy" values in **Foi** -- coercion to boolean is never done implicitly.

However, most non-boolean values can be explicitly converted to a `true` or `false` equivalent, via the unary `?` operator. And the `!` unary operator does the same as `?`, but also negates, from `true` to a `false` or vice versa:

```java
?0;             // false
!0;             // true

? 1;            // true
! 1;            // false

?(empty);       // false
!(empty);       // true

? "hello";      // true
! "hello";      // false
```

It's much more common/advisable to use `?` / `!` operators with a variable (e.g., `?customerOrder`, `!qty`, etc) than with a literal value.

**Note:** Whitespace is optional between `?` and the value/variable, as shown above. However, since `?empty` / `!empty` (with no whitespace) are actual operators, either whitespace (`! empty`) or parentheses (`?(empty)`, as above) are necessary to distinguish the intended expression from the operators. That said, `?(empty)` / `!(empty)` are pretty uncommon/unnecessary expressions in programs; `true` and `false` are shorter, respectively, *and* more semantic.

### Strings

Strings are always delimited with `"    "` (double-quotes).

To include a `"` character inside a string literal, it must be escaped with a second `"`, like this: `"Hello, ""friend""!"`; when printed, that value would look like `Hello, "friend"!`.

Other than the `""` escaping, the contents of strings are not, by default, parsed/processed in any way.

That is, if you spread a string across multiple lines, those new lines (and any leading whitespace) are included in the string's contents:

```java
"This is line one,
   and this is line two.";
// This is line one,
//    and this is line two.
```

If you want to collapse new lines (and line trailing/leading whitespace) into a single whitespace character, prefix the string literal with a `\`, like this:

```java
\"This is all on
   the same
 line.";
// This is all on the same line.
```

To interpolate expressions inside of a string, immediately prefix the string literal with a `` \` `` (backslash + back-tick), and then also delimit each expression inside the string with `` ` `` back-ticks:

```java
\`"My name is `name`.";
// My name is Kyle.
```

The interpolated expression can be any valid **Foi** expression.

Interpolation is the best way to include a unicode character in a string literal, via its hexadecimal code, using the `\u` numeric prefix:

```java
\`"I was happy `\u263A` to see you!";
// I was happy ☺ to see you!
```

To include a literal `` ` `` back-tick by itself (not as an interpolation delimiter) in an interpolation-parsed string, escape it with a second `` ` ``, like this:

```java
\`"Here is a single `` back-tick, `name`.";
// Here is a single ` back-tick, Kyle.
```

To both collapse whitespace, as well as allow interpolation, combine the `\` and  `` \` ``, in order, like this:

```java
\\`"This is
   one line, written
 by `uppercase(name)`.";
// This is one line, written by KYLE.
```

## Identifiers

In **Foi**, identifiers are case-sensitive, and can be comprised of any of these characters (with no whitespace):

* `A` - `Z`, `a` - `Z`
* `0` - `9`
* `_`
* `~`

**Note:** There is no restriction on the first character of identifiers, as in some languages; so `3stars` is a valid identifier name. However, an identifier must have at least one non-digit (`0` - `9`) character somewhere in it. Otherwise, it's just a number.

Identifiers can be any length.

Identifiers cannot conflict with keywords: `def`, `defn`, `deft`, `import`, `export`, etc. They also cannot conflict with named operators: `~each`, `~chain`, `~fold`, etc.

## Code Comments

Adding comments to **Foi** code takes two forms, single-line and multi-line:

````java
// this is a single line comment

whatever;   // so is this one

another; /// But...
   this is a block comment, and
   can span as many lines
     as needed.
///
````

The triple-slash `///` comment block can start anywhere on a line, span as many lines as needed (including just a single line), and must end with another triple-slash `///`.

## Imports And Exports

To import named dependencies (including "globals" from the built-in `#Std`), use the `import` keyword:

```java
def Std: import "#Std";

Std.log("Hello");               // "Hello"

Std.log(6 + 12);                // 18
```

Or import specific members from a dependency:

```java
def < :log >: import "#Std";

log("Hello");                   // "Hello"
```

The project manifest maps dependency files to identifiers, which can then be imported by name:

```java
def < :myHelper >: import "Utils";
```

----

To export lexical members from a module (no renaming, source and target names match):

```java
export { :login, :logout };
```

To rename lexical exports (e.g., from lexical `login` / `logout` to exported names `doLogin` / `doLogout`):

```java
export { doLogin: login, doLogout: logout };
```

When exporting, keep in mind that `defn` function definitions *do* hoist -- so they can appear anywhere in the outmost module scope, even below an `export` statement. This is the most common type of value to export from a module.

However, `def` variable definitions (even if they hold functions) do not hoist. Those must always appear at the top of a scope, thus before any `export` statements. Those variables must be assigned an intended value to *export* before an `export` statement; subsequent variable assignments are not retroactively exported.

## Function Calls

The traditional function call-form (e.g., `log("Hello")`) always requires `(    )` around the argument list, and must immediately follow the function name (no whitespace). If there are no arguments to pass, the call looks like `someFn()`.

Inside the argument list, assigning arguments to the function's parameters is done positionally, from left-to-right. To skip an argument/parameter position, simply omit anything (exception optional whitespace) between two successive `,` commas:

```java
myFn(1,,3,,,6);
```

Omitting an argument is the same as specifying `empty` for that argument:

```java
myFn(1,empty,3,empty,empty,6);
```

**Note:** Trailing comma(s) are allowed (and ignored).

## Evaluation-Expression Form

All function calls and operators can optionally be evaluated in a lisp-like evaluation-expression form (aka [S-expressions](https://en.wikipedia.org/wiki/S-expression)), with `| |` delimiters, instead of the canonical `( )` Lisp parentheses:

```java
def < :log >: import "#Std";

| log "Hello" |;               // "Hello"

| log | + 6, 12 ||;            // 18
```

**Note:** Whitespace is optional inside the `|    |` form, except as required between the first element -- this must be function-resolving expression, including operators -- and the comma-separated argument list (e.g., `6, 12` above). So, `|log "Hello"|` and `|log |+ 6,12||` are the minimal whitespace forms of the above code snippet.

Any function name -- or expression that evaluates to a function -- can be used in the first (callee) position of the `|    |` expression. However, with one exception, keywords (`import`, `def`, etc) may not be used in the callee position.

----

The `defn` keyword (for function definitions -- see later) is the single exception; this keyword can be used as a (function-defining!) function in the evaluation-expression form. The single argument to `defn` is the full function signature (including any optional whitespace):

```js
def fn: | defn myFn(x) ^x + 1 |;

// equivalent to:
def fn: defn myFn(x) ^x + 1;
```

In the above snippet, it doesn't seem like the special case `| defn    |` form offers any benefit, as the shorter form without the `|    |` is identical.

It can be helpful for readability sake, when visually delimiting an inline function definition (aka, a "lambda") inside another evaluation-expression. In fairness, perhaps a wrapping `(    )` would be better here than a wrapping `|    |`. More on this in the following sections.

### Expression Readability

It's clearly an idiosyncratic choice for **Foi** to use `|    |` [for its S-expressions](https://en.wikipedia.org/wiki/S-expression), instead of the nearly-universal `(    )` Lisp form.

I wanted to take a moment to explain my justification for this choice.

Unlike all those other Lisp languages, **Foi** takes the very unusual step of combining a prefix-Lisp form in the same language as an infix function call form.

And because of the strong familiarity (from non-Lisp languages) of function calls looking like `whatever(42)`, the `( )` must be used for delimiting the argument list, with the function callee prefixed on the outside. Moreover, `(    )` is extremely common (in infix language form) for expression grouping, providing yet another complication.

Syntactically/grammatically, it's thus challenging to support variations like `whatever(42)` (infix function call form), `(whatever 42)` (prefix Lisp form), and `(42 + whatever())` (infix expression grouping)... all in the same language, especially as these expressions nest inside each other in complicated ways.

Simpler Lisp languages are free to use `(    )` for their S-expressions mostly because they don't really use `(    )` for any of these other purposes. However, **Foi** must address the conflation issue.

Even if it wasn't grammatically hard for all these forms to co-exist in **Foi**, I believe it would be a significant readability barrier for authors and readers of such code, to be encountering frequent `(    )` conflation (use for different purposes) in the same program.

As such, I've chosen for **Foi** to use something other than `(    )` for delimiting its evaluation-expression (Lisp) form.

----

Unfortunately, `[    ]`, `<    >`, and (especially!) `{    }` pairs are already reserved for other important purposes, so they're out of consideration.

There's no more standard character pairs that have visually-obvious opening and closing characters (`/` and `\` have obvious conflicts). We've also [explored a variety of other options](https://github.com/getify/foi-lang/discussions/4), including arbitrary forms like `$    |`, or multi-character delimiters (`[|    |]`, etc).

Ultimately, it seems to me like the best option is just bare `|    |`, given their "tall" visual distinctiveness.

Regrettably, this choice doesn't provide any distinction between opening and closing characters in a pair. It means that in some cases, sequences where a `|` appears adjacent to another `|` may be a bit confusing/visually ambiguous; they could be two opening or two closing characters (most likely), or it could be a closing followed by an opening, or the reverse.

Still, there's some visual ambiguity downsides to `|` being used as an enclosing pair this way.

----

Let's consider a particularly nasty looking (albeit contrived/pathological) example:

```java
||defn(x,y)^x+y| |+ 1,2,whatever(3)|,|* |+ 1,2,|/ 1,2||,3||;
```

That's a bit tough to visually parse out, right!?

Indentation/new-lines are often used in Lisp-style to aid readability:

```java
|defn(x,y)^x+y
    |+ 1,2,whatever(3)|,
    |* |+ 1,2,
        |/ 1,2||,
        3||;
```

**Note:** with this formatting, the `defn ..` expression can visually stand alone on the first line, without *needing* its own `|    |` (or `(    )`) wrapping around it.

Better? A bit.

Here's what the typical Lisp-style version with `(    )` would look like (but still with **Foi** commas between arguments, instead of only whitespace):

```java
(defn(x,y)^x+y
    (+ 1,2,whatever(3)),
    (* (+ 1,2,
        (/ 1,2)),
        3));

// without newlines/indentation:
((defn(x,y)^x+y) (+ 1,2,whatever(3)),(* (+ 1,2,(/ 1,2)), 3));
```

**Note:** In the single-line form, I added `(    )` around the `defn ..` just for a bit of extra clarity.

TBH, I'm not sure canonical `(    )` Lisp form is *significantly* more readable. But at least it's clear what's opening `(`, and what's closing `)`.

----

Let's finally look at what I'm proposing as **Foi**'s readability compromise. In any case where two or more `|` characters might appear adjacent to each other, and there's a concern that it's not clear what each one means, use `(    )` grouping around each `|    |` expression -- always valid grammatically -- to visually disambiguate.

Consider:

```java
(|defn(x,y)^x+y
    (|+ 1,2,whatever(3)|),
    (|* (|+ 1,2,
        (|/ 1,2|)|),
        3|)|);

// without newlines/indentation:
(|(defn(x,y)^x+y) (|+ 1,2,whatever(3)|),(|* (|+ 1,2,(|/ 1,2|)|),3|)|);
```

Take a few moments for that to sink in.

This approach essentially means `(|    |)` is an optional evaluation-expression form, as opposed to the bare `|    |` form.

Even though it's more characters to type and read, I honestly kind of like it; I find it a bit easier to scan. It's not only clear where opening and closing are happening, but *also* clear where evaluation-expression is happening separate from any other use of `(    )`.

You could even just think of `(|` as the opening tag and `|)` as the closing tag -- perhaps even "requiring" them in your own code/style guide/linting -- if you favor the consistency and don't mind the double-character delimiters.

That's probably what I will do, personally. Given the above analysis and constraints, I think this is a reasonable compromise for **Foi** to make.

### Motivations

So... why does **Foi** even include this optional Lisp-like form? It adds complexity to the language -- for implementers, authors, and readers! -- but for what purpose?

The primary reason for the `|    |` evaluation-expression form in **Foi** is that it allows quite a bit of additional flexibility/capability at the call-site that isn't possible with the traditional infix call-site form (e.g., `whatever(1,2,3)`). In particular, it allows operators to be treated as more general function calls.

We'll cover many of those capabilities over several following sub-sections.

### Reversing Argument Order

One such flexibility is that we can control the treatment of input arguments in various ways.

Some operators like `+` are commutative, so the operand/argument order doesn't matter. But other operators, like `-`, are not commutative, so the order matters.

To reverse the order of applied arguments of the operator-function in question, we can use the `'` prime operator, applied first to the function:

```java
| | ' - | 1, 6 |;               // 5
```

Yes, with `| ' - |`, we just applied one operator against another operator!

**Note:** The `'` prime operator has no prefix-operator form (like `'something(42)` or `1 '- 6`); that sort of syntax could cause chaos for readability. Thus, it can only be used inside an evaluation-expression form, as shown above.

Since this operation will be extremely common, a special sugar short-hand is available. The `'` prime operator may appear immediately preceding (no whitespace) the operator/function (or expression) it's modifying:

```java
| '- 1, 6 |;                    // 5
```

This short-hand form of `'` should be preferred for readability sake wherever practical.

### Partial Application

It's common in functional programming to produce more specialized functions by applying only some inputs to a more generalized (higher-arity) function; the result is a another function that expects the subsequent arguments.

This is referred to as partial application, and is another flexible capability afforded by the evaluation-expression form.

Consider the `+` mathematical operator, which has a minimum arity of 2. If we provide it only one argument, the result is a partially applied function that's still waiting for the second argument:

```java
| | + 6 | 12 |;                 // 18
```

Here, the `| + 6 |` creates the partially applied (operator) function, which is then provided a second argument `12` in the outer `|    12 |` expression.

**Note:** As with the traditional call form, arguments can be skipped with successive `,` commas with nothing (except optional whitespace) between them. Also trailing comma(s) are allowed (but ignored). As such, an expression like `| myFn 1, 2, |` will be treated (for partial application argument counting purposes) as 2 arguments, not 3. To specify a third affirmative empty argument at the end of the list, add an additional trailing `,` comma (e.g., `| myFn 1, 2,, |`) or specify `empty` explicitly (e.g., `| myFn 1, 2, empty |`).

Partial application operates according to the default argument ordering, which is left-to-right. However, it's quite common (especially with operators) to want to reverse the partial application order (right-to-left). This is most useful for producing point-free expressions.

For example, let's say we want to produce a function (from the `-` operator) that will subtract `1` from its next input value. How do we partially apply the `1` when it's the second/right-most argument?

To accomplish this, recall the `'` prime operator, which reverses the argument ordering:

```java
| | '- 1 | 6 |;                 // 5
```

The `| '- 1 |` evaluation-expression applies `1` as the right-most argument to `-`, and since that's the only argument provided, the result is a right-partially applied function.

*That* function -- which is back to regular left-to-right ordering, by the way -- is then expecting its final argument, which is then provided by the outer `|    6 |` evaluation-expression.

### N-Ary Operators

Another advantage of this form is that it allows n-ary operators -- operators accepting 3 or more operand inputs -- where typically prefix/infix/suffix operators would be limited to unary (single operand) or binary (two operands) usage.

Many operators in **Foi** are n-ary, such as the `+` operator, `+>` flow (composition) operator, and `?<=>` range-check operator.

For example, say you want to add 5 numbers together. You can obviously do:

```java
1 + 2 + 3 + 4 + 5;
```

But because `+` is an n-ary operator, you can also do:

```java
| + 1, 2, 3, 4, 5 |;
```

It's nice to only need to list the operator once instead of 4 times!

Still, as the `+` operator is a single symbol, this example (including padded whitespace) yields a slightly longer expression, which may seem disfavorable.

However, other operators are comprised of two or more symbols, so the length of the evaluation-expression form will likely end up shorter depending on how many arguments are provided.

Also, some operators may result in a change of value-type from the operand(s) to the result. In those cases, you cannot simply combine multiple infix operator usages like we did with `+`.

For example, say you wanted to test 3 variables as all being equal to each other. The `?=` infix operator can only accept two operands (left and right), so we're forced to do multiple expressions, and combine their results with the logical-AND `?and` operator:

```java
(x ?= y) ?and (y ?= z) ?and (x ?= z);
```

**Note:** The `?=` equality comparison may not be transitive, depending on the types being compared, hence why we included the `x ?= z` check for good measure.

But since the `?=` operator is n-ary, we can provide it 3 or more arguments using the evaluation-expression form, resulting in a much shorter/nicer-to-read expression:

```java
| ?= x, y, z |;
```

It should be clear how much more preferable n-ary operator evaluation can be!

### Apply (aka Spread)

Say we have a list of values (a Tuple, as we'll see later) called `numbers`, and we want to "spread them out" as arguments to an operator/function. We can use the `...` operator (only available in the evaluation-expression form):

```java
| + ...numbers |;
| + 0, ...numbers, 1000 |;
```

OK, that's useful. But what about modifying an operator/function to automatically accept its inputs as a list?

```java
| | ... + | numbers |;
```

Since `...` is an operator, when applied against an operator/function like `+`, it produces a new function that will expect a single (Tuple) argument that's then *spread out* to the underlying operator/function.

----

As you can see from the last several sections, there's lots of additional power in the evaluation-expression form, but there are yet still other capabilities that we'll encounter later in this guide.

## Defining Variables

To define variables, use the `def` keyword (not an operator/function).

```java
def age: 42;
```

All definitions need a value initialization, but you can use the `empty` value if there's no other value to specify.

`def` definitions do not hoist, so to avoid confusion, they *must not* be preceded in any scope (module, function, or block) by any other non-definition (besides `def`, `deft`, `defn`, and `import`) statements.

To reassign a variable:

```java
def age: 41;

// later
age := 42;
```

Unlike `def` definitions, `:=` re-assignments are allowed anywhere in the scope after the associated `def` definition.

Multiple re-assignments (of the same value) can also be chained:

```java
x := y := z := 0;

| := x, y, z, 0 |;
```

`def` definitions attach to the nearest enclosing scope, whether that be module, function, or block. A block-scoped variable definition is thus:

```java
{
    def tmp: 42;
    tmp := 43;
}
```

However, since `def` definitions must appear at the top of their respective scopes, and there may be multiple such definitions in a block, the definitions-block form should be preferred for readability sake:

```java
def (tmp: 42) {
    tmp := 43;
};
```

Moreover, this definitions-block form is allowed anywhere in its enclosing scope, so it's more flexible than a non-block `def` declaration.

### Block-Definitions Clause

In addition to the definitions-block form just shown, several other expressions in **Foi** allow a `{    }` block to be declared as part of the larger expression. For syntactic convenience, many of these expressions' blocks can be prefaced by the optional `(   )` block-definitions clause:

* A [guard block](#guard-expressions) with block-definitions clause:

    ```java
    // if x > y, swap them
    // (tmp is block-scoped)
    ?[x ?> y]: (tmp: x) {
        x := y;
        y := tmp;
    };
    ```

* A [pattern matching clause block](#pattern-matching) with block-definitions clause:

    ```java
    ?{
        // x is odd?
        // (tmp is block-scoped to the clause)
        ?[mod(x,2) ?= 1]: (tmp) {
            tmp := (x * 3) + 1;
            ?[tmp ?> 100]: tmp := 100
            myFn(tmp);
        }

        // x is non-zero (and even)?
        ?[x != 0]: myFn(x)

        // otherwise, x must be zero,
        // so skip calling function and
        // default to fixed value 1
        ?: 1
    };
    ```

* A [loop iteration block](#loops-and-comprehensions) with block-definitions clause:

    ```java
    0..3 ~each (v, idx) {
        log(v + ": " + idx);
    };
    ```

**Note:** While function body definitions, and the Record/Tuple *def*-block, both have `{    }` blocks, these *cannot* be prefaced by a block definitions clause.

### Destructured Definitions

When defining a variable where the assignment is intended to step into (aka, "destructure") a [Record/Tuple value](#records-and-tuples)'s contents, we can use a dedicated destructuring form:

```java
def <
    :items.0.price,
    firstItem: items.0,
    #order
>: getOrder(123);

price;          // 29.97
firstItem;      // < price: 29.97, label: ... >
order;          // < id: 123, items: < < price: 29.97, ... >
```

The `:items.0.price` syntax form implicitly assumes a target variable name from the final source property name (`price`) above; for this syntax to be valid, the source property name must be fixed (cannot be a dynamic expression) and a valid identifier (cannot be a number like `0`).

If you need to "rename" the target variable -- for example, if the source property name isn't fixed or a suitable identifier -- the target name (`firstItem` above) may be specified before the `:`.

Finally, to capture the entire value being destructured, use `#` prefixing a target variable name.

This destructuring form is also allowed in a block-definitions clause:

```java
def (< :items.0.price >: getOrder(123)) {
    price;          // 29.97
};

order.items ~each (< itemPrice: price >) {
    itemPrice;      // 29.97
};
```

## Boolean Logic

The `true` and `false` boolean values are used primarily for decision making. Accordingly, non-negated, boolean-returning operators, aka logical operators, begin with the `?` character (to signal asking a question to make a decision).

To combine two or more boolean values with logical-AND (`?and`):

```java
def isValid: true;
def isComplete: true;
def isSuccess: false;

isValid ?and isComplete;                    // true
isValid ?and isComplete ?and isSuccess;     // false

| ?and isValid, isComplete, isSuccess |;    // false
```

And for logical-OR (`?or`):

```java
def isValid: true;
def isComplete: true;
def isSuccess: false;

isValid ?or isComplete ?or isSuccess;       // true

| ?or isValid, isComplete, isSuccess |;     // true
```

**Note:** As you can see, the `?and` and `?or` operators are n-ary, meaning they can take 2 or more arguments -- but only in the evaluation-expression form.

Any `?`-prefixed logical boolean operator can be flipped/negated by swapping the `?` character with `!` in the operator. For example, `!and` is *NAND* (not-and) and `!or` is *NOR* (not-or):

```java
// instead of these:
!(true ?and false);             // true
!true ?or !false;               // true
!(true ?and true);              // false
!true ?or !true;                // false

// or these:
!(false ?or false);             // true
!false ?and !false;             // true
!(true ?or false);              // false
!true ?and !false;              // false

// use negated operators:
true !and false;                // true
true !and true;                 // false
false !or false;                // true
true !or false;                 // false
```

We'll see more `?`-prefixed, boolean-returning operators in the next section, all of which can also be negated by swapping `?` for `!`.

## Equality And Comparison

The `?=` operator checks for equality:

```java
def x: 42;
def y: 42;
def z: 100;

x ?= 42;                    // true

| ?= x, y, z |;             // false
```

**Note:** `?=` is another n-ary operator in the evaluation-expression form. Keep in mind, equality comparison in **Foi** is not necessarily transitive.

The `empty` value semantically means *absence of a value*. As such, checking for equality with `empty` is somewhat of a confusing construct. It's valid to do so (`x ?= empty`), but a dedicated `?empty` prefix boolean operator is provided to reduce confusion:

```java
def x: 42;
def y: empty;
def z: empty;

x ?= empty;                 // false
y ?= empty;                 // true  -- but a bit confusing!

?empty x;                   // false
?empty y;                   // true  -- clearer!

| ?empty x, y, z |;         // false
| ?empty y, z |;            // true
```

----

To relationally compare (`?<` less-than, `?>` greater-than):

```java
def x: 100;
def y: 200;

x ?< y;                     // true
x ?> y;                     // false
```

And for the inclusive comparisons (`?<=` less-than-or-equal, `?>=` greater-than-or-equal):

```java
def x: 100;
def y: 200;

x ?<= x;                    // true
y ?>= y;                    // true
```

**Note:** These four operators are also n-ary operator in the evaluation-expression form. They compare the first operand against all other operands/inputs. For example, `| ?< x, y, z |` is the equivalent of `(x ?< y) ?and (x ?< z)`, but *does not* compare `y ?< z`.

A very common task is to check if a value is in a range between two other values:

```java
def x: 100;

(x ?> 0) ?and (x ?< 500);   // true
```

However, this can be done more idiomatically with the range-check operators, `?<>` (non-inclusive) and `?<=>` (inclusive):

```java
def x: 100;

| ?<>  0,   x, 500 |;   // true
| ?<=> 100, x, 100 |;   // true
```

**Note:** Because these two operators have an arity of exactly 3, they cannot be used in the typical infix expression form, which would only allow two operands (left and right).

----

As mentioned in the previous section, all these `?`-prefixed comparison operators can also be flipped/negated by swapping the `?` with `!`:

```java
def x: 42;
def y: 100;
def z: empty;

x ?= 42;                // true
x != 42;                // false

?empty z;               // true
!empty z;               // false

x ?> y;                 // false
x !> y;                 // true
x ?>= y;                // false
x !>= y;                // true

x ?< y;                 // true
x !< y;                 // false
x ?<= y;                // true
x !<= y;                // false

| ?<>  40,  x, 50  |;   // true
| !<>  40,  x, 50  |;   // false
| ?<=> 100, y, 100 |;   // true
| !<=> 100, y, 100 |;   // false
```

## Pattern Matching

To make decisions (with booleans!), use pattern matching. There are two forms:

1. Dependent: each pattern is matched against (dependent on) a single topic; delimited with an opening of `?(    ){`, and closed with `}`.

2. Independent: each pattern has its own independent topic; delimited with an opening of `?{`, and closed with `}`.

Each pattern clause is defined by `?[    ]: consq`, where the pattern is defined inside the `[    ]`. A pattern can be negated as `![    ]`. The pattern match clause's consequent (`consq`) can either be a single expression, or a `{   }` block; either way, it's only evaluated if the pattern is matched via the conditional.

Let's examine each pattern matching form separately, starting with dependent pattern matching. The topic of the match is any arbitrary expression, defined in the `?(    ){` clause.

Consider:

```java
def myName: "Kyle";

?(myName){
    ?["Kyle"]: log("Hello!")
    !["Kyle"]: log("Goodbye!")
}
// Hello!
```

In this example, the topic is the `myName` variable, which is evaluated once. Each pattern clause is evaluated, in order, and compared for equality with the topic. For the first clause whose pattern is matched, its consequent is evaluated and the result returned for the overall pattern match expression.

Dependent pattern matching expressions *should be* determinate, in that all possible conditional branches are defined. The result of a pattern matching expression is thus the consequent expression of whichever conditional clause was matched:

```java
def myName: "Kyle";

def greeting: ?(myName){
    ?["Kyle"]: "Hello!"
    !["Kyle"]: "Goodbye!"
};

greeting;               // "Hello!"
```

However, if no pattern matches, the default result of the expression is an `empty`.

**Note:** You can configure **Foi** to issue a warning notice in such a case.

To explicitly define a default pattern -- like an "else" clause, that matches if nothing else previously matched -- use `?:` as the last clause in the pattern matching expression:

```java
def myName: "Kyle";

def greeting: ?(myName){
    ?["Kyle"]: "Hello!"
    ?: "Goodbye!"
};

greeting;               // "Hello!"
```

**Note:** Comparing this example to the previous one, `?:` is equivalent to the `!["Kyle"]` pattern. Readability preferences may dictate either style, depending on the circumstances.

A dependent style pattern can include a `,` comma separated list of multiple values, any of which may match the topic:

```java
def myName: "Kyle";

def greeting: ?(myName){
    ?["Kyle","Fred"]: "Hello!"
    ?: "Goodbye!"
};

greeting;               // "Hello!"
```

It may also be useful to access the topic of a pattern matching expression inside its clause(s); the topic is bound to the `#` symbol:

```java
def myName: "Kyle";

def greeting: ?(myName){
    ?["Kyle"]: | + "Hello ", #, "!" |
    ?: "Goodbye!"
};

greeting;               // "Hello Kyle!"
```

Dependent pattern matching should only be used if the patterns only need equality-comparison of one or more discrete value(s) against the topic.

----

For more complex boolean-logic matching patterns, the independent pattern matching form is appropriate. Independent pattern matching has no topic, and thus begins with a `?{` instead of a `?(    ){`.

In this form, each clause matches only if the pattern is a conditional that evaluates to `true`. You could thus mentally model `?{` as if it was shorthand for `?(true){`:

```java
def myName: "Kyle";

def greeting: ?{
    ?[myName ?= "Kyle"]: "Hello!"
    ![myName ?= "Kyle"]: "Goodbye!"
};

greeting;               // "Hello!"
```

**Note:** The pattern-match conditional `![myName ?= "Kyle"]` is equivalent to `?[myName != "Kyle"]`. Readability preferences may dictate either style, depending on the circumstances.

Just as with dependent pattern matching, it's preferable for the overall independent pattern matching expression to be determinate, in that all conditional branches are covered. Again, to define a default (final) clause, `?:` may be used:

```java
def myName: "Kyle";

def greeting: ?{
    ?[myName ?= "Kyle"]: "Hello!"
    ?: "Goodbye!"
};

greeting;               // "Hello!"
```

**Note:** Again comparing this example to the previous one, `?:` is equivalent to the previous snippet's `![myName ?= "Kyle"]` conditional, or even `?[myName != "Kyle"]`. Moreover, `?:` is also equivalent to `?[true]`, which clearly would *always match*. Readability preferences may dictate any of those style options, depending on the circumstances, but generally the shorter `?:` form is most idiomatic.

### Guard Expressions

When an independent pattern matching expression would only have one clause, the clause can be specified standalone, as a *guard* expression.

For example:

```java
def myName: "Kyle";

// full pattern matching expression:
?{
    ?[!empty myName]: printGreeting(myName)
}

// standalone guard expression:
?[!empty myName]: printGreeting(myName);

// or:
![?empty myName]: printGreeting(myName);
```

## Records And Tuples

Records are immutable collections of values, delimited by `<    >`.

You can name each field of a record, but if you omit a name, numeric indexing is automatically applied. Any record with all numerically indexed fields (implicitly or explicitly defined) is a special case called a Tuple.

```java
def idx: 2;
def prop: "last";

def numbers: < 4, 5, 6 >;
numbers.1;                      // 5
numbers[idx];                   // 6

def person: < first: "Kyle", last: "Simpson" >;
person.first;                   // "Kyle"
person[prop];                   // "Simpson"
```

Above, Record/Tuple fields are accessed with `.` operator, whether numeric or lexical-identifier. `[    ]` field access syntax evaluates field-name expressions (including strings that may include non-identifier characters).

Since `.` is an operator, Record/Tuple field access can also be performed in the evaluation-expression form, in which case it evaluates the second argument as an expression (like the `[    ]` form does):

```java
def idx: 2;
def prop: "last";

def numbers: < 4, 5, 6 >;

| . numbers, 1 |;               // 5
| . numbers, idx |;             // 6

def person: < first: "Kyle", last: "Simpson" >;

| . person, "first" |;          // "Kyle"
| . person, prop |;             // "Simpson"
```

To define Records/Tuples using arbitrary expressions (other than simple identifiers) for the values, use the evaluation-expression form:

```java
def < :String.uppercase >: import "#Std";

def five: 5;
def numbers: < 4, five, 6 >;

def surname: "Simpson";
def person: < first: "Kyle", last: |uppercase surname| >;
```

**Note:** To keep Record/Tuple syntax complexity to a minimum, *only* the `|    |` form of evaluation-expression (function invocation, operators, etc) is allowed inside the `<    >` literal definition.

Strings are just syntax sugar for Tuples of characters. Once defined, a string and a Tuple of characters will behave the same.

```java
def chars: < "H", "e", "l", "l", "o" >;
def str: "Hello";

chars.1;                    // "e"
str.1;                      // "e"
```

To determine the length of a string (or a Tuple), or the count of fields in a Record, use the `size()` function:

```java
def < :size >: import "#Std";

size("Hello");              // 5
size(< "O", "K" >);         // 2
size(< a: 1 >);             // 1
```

If the desired index or field name is held in a variable, you can use it in the Record/Tuple literal definition by prefixing the variable name with the `%` sigil:

```java
def idx: 3;
def nums: < 5, 10, 15, %idx: 20, 25 >;
// < 5, 10, 15, 20, 25 >

def field: "first";
def person: < %field: "Kyle", last: "Simpson" >;
// < first: "Kyle", last: "Simpson" >
```

When defining a Record and the field name matches the variable holding the value, a concise syntax is allowed:

```java
def first: "Kyle";
def last: "Simpson";

def person: < :first, :last >;

// instead of:
def person: < first: first, last: last >;
```

To progressively define the contents of a Record/Tuple across an arbitrary number of statements/operations, use a Record/Tuple *def-block* `<{    }>`:

```java
def numbers: <{
    def five: 5;
    def six: empty;

    #1 := 4;
    #2 := five;

    six := #1 + #2 - 3;
    #3 := six;
}>;

def person: <{
    #first := "Kyle";
    #last := "Simpson";
}>;
```

As shown, the `<{    }>` *def-block* can contain any arbitrary logic for determining the contents, including traditional function calls, loops, etc. Once the block closes, the computed value is frozen as immutable.

Inside a `<{    }>` *def-block*, the `#` sigil indicates a self-reference to the current Record/Tuple context that's being defined, and can be used either in l-value (assignment target) or r-value (value expression) positions. However, these special self-references cannot cross inner-function boundaries.

### Equality Comparison

Since Records/Tuples are primitive (and immutable) value types in **Foi**, equality comparison is structural (meaning deep contents comparison rather than reference identity).

```java
def a: < 4, 5, 6 >;
def b: < 4, 5, 6 >;
def c: < 5, 4, 6 >;

a ?= b;         // true
b ?= c;         // false
c ?= a;         // false
```

```java
def a: < one: "hello", two: "world" >;
def b: < one: "hello", two: "world" >;
def c: < two: "world", one: "hello" >;

a ?= b;         // true
b ?= c;         // true
c ?= a;         // true
```

### Inspecting

You can determine if a value is *in* a Tuple with the `?in` / `!in` operator:

```java
def numbers: < 4, 5, 6 >;

7 ?in numbers;                  // false
| ?in 4 numbers |;              // true

7 !in numbers;                  // true
| !in 4 numbers |;              // false
```

**Note:** The `in` operator only inspects numerically indexed fields.

You can determine if a field is defined in a Record with the `?has` / `!has` operator:

```java
def person: < first: "Kyle", last: "Simpson" >;

person ?has "first";            // true
person ?has "middle";           // false

person !has "nickname";         // true
```

### Generating Sequences (Ranges)

If you want to generate a list (Tuple) of sequential ([aka "interval"](https://www.graphpad.com/support/faq/what-is-the-difference-between-ordinal-interval-and-ratio-variables-why-should-i-care/)) data, you can use the binary `..` range operator (either infix or evaluation-expression form).

This usage of the `..` operator is valid with naturally sequential (ordered, fixed interval) values, such as integers and characters:

```java
def someInts: 2..13;

someInts.5;                     // 7

def alphabet: | .. "a", "z" |;

alphabet.5;                     // "e"
```

The bounds (start and end) of a sequence/range can be held in variables:

```java
def two: 2;
def thirteen: 13;
def someInts: two..thirteen;

def a: "a";
def z: "z";
def alphabet: a..z;
```

The start/end values must be of the same data type; `3.."g"` will not work.

### Deriving Instead Of Mutating

Since Records/Tuples are immutable, to "change" their contents requires you to derive a new Record/Tuple.

The most basic way to derive a new Tuple is to concatenate two (or more) Tuples with `+`:

```java
def odds: < 1, 3, 5, 7, 9 >;
def evens: < 0, 2, 4, 6, 8 >;

odds + evens;
// < 1, 3, 5, 7, 9, 0, 2, 4, 6, 8 >
```

Another way to derive a new Record/Tuple is to select multiple elements using the `.<    >` syntax -- should be treated as a singular compound operator -- with one or more source indices/keys separated by commas:

```java
def numbers: < 3, 4, 5, 6, 7 >;
def evenDigits: numbers.<1,3>;
// < 4, 6 >

| .<1,3> numbers |;
// < 4, 6 >

def person: < first: "Kyle", last: "Simpson", nickname: "getify" >;
def profile: person.<first,nickname>;
// < first: "Kyle", nickname: "getify" >

| .<first,nickname> person |;
// < first: "Kyle", nickname: "getify" >
```

**Note:** The `.<` of this operator cannot have any whitespace between the two symbols, but whitespace is allowed inside the `.<    >`.

You can also select a ranged Tuple subset, with the `.[  ..  ]` syntax -- should be treated as a singular compound operator -- as a shorthand for the `.<    >` form:

```java
def numbers: < 3, 4, 5, 6, 7 >;

def head: numbers.0;                // 3
def first: numbers.[..0];           // < 3 >
def leading: numbers.[..3];         // < 3, 4, 5, 6 >

def last: numbers.-1;               // 7
def trailing: numbers.[-1..];       // < 7 >
def tail: numbers.[1..];            // < 4, 5, 6, 7 >

def middle: numbers.[1..3];         // < 4, 5, 6 >

| .[1..3] numbers |;                // < 4, 5, 6 >
```

**Note:** The `.[` of this pick syntax form cannot have any whitespace between the two symbols, nor can the `1..3` expression have whitespace; however, whitespace is allowed *around* the range (`.[ 1..3 ]`).

A ranged Tuple subset such as `.[2..5]` is a shorthand equivalent for `.<2,3,4,5>` -- selecting out specific indices `2`, `3`, `4`, and `5`.

Certain ranges (e.g., `.[0..]`, `.[..-1]`, and `.[0..-1]`) are no-op expressions, since they result in the same Tuple; as immutable values, there's no reason for **Foi** to actually copy the Tuple in these cases.

----

Additionally, inside the `<    >` syntactic definition of a Record/Tuple, a special `&` pick sigil prefixed on a variable name (not an arbitrary expression) *picks and includes* some or all of the contents of that other Record/Tuple:

```java
def numbers: < 4, 5, 6 >;
def allDigits: < 0, 1, 2, 3, &numbers, 7, 8, 9 >;
// < 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 >

def person: < first: "Kyle", last: "Simpson" >;
def friend: < &person, first: "Jenny" >;
// < first: "Jenny", last: "Simpson" >
```

**Note:** `&` (*pick*) is a sigil, not an operator, and only has meaning inside a static `<    >` Record/Tuple definition; it cannot be used in a `<{    }>` *def-block* or the evaluation-expression `|    |`.

The annotations `&numbers` and `&person` *pick* the entire contents of `numbers` and `person`, respectively, to be included in the new Tuple and Record values. The order of field definitions is always left-to-right, so subsequent field definitions override previous ones; thus, `first: "Kyle"` is reassigned to `first: "Jenny"` above.

Picking is useful for merging multiple sequences. For example, to define a Tuple holding all the base-64 characters:

```java
def upper: "A".."Z";
def lower: "a".."z";
def digits: "0".."9";

def base64: < &upper, &lower, &digits, "+", "/" >;
// < "A", "B", "C", "D", ... "8", "9", "+", "/" >
```

**Note:** For type consistency, we intentionally defined `digits` as the character sequence `"0".."9"` instead of the integer sequence `0..9`.

To *pick* only a specific element:

```java
def numbers: < 4, 5, 6 >;
def oddDigits: < 1, 3, &numbers.1, 7, 9 >;
// < 1, 3, 5, 7, 9 >

def person: < first: "Kyle", last: "Simpson" >;
def friend: < first: "Jenny", &person.last >;
// < first: "Jenny", last: "Simpson" >
```

Moreover, the *picks* `&numbers.1` and `&person.last` are shorthand equivalents for:

```java
def numbers: < 4, 5, 6 >;
def oddDigits: < 1, 3, 2: numbers.1, 7, 9 >;
// < 1, 3, 5, 7, 9 >

def person: < first: "Kyle", last: "Simpson" >;
def friend: < first: "Jenny", last: person.last >;
// < first: "Jenny", last: "Simpson" >
```

One advantage of this more verbose form is, you can re-index/rename the field (something other than `2` or `last`, respectively) in the target Record/Tuple.

The `.<    >` and `.[  ..  ]` syntaxes also work with the `&` pick sigil inside a `<     >` Record/Tuple literal:

```java
def numbers: < 3, 4, 5, 6, 7 >;
def evenDigits: < 2, &numbers.<1,3>, 8 >;
// < 2, 4, 6, 8 >
def fiveBelow: < 0, 1, 2, &numbers.[..2] >;
// < 0, 1, 2, 3, 4, 5 >

def person: < first: "Kyle", last: "Simpson", nickname: "getify" >;
def profile: < &person.<first,nickname> >;
// < first: "Kyle", nickname: "getify">
```

----

There is no dedicated *exclude* syntactic form for "skipping" an index/field while deriving a new Record/Tuple. However, subsequently assigning an `empty` value to an index or field has the same effect:

```java
def numbers: < 3, 4, 5, 6, 7 >;
def fewer: < 0, &numbers, 2: empty, 4: empty >;
// < 0, 3, 5, 6 >

def person: < first: "Kyle", last: "Simpson", nickname: "getify" >;
def entry: < &person, nickname: empty >;
// < first: "Kyle", last: "Simpson" >
```

As shown, `empty` means the *absence of a value*, and thus cannot actually be held in a Record/Tuple.

### Maps

A Record can also act as a *map*, in that you can use another Record/Tuple *as a field* (not just as a value), using the `%` sigil to start the field name:

```java
def numbers: < 4, 5, 6 >;
def dataMap: < %numbers: "my favorites" >;

dataMap[numbers];           // "my favorites"
```

**Note:** Like `&`, the `%` (map-field) sigil is not an operator, and can only be used inside a `<    >` Record definition.

### Sets

A Set is a Tuple that only has unique values. An alternate Tuple definition form, delimited with `<[    ]>` instead, is provided for convenience, to ensure each unique value is only stored once:

```java
def something: < 4, 5, 6 >;
def another: < 6, 7 >;
def uniques: <[ &something, &another ]>;
// < 4, 5, 6, 7 >
```

All syntax rules of Tuple definition `<    >` still apply inside the `<[    ]>`, including use of the `&` and `%` sigils; as Sets *are* Tuples, not Records, field names are not allowed.

Unlike `+` which merely concatenates, the `$+` operator acts as a unique-only Set-append operation:

```java
def numbers: <[ 4, 5, 5, 6 ]>;

def moreNumbers: numbers $+ < 6, 7 >;

moreNumbers;                // < 4, 5, 6, 7 >
```

**Warning** The `$+` operator only works on Tuples (Sets), not Records.

Set equality comparison deserves special attention. Since Sets are merely a construction form for Tuples, the `?=` will perform Tuple equality comparison, where order matters. This may produce undesired results (false negatives).

As such, the `?$=` (set equality) operator, and the corresponding `!$=` (set non-equality), perform unordered comparison of Sets (Tuples):

```java
def set1: <[ 4, 5, 5, 6 ]>;     // < 4, 5, 6 >
def set2: <[ 5, 5, 6, 4 ]>;     // < 5, 6, 4 >
def set3: <[ 6, 4, 5, 0 ]>;     // < 6, 4, 5, 0 >

set1 ?= set2;                   // false

set1 ?$= set2;                  // true
set1 !$= set3;                  // true
```

**Note:** Unordered (Set equality) comparison is slower than ordered comparison (Tuple equality). This cost is worth paying if you really need to compare two Sets, but it may be worth examining if a different approach is feasible.

## Functions

To define a function, use the `defn` keyword. To return a value from anywhere inside the function body, use the `^` sigil:

```java
defn add(x,y) { ^x + y; }
```

Function definitions are always hoisted to their enclosing scope:

```java
add(6,12);                          // 18
| add 6, 12 |;                      // 18

defn add(x,y) { ^x + y; }
```

Function definitions are also expressions (first-class values), so they can be assigned and passed around:

```java
def myFn: defn add(x,y) { ^x + y; };

myFn(6,12);                         // 18
add(6,12);                          // 18

somethingElse(myFn);
```

Function definition expressions can also be immediately invoked, using the evaluation-expression:

```java
|
    |defn add(x,y) { ^x + y; }| 6, 12
|;                                  // 18
```

**Note:** In the above example, the `|` pair surrounding the `defn` expression is technically optional. But it's recommended for readability sake, to visually disambiguate where the inline function expression begins and ends.

Concise function definitions may omit the name and/or the `{    }` around the body, but the concise body must be an expression marked by the initial `^` return sigil:

```java
def myFn: defn(x,y) ^x + y;

|
    |defn(x,y) ^x + y| 6, 12
|;                                  // 18
```

### Default Parameter Values

To default a function parameter value:

```java
defn add(x: 0, y: 0) ^x + y;
```

The default is applied if the corresponding argument supplied has the `empty` value, or if omitted.

### Gathering Arguments

To specify a function that collects all individual/positional arguments into a single list (Tuple):

```java
defn add(*nums) ^| + ...nums |;
```

The `*` must immediately precede a single identifier (with no default value), and must be the only listed parameter in the function definition. All arguments passed to this function will be gathered in a single `nums` value.

**Note:** Functions with this variadic parameter gathering in the signature cannot be partially applied.

### Negating A Predicate

A predicate is a boolean-returning function. For example:

```java
defn isOdd(v) ^mod(v,2) ?= 0;
```

It can be quite useful to negate a predicate; this can be expressed with the unary `!` prefixing a function value:

```java
def isEven: !isOdd;

// or:
def isEven: | ! isOdd |;
```

**Note:** `!` is overloaded to produce a negated (aka, complement) function if used against a function value. Otherwise, it acts to flip/negate a boolean value.

### Function Pre-conditions

It's common that we write function logic while making certain assumptions (aka: expectations, requirements, pre-requisites) for the parameter inputs.

Functions should be as obvious as possible in surfacing such assumptions, rather than merely embedding this logic into the function body's runtime. Ideally, these pre-conditions are part of the explicit function signature, so a reader doesn't need to inspect and mentally execute the function's implementation.

Additionally, some pre-conditions may be verifiable at compile time. And even more importantly, pre-conditions can be evaluated *before* the function has been invoked, where a function might not even need to be invoked!

----

In most other programming languages, a pre-condition means: "if this condition *is not met*, the function cannot run". We might even call this an "assertion". And in some languages, exceptions might be thrown to indicate this failure.

In **Foi**, it's the opposite (indeed, **Foi** doesn't have exceptions).

We're intentionally flipping the mental model from "the function runs only if it *can*" to "the function runs only if it needs to". If a function's pre-condition *is met*, the function **doesn't need to run**; its result value is already explicitly known.

----

These aspects of the function's signature go beyond parameter [type annotations](#type-annotations). It's more than, "is this parameter always an `int`?"; pre-conditions are lifted to the call-site, applied against the function's argument input *value(s)*, and indeed the *relationship(s)* between such argument values.

Consider a function that returns `1` if its argument is less than or equal to `1`. We might call this a "base condition" or an "early return" in certain styles of programming.

You *could* write it this way:

```java
defn myFn(x) {
    ?{
        ?[x ?<= 1]: ^1
        ?: empty
    }

    // ..
}
```

The problem is, this `^1` "early return" isn't particularly obvious, and requires reading into the body to determine.

**Foi** functions ***can and should do better***.

Pre-conditions are [guard expressions](#guard-expressions), of the form `?[    ]: expr` or `![    ]: expr`, which are applied to *guard* against the need to run the function. One or more of these pre-conditions may appear in the function definition, between the `(    )` parameter list and the body of the function -- either the `{    }` full body, or the the `^`-denoted concise expression-body.

Thus, the above `myFn()` function could be more appropriately defined as:

```java
defn myFn(x) ?[x ?<= 1]: 1 {
    // ..
}
```

Pre-conditions are evaluated -- hoisted to the call-site -- before actual function invocation. If a pre-condition matches, the consequent `expr` is evaluated and returned (no `^` return sigil) and thus the function invocation is skipped.

----

Just like with pattern matching expressions, a preceding `!` (in place of the `?`) negates the pre-condition. By using this form of a pre-condition, you somewhat conform it to the typical mental model of pre-conditions (as discussed earlier).

For example, if you want to define a function that only computes its result when the input is greater than `10`:

```java
defn myFn(x) ![x ?> 10]: empty {
    // ..
}
```

You can read/interpret the `![x ?> 10]: empty` pre-condition as: "x must be greater than 10; if it's not, return `empty` instead". That's basically the way we interpret pre-conditions in any programming language.

**Note:** In this usage, `empty` indicates to the calling code that the function had no valid computation to perform. However, there are other types of values that could (should!?) be returned here, such as a `None` or `Left` monads (more later).

----

If a function has multiple parameters, a pre-condition may imply a *relationship* between them. For example, to define a function where the first parameter must be larger than the second:

```java
defn myFn(x,y) ![x ?> y]: empty {
    ^(x - y);
}
```

Here, if `myFn(5,2)` is called, the result will be `3`. But if `myFn(2,5)` is called, the function won't be invoked at all, and the result (from the pre-condition) will be `empty`:

```java
defn myFn(x,y) ![x ?> y]: empty {
    ^(x - y);
}

def result1: ?(myFn(5,2)){
    ![empty]: #
    ?: 0
};
def result2: ?(myFn(2,5)){
    ![empty]: #
    ?: 0
/?;

result1;            // 3
result2;            // 0
```

### Named Arguments

To override positional argument-parameter binding at a function call-site, the evaluation-expression form can specify which parameter name each argument corresponds to (in any order):

```java
defn add(x: 0, y) ^x + y;

| add x:3, y:4 |;               // 7
| add y:5 |;                    // 5
```

**Warning:** Be careful in relying on this capability, as it creates an additional refactoring dependency burden between the function definition and its call-sites. If you change the name of a parameter in a function definition, any named-argument references at call-sites must be updated.

This code style means that function parameter naming has readability implications at call-sites. If you are using this code style at call-sites, consider "standardizing" certain generic variable naming conventions in your function definitions, to make using such functions predictable.

For example:

* a general value parameter might always be named `v` or `val`
* a function parameter might always be named `cb` or `fn`
* a list/array parameter might always be named `list`, `arr`, or even `vs` (i.e., the plural of `v`)

### Function Recursion

Function recursion is supported:

```java
defn factorial(v) ![v ?> 1]: 1 {
    ^v * factorial(v - 1);
}

factorial(5);                   // 120
```

Tail-calls (recursive or not) are automatically optimized by the **Foi** compiler to save call-stack resources:

```java
defn factorial(v,tot: 1) ![v ?> 1]: tot {
    ^factorial(v - 1,tot * v);
}

factorial(5);                   // 120
```

### Function Currying

Function definitions can optionally be curried:

```java
defn add(x)(y) ^x + y;

def add6: add(6);

add6(12);                           // 18
add(6)(12);                         // 18

// loose currying:
add(6,12);                          // 18
```

### Function Overs

Function definitions must declare side-effects (reassignment of free/outer variables) using the `:over` keyword:

```java
def customerCache: empty;
def count: 0;

defn lookupCustomer(id) :over (customerCache) {
    // ..

    // this reassignment side-effect allowed:
    customerCache := cacheAppend(customerCache,customer);

    // but this is disallowed because `count`
    // isn't listed in the `:over` clause:
    count := count + 1;
}
```

**Note:** Closure over free/outer variables -- specifically, (r-value) read-only access -- is allowed without being listed in the `:over` clause. The `:over` clause must only list free/outer variables that will appear in an (l-value) assignment-target position.

### Function Composition

Function composition can be defined in left-to-right style, with the `+>` flow operator:

```java
defn inc(v) ^v + 1;
defn triple(v) ^v * 3;
defn half(v) ^v / 2;

def compute1: inc +> triple +> half;
def compute2: | +> inc, triple, half |;

compute1(11);           // 18
compute2(11);           // 18
```

**Note:** The `+>` flow operator produces a unary function, meaning it will only accept and pass-along a single argument; any additional passed arguments are ignored.

It's also very common in FP to prefer right-to-left style composition. Probably the most obvious reason is the visual-ordering coherency between `half(triple(inc(v)))` and a composition argument list like `half, triple, inc`.

The `<+` compose-right operator is equivalent to using the `'` prime operator to reverse the order of the `+>` operator's arguments, as `'+>`:

```java
defn inc(v) ^v + 1;
defn triple(v) ^v * 3;
defn half(v) ^v / 2;

def compute1: | <+ half, triple, inc |;
def compute2: | '+> half, triple, inc |;

compute1(11);           // 18
compute2(11);           // 18
```

### Function Pipelines

By contrast, the `#>` pipeline operator (F#-style) operates left-to-right like this:

```java
defn inc(v) ^v + 1;
defn triple(v) ^v * 3;
defn half(v) ^v / 2;

11 #> inc #> triple #> half;        // 18

11 #> | +> inc, triple, half |;     // 18
```

The first expression in a pipeline must be a value (or an expression that produces a value). Each subsequent step must resolve to a function, which when invoked produces a value to pass on to the next step.

Since the `#>` operator is n-ary, multiple steps can also be used in the evaluation-expression form:

```java
defn inc(v) ^v + 1;
defn triple(v) ^v * 3;
defn half(v) ^v / 2;

| #> 11, inc, triple, half |;       // 18
```

Recall that we can reverse the order of arguments with the `'` prime operator, allowing us to do right-to-left pipelining (if we wanted to for some reason):

```java
defn inc(v) ^v + 1;
defn triple(v) ^v * 3;
defn half(v) ^v / 2;

| '#> half, triple, inc, 11 |;       // 18
```

The *topic* of a pipeline step is the result of the previous step, and is implicitly passed as the single argument to the step's function. But the *topic* (i.e., the previous step's result value) can be explicitly referred to with the `#` sigil:

```java
defn add(x,y) ^x + y;
defn triple(v) ^v * 3;
defn half(v) ^v / 2;

11 #> add(1,#) #> triple #> half;        // 18
11 #> | add 1, # | #> triple #> half;    // 18
```

Of course, if the `add()` function is curried, we can get back to point-free style (no need for the explicit `#` topic):

```java
defn add(x)(y) ^x + y;
defn triple(v) ^v * 3;
defn half(v) ^v / 2;

11 #> add(1) #> triple #> half;        // 18
11 #> | add 1 | #> triple #> half;     // 18
```

A *pipeline function* is a specialized function definition form that replaces the `^` return sigil with a `#>` pipeline as its concise body. The *topic* of the first step is automatically bound to the first parameter of the function:

```java
defn add(x,y) ^x + y;
defn triple(v) ^v * 3;
defn half(v) ^v / 2;

defn compute(x) #> add(1,#) #> triple #> half;

compute(11);    // 18
```

And again, if we define `add()` as curried function, we can avoid the `#` topic reference:

```java
defn add(x)(y) ^x + y;
defn triple(v) ^v * 3;
defn half(v) ^v / 2;

defn compute(x) #> add(1) #> triple #> half;

compute(11);    // 18
```

Compare this `#>` *pipeline function* form to the previously-discussed `+>` flow operator:

```java
defn add(x)(y) ^x + y;
defn triple(v) ^v * 3;
defn half(v) ^v / 2;

def compute: | +> add(1), triple, half |;

compute(11);    // 18
```

The previous `#>` *pipeline function* form is more powerful/flexible than the `+>` approach, in that a pipeline function can declare multiple parameters, and access any of them throughout the pipeline via `#`.

## Base Unit Functions

A unit function, also known as a unit constructor, is a utility for "constructing" a value. There are two helpful utility unit functions that are nearly ubiquitous in FP programs.

**Foi** provides both of these unit function utilities built-in.

### Value Identity Function

The first is often referred to as "identity". It's a function that takes a single input and simply returns it untouched. You could define your own like this:

```java
defn identity(v) ^v;

identity(42);       // 42
```

However, this utility is so commonly needed that **Foi** provides it built-in, as well as its shorthand:

```java
Value@42;           // 42
Value@ 42;          // 42
Value@(42);         // 42

@42;                // 42
@ 42;               // 42
@(42);              // 42

| @ 42 |;           // 42
```

Both `Value@` and the abbreviated `@` are defined as the identity function. They are interchangeable, so the choice between them is primarily a readability/style preference.

Additionally, as shown, **Foi** provides a shorthand when a single-argument function name ends in `@`; its argument can be applied without surrounding `(    )`, with or without whitespace between `@` and the argument, saving 1-2 characters.

**Note:** For readability/operator-precedence sake, you may still optionally use the `(    )` around the argument; it's technically treated as an expression grouping rather than part of the required infix function invocation syntax.

Here's an example of how you might use the identity unit function:

```java
defn formatRecord(record,formatFn: @) {
    // ..
    ^formatFn(record);
};
```

The `@` serves as a default pass-through function here.

### Null-Application Function

It's also very common to need a function which takes no argument, but returns some fixed value. There are places where a function is expected -- for example, operators like `~cata` or `~map` -- and yet we want to provide a placeholder function that returns a fixed value.

In **Foi** this is called "null application".

We can define either dedicated functions or inline functions for this purpose. For example:

```java
defn fortyTwo() ^42;

fortyTwo();             // 42
```

But since that may happen a lot, we might want a utility for constructing such functions:

```java
defn constant(v)() ^v;

def fortyTwo: constant(42);

fortyTwo();             // 42
```

**Note:** Notice the `()` in the `constant()` function definition, which matches with the `()` at the `fortyTwo()` call-site. That's where the label "null application" comes from.

Since this is so common in FP programs, **Foi** provides a null-application unit function:

```java
def fortyTwo: Function@42;
def hello: Function@ "Hello!";
def yes: Function@(true);

fortyTwo();             // 42
hello();                // "Hello!"
yes();                  // true
```

The name of this utility (`Function@`) is a nod to the fact that it's a unit constructor for a function, specifically a null-application function. But `Function@` is a little verbose, so there's also a shorthand: `@@`.

```java
def fortyTwo: @@42;
def hello: @@ "Hello!";
def yes: @@(true);

fortyTwo();             // 42
hello();                // "Hello!"
yes();                  // true
```

Here's an example of how you might use this utility:

```java
defn getName(record,getLabel: @@"Default") {
    ^(getLabel(record) + ": " + record.name);
};
```

The `@@` here constructs a default `getLabel` function that just returns the value `"Default"`.

## Loops

Perhaps some of the most distinctive features in various programming languages (FP-oriented versus more general) is the mechanics of looping/iteration. Imperative languages tend to have a variety of loop types (`for`, `while`, `do..while`, etc), whereas FP languages favor iterations/comprehensions (`map`, `filter`, `reduce` / `fold`, etc).

**Foi** is unquestionably an FP-oriented language, but tries (to an extent!) to cast a wider, more pragmatic net, in hopes of being inclusive of broader programming styles. As such, there's a unified syntax which can be used for both imperative looping and declarative iteration/comprehension.

Let's start with the typical imperative loop approach. Here's a loop that prints `"Hello!"` four times, using the `~each` loop operator:

```java
0..3 ~each {
    log("Hello!");
};
// Hello!
// Hello!
// Hello!
// Hello!
```

`~each` is a operator/function that can be used either in the infix form (shown above) or the evaluation-expression form. The first operand to `~each` defines the *range*, and the second operand defines the *iteration* operation(s).

1. The *range* is an expression that determines the *bounds* of the loop processing; this expression can take two forms:

    - If the *range* expression resolves to a Record/Tuple, the contents of the value are set as fixed *bounds* for loop processing. Examples of such an expression: an identifier, a function call, generated (`0..3`, as above), or explicit inline (such as `< 0, 1, 2, 3 >`).

    - If the *range* expression is a conditional of the form `?[    ]` or `![    ]` -- same as the conditional of an independent [pattern matching](#pattern-matching) clause -- the expression will be evaluated *before* each iteration, and will only proceed with the iteration if `true`; `false` signals the end of the *range* and terminates the loop.

        For example:

        ```java
        def done: false;

        ![done] ~each {
            // ..
        };
        ```

        This loop will keep running as long as `done` is false. The *range* could also have been written as `?[!done]`, but the former should generally be preferred as easier to read.

    - If the `range` expression is omitted, `~each` returns another function that expects a single argument defining the *range*. For example:

        ```java
        def printAll: ~each log;

        printAll(< 1, 3, 5, 7, 9 >);
        // 1
        // 3
        // 5
        // 7
        // 9
        ```

2. The *iteration* is an expression that defines what operation(s) to perform for each iteration. This expression can take several forms:

    - an expression that evaluates to a function to invoke for each iteration. For example:

        ```java
        0..3 ~each log;
        // 0
        // 1
        // 2
        // 3
        ```

    - an inline block with a `(    )` block-definitions clause (list of comma-separated definitions). For example:

        ```java
        2..5 ~each (v, idx) {
            log(idx + ": " + v);
        };
        // 0: 2
        // 1: 3
        // 2: 4
        // 3: 5
        ```

        **Warning:** Beware that any initializations of these definitions (e.g., `(v: 3, idx: 7)`) may very well be overwritten immediately, as they are assigned per-iteration according to the loop `range` and the iteration-type.

        If the loop iteration doesn't need any block-scoped definitions, omit the `(    )` block-definitions clause:

        ```java
        0..3 ~each {
            log("Hello!");
        };
        // Hello!
        // Hello!
        // Hello!
        // Hello!
        ```

In general, the result of the `~each` operation is another *range* (e.g., Record/Tuple), such that multiple `~each` expressions can be chained together. For example, `a ~each b ~each c`, which would loop performing `b` over the `a` *range*, then loop performing `c` over the resultant *range* from the first `~each` operation. The same would be true of `| ~each a, b, c |`.

**Note:** For `~each` looping over a Record/Tuple *range*, `~each` will result in the same Record/Tuple. But in the case where the *range* was a conditional, the result of `~each` will be the final boolean `false` that terminated the *range*.

## List Comprehensions

However, moving beyond imperative `~each` looping of Records/Tuples, **Foi** provides a variety of list comprehensions, including: `~map`, `~flatMap`, `~filter`, `~fold`, and `~foldR`.

These must all have a non-conditional *range* operand; when the *range* is a list (Tuple), they act as *list comprehensions*.

### Filter Comprehension (List)

The *iteration* operand for the `~filter` comprehension is a *predicate*, meaning for each value in the list (Tuple), it must compute a `true` to keep (aka, "filter in") the value, or `false` to discard (aka, "filter out") the value:

```java
defn isEven(v) ^mod(v,2) ?= 0;

def evens: 0..9 ~filter isEven;
// < 0, 2, 4, 6, 8 >

def odds: 0..9 ~filter (v) {
    !isEven(v);
};
// < 1, 3, 5, 7, 9 >
```

The `~filter` comprehension requires a list (Tuple) *range*, and its final result is always another list (Tuple).

**Note:** Just like with loops, all these comprehensions support the *iteration* operand being a function, an inline function definition, or an inline-block.

### Map Comprehension (List)

Perhaps one of the most common/recognizable list comprehensions is *map*:

```java
defn double(v) ^v * 2;

def evens: 0..5 ~map double;
// < 0, 2, 4, 6, 8, 10 >
```

```java
def evens: 0..5 ~map (v) { v * 2; };
// < 0, 2, 4, 6, 8, 10 >
```

**Note:** The `~map` comprehension expresses Functor/Mappable behavior from [broader Category Theory](#broader-category-theory-capabilities) (more later).

----

To compose multiple (`~map`) comprehensions:

```java
defn inc(v) ^v + 1;
defn triple(v) ^v * 3;
defn half(v) ^v / 2;

def odds: < 1, 3, 5, 7, 9 >;

odds ~map inc ~map triple ~map half;
// < 3, 6, 9, 12, 15 >

| ~map odds, inc, triple, half |;
// < 3, 6, 9, 12, 15 >

odds ~map | +> inc, triple, half |;
// < 3, 6, 9, 12, 15 >
```

Further, we can take advantage of omitting the *range* to create a function out of the comprehension composition:

```java
defn inc(v) ^v + 1;
defn triple(v) ^v * 3;
defn half(v) ^v / 2;

def compute1: ~map inc ~map triple ~map half;
def compute2: | ~map , inc, triple, half |;

compute1(< 1, 3, 5, 7, 9 >);
// < 3, 6, 9, 12, 15 >

compute2(< 1, 3, 5, 7, 9 >);
// < 3, 6, 9, 12, 15 >
```

**Note:** In the evaluation-expression form (for the `compute2` definition), the leading `,` in the arguments list indicates explicitly omitting the *range*, thereby producing a right-partially-applied function that expects the *range*.

----

A *map iteration* doesn't always have to produce a single discrete value.

Let's consider the case where a *map* operation itself returns a list (Tuple) instead of a single discrete value; the result is a list of sub-lists. This operation is typically called a *zip*:

```java
defn zip(xs,ys) ^xs ~map (x,i) { < x, ys[i] >; };

zip(< 1, 2, 3 >,< 4, 5, 6 >);
// < <1,4>, <2,5>, <3,6> >
```

As shown, returning a list (Tuple) from the *map iteration* ends up with a list of sub-lists. When a *zip* is called for, this is the approach.

### FlatMap Comprehension (List)

When *mapping* two or more lists (Tuples) together, sometimes what we want is a single-level list, with all the sub-lists flattened out. This operation can be referred to as a *merge*.

To perform *merge* (aka, flattening while mapping), we can use the `~flatMap` comprehension:

```java
defn merge(xs,ys) ^xs ~flatMap (x,i) { < x, ys[i] >; };

merge(< 1, 2, 3 >,< 4, 5, 6 >);
// < 1, 4, 2, 5, 3, 6 >
```

**Note:** You may recognize that "flatMap" often goes by alternate names in other contexts/languages: "bind", "chain", etc. As we'll see later with [Monadic Bind](#monadic-bind), `~flatMap` has various aliases: `~bind`, `~chain`, and the shorter/generally more preferred `~<`. They all work identically, so readability preferences dictate which to use.

### Do Comprehension (List)

It may not seem obvious yet, but `~<` (aka, `~flatMap`, `~bind`, or `~chain`) is likely to be a fairly common list (Tuple) comprehension operation in your programs.

For example, it's common to *flat-map* with multiple lists, and need access to a value from each list to perform the mapping:

```java
def xs: < 2, 4 >;
def ys: < 7, 8 >;

xs ~< (x) {
    ys ~< (y) {
        < x + y >;
    };
};
// < 9, 10, 11, 12 >
```

Notice that to pull this off -- accessing both `x` and `y` in the same scope! -- we had to nest the second `~<` operation inside the *iteration* scope of the first `~<` operation. That works, but it should *smell* a little bit.

----

By the way, we illustrated the above with two `~<` *flat-map*s, but there's an alternate way to think about it:

```java
def xs: < 2, 4 >;
def ys: < 7, 8 >;

xs ~< (x) {
    ys ~map (y) {
        x + y;
    };
};
// < 9, 10, 11, 12 >
```

Notice that here, I replaced the innermost `~<` with a `~map`, and then omitted the `<    >` Tuple wrapped around the `x + y` computation.

The end result is equivalent. Generally, it doesn't really matter which way makes most sense to you; pick one and go with it. But for this guide, I'll stick with the former double `~<` processing model.

----

Whichever way we mentally model this task, we'll likely encounter it rather often. And as ugly as multiple nested scopes can get (especially if there's 3 or more of these steps/nestings involved), **Foi** provides a special comprehension: `~<<`, called the *do comprehension*.

**Note:** In other languages (e.g., Haskell, Scala, etc), you may see something similar referred to as the "do syntax", most commonly in a monadic context. However, here we're broadening/generalizing the concept to include lists (Tuples).

Consider:

```java
def xs: < 2, 4 >;
def ys: < 7, 8 >;

List ~<< {
    def x:: xs;
    def y:: ys;
    x + y;
};
// < 9, 10, 11, 12 >
```

There are several *special* things going on here. But hopefully once I describe the processing steps, you'll recognize it as the same as the first snippet (double `~<` version) at the top of this section.

First off, the `~<<` operator's "range" operand is [`List`](#list-monad), the formal type for a Tuple. That should seem a bit strange, since nothing should really happen if you perform a comprehension/iteration across an empty list... right? But it's even more unusual, in that this is a **type** for the *range* rather than a concrete value. Just hang with me.

Notice the `::` instead of the typical `:` in the `def` statement. What `def x:: xs` is saying is, pull out each value from `xs`, one at a time, and assign each value to `x`, on successive iterations of the block.

*THAT* you should recognize as a comprehension. Moreover, the second `def y:: ys` is pulling out each value from `y` one at a time, and assigning it to `y`. Again: comprehension.

Here's the tricky part: the second comprehension is happening **for each iteration of the first comprehension**. In other words, the second is *nested in* the first. Just like the earlier double `~<` snippet, right?

When each *iteration*'s `x + y` operation is performed, the result is automatically wrapped in **the same type as** the *range* context -- remember: if omitted, an empty `<>` tuple is assumed for the *range*. Finally, that result is *flat-map*ped into the overall result.

We can also just include the special iterative assignments in in a [block-definitions clause](#block-definitions-clause):

```java
~<< (x:: xs, y:: ys) {
    x + y;
};
// < 9, 10, 11, 12 >
```

As both forms are equivalent, readability and other code maintenance concerns dictate the appropriate style to choose.

----

Remember, the final value produced in the iteration block is automatically *wrapped* in the comprehension *range* type (list/Tuple here). That means a *double-wrapping* effect would occur if the value produced is already a list (Tuple); this might be desired but often is a mistake.

Consider the non-*do comprehension* form:

```java
def xs: < 2, 4 >;
def ys: < 7, 8 >;

xs ~< (x) {
    ys ~map (y) {
        < x + y >;   // <-- notice the tuple and ~map
    };
};
// < < 9 >, < 10 >, < 11 >, < 12 > >     <-- oops!
```

When it's a literal value like this, you can omit the `<    >`.

But when that "final" value comes from an outside computation, we can't just omit the `<    >`. So we're forced to use `~<` instead of `~map`:

```java
defn tup(x,y) ^(< x + y >);

def xs: < 2, 4 >;
def ys: < 7, 8 >;

xs ~< (x) {
    ys ~< (y) {      // <-- notice ~< instead of ~map
        tup(x,y);
    };
};
// < 9, 10, 11, 12 >
```

But for the *do comprehension* form, we ostensibly cannot control the final step's behavior:

```java
~<< (x:: xs, y:: ys) {
    tup(x,y);
};
// < < 9 >, < 10 >, < 11 >, < 12 > >     <-- oops!
```

So here's some workarounds:

```java
~<< (x:: xs, y:: ys) {
    tup(x,y).0;         // ugh!
};
// < 9, 10, 11, 12 >


~<< (x:: xs, y:: ys) {
    def v:: tup(x,y);   // meh
    v;
};
// < 9, 10, 11, 12 >
```

But these are a bit annoying, right?!

When you need to *unwrap* the final value, there's a special syntactic shortcut, a prefix of `::` on the final expression:

```java
~<< (x:: xs, y:: ys) {
    ::tup(x,y);         // <-- notice the ::
};
// < 9, 10, 11, 12 >
```

This special `::` usage (without a `def`) may only prefix the **final expression** in the *iteration* block of the *do comprehension*.

You can think about this `::` as instructing the *do comprehension* to perform a final `~<` (instead of `~map`). Or you can think about it as skipping the automatic value wrapping that would otherwise occur.

----

You may have noticed that a single list (Tuple) in this `~<<` *do comprehension* form is equivalent to the `~map` comprehension on a single list:

```java
def xs: < 2, 4 >;

~<< (x:: xs) {
    x * 10;
}
// < 20, 40 >


xs ~map (x) {
    x * 10;
}
// < 20, 40 >
```

The `~map` form is clearer here -- seems a bit more obvious and less magical -- and should probably be preferred in the specific case of having a single comprehension.

By contrast, the *do comprehension* form really shines when there are multiple comprehensions composed together, especially if computations need to access values from each of them, in a single scope. That's precisely what the magical *do comprehension* is all about.

### Fold Comprehensions (List)

The `~fold` comprehension (left-to-right), often referred to as *reduce*, works like this with lists (Tuples):

```java
defn add(x,y) ^x + y;

0..9 ~fold add;
// 45   (0 + 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9)

0..9 ~fold (acc,v) {
    acc + v;
};
// 45   (0 + 1 + 2 + 3 + 4 + 5 + 6 + 7 + 8 + 9)
```

**Note:** The final result (and type) of the `~fold` comprehension is determined by the return value of the final *iteration* evaluation.

In this form of the `~fold` comprehension, the first *iteration* evaluation will receive the first value from the list (Tuple) as the *accumulator* (above: `x`, `acc`) argument, and the second value from the list as the *iteration-value* (above: `y`, `v`) argument.

So in the above snippet, the first invocation of `add()` in each comprehension will have `x` set as `0` and `y` set as `1`. That return result (`1`) comes in as `x` for the second invocation of `add()`, with `y` being `2`. And so on.

----

But, folds/reductions sometimes need to start with a specific *initial value* rather than with just the first value in the list (Tuple). In this usage, the `~fold` comprehension is a representation of generalized [Foldable behavior](#foldable--catamorphism) (covered later).

If the `~fold` comprehension's *range* argument is a list (Tuple), and there are at least two other operands specified (i.e., 3 or more operands total), then the second operand will be interpreted as the *initial-value* (or *default-value*), and the third operand is an *iteration* expression.

For example:

```java
defn sub(x,y) ^x - y;

| ~fold 1..5, 100, sub |;
// 85   (100 - 1 - 2 - 3 - 4 - 5)
```

**Note:** As shown, 3 or more operands require the evaluation-expression form.

When an initial value is provided to a fold/reduce, it is set as the *accumulator* argument to the first *iteration*, and the first value in the list (Tuple) is the *iteration-value* argument.

So in the snippet above, `100` is the *initial value*, and comes in as `x` in the first invocation of `sub()`, with `y` being `1`. That return result (`99`) comes in as `x` for the second invocation of `sub()`, with `y` being `2`. And so on.

For a `~fold` comprehension to execute, there should be at least two values to operate upon. If the list (Tuple) only has one value in it, then specifying the *initial-value* provides this second value.

If there's only one value (either in the list or as the specified *initial-value*), the `~fold` comprehension short-circuits (i.e., skips evaluating the *iteration* operand), returning that single value.

But if there isn't at least one value provided, the `~fold` operation is invalid, and `empty` is the result.

**Note:** You can configure **Foi** to issue a warning notice in such a case.

----

Folds *can even* produce another Record/Tuple result. One typical way to accomplish this is for the *initial-value* operand to be an empty Record/Tuple (`<>`):

```java
defn onlyOdds(list,v)
    ![mod(v,2) ?= 1]: list
        ^list + < v >

| ~fold 0..9, <>, onlyOdds |;
// < 1, 3, 5, 7, 9 >
```

The `~foldR` comprehension works identically to the `~fold` comprehension, but in right-to-left order. Compare the two comprehensions here:

```java
defn sub(x,y) ^x - y;

1..5 ~foldR sub;
// -5    (5 - 4 - 3 - 2 - 1)

1..5 ~fold sub;
// -13   (1 - 2 - 3 - 4 - 5)
```

## Monads (And Friends)

The identity monad in **Foi** is called `Id`, and the empty monad is called `None`. These two are actually paired in the [`Maybe` monad type](#maybe-monad), as discussed later.

The `@` operator applies the "unit constructor" for any monad type, thus a monadic value can be constructed like this:

```java
def m: Id@ 42;          // Id{42}
| @ Id, 42 |;           // Id{42}

def nil: None@;         // None
| @ None |;             // None
```

A monadic value is a primitive, immutable value type in **Foi**, meaning equality comparison is structural (just like Records/Tuples). As such:

```java
def m: Id@ 42;
def g: Id@ 42;

m ?= g;                 // true
```

For any monadic value besides instances of `None`, if `@` is partially applied with a monad type, you get a regular factory function for constructing that specific monad instance:

```java
def ofId: Id@;

ofId(42);               // Id{42}
```

A monadic value is a valid *range* for certain comprehensions, as we'll now explore.

### Monadic Map

Since [broader Category Theory](#broader-category-theory-capabilities) shows that Monads are Functors/Mappables, monadic values can also be used as the *range* for a `~map` comprehension:

```java
defn double(v) ^v * 2;

def m: Id@ 21;

m ~map double;                  // Id{42}

m ~map (v) { v * 2; };          // Id{42}
```

**Note:** Unless the *range* is a [`List` monad](#list-monad), the *iteration* for monadic `~map` will always be provided the instance's single held value as argument; for list (Tuple) and the `List` monad, the *iteration* is provided both value and index arguments.

----

Comprehension operations for `None` are all no-ops, meaning they do nothing but simply return the same `None` value again:

```java
defn double(v) ^v * 2;

Id@ 21 ~map double;     // 42
None@ ~map double;      // None
```

**Note:** The `double()` invocation won't happen for the `None@`-constructed monadic value.

### Monadic Bind

Monadic values also (obviously!) support the `~bind` comprehension:

```java
defn double(v) ^v * 2;

def m: Id@ 21;

m ~bind (double +> Id@);        // Id{42}
m ~flatMap (double +> Id@);     // Id{42}
m ~chain (double +> Id@);       // Id{42}
m ~< (double +> Id@);           // Id{42}
```

**Note:** As shown, for convenience/familiarity sake, `~flatMap`, `~chain` and `~<` are all aliases for the `~bind` comprehension. All 4 are interchangeable, but for brevity sake, `~<` is generally most preferable.

### The Monad Laws

For formality sake, here are the 3 monad laws demonstrated, using the `Id` monad (via its `@` unit-constructor) and the `~<` *bind* operator:

1. **Left Identity:**

    ```java
    defn incM(v) ^(Id@ v + 1);

    Id@ 41 ~< incM;
    // Id{42}
    ```

2. **Right Identity:**

    ```java
    Id@ 42 ~< Id@;
    // Id{42}
    ```

3. **Associativity:**

    ```java
    defn incM(v) ^(Id@ v + 1);
    defn doubleM(v) ^(Id@ v * 2);

    Id@ 20 ~< incM ~< doubleM;
    // Id{42}

    Id@ 20 ~< (v) {
        incM(v) ~< doubleM;
    };
    // Id{42}
    ```

### Monadic Do Comprehension

Recall the [*do comprehension* for lists](#do-comprehension-list) with the `~<<` operator.

In the same way that `~<<` makes it convenient to compose multiple list comprehensions (via `~<` *chain*ing) while accessing values from each within a single scope, the same is possible and useful for monadic comprehensions.

Here's the nested `~<` form:

```java
defn inc(v) ^v + 1;
defn double(v) ^v * 2;

def incM: inc +> Id@;
def doubleM: double +> Id@;

incM(1) ~< (x) {
    doubleM(x) ~< (y) {
        Id@ (3 * x) + y;
    }
};
// Id{10}
```

To have access to both `x` and `y` in the final *chain* step, we used a nested scope. Alternatively, you could pack both values into a Record/Tuple to pass into the final step, but that's even uglier.

Here's the more ergonomic `~<<` *do comprehension* form:

```java
Id ~<< (x:: incM(1), y:: doubleM(x)) {
    (3 * x) + y;
};
// Id{10}


Id ~<< {
    def x:: incM(1);
    def y:: doubleM(x);
    (3 * x) + y;
};
// Id{10}
```

**Note:** These should look familiar to the various styles presented in the [Do Comprehension (List)](#do-comprehension-list) section earlier.

In these snippets, the only substantive difference from the list comprehension form is the `Id` (monad type) being passed as the first (*range*) operand to `~<<`. This provides the *type* of monad to wrap (via its `@` unit constructor) around the final computed value. Remember: if omitted, a general list (Tuple) type is assumed, which is *not* desired here.

Don't forget the special `::` prefix on the final expression, in case you need to omit the automatic monadic type wrapping:

```java
defn compute(x,y) ^Id@ ((3 * x) + y);

Id ~<< {
    def x:: incM(1);
    def y:: doubleM(x);
    compute(x,y);
};
// Id{Id{10}}            <-- oops!

Id ~<< {
    def x:: incM(1);
    def y:: doubleM(x);
    ::compute(x,y);   // <-- notice ::
};
// Id{10}
```

### Monadic Function Returns

A function's return value can be explicitly expressed monadically:

```java
defn incM(v) ^(Id@ v + 1);

incM(3);                // Id{4}
```

Non-monadic-returning functions can also be composed with the intended unit constructor:

```java
defn inc(v) ^v + 1;
def incM: inc +> Id@;

incM(3);                // Id{4}
```

That approach is often useful if the non-monadic-returning function is already independently defined, so effectively a monad is just being *wrapped* around the function's return value.

Monadic function returns can also be integrated directly into a function's logic:

```java
defn factorialM(v,tot: Id@ 1) ![v ?> 1]: tot {
    tot := tot ~map (t) { t * v; };
    ^factorialM(v - 1,tot);
}

factorialM(5);                   // Id{120}
```

### Pattern Matching Monads

You can use [pattern matching](#pattern-matching) to determine which type of monad instance is being dealt with:

```java
def m: getSomeMonad(42);

?(m){
    ?[?as Id, ?as Right]: something(m)
    ?[?as None]: something(m,"default!")
    ?[?as Left]: something(m,"oops")
    ?: something(Left@ "Unknown!","oops")
};
```

**Note:** More on [types and the `?as` operator](#type-annotations) later.

### `List` Monad

For something to be a monad, we need it to be able to satisfy [the 3 monadic laws](#the-monad-laws). In particular, it needs a *bind* operation and it needs a unit constructor.

Well... we've already seen that lists (Tuples) have the `~<` (aka `~flatMap`, `~bind`, or `~chain`) operator defined. So, all we're missing is a unit constructor for a list (Tuple).

Thankfully, **Foi** provides `List`, such that `List@` produces `<>` and `List @ 42` produces `< 42 >`.

We can thus prove `List` (any Tuple) is a monad:

```java
defn incM(v) ^(List@ v + 1);
defn doubleM(v) ^(List@ v * 2);

// (1) Left Identity:
List@ 41 ~< incM;
// < 42 >

// (2) Right Identity:
List@ 42 ~< List@;
// < 42 >

// (3) Associativity:
List@ 20 ~< incM ~< doubleM;
// < 42 >

List@ 20 ~< (v) {
    incM(v) ~< doubleM;
};
// < 42 >
```

So, there you go: the `List` monad.

### `Maybe` Monad

The `Id` and `None` monads are paired in the `Maybe` Sum type monad. For readability preferences, `Maybe.None` is an alias for `None`, and `Maybe.Id` is an alias for `Id`. Moreover, to adhere to the monadic laws, `Maybe@` is the same as `Id@`:

```java
Maybe@ 42;              // Id{42}
Maybe@ empty;           // Id{empty}
Id@ empty;              // Id{empty}
None@;                  // None
```

But that's not particularly useful or interesting: we could already select `Id` or `None` explicitly.

A special `Maybe._` constructor (not the main unit constructor) inspects the provided value and selects `None` when it encounters the `empty` value, and `Id` for any other value:

```java
Maybe._(42);            // Id{42}
Maybe._(empty);         // None
```

For syntactic consistency, you can still use the `@` form:

```java
Maybe._ @ 42;           // Id{42}
Maybe._ @ empty;        // None
```

For further convenience, you may choose to do this:

```java
def May: Maybe._@;

May(42);                // Id{42}
May(empty);             // None
May@ 42;                // Id{42}
May@ empty;             // None
```

Instead of using `Maybe._`, you can define custom constructor functions that select `None` or `Id` for various values:

```java
defn nonZero(v)
    ?[v ?= 0]: None@
    ^Id@ v;

def qty: nonZero(0);            // None
def cost: nonZero@ 1.99;        // Id{1.99}
```

One common idiom where `Maybe` is used is conditional property access:

```java
defn prop(name)(obj) ^Maybe._ @ obj[name];

def order: Id@ <
    shippingAddress: <
        street: "123 Easy St",
        city: "TX",
        zip: "78889"
    >
>;

order
~< prop("shippingAddress")
~< prop("street");
// Id{"123 Easy St"}

order
~< prop("billingAddress")
~< prop("street");
// None
```

----

As with other monad kinds, `Maybe` has a `~<<` *do comprehension* form:

```java
Maybe ~<< {
    def shipAddr:: prop("shippingAddress",order);
    def street:: prop("street",shipAddr);
    Id ~<< (streetV:: street) {
        log("Street: " + streetV);
    };
};
// Street: 123 Easy St


Maybe ~<< {
    def billAddr:: prop("billingAddress",order);
    def street:: prop("street",billAddr);
    Id ~<< (streetV:: street) {
        log("Street: " + streetV);
    };
};
// None
```

Since the `prop("billingAddress",order)` returns a `None`, the rest of that second `Maybe ~<< {    }` *do comprehension* will short-circuit exit.

### Foldable / Catamorphism

We briefly mentioned Foldable earlier, with the `~fold` comprehension on lists (Tuples). We'll revisit the generalized Foldable (more broadly, Catamorphism) behavior now, in the context of monads.

Sum type monads in **Foi** have Foldable behavior built-in, expressed with the `~fold` / `~cata` (catamorphism) comprehensions.

For a monadic value *range*, the `~fold` / `~cata` comprehensions require multiple *iteration* expressions, one for each component of the Sum type; again, because this means 3 or more operands, the evaluation-expression form is required.

The difference between monadic `~fold` and `~cata` is with the *default iteration* operand. For `~fold`, this operand is an already computed *default-value*, whereas for `~cata` it's a function that computes a value. In either case, this operand is evaluated/used when the first/left-most component of the Sum type is encountered.

Let's consider a binary Sum type like `Maybe` (comprised of `None` and `Id`), which thus requires a *range* expression and **two** *iteration* expressions. For `None`, the *default iteration* expression (second operand, `"defaultMsg`s below) is evaluated; for `Id`, the *alternate iteration* expression (third operand, `@`s below) is invoked.

For example:

```java
def defaultMsg: @@"default!";

def m: Maybe._@ 42;                 // Id{42}
def g: Maybe._@ empty;              // None

| ~fold m, defaultMsg(), @ |;       // 42
| ~cata g, defaultMsg,   @ |;       // default!
```

As shown, for the `~fold` comprehension, we provide the already-computed result of the `defaultMsg()` function for the *default iteration* operand. With `~cata`, we provide that `defaultMsg` function reference, for it to be invoked only if needed.

**Note:** Folding (Catamorphism) of a monadic value *often* performs a *natural transformation* to another monadic type (via its unit constructor). Extracting a value (via identity `@` as shown) is a much less typical usage; that's merely convenient for illustration purposes here.

----

There's some useful conceptual symmetry to recognize, specifically between `~fold`ing a list (Tuple) with an *initial-value*, versus `~fold`ing a binary Sum type:

```java
defn two() ^2;
defn mult(x,y) ^x * y;

def list: < 1, 3, 7 >;
def m: Maybe._ @ 42;

| ~fold list,  two(), mult |;    // 42

| ~fold m,     two(), @    |;    // 42
| ~fold None@, two(), @    |;    // 2
```

For the list (Tuple) `~fold`, the `two()` function is eagerly invoked to provide the *initial-value* operand for the folding.

For the monadic `~fold`, the `two()` instead provides the *default-value* operand (used in the case of a `None` instance).

### `Either` Monad

The `Either` monad is another binary Sum type, pairing `Left` and `Right`. For readability preferences, `Either.Left` is an alias for `Left`, and `Either.Right` is an alias for `Right`. Also, the `Either@` unit constructor is an alias for `Right@`.

`Either` is typically used for error flow-control in FP programs (as opposed to exception handling).

By holding an error message in a `Left` -- similar to `None`, except it can represent an affirmative value -- this error message can propagate through monadic operations (which are skipped), until the program handles the message. `Right` is like `Id`, in that it only holds some *success* value, and all its comprehensions are valid.

In this example, the error message specified for `halve(0)` sets `e1` as a `Left`, and thus its associated `~map` comprehension does nothing:

```java
defn print(v) ^log("Value: " + v);
defn halve(v)
    ![v ?> 1]: Left@ "Value must be greater than 1"
    ![mod(v,2) ?= 0]: Right@ (v + 1) / 2
    ^Right@ v / 2;

def e1: halve(0);   // Left{"Value must be greater than 1"}
def e2: halve(4);   // Right{2}

e1 ~map print;      // Left{"Value must be greater than 1"}
e2 ~map print;      // Value: 2
```

Both `Maybe` and `Either` are foldable, so we can define *natural transformations* between them:

```java
defn halve(v)
    ![v ?> 1]: Left@ "Value must be greater than 1"
    ![mod(v,2) ?= 0]: Right@ (v + 1) / 2
    ^Right@ v / 2;

// natural transformation utilities
defn maybeFromEither(e)
    ^| ~fold e, None@, Id@ |;
defn eitherFromMaybe(m)
    ^| ~fold m, Left@ "Missing!", Right@ |;

def m1: halve(0) #> maybeFromEither;    // None
def m2: halve(4) #> maybeFromEither;    // Id{2}

| ~cata eitherFromMaybe(m1), @, @ |;    // "Missing!"
| ~cata eitherFromMaybe(m2), @, @ |;    // 2
```

Above, `halve(0)` returns a `Left` holding the error message, which we then transform to a `None` with the `maybeFromEither()` utility. `halve(4)` produces a `Right{2}`, which is transformed to `Id{2}`.

Then, for `m1` and `m2` instances, we perform a *natural* transformation back to `Either`, with the `eitherFromMaybe()` utility. We fold the resulting `Either` instances, extracting the values `"Missing!"` and `2`, respectively.

## Broader Category Theory Capabilities

So far, we've seen several behaviors/capabilities that are organized within broader Category Theory, such as Functor/Mappable (with `~map` comprehension), [Monad](#monads-and-friends) (with `~<` bind/chain comprehension), and [Foldable](#foldable--catamorphism) (with `~fold` / `~cata` comprehension).

But there are certainly other capabilities/behaviors to consider. While they may often show up adjacent to monads, these are separate topics.

### Applicative

Applicative is a pattern for holding a function in a monadic instance, then "applying" the underlying value from another monadic instance as an input to this function, returning the result back as another monadic value.

If the function requires multiple inputs, this "application" can be performed multiple times, providing one input at a time.

Here's how we can perform *Applicative* with `~<` and `~map`:

```java
defn add(x)(y) ^x + y;

def three: Id@ 3;
def four: Id@ 4;

(Id@ add)
    ~< (fn) {
        three ~map fn
    }
    ~< (fn) {
        four ~map fn
    };
// Id{7}
```

In this snippet, the `add()` function is wrapped in an `Id`, and then `~<` chained to access the `fn` it holds. That function is used to *map* the `three` monad, which calls `add(3)` and wraps the curried function back in another `Id`.

*That* `Id` is then `~<` chained again to access the curried function `fn`, and *that* function is used to *map* the `four` monad. This invokes `add(3)(4)` producing `7`, and then wraps that in yet another `Id`.

Restating: *Applicative* is pattern to *apply* the value(s) held in one or more monads, one at a time, as the inputs to a curried function (also held in a monad).

Of course, we could define a function helper to make this process a little cleaner:

```java
defn add(x)(y) ^x + y;
defn ap(m2)(m1) ^m1 ~< (fn) { m2 ~map fn; };

def three: Id@ 3;
def four: Id@ 4;

(Id@ add)
    #> ap(three)
    #> ap(four);
// Id{7}
```

However, for further built-in convenience and expressiveness, **Foi** provides the `~ap` operator:

```java
defn add(x)(y) ^x + y;

def three: Id@ 3;
def four: Id@ 4;

(Id@ add)
    ~ap three
    ~ap four;
// Id{7}
```

### Concatable / Semigroup

Concatable (formally, Semigroup) defines the ability for values to be "concatenated" (combined with each other).

We've already seen a number of value types in **Foi** that are concatable. For example: strings and numbers. In fact, the `+` operator invokes the underlying *concatable* behavior for all such value types:

```java
1 + 2 + 3 + 4 + 5;                  // 15
| + 1, 2, 3, 4, 5 |;                // 15

"Hello" + " " + "world";            // "Hello world"
| + "Hello", " ", "world" |;        // "Hello world"
```

Most monadic values in **Foi** also implement Concatable, meaning that if used with `+`, they will delegate concatenation to their underlying value (if it is also Concatable):

```java
(Id@ 30) + (Id@ 12);
// Id{42}

| + Id@ "Hello", Id@ " ", Id@ "world" |;
// Id{"Hello world"}
```

### Monoid

Monoid is a Semigroup plus an "empty value" -- such that when concatenated with, the original value is unchanged. For example, these are all monoids:

* string concatenation with the `""` (empty string)
* numeric addition with the `0` (empty number)
* list (Tuple) concatenation with the `<>` (empty list/Tuple)

Revisiting concatenation from the previous section, a concatenation is a specialized type of fold, so we can use the list (Tuple) `~fold` comprehension together with the `+` operator:

```java
| ~fold < "Hello", " ", "world" >, + |;
// "Hello world"
```

We can do the same with a list (Tuple) of monadic (monoidal) values:

```java
| ~fold < Id@ "Hello", Id@ " ", Id@ "world" >, + |;
// Id{"Hello world"}
```

It can be useful to perform a mapping on each of a list's values as they're being folded/concatenated, especially when that mapping is to lift non-monadic-but-monoidal values into monads.

The `~foldMap` comprehension does so, while applying the `+` concatenation as its *fold*:

```java
< "Hello", " ", "world" > ~foldMap Id@;
// Id{"Hello world"}
```

As shown, the `Id@` unit constructor maps each string to a monad, and then since the `Id` monad has a *concatenation* monoid (`+`) defined -- and the underlying string values are monoidal with a `+`string *concatenation* -- `~foldMap` then folds the monads (monoids) together, producing the single `Id{"Hello world"}` value.

----

For the list (Tuple) `~fold` comprehension, recall that:

* To perform a *fold*, at least two values are necessary, either both in the list (Tuple) to *fold*, or one value in the list and the *initial-value* provided in the expanded `~fold` form.

* If there's only one value provided, the *iteration* is skipped and that value is simply returned.

* If there's no value available, the `~fold` operation is invalid and returns `empty`; configure **Foi** to report a warning in such a case.

Thankfully, this familiar behavior is defined the same for `~foldMap`. Thus, if the list (Tuple) only has one element, the "empty value" of a monoid (`""` below) is a useful candidate to provide as the *default-value*, to ensure a *fold* actually occurs:

```java
| ~foldMap < "Hello" >, "", Id@ |;
// Id{"Hello"}
```

----

To define any custom monoid, provide a suitable "empty value", as an *initial-value* to the `~fold` / `~foldMap` operations.

For example, we could define a monoid as the boolean-AND (`?and`) *concatenation* of two boolean values, with `true` as the "empty value". And we can do similarly for the boolean-OR (`?or`) operation, with `false`.

Consider:

```java
defn all(bools) ^| ~fold bools, true, ?and |;
defn any(bools) ^| ~fold bools, false, ?or |;

def a: < true, true, true, true >;
def b: < true, false, true, true >;

all(a);     // true
all(b);     // false

any(b);     // true
```

The above is a nice application of a monoid, but the `?and` and `?or` operators are already n-ary, thus we could have done:

```java
defn all(bools) ^| ~fold bools, true, ?and |;
defn any(bools) ^| ~fold bools, false, ?or |;

def a: < true, true, true, true >;
def b: < true, false, true, true >;

all(a);     // true
all(b);     // false

any(b);     // true
```

So how do we apply this approach for monadic values such as `Id{true}`?

We *have* already identified the necessary monoidal empty value (`true` or `false`), which can be wrapped in `Id` (i.e., `Id{true}` and `Id{false}`). However, there's no default *concatenation* operation (`+`) defined for booleans in **Foi**; how could it know automatically whether the *concatenation* of booleans should be computed with logical-AND or logical-OR!?

Moreover, even if there were such a default `+` concatenation for booleans, the `?and` and `?or` operators are not monad-aware.

So, let's instead define custom, monad-aware concatenation logic (`andM()` and `orM()` below):

```java
defn andM(x,y) ^x ~map (xv) { xv ?and y; };
defn orM(x,y) ^x ~map (xv) { xv ?or y; };

defn all(bools) ^| ~fold bools, Id@ true, andM |;
defn any(bools) ^| ~fold bools, Id@ false, orM |;

def a: < true, true, true, true >;
def b: < true, false, true, true >;

all(a);     // Id{true}
all(b);     // Id{false}

any(b);     // Id{true}
```

`~fold` is more flexible in letting you specify custom *fold*ing (concatenation) logic for values. By contrast, `~foldMap` assumes/relies on the built-in `+` operation for values being *fold*ed (and recursively, concatenating underlying values).

## Concurrency / Asynchrony

**Foi** does not have any concurrency or asynchrony built natively into it. However, programs absolutely perform and respond to external operations that are inherently concurrent/asynchronous -- network calls, file system, timers, etc.

As such, there are a variety of language features for *managing* concurrency. These features are oriented around data transmission, but they can also be used more generally for asynchronous flow control.

**Note:** all of these features inherently operate synchronously in the language. Only external mechanisms outside the program can influence a non-synchronous delay into the program.

### `Promise` Monad

The most basic component of **Foi** concurrency/asynchrony is `Promise`. It resembles promises in other languages (JS, etc), but has some important differences.

Most importantly, `Promise` is a monad. It's kind of like the `Id` monad (it just holds a single value), except it starts out in a *pending* state where it doesn't yet hold any value. Any operations (`~map`, `~<`) are deferred while a promise is pending.

Once the promise is resolved, any deferred operations are immediately (synchronously) performed. From then onward, the promise remains permanently in the *resolved* state, and any operations against it are evaluated synchronously.

You *can* construct a `Promise` instance that's already resolved with the unit constructor:

```java
def pr: Promise@ 21;
// Promise{21}

pr.resolved();
// true

pr ~map (v) {
    v * 2;
};
// Promise{42}
```

If you're constructing a promise that will be resolved later, you actually need to construct a *subject*.

A subject is a record that contains a `pr` holding the promise, and a `resolve()` function for resolving the associated promise:

```java
def subj: PromiseSubject@;

subj.pr.resolved();
// false

def pr2: subj.pr ~map (v) {
    v * 2;
};
// Promise{..pending..}

pr2 ~map (v) {
    log("v: " + v);
};
// Promise{..pending..}

subj.resolve(21);
// Right{21}
// v: 42
```

Promises in **Foi** only have a single resolved state, unlike in JS where they can be fulfilled or rejected. To communicate something like "success" or "failure", the most appropriate (FP) way is to resolve the promise with an `Either`. This can then be folded into one promise or another, to fork different code paths.

Consider:

```java
defn fetchCustomers() { ..returns promise.. };

defn getCacheData(key)
    ![cache ?has key]: Promise@ (Left@ "Not in cache")
    ^Promise@ (Right@ cache[key]);


getCacheData("customers")
~< (resE) {
    (| ~cata resE, fetchCustomers, Promise@ |);
}
~map printRecords;
// Promise{..pending..}
```

The `getCacheData()` function produces a `Promise{Either}` value, which resolves to a `Left` on failure or `Right` on success.

We use the `Left` to fetch the customers remotely, or the `Right` to simply pass-through to the next step, which invokes `printRecords()`.

----

Instead of constructing multi-step `~<` / `~map` chains, `Promise` also supports the helpful [`~<<` *do comprehension*](#monadic-do-comprehension). The promise chain from the above snippet could instead be expressed as:

```java
Promise ~<< {
    def resE:: getCacheData("customers");
    def customers:: (| ~cata
        resE, fetchCustomers, Promise@
    |);
    printRecords(customers);
};
// Promise{..pending..}
```

If the promises returned from `getCacheData()` or the `~cata` operation are pending when encountered, the rest of the *do comprehension* block is suspended until the promise resolves.

**Note:** This kind of code may be familiar/recognizable as "async..await" style in JS.

----

One question that may now come to mind: how can you perform *asynchronous comprehensions* (looping), where a pending promise suspends the iteration/looping until it's resolved?

Here's one way:

```java
defn fetch(url) { ..returns promise.. };

defn printResponses(prs)
    ![size(prs) ?> 0]: Promise@ "Complete."
{
    ^prs.0 ~< (resp) {
        log("resp: " + resp);
        printResponses(prs.[1..]);
    };
};

def urls: <
    "https://some.url/1",
    "https://some.url/2",
    "https://some.url/3",
>;

printResponses(urls ~map fetch)
~map log;
// Promise{..pending..}
//
// ... eventually ...
//
// resp: response 1
// resp: response 2
// resp: response 3
// Complete.
```

The `printResponses()` function "asynchronously loops" through the `prs` list of promises, with the recursive call to `printResponses()` chained off each promise.

That approach works OK, but it's unfortunately a bit convoluted.

The `~<*` operator extends `Promise` *do comprehension*, to loop asynchronously over (a list of) `Promise` instances. The above `printResponses()` function can thus be expressed as:

```java
defn printResponses(prs) {
    ^prs ~<* (resp) {
        log("resp: " + resp);
    }
    ~map { "Complete."; };
};
```

If (as above) the provided *range* operand is a list of promises, iterating will proceed (synchronously or asynchronously) as each next promise resolves. The result of the comprehension is a pending promise that resolves once the iterating has completed.

This form can be thought of as like an asynchronous `~each` comprehension. That's a fair bit cleaner than the async-recursion approach, right!?

**Note:** Any non-promise values in the *range* list will be "lifted" to a resolved promise for the purposes of the iteration handling.

Moreover, as the `~<` part of the `~<*` operator suggests, the *iteration* clause is also a *do comprehension* block over promises (i.e., `Promise ~<< {    }`):

```java
urls ~map fetch
~<* (resp) {
    def v:: processResp(resp);
    def success:: storeVal(v);
    ?[success]: log("Stored: " + v);
};
// Promise{..pending..}
```

**Note:** Any value unwrapped via `def ::` in a `~<*` comprehension must be a Promise.

----

If the *range* operand provided to `~<*` is not a concrete value but instead a type (like `Promise`) -- as *range* always is with regular `~<<` do comprehensions -- the iteration looping will continue until a `Left` is produced as the **final result** of an iteration:

```java
defn printResp(v) { log("Resp: " + v); };
defn fetchMoreData() { ... Promise<Either> ... };

Promise ~<* {
    def respE:: fetchMoreData();
    Either ~<< (resp:: respE) {
        printResp(resp);
    };
};
// Promise{..pending..}
```

The outer `~<*` *looping do comprehension* above pauses each iteration while the promise returned from `fetchMoreData()` is waiting to resolve.

Once it does, the inner `Either ~<< {    }` *do comprehension* attempts to unwrap the `respE` Either value. If a `Left` is encountered, this inner *do comprehension* short-circuits out, and its resultant `Left` value terminates the loop. Otherwise, `printResp(resp)` is called, and the outer looping is allowed to keep going.

----

It's common to want to perform operations across multiple promises. Two such promise combinators come built into **Foi**.

`race()` creates a promise that will resolve as soon as the first promise in the provided list resolves (left-to-right ordering):

```java
def subj1: PromiseSubject@;
def subj2: PromiseSubject@;

race(< subj1.pr, subj2.pr >)
~map (v) {
    log("Value: " + v);
};
// Promise{..pending..}

subj2.resolve(42);
// Right{true}
// Value: 42

subj1.resolve(10);
// Right{true}
```

`all()` creates a promise that will resolve once all promises in the provided list have resolved; the resolved value will be a list of those source promise resolutions in the same order as the list provided to `all()`, regardless of the order of resolution operations:

```java
def subj1: PromiseSubject@;
def subj2: PromiseSubject@;

all(< subj1.pr, subj2.pr >)
~map (vals) {
    log("Values: " + vals);
};
// Promise{..pending..}

subj2.resolve(42);
// Right{true}

subj1.resolve(10);
// Right{true}
// Values: < 10, 42 >
```

### `Channel`

CSP (Communicating Sequential Processes) is a classic pattern for coordinating data transmission. It's primarily used in languages like Go and Clojure, though it's been implemented in many others (including JS).

CSP Channels are a mechanism primarily useful for coordinating communication of values between separate (independent) aspects of a program, without these aspects needing to *know* about each other explicitly; the only thing they *share* is the *channel* communication conduit.

One such common coordination pattern is referred to as "back pressure", in that the producer(s) of a value into a channel is throttled to produce no faster than the consumer(s) of that channel are willing to take these values out.

----

**Foi** defines CSP as a first class feature, via the `Channel` type.

It's important to distinguish: a `Channel` instance is not *itself* monadic -- it has no `~<` / `~map` defined. However, all the operations on `Channel` instances produce `Promise` monad instances, so you still interact with `Channel` in a monadic-oriented way.

A channel is a container (by default, with no internal buffer) where both the putting-in of a value (`put()`) and reading-out of a value (`take()`) are *coordinated*. Both operations produce a `Promise` instance, and these promises won't resolve until both operations have occurred (regardless of ordering).

Both operations' promises resolve with `Either` values, indicating (eventual) success/failure of the operation.

Consider these illustrative helpers, which we'll use throughout the rest of our CSP examples:

```java
defn putVal(ch,v) ^(
    ch.push(v)
    ~map (ev) {
        Either ~<< (v:: ev) {
            log("Value put: " + v);
        };
    }
);

defn takeVal(ch) ^(
    ch.take()
    ~map (ev) {
        Either ~<< (v:: ev) {
            log("Value taken: " + v);
        };
    }
);
```

**Note:** Notice the `Either ~<< {    }` *do comprehensions* again neatly and conditionally handle the `Either` instance, either invoking `log()` or short-circuiting out of the block with a `Left`.

Now, let's create a channel and use those helpers:

```java
def ch: Channel@;

putVal(ch,42);
// Promise{..pending..}

takeVal(ch);
// Promise{}
// Value put: 42
// Value taken: 42
```

As you can see, the initial `put()` is pending until the corresponding `take()` occurs.

Multiple `put()` operations can be queued up, and each subsequent `take()` will pull the next value in order off the "queue".

```java
def ch: Channel@;

putVal(ch,42);
// Promise{..pending..}

putVal(ch,100);
// Promise{..pending..}

takeVal(ch);
// Promise{}
// Value put: 42
// Value taken: 42

takeVal(ch);
// Promise{}
// Value put: 100
// Value taken: 100
```

**Note:** Don't think of queued `put()` operations as a *buffer*. Regardless of implementation details, the best way to think of these queued `put()`s, are as values that are waiting to go into the channel, rather than values that are already in the channel. That's why we don't see the `"Value put: 100"` confirmation above until the `100` is being taken from the channel. In other words, channel is creating an implicit *dependent relationship* between each attempted `put()` and its corresponding `take()`. Channel buffering is a separate concept we'll cover in a bit.

We can also attempt a `take()` before a corresponding `put()`:

```java
def ch: Channel@;

takeVal(ch);
// Promise{..pending..}

putVal(ch,42);
// Promise{}
// Value taken: 42
// Value put: 42
```

And of course, multiple `take()` operations can be queued, satisfied sequentially by subsequent `put()` operations:

```java
def ch: Channel@;

takeVal(ch);
// Promise{..pending..}

takeVal(ch);
// Promise{..pending..}

putVal(ch,42);
// Promise{}
// Value taken: 42
// Value put: 42

putVal(ch,100);
// Promise{}
// Value taken: 100
// Value put: 100
```

**Note:** Again, don't think of multiple pending `take()`s as *buffered*, but rather as externally pending operations, until satisfied by corresponding `put()`s.

----

Recall the `~<*` async-comprehension discussed in the `Promise` section. It's quite useful for explicitly coordinating a queue of `put()`s and/or `take()`s:

```java
def ch: Channel@;

1..3 ~<* (v) {
    ::putVal(ch,v);
};
// Promise{..pending..}

// elsewhere:
Promise ~<* {
    ::takeVal(ch);
};
// Value put: 1
// Value taken: 1
// Value put: 2
// Value taken: 2
// Value put: 3
// Value taken: 3
```

The `putVal()` loop above terminates once the `1..3` *range* is eventually completed. However, the `Promise ~<* {    }` loop will remain in a paused state, waiting for the next value to come into the channel.

You may recall that `~<*` terminates only when the **final expression** resolves to a `Left`. The only way a `Channel` instance's `take()` can resolve to a `Left` is when the channel itself is closed.

So, if we issued `ch.close()` in the above program:

```java
ch.close();
// Right{true}
// Error: Channel already closed
```

The `close()` above returns `Right{true}` indicating it was successful in closing the channel.

**Note:** Subsequent `close()` calls on an already closed channel will return `Left{"Channel already closed"}`.

After closing, the pending `take()` (from inside `takeVal()`) is resolved with a `Left{"Channel already closed"}`; that short-circuits out of the `Either` *do comprehension*..

Finally, the paused `~<*` loop will terminate, when encountering that `Left{"Channel already closed"}` value.

----

As illustrated so far, channels by default have no internal buffering.

But you can override this when you construct a channel; if you set a positive integer for a buffer size, that many values may be accepted into the channel (immediately resolving the `put()`s) even before the corresponding `take()`s are received.

Buffering a channel is useful for allowing the producer side of a channel to be less back-pressure bounded to the consumer.

**Warning:** A very high value of buffer size effectively eliminates back pressure from the channel. However, you should reconsider doing so; this coordination between producer and consumer is the main point and spirit of CSP. If you really need unbounded producer-side messaging without such coordination, a `PushStream` (covered later) is a better fit.

Consider:

```java
def ch: Channel@ 3;

1..3 ~each (v) { putVal(ch,v); };
// Value put: 1
// Value put: 2
// Value put: 3

putVal(ch,4);
// Promise{..pending..}

Promise ~<* {
    ::takeVal(ch);
};
// Value put: 4
// Value taken: 1
// Value taken: 2
// Value taken: 3
// Value taken: 4

ch.close();
// Right{true}
// Error: Channel already closed
```

Any `put()` that occurs while a buffer has capacity produces an immediately resolved promise, as shown by the messages printed during the `~each` loop.

But as shown by `putVal(ch,4)` above, any `put()` that occurs while a buffer is at capacity (and no `take()`s are pending) produces a pending promise. In other words, it behaves exactly the same as on a channel with buffer size of `0` (as default).

If a `put()` is pending when a `take()` frees up capacity in the buffer, that pending `put()` is immediately resolved and its value is treated as being placed into the internal buffer. That's illustrated by the timing of the `"Value put: 4"` message above.

----

Aside from `take()`, it can also be useful to `peek()` at a channel, and *see* a value that's `put()` into it, even if there's no `take()` yet to actually retrieve it.

A `peek()` operation also produces a pending promise (until any attempted `put()`), but all *peeks* will see the same value once the next `put()` is queued and until that value is `take()`n out of the channel. In other words, `peek()` operations don't *queue* (stack on each other sequentially) the way `puts()` and `takes()` do.

Consider:

```java
defn peekAt(ch,idx) ^(
    ch.peek()
    ~map (v) {
        log("(" + idx + ") Peeking at value: " + v);
        v;
    }
);

def ch: Channel@;

1..3 ~each (idx) {
    peekAt(idx,ch);
};

putVal(ch,42);
// Promise{..pending..}
// (1) Peeking at value: 42
// (2) Peeking at value: 42
// (3) Peeking at value: 42

peekAt(ch,4);
// Promise{42}
// (4) Peeking at value: 42

takeVal(ch);
// Promise{}
// Value put: 42
// Value taken: 42

peekAt(ch,5);
// Promise{..pending..}
```

The spirit of CSP channels is that the primary *read* operation should be `take()`, not `peek()`.

But peeking is especially useful when coordinating (without side effects) `take()`s from multiple channels. For example: a merge operation (i.e., first-come-first-served race), or a zip operation (i.e., wait for all channels to have a value before taking them).

The merge/race operation can be performed with the `alts()` utility:

```java
def nextVal: alts(< ch1, ch2, ch3, ch4 >);
// Promise{..pending..}
```

The first channel (left to right) that has a value available (via a `put()`) will resolve the pending promise, with a record that has a `value` property with the value, and a `channel` property holding the channel the value came from.

Races like this are often used for "timeouts", where you wait for a value from a channel for only a certain amount of time, and if the timeout expires, a value comes in on a second channel, thereby short-circuiting the waiting for a value from the first channel.

**Warning:** This ordered behavior means that if `ch1` always has values available, it can *starve out* any attempt for `ch2`, `ch3`, and `ch4` channels to transmit their values. To guard against such starvation, programs should be careful to perform some shuffling of the list order provided to `alts()`, perhaps just a simple round-robin strategy.

The zip/all operation can be performed with `every()`:

```java
def nextVal: every(< ch1, ch2, ch3, ch4 >);
// Promise{..pending..}
```

The promise from `every()` will resolve once a value is available from all channels, and will contain a list of these values in the same order as provided to `every()`, regardless of what order the arrive in.

### `PushStream` Monad

If you need a producer side to be unrestrained from the consumer side, the data transmission mechanism best suited is the `PushStream` monad.

The most trivial example is a single-value stream, via the unit constructor:

```java
def s: PushStream@ 21;

s ~< (v) {
    PushStream@ (v * 2);
}
~map (v) {
    log("Value: " + v);
};
// Value: 42
```

**Note:** A single-value stream (as above) seems similar conceptually to an already resolved Promise. One specific difference is that streams don't hold onto their values like promises do. So once a value is *observed* (another stream "subscribing" to it via `~<` or `~map`), that value is no longer in the original stream. Moreover, streams are either *open* or *closed* (`close()` and `closed()`), whereas promises are either *pending* or *resolved* (`resolve()` and `resolved()`).

Like promises, you typically construct a `PushSubject` to control the stream:

```java
def subj: PushSubject@;

def s2:
    subj.stream ~< (v) {
        PushStream@ (v * 2);
    }
    ~map (v) {
        log("(1) Value: " + v);
        v + 1;
    };
// PushStream{}

1..3 ~each subj.push;
// (1) Value: 1
// (1) Value: 2
// (1) Value: 3

s2 ~map (v) {
    log("(2) Value: " + v);
};
// PushStream{}
// (2) Value: 2
// (2) Value: 3
// (2) Value: 4
```

As shown, the `~<` / `~map` comprehensions remain active on an open stream, and are repeated for each new value pushed through the stream.

`push()` returns a `Right` with the pushed value, if successful; `Left` is returned if pushing failed (i.e., the stream has been closed).

A stream remains open and active until `close()` is called on it, or is called upstream; close signals actively propagate downstream, since no more values ever propagate through closed streams.

Consider:

```java
def subj: PushStream@;

1..3 ~each subj.push;

def another: subj.stream ~map (v) {
    log("Value: " + v);
};
// Value: 1
// Value: 2
// Value: 3

subj.push(4);
// Right{true}
// Value: 4

subj.stream.closed();
// true

another.closed();
// true

// ------------

subj.stream.close();
// Right{true}

subj.stream.closed();
// true

another.closed();
// true

subj.push(10);
// Left{"Stream already closed"}
```

**Note:** Closing a stream not only shuts down any further value propagations, but also cleans up memory used by the stream (and any downstream instances). As such, streams should always be closed when no longer needed.

----

`PushStream` supports the `~<<` *do comprehension* form:

```java
def subj: PushStream@;

1..3 ~each subj.push;

def s: PushStream ~<< (v:: subj.stream) {
    log("(1) Value: " + v);
    v * 2;
};
// PushStream{}
// (1) Value: 1
// (1) Value: 2
// (1) Value: 3

subj.push(4);
// Right{true}
// (1) Value: 4

s ~map (v) {
    log("(2) Value: " + v);
};
// PushStream{}
// (2) Value: 2
// (2) Value: 4
// (2) Value: 6
// (2) Value: 8

5..6 ~each subj.push;
// (1) Value: 5
// (2) Value: 10
// (1) Value: 6
// (2) Value: 12
```

As you can see, just like `~<` / `~map`, this form automatically *repeats* -- not really looping! -- each time a new value is pushed into the stream.

**Note:** As stream inherently repeats operations for each value, there is no corresponding `~<*` *looping do comprehension* form for streams; that's only for `Promise`.

### `PullStream` Monad

Where a `PushStream` is controlled by the producer, a `PullStream` is more like a CSP `Channel` where control is shared by producer and consumer.

However, unlike `Channel`, a `PullStream` offers no *back pressure*, in that it does not block the producer *pushing* on the consumer *pulling* the value. That is, a `push()` does not wait for a corresponding `pullInto()`.

We can again create a single-value stream instance, via the unit constructor:

```java
def s: PullStream@ 21;

s ~< (v) {
    PullStream@ (v * 2);
}
~map (v) {
    log("Value: " + v);
};
// PullStream{}
```

As shown, the `21` (and subsequent `42`) values don't actively propagate through the `PullStream` instances, which is why nothing is logged.

To trigger the propagation, use `pullInto()`:

```java
def s: PullStream@ 21;

def t:
    s ~< (v) {
        PullStream@ (v * 2);
    }
    ~map (v) {
        log("Value: " + v);
    };
// PullStream{}

t.pullInto(1);
// Right{1}
// Value: 42
```

The `pullInto()` function takes an optional integer argument for the count of how many values to attempt to pull down the stream. If the attempt is successful (as above), the result will be a `Right` holding the same numeric value.

Otherwise, if insufficient pending values are found anywhere upstream, the return will be a `Left` holding the count of values *not* able to be pulled. Those pending requests remain active upstream, and will be fulfilled as any new values are pushed in.

You will typically construct a `PullSubject` to control the stream:

```java
def subj: PullSubject@;

def s2: subj.stream
    ~map (v) {
        log("(1) Value: " + v);
        v * 2;
    };
// PullStream{}

1..3 ~each subj.push;
// (nothing)

s2 ~map (v) {
    log("(2) Value: " + v);
};
// PullStream{}

s2.pullInto(2);
// Right{2}
// (1) Value: 1
// (2) Value: 2
// (1) Value: 2
// (2) Value: 4

s2.pullInto(3);
// Left{1}
// (1) Value: 3
// (2) Value: 6
// (1) Value: 4
// (2) Value: 8

5..6 ~each subj.push;
// (1) Value: 5
// (2) Value: 10

s2.pullInto(1);
// Right{1}
// (1) Value: 6
// (2) Value: 12
```

----

`PullStream` also supports the `~<<` *do comprehension* form:

```java
def subj: PushStream@;

1..3 ~each subj.push;

PullStream ~<< (v:: subj.stream) {
    log("Value: " + v);
};
// PullStream{}

s.pullInto(3);
// Right{3}
// Value: 1
// Value: 2
// Value: 3
```

**Note:** Since the `~<<` block automatically repeats as each value is pulled through, there's no need for a separate `~<*` *looping do comprehension* form for `PullStream`.

## `IO` Monad

The first and most important *rule* of FP is that you have to be very careful to minimize and control side-effects whenever and wherever possible. Mismanaged side effects are the single greatest source of bug infection in our code.

One powerful tool for managing side effects in a mathematical, predictable way is the `IO` monad.

In **Foi**, the `IO` monad is special, in that it composes behaviors of multiple monad types together: IO, Task, and Reader.

Let's explore each of these capabilities separately.

### Task

Task is a pattern for lazily defining a set of actions, usually performing side-effects outside the program -- printing to the console, performing network requests, reading or writing to a file system, waiting on external async operations, generating random numbers, etc.

The key is, `IO` instances are lazy; these actions **do not run automatically**. Moreover, multiple `IO` monads are chained together to compose separate units of action into a single lazy action.

To construct an `IO` instance, we give it an *executor* function that will perform the action(s) (side effects):

```java
def task = IO@ (defn someTask(){
    log("Log messages are a side effect!");
});
// (nothing)
```

Notice that the log message didn't actually happen. `IO` instances are lazy. An `IO` instance (which may be a composed chain of many `IO`s) is run on-demand (one or more times), using the `run()` method on the instance:

```java
def task = IO@ (defn someTask(){
    log("Log messages are a side effect!");
});

task.run();
// Log messages are a side effect!
```

When you simply want to hold a value in an `IO` instance, instead of providing a function that only returns the value, we can use a special unit constructor as a shortcut:

```java
def specialNumber = IOof@ 42;

specialNumber.run();   // 42
```

As with all monads, we can compose instances together via comprehensions like `~map` and `~<` (chain):

```java
defn doubleIO(v) ^IO@ (v * 2);
defn incIO(v) ^ IO@ (v + 1);
defn finish(v) {
    log("v: " + v);
    ^incIO(v);
};

def num = IOof@ 21;

def task = num
    ~map doubleIO
    ~< finish;

task.run();   // 43
// v: 42
```

----

Recall the [`~<<` do-comprehension](#monadic-do-comprehension) (for monads), which gives a special syntax for chaining monadic values together a more familiar imperative-style. It's especially convenient when you might otherwise need to nest `~<` chain steps to create a shared scope for accessing values from each step together.

Because this is so common with `IO`, the *do comprehension* form is most common. The previous snippet could be done like this:

```java
(IO ~<< (v:: num) {
    def x:: doubleIO(v);
    ::finish(x);
}).run();   // 43
// v: 42
```

As you can see, the `v:: num` statement unwraps the `IO` instance `num`, and assigns its value to `v`. Likewise, the `def x:: doubleIO(v)` unwraps the `IO` instance that comes back from the function call, and assigns the result to `x`.

Finally, the `IO` instance from `finish(x)` is returned (without wrapping, due to the `::` prefix).

You can interleave `def ::` and `def :` style definitions:

```java
defn readFile(filename) {
    // ..
    ^IO@ fileContents;
};
defn writeFile(filename,contents) {
    // ..
    ^IO@ res;
};
defn processFile(filename) ^IO ~<< {
    def text:: readFile(filename);
    def uptext: uppercase(text);
    def res:: writeFile("upper.txt",uptext);
    < :res, :text >;
};

processFile("my-file.txt")
.run();
// < res: .., text: ... >
```

### Reader

Reader is a pattern for carrying a value across monadic operations, without needing a shared outer scope to access it. We typically treat the Reader value as an *environment* context that is parameterized for the IO to run against. This enables a *pure* IO that doesn't need to access anything from its outer context.

`IO` implements Reader by allowing a single argument (optional) to the `run()` function. If a Reader value is provided, it's automatically passed as the first argument to the *executor* function:

```java
def task: IO@ (defn(readerEnv){
    log("X: " + readerEnv.x);
});

task.run(< x: 42 >);
// X: 42
```

**Note:** The Reader value can be anything, but it's most commonly a Record (or Tuple).

Inside a `~<` chain step, the carried Reader value can be *accessed* as so:

```java
def task:
    IOof@ 42
    ~< (v) {
        IO@ (defn(env){
            log("Value: " + v, "Env.x: " + env.x);
        });
    };

task.run(< x: 3 >);
// Value: 42, Env.x: 3
```

This is a bit ugly/awkward, but is cleaner in *do comprehension* form to *extract* the Reader value:

```java
defn getEnv() ^IO @(defn(env) ^env);

def fortyTwo: IOof@ 42;

def task: IO ~<< {
    def v:: fortyTwo;
    def env:: getEnv();
    log("Value: " + v, "Env.x: " + env.x);
};

task.run(< x: 3 >);
// Value: 42, Env.x: 3
```

In fact, this is even cleaner:

```java
def fortyTwo: IOof@ 42;

def task: IO ~<< (env, v:: fortyTwo) {
    log("Value: " + v, "Env.x: " + env.x);
};

task.run(< x: 3 >);
// Value: 42, Env.x: 3
```

As shown, the Reader value is automatically provided to the *do comprehension* block, as exposed by a block definitions clause.

### Transforming Over Concurrency

The final super power of `IO` is that it automatically acts as a transformer over each of the previous concurrency/asynchrony mechanisms: `Promise`, `Channel`, `PushStream`, and `PullStream`. That means that if an `IO` operation encounters an instance of any of these, it will automatically *lift* to that space, thereby acting to *unwrap* such values.

Consider:

```java
defn getValue() ^Promise@ 42;
defn printValue(v) ^IO@ (defn(){
    log("Value: " + v);
});

def task: IO ~<< {
    def v:: getValue();
    ::printValue(v);
};

task.run();
// Promise{}
// Value: 42
```

As illustrated, when the promise instance from `getValue()` was encountered, the rest of the IO evaluation -- in other words, what's returned from the `run()` call -- was *lifted* to a promise. Subsequent steps in the IO chain wait for a previous promise to resolve before proceeding.

That's basically the `Promise ~<< {    }` behavior combined automatically into `IO`'s `~<<` *do comprehension*.

If an `IO` instance holds a `Promise` instance, that's unwrapped automatically:

```java
defn readValue() ^IOof@ (Promise@ 42);
defn printValue(v) ^IO@ (defn(){
    log("Value: " + v);
});

def task: IO ~<< {
    def v:: readValue();
    ::printValue(v);
};

task.run();
// Promise{}
// Value: 42
```

Even in the inverse scenario -- where a `Promise` instance holds an `IO` instance -- the transformation still occurs:

```java
defn readValue() ^Promise@ (IOof@ 42);
defn printValue(v) ^IO@ (defn(){
    log("Value: " + v);
});

def task: IO ~<< {
    def v:: readValue();
    ::printValue(v);
};

task.run();
// Promise{}
// Value: 42
```

----

Here's `IO` transforming over a `Channel`:

```java
defn getValue() ^Channel@ 42;
defn printValue(v) ^IO@ (defn(){
    log("Value: " + v);
});

def task: IO ~<< {
    def ev:: getValue();
    ::(| ~cata ev, IOof@, printValue |);
};

task.run();
// Promise{}
// Value: 42
```

The `def ev:: getValue()` unwrapping automatically calls `take()` on the channel, which produces a `Promise` instance; that *lifts* the `IO` evaluation into the promise space.

----

Transformation is not limited to promises. It also applies to streams. For example, here's an `IO` *do comprehension* transforming over a `PushStream`:

```java
defn getValues() {
    def subj: PushSubject@;
    1..3 ~each subj.push;
    ^subj.stream;
};
defn printValue(v) ^IO@ (defn(){
    log("Value: " + v);
});

def task: IO ~<< {
    def v:: getValues();
    ::printValue(v);
};

task.run();
// PushStream{}
// Value: 1
// Value: 2
// Value: 3
```

Notice how `run()` this time returned a `PushStream` instance instead of a `Promise`, since the `IO` was *lifted* into that space for its evaluation.

Here's the same, but with a `PullStream`:

```java
defn getValues() {
    def subj: PullSubject@;
    1..3 ~each subj.push;
    ^subj.stream;
};
defn printValue(v) ^IO@ (defn(){
    log("Value: " + v);
});

def task: IO ~<< {
    def v:: getValues();
    ::printValue(v);
};

def s: task.run();
// PullStream{}

s.pullInto(3);
// Right{3}
// Value: 1
// Value: 2
// Value: 3
```

Since we're using a `PullStream`, we have to call `pullInto(3)` to actually pull those values into the `IO` evaluation.

**Note:** With `IO`, typically `PushStream` will be preferred over `PullStream`, given this extra step of needing to call the `pullInto()` on the returned stream.

----

Be aware that the order of transformation in an IO `~<<` *do comprehension* matters, with respect to what kinds of *lifting* is supported:

* A concrete/synchronous IO can be lifted to a promise IO evaluation (as shown earlier). This includes a channel transformation, since the `take()` produces a promise.

* A stream IO evaluation can lift concrete/synchronous values, or promises.

* A `PullStream` can be lifted to a `PushStream`, or vice versa.

But importantly, a stream cannot be lifted to a promise evaluation (only vice versa). Perhaps counterintuitively, that means the following is invalid:

```java
def getValue() ^Promise@ 42;
def getValues() ^PushStream@ 10;

IO ~<< {
    def v:: getValue();
    def x:: getValues();
    log("v: " + v + ", x: " + x);
}.run();
// (error)
```

The `getValue()` lifts the `IO` evaluation into a promise, but then a subsequent `PushStream` instance from `getValues()` cannot be lifted into this promise evaluation.

But the reverse order *is* supported, so this is valid:

```java
def getValue() ^Promise@ 42;
def getValues() ^PushStream@ 10;

IO ~<< {
    def x:: getValues();
    def v:: getValue();
    log("v: " + v + ", x: " + x);
}.run();
// v: 42, x: 10
```

In other words, we can resolve a singular-value promise from `getValue()` for each time we get a value from the stream returned by `getValues()`. But we cannot do the reverse: pushing each value from the `getValues()` stream into a single promise from `getValue()`.

It's not common that you'll want to mix/compose promises and streams in the same `IO` *do comprehension* -- usually just one or the other -- so this kind of confusing nuance won't be encountered very often.

## Generator Monad

Generators are a specialization of `IO`, which behaves somewhat like a CSP `Channel`.

Generators are useful for expressing "lazy" computations, meaning that they step through producing a single value at a time, and only compute the next value after the previous value has been requested.

The `Gen@` unit constructor expects a single argument: a function that will receive the Reader value (`env` below), and a function (`yield` below) to produce values from the generator. Calling `yield()` queues up a value to send through the iterator, and produces a `Promise` instance that will resolve once the value has been taken from the iterator. If successful, the resolved value with be a `Right`, and `Left` otherwise.

```java
def genIO: Gen@ (defn(env,yield){
    def pr: yield(42);
    // ..
});
```

Calling the unit constructor produces an `IO` instance (`genIO` above); when `run()` is invoked on the `IO`, the generator function starts running, and an iterator is returned to retrieve the generator instance's value(s).

The iterator holds two functions: `next()` for retrieving values, and `close()` for closing the iterator (and thus stopping the `IO` instance of the generator).

The most classic example of a generator is computing the Fibonacci sequence:

```java
def fib: Gen@ (defn(env,yield){
    def a: 0;
    def b: 1;
    ^Promise ~<* {
        def cur: a;
        def res:: yield(cur);
        a := b;
        b := cur + a;
    };
});

def it: fib.run();


// print the first 10 Fibonacci numbers
0..9 ~<* {
    def ev:: it.next();
    (| ~cata ev, Left@, log |);
};
// 0
// 1
// 1
// 2
// 3
// 5
// 8
// 13
// 21
// 34
```

**Warning:** The above generator is designed to run perpetually (doesn't stop itself), and without any delay, so be careful about using an unbounded looping to consume values from it; such a loop will also run synchronously, forever. The `0..9 ~<* {    }` approach above limits how many values to *take* from the iterator.

Even though the iterator interface (`yield()` and `next()`, above) responds with Promises, this Fibonacci generator is fully synchronous; remember that **Foi** `Promise` instances are not inherently asynchronous (as in some other languages).

Here's another example, of a generator that only produces a fixed number of values through its iterator (and then closes it):

```java
def someNums: Gen@ (defn(env,yield){
    ^((env.start..env.end) ~<* yield)
        ~map { "Complete." };
});

def it: someNums.run(< start: 4, end: 7 >);


// consume all the values from this
// iterator
def pr: it ~<* (ev) {
    (| ~cata ev, Left@, log |);
};
// Promise{..pending..}
// 4
// 5
// 6
// 7

pr ~map log;
// Left{"Complete."}
```

**Note:** The `~<*` knows how to consume an iterator. It calls `next()` under the covers, which produces a promise. When unwrapped, this value is a `Right` or `Left`; if it's a `Left`, the looping will terminate. Above, we unwrap the `Right` with `~cata`, but in the next snippet, we'll use an `Either` *do comprehension* to process `ev`.

You can also manually terminate an iteration early by closing the iterator:

```java
def someNums: Gen@ (defn(env,yield){
    ^((env.start..env.end) ~<* yield)
        ~map { "Complete." };
});

def it: someNums.run(< start: 1, end: 10 >);


// consume all the values from this
// iterator
def pr: it ~<* (ev) {
    Either ~<< (v:: ev) {
        log(v);

        // shall we terminate early?
        ::?[v ?= 3]: {
            it.close();
            Left@ "Terminated.";
        };
    };
};
// Promise{..pending..}
// 1
// 2
// 3

pr ~map log;
// Left{"Terminated."}
```

**Note:** The `Either ~<< {    }` *do comprehension* block is nested inside the outer `Promise ~<* {    }` *looping do comprehension* block. This allows us to ergonomically unwrap the Either value (`ev`) that came back from `next()`. If the `v:: ev` unwrapping encounters a `Left`, it short-circuits to skip the `Either` comprehension block.

The manually produced `Left@ "Terminated."` value forcibly terminates first the inner `Either ~<< {    }` block, and then the outer `Promise ~<* {    }` loop.

However, if that value were omitted (but the iterator was still closed), the loop would normally start a next (final) iteration, yet the `next()` call would immediately fail with a `Left@ "Complete."` -- from the final `"Complete."` return value of the generator -- and that would terminate the loop.

## Type Annotations

Type annotations in **Foi** are applied to values/expressions (not to variables, etc). These are optional, as **Foi** uses type inference wherever possible. But applying them can often improve the performance optimizations the **Foi** compiler can produce.

A type annotation always begins with the `:as` keyword:

```java
def age: 42 :as int;

def cost: | * getQty(order,item), getPrice(item) | :as float;
```

Custom types can be defined, for use in subsequent annotations, with the `deft` keyword:

```java
deft OrderStatus { empty, "pending", "shipped" }

def myStatus: getOrderStatus(order) :as OrderStatus;
```

Function signatures may optionally be typed via custom types:

```java
deft InterestingFunc (int,string) -> empty;

defn whatever(id,name) :as InterestingFunc {
    // ..
}
```

The `?as` operator corresponds to the `:as` type annotation keyword; it's a boolean operator that returns `true` if a value/expression matches the indicated type, `false` otherwise:

```java
def age: 42;

age ?as int;                // true

(age :as bool) ?as int;     // false
```

This operator is useful in pattern matching:

```java
deft SimpleFunc (int) -> empty;
deft InterestingFunc (int,string) -> empty;

def result1: ?(myFn){
    ?[?as SimpleFunc]: myFn(42)
    ?[?as InterestingFunction]: myFn(42,"Kyle")
    ?: myFn()
};

// or:

def result2: ?{
    ?[myFn ?as SimpleFunc]: myFn(42)
    ?[myFn ?as InterestingFunction]: myFn(42,"Kyle")
    ?: myFn()
};
```

## License

[![License](https://img.shields.io/badge/license-MIT-a1356a)](LICENSE.txt)

All code and documentation are (c) 2022 Kyle Simpson and released under the [MIT License](http://getify.mit-license.org/). A copy of the MIT License [is also included](LICENSE.txt).
