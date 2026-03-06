# ⚙️ RedLang: A Custom OOP Compiler (C# to LLVM IR)

> A complete, fully functional compiler for a custom statically-typed, object-oriented programming language ("RedLang"), engineered from scratch using **C#**, **ANTLR4**, and **LLVMSharp**.

*Note: This project was co-developed as an academic requirement for the Software Engineering program at INTEC. This repository serves as a portfolio showcase of my specific architectural and implementation contributions.*

## 🚀 Overview & Capabilities
RedLang is an imperative, class-based language compiled directly to native machine code via LLVM. It features a custom syntax (`.red` files) and supports:
* **Object-Oriented Design:** Classes, fields, methods, and user-defined types.
* **Static Typing:** Primitives (`i`, `f`, `b`, `s`), nullable types (`?`), and fixed-size arrays.
* **Control Flow & Modules:** Loops (`loop`, `repeat`), conditionals (`check`/`otherwise`), and cross-file imports (`use ClassName;`) with DFS circular dependency detection.

## 🛠️ Tech Stack
* **Language & Runtime:** C#, .NET 9.0
* **Frontend (Lexer/Parser):** ANTLR4 (`Antlr4.Runtime.Standard`)
* **Backend (Code Generation):** LLVMSharp, Clang Toolchain
* **Architecture:** Visitor Design Pattern, Multi-pass Semantic Pipeline

---

## 🧠 Architectural Deep Dive

The system follows a classic multi-phase pipeline, decoupling grammar definitions from business logic.

### 1. Lexical & Syntactic Analysis (ANTLR4)
The frontend relies on two split grammar files (`Lexer.g4` and `Parser.g4`) defining over 40 token types and 30 syntax rules. ANTLR4 generates the recursive-descent parser, producing a Concrete Syntax Tree (CST).

### 2. AST Transformation (Visitor Pattern)
To decouple the auto-generated ANTLR code from the compiler logic, the system strictly implements the **Visitor Design Pattern**. 
The `AstBuilderVisitor` class intercepts CST nodes and transforms them into a strongly-typed, flat-hierarchy Abstract Syntax Tree (AST) (e.g., `ProgramNode`, `BinaryExpressionNode`, `CallNode`). This ensures the semantic analyzer is completely unaware of ANTLR's internal `*Context` classes.

### 3. Semantic Analysis & Scope Management
The `SemanticAnalyzer` executes a rigorous multi-pass validation:
* **Hierarchical Symbol Tables:** Implements lexical scoping by chaining dictionaries (`Function Scope -> Class Scope -> Global Scope`).
* **Cross-Unit Resolution:** Builds a directed dependency graph from `use` statements, running a DFS algorithm to detect and prevent circular dependencies.
* **Static Type Checking:** Validates all assignments, resolves operators based on numeric widening ranks (e.g., `i` promotes to `f`), validates function arity, and performs type inference on array literals.

### 4. Code Generation (LLVM IR)
The `CodeGenerator` traverses the validated AST and emits LLVM Intermediate Representation via LLVMSharp bindings. 
* Classes are mapped to LLVM named structs.
* It uses a two-pass approach to register function prototypes before emitting basic blocks, enabling mutual recursion.
* The resulting `.ll` file is passed to a Clang subprocess (`clang output.ll -o programa.exe`) to produce a standalone executable binary.

---

## 💻 Sample Code (RedLang)
```red
object Program
{
    entry func Main():i
    {
        declare x:i = 5;
        declare y:i = 3;

        loop (declare j:i = 0; j < 10; set j = j + 1)
        {
            show(j + y);
        }

        check (x > y)
        {
            show("x is greater");
        }
        otherwise
        {
            show("y is greater");
        }

        gives 0;
    }
}
