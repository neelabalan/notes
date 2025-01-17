> to bytecode

> from - https://github.com/python/cpython/blob/main/InternalDocs/compiler.md

## Compiler design

In CPython, the compilation from source code to bytecode involves several steps:

- Tokenize the source code Parser/lexer/ and Parser/tokenizer/.
- Parse the stream of tokens into an Abstract Syntax Tree Parser/parser.c.
- Transform AST into an instruction sequence Python/compile.c.
- Construct a Control Flow Graph and apply optimizations to it Python/flowgraph.c.
- Emit bytecode based on the Control Flow Graph Python/assemble.c.