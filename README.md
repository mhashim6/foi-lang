# Foi: a different kind of functional programming language

**Foi** is a programming language that pragmatically balances Functional Programming (FP) and imperative programming techniques. It pulls inspiration for various syntax and behaviors from a variety of languages, including: JS, Scala, Haskell, F#, Go, Clojure, CommonLisp, and others.

The language is designed for general application programming purposes, but with a clear emphasis on FP (and de-emphasis on OOP). It's not trying to compete in performance or capability with systems languages like C or Rust. Eventually, **Foi** will compile to WASM so it should be usable for applications in a variety of environments, from the web to the server to mobile devices.

An important design goal is that a somewhat experienced developer -- especially one with both FP and imperative experience/curiosity -- should be able to sufficiently/fully learn **Foi** in a few days. It also shouldn't take thousands of pages of books or months of workshop videos to fully detail the surface area of **Foi**.

## Mission

**Foi** promotes coherence and safety through declarative Functional Programming (FP) patterns as first class language features, while bridging (with familiar idioms) to those more experienced in traditionally imperative programming languages.

It should feel intuitive when doing the *right*&trade; things, appear obvious when doing the *risky* things, and discourage doing the *dangerous* things.

**Foi** is a language you write for other humans to read first. It's only a secondary benefit that the computer can understand the code and execute the desired operations.

## Design Summary

**Foi** aims to be a novel mix of a variety of syntactic styles and ideas from various languages. One primary goal is for **Foi** features to have internal consistency with each other, built on self-evident (as much as possible!) semantics and mental models. As such, there will be parts of **Foi** that look familiar and other parts that feel quite unfamiliar.

Ultimately, the hope is these design decisions diverge enough from familiar languages to take useful steps forward, but not too much that **Foi** is unuseful or impractical.

A programming language design is a delicate balance, and inevitably will be judged both on aesthetics and on empirical outcomes. It's impossible to design a perfect language that everyone loves at first glance.

That said, here are a few explanations to help bring some clarity to **Foi**'s particular design decisions:

