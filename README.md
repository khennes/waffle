[In progress]

## A simplified C-style programming language, parser, and compiler
For my final project at [Hackbright Academy](http://www.hackbrightacademy.com), I decided to write a simple programming language called Waffle, along with a Pratt-style parser and a compiler (nicknamed Crumpet), both coded by hand, in order to gain a better fundamental understanding of how programming languages are implemented. After considering a few different targets for compilation (x86 and LLVM, among others), I ultimately settled on [asm.js](http://www.asmjs.org), a low-level subset of Javascript that is in the process of being formalized by a team at Mozilla.

### Language design
Waffle is a statically typed, C-style programming language. Variable types are explicitly declared; I borrowed Go’s type declaration syntax and C’s structures and statement blocks. Waffle supports functions, recursion, loops, conditional statements, inline and multiline comments, and the following data types: integers, floats, strings, arrays, structures, and booleans.   

Because I wanted to focus most heavily on the processes of parsing and code generation in the four weeks I had to complete the project, and because Waffle was intended as a personal learning exercise rather than  a usable scripting language, along the way I decided to make a few expedient choices when it came to defining the syntax. These include:
* Adding a "var" keyword to denote new variable declarations
* Using C-style statement block delimiters { and }
* Ignoring lexical block scope during interpretation (for now). As function scope is handled implicitly by stack frame allocation, and Javascript has native block scope, I wasn't especially concerned about this, but I'll be looking to add true lexical scoping as a next step. 

### Parsing
After using the [PLY](http://www.dabeaz.com/ply/ply.html) library to define my language's tokens, I opted to hand-code a parser rather than use a generator like Yacc to better understand what happens during the parsing process. I wrote an implementation of Pratt’s parsing algorithm based on [Frederic Lundh](http://effbot.org/zone/simple-top-down-parsing.htm)'s and [Douglas Crockford](http://javascript.crockford.com/tdop/tdop.html)'s Python and Javascript implementations, including Crockford’s extensions for statement parsing. 

First, `main()` calls the `start_lex(filename)` function, which reads a user-specified Waffle file and slices it into tokens based on my articulated grammar, outputting a `token_stream`. In `tokenize()`, tokens are assigned to their unique symbol classes, which are dynamically generated by the`symbol(ttype, bp=0)` method; added to the `token_stack`; and registered to the global `symbol_table` dictionary. At the same time, each symbol is given a relative binding power (`bp`), based on how strongly it “attracts” the tokens to the right of it.

Next, `main()` instantiates the `Program` parent class and calls its chain of parsing methods: `stmtd()`, which calls `parse_expression()` (or `parse_statement()`) until the entire token stack is consumed, then returns an abstract syntax tree (AST). In the meantime, the `advance()` helper function checks that the current token type is valid/expected before fetching the next one. 

Almost all symbols inherit a null (`nulld`), left (`leftd`), or statement (`stmtd`) denotation method from the parent class `BaseSymbol`, based on their position within an expression. “Prefix” tokens appear at the beginning of an expression and receive nulld; “infix” (regular binary operators like `+`, `-`, `%`, etc.), and “infix_r” (for right-associative tokens like `**` and the assignment operators `=`, `-=`, and `+=`) receive leftd. Finally, statements (`return`, `print`, `break`, etc.) receive stmtd. The token stack and current token are stored as global variables. (When I implement lexical scope, the current scope object will similarly inherit from a base scope class and be held in a global variable.)

I took the intermediate step of coding a Python interpreter to check my work along the way. To that end, `main()` next calls the program node’s `eval(env)` method, traverses the AST recursively, and returns the result of the program.