* **Words vs Symbols** I don't think a language should be all symbols. I struggle to memorize arbitrary symbol combinations just as much as anyone. But I similarly feel overburdened when a language is full of long lists of reserved keywords.

    I dislike how these reserved keywords can visually appear indistinct from our variables/identifiers (save for syntax highlighting). I also dislike how keywords can conflict with very useful variable names in certain contexts.

    **Foi** has a pretty short list of bare keywords, most of which are actually part of the type annotation system (`int`, `bool`, etc). **Foi** has a variety of purely symbolic operators, like `+` (addition, concatenation) and `+>` (compose-left).

    But in compromise in between, there are several "named operators", that combine both a symbol and a word. In such cases, the symbol is always the first character, and helps visually distinguish from any other identifier (either built-in or user-defined).

    Another example: `^` replaces the `return` keyword. This is designed to be a single "return" signifier that stands out (it's vertically top-aligned), has a semantic signal (sending a value "up and out" from a function), but is short enough to not burden even the most concise of inline function expressions.

* **Consistency** Building on the last point, all boolean operators begin with the `?` symbol (or `!` for the negated form), so they should be easily recognizable as such. This includes pure symbolic operators, like `?=` (equality) / `!=` (inequality) or `?>` (greater-than), as well as named operators, like `?and` (boolean-AND) and `!or` (boolean-NOR) and `?in` (included-in).

    The `?` and `!` symbols can also serve as unary prefix operators on any identifier (or value), to cast the value to a boolean (and negate it, for `!`). Again, these symbols always indicate boolean purposes. Consistency.

    Lastly, all decision making in **Foi** uses `?` and `!`. For example, there's pattern-matching `?{ .. }` / `?(..){ .. }` -- a more powerful combination form of `if`..`else if` and `switch`. There's also guard clauses `?[ .. ]:` -- like `if` statements in front of statements. Finally, loop conditionals use the same form as guard clauses.

    Again, all these decision-making features consistently use the `?` / `!` signifiers.

    Loops/comprehensions all use `~` as the first symbol, from the `~each` / `~map` named-comprehensions to the `~<` chain/bind/flatMap operator.

    Lexical definitions come in three forms: variables, functions, and types. Accordingly, there's three corresponding keywords: `def` (variables), `defn` (functions), and `deft` (types).

    These kinds of choices help visual distinction (from other identifiers/syntax) but also conceptually group related features/capabilities together, and hint/signal semantics.

* **Visual Semantics** Many of the operators use symbol(s) with intended visual semantics.

    For example, `+>` (compose left-to-right operator) has two semantic signals in it. First, the `+` signifies a concatenation of functions (aka "composition"). Secondly, the `>` symbol is pointing in the direction of data-flow through the functions. By contrast, `<+` is the compose-right (right-to-left), meaning the data flows in the opposite direction.

    As mentioned previously, `^` points upward, to semantically hint "return value up/out".

* **Infix + Lisp** Pretty much all programming languages either use infix form or lisp form.

    The infix form (`x + 3`, `myFn(1,2,3)`) is extremely common and familiar. The lisp form (`(+ x 3)`, `(myFn 1 2 3)`) is much more powerful/flexible. But why do we have to choose? I believe we should be able to combine these capabilities into a single language.

    In **Foi**, an optional lisp-like form, denoted with `| .. |` (pipes) instead of `( .. )` (parentheses) lets you opt into lisp expression semantics, treating all operators and functions alike.

    In infix form, a single operator symbol can only ever have at most two operands (left and right). Ternary `? :` is a special trick that has 3 operands, but has to split up two symbols.

    Lisp lets us express n-ary operators with 3 or more operands. For example, `| +> fn1, fn2, fn3, fn4 |` is cleaner than repeating the operator `fn1 +> fn2 +> fn3 +> fn4`.

    In some usages, a `| .. |` expression can benefit readability-wise from optional parentheses: `(| .. |)`.

* **Immutable Objects** In **Foi**, Records / Tuples (both use `< .. >` syntax) are immutable.

    That means we don't overload `{ .. }` to mean both blocks and object literals -- a perpetual confusion in languages like JS -- nor `[ .. ]` to mean both property accesses and array literals -- yet more confusion spawning.

    We also get rid of any syntax/capabilities around mutation of these values. But in place, we need ergonomic facilities for deriving new immutable Records/Tuples from existing ones.

    For example, `< &one, &two, 10, 20 >` is a Tuple that *includes* the contents of `one` and `two` Tuples, along with the values `10` and `20`. The `&` operator (known as "pick") might evoke "pointer" / "reference" semantics from other languages. That's intentional here, as the mental model should be that those `one` / `two` immutable values are being *linked* to the new value rather than being copied (as `...` does in JS). Similar for Records.

    The `&` capability also allows selective picking/linking, like `< &two.3 >` `< &one.something >`.

    Computed property names in JS again overload the `[ .. ]` syntax, like `{ [someProp]: 42 }`. But in **Foi**, we simplify this with a single `%` symbol: `< %someProp: 42 >`. This is also how Records act as Maps (non-primitive keys): `< %otherObj: .. >`.

    **Foi** treats a "Set" as just a filtered construction (removing duplicates) of a Tuple.

    All these capabilities thus work identically for Records, Maps, Tuples, and Sets. That means you really just need to learn one data structure form: `< .. >`.

* **Types** In the **Foi** language, you don't annotate "container types" -- types on variables, properties, etc. Instead, you annotate "value types" -- types on values, and on expressions.

    I believe we'll get most of the benefits of "typing" with much less syntactic and mental overhead compared to traditional language static types.

This is not an exhaustive list of design persuasions, but it should help set the right perspective for evaluating/analyzing **Foi** code.

### Original Design Motivations

Click to expand the following list of ideals that initially inspired the design for **Foi**.

<details>
    <summary>Inspirational/Aspirational Design Ideas for Foi</summary>
    <p></p>
    <ul>
        <li>versioned, with no backwards-compat guarantee</li>
        <li>parsed, JIT'd, compiled (probably WASM), with full-fidelity AST (preserves everything including whitespace, comments, etc)</li>
        <li>semicolons and braces (both required) -- no ASI, no implicit blocks</li>
        <li>lexical and block scoped, functions are first-class and have closure</li>
        <li>side effects (non-local reassignments) must be explicitly declared in function signature</li>
        <li>no global scope (everything is in a module scope)</li>
        <li>no circular dependencies, only synchronous module initialization</li>
        <li>no references or pointers</li>
        <li>no class, nor prototype, nor `this`-awareness</li>
        <li>functional (tail-call optimized, pattern matching, curried function definitions, composition syntax, native monads, no exception handling, etc)</li>
        <li>no `const`</li>
        <li>no `let` -- block-scoped declarations are explicit syntax as part of the block</li>
        <li>function auto-hoisting, but no variable hoisting</li>
        <li>only one empty value</li>
        <li>numeric types: int, float, bigint, bigfloat</li>
        <li>everything is an expression (no statements)</li>
        <li>iteration/looping (for/foreach/filter/map) are syntactic expressions, but accept functions</li>
        <li>all keywords and operators are functions (with optional lisp-like call syntax)</li>
        <li>records/tuples (instead of objects/arrays) that are immutable and by-value primitives</li>
        <li>syntax for mutable data collection (dynamically define props/indices, like objects or arrays), but in order to use/read/pass-around, must first be "frozen" into an immutable record or tuple -- somewhat like a heap-allocated typed-array that's then accessed by a "view" (a record or tuple)</li>
        <li>strings are sugar for tuples of characters, and are interoperable as such</li>
        <li>optional named-argument call syntax</li>
        <li>asynchrony built in (syntax for future values and reactivity/streams)</li>
        <li>garbage collected</li>
        <li>type awareness: weakly typed (small, limited set of type coercions), with dynamic type inferencing as well as optional type annotations on values/expressions</li>
    </ul>
    <p>
        In addition to the above, I may pull parts of a long-ago description of [earlier ideas for this language (then called "FoilScript")](https://github.com/getify/FoilScript#whats-in).
    </p>
</details>

## Exploring Foi

If you have experience or familiarity with JS, check out the [Foi vs JS Cheatsheet](Cheatsheet.md), as well as some [screenshots of Foi vs JS code snippets](Cheatsheet.md#comparison-examples).

Additionally, [Foi Guide](Foi-Guide.md) is a detailed exploration of the language.

For implementers or language design enthusiasts, a [formal grammar specification](Grammar.md) is in progress.

## Tools

**Foi** is still being designed. As such, there's no official compiler/interpreter yet.

However, [Foi-Toy](foi-toy/README.md) is an experimental tool for playing around with **Foi** code, prior to there being an official compiler.

Foi-Toy currently supports tokenizing **Foi** code, and syntax highlighting (via HTML/CSS) as shown in the above screenshots.

## License

[![License](https://img.shields.io/badge/license-MIT-a1356a)](LICENSE.txt)

All code and documentation are (c) 2022 Kyle Simpson and released under the [MIT License](http://getify.mit-license.org/). A copy of the MIT License [is also included](LICENSE.txt).
