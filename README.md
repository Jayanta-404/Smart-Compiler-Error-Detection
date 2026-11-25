# 🛠️ Smart Compiler Error Detection and Correction System

> *Detecting errors and suggesting meaningful corrections* — a Compiler Design project that analyses a subset of the C language, finds **lexical, syntax and semantic errors**, and suggests **ranked, context-aware fixes** you can review, apply, reject, undo and re-check.

---

## 📌 About the Project

Normal compilers tell you *what* is wrong. This project also tells you **how to fix it**.

You paste or upload C code, press **Analyze**, and the system:

1. Breaks the code into tokens (lexical analysis)
2. Builds a syntax tree (syntax analysis)
3. Checks meaning, types and scopes (semantic analysis)
4. Suggests the best correction for every error, explains why, and lets you apply it with one click

---

## ✨ Features

| Area | What it does |
|---|---|
| **Lexical analysis** | Finds invalid characters, malformed numbers, unterminated strings/comments and bad character literals, with exact line and column |
| **Syntax analysis** | LALR(1) parser built with PLY. Gives one precise error per real mistake (no confusing chain of errors) |
| **Semantic analysis** | Symbol table with scopes, type checking, function call checks, array checks, `printf`/`scanf` format checks |
| **Correction engine** | Rule-based fixes, edit distance (Levenshtein / Damerau) for typos like `itn` → `int`, grammar-based and symbol-table-based suggestions. Every suggestion is **verified by re-analysing** the fixed code |
| **Safe code fixer** | Apply, reject, undo, reset. The original code is always kept. Fixes that make assumptions need your confirmation |
| **Web UI** | Code editor, error dashboard, correction workspace, side-by-side diff, final report (downloadable JSON) |

**Error codes:** `LEX001–LEX006`, `SYN001–SYN010`, `SEM001–SEM022`
(full table in [docs/user_guide.md](docs/user_guide.md))

---

## 🧪 Example

**Input**
```c
int main() {
    int x = 10
    int y = 5;
    itn z = x + y;
    return 0;
}
```

**Output**
```
SYN001  line 2, col 15  Missing ';' before keyword 'int'  ->  Insert ';' after number '10'
SYN004  line 4, col 9   Unexpected identifier 'z'         ->  Replace 'itn' with the keyword 'int'
```

---

## 🧱 Supported C Subset

**Supported**
- Types: `int`, `float`, `double`, `char`, `void`, one-dimensional arrays, string and character literals
- Variable declarations (global and local) with initialisers
- Operators: arithmetic, relational, logical, assignment (`= += -= *= /= %=`), `++`, `--`, unary `- + ! &`
- Statements: `if/else`, `for`, `while`, `do-while`, `break`, `continue`, `return`, nested blocks
- Functions: prototypes, definitions, calls, parameters, built-in `printf` and `scanf`
- Comments (`//` and `/* */`); `#include` lines are ignored with a note

**Not supported yet (future work)**
Pointers, `struct`/`union`/`enum`, `switch`, `goto`, ternary `?:`, bitwise operators, casts, `sizeof`, multi-dimensional arrays, `const`/`static`, macros.

> This is **not** a full ISO C compiler. "Passed" means the code passed the checks implemented in this project.

---

## 🏗️ How It Works

```
C source code
     │
     ▼
 Lexer (PLY)  ──►  Parser (PLY, LALR)  ──►  AST
                                             │
                                             ▼
                          Semantic Analyzer + Symbol Table + Type Checker
                                             │
                                             ▼
                              Error Classifier & Error Manager
                                             │
                                             ▼
                    Correction Engine  ──►  Code Fixer (apply / undo)
                                             │
                                             ▼
                                   Streamlit User Interface
```

---

## 📁 Project Structure

```
Smart-Compiler/
├── app.py                     # Streamlit entry point
├── requirements.txt           # Python dependencies
├── pytest.ini                 # Test configuration
├── compiler/                  # Compiler logic (independent of the UI)
│   ├── tokens.py, lexer.py          # Tokens and lexical analysis
│   ├── grammar.py, parser.py        # Grammar rules and parser with error repair
│   ├── ast_nodes.py                 # Syntax tree nodes
│   ├── symbol_table.py, type_checker.py, semantic_analyzer.py
│   ├── error_manager.py, error_classifier.py
│   ├── correction_rules.py, correction_engine.py, code_fixer.py
│   └── compiler_pipeline.py         # Runs all phases in order
├── ui/                        # Streamlit panels and session logic
├── utils/                     # Source locations, diff, validators
├── samples/                   # Example C files (valid and with errors)
├── tests/                     # Automated tests
└── docs/                      # Architecture, grammar, algorithms, report
```

---

## 🚀 Getting Started

### Requirements
- Python **3.10 or newer**
- Git

### Installation

**Windows**
```bat
git clone https://github.com/YOUR-USERNAME/Smart-Compiler.git
cd Smart-Compiler
py -3 -m venv .venv
.venv\Scripts\activate
python -m pip install --upgrade pip
pip install -r requirements.txt
```

**Mac / Linux**
```bash
git clone https://github.com/YOUR-USERNAME/Smart-Compiler.git
cd Smart-Compiler
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

> On PowerShell, if activation is blocked, run `Set-ExecutionPolicy -Scope Process RemoteSigned` once and activate again.

### Run the app
```bash
streamlit run app.py
```
Open **http://localhost:8501** in your browser.

### Run the tests
```bash
pytest -v
```
Expected result: `226 passed`.

---

## 🖥️ How to Use the App

1. **Choose code** — pick a sample in the sidebar and press *Load sample*, upload a `.c` file, or type/paste code.
2. **Analyze code** — the *Error analysis* tab shows every error with its ID, line, column, explanation and suggested fix.
3. **Correction workspace** — apply, reject or undo fixes. The code is re-analysed after every change.
4. **Final compilation report** — see what was found, what was fixed and what remains. Download it as JSON.
5. **Compiler internals** — view the token stream, abstract syntax tree and symbol table.

### Sample files

| File | Purpose |
|---|---|
| `samples/valid_program.c` | Clean program, no errors |
| `samples/lexical_errors.c` | Invalid characters, bad literals |
| `samples/syntax_errors.c` | Missing `;`, missing `)`, misspelled keyword |
| `samples/semantic_errors.c` | Undeclared variables, type mismatch, wrong argument count, out-of-scope use |

---

## 💻 Use Without the UI

```python
from compiler.compiler_pipeline import CompilerPipeline
from compiler.code_fixer import CodeFixer

pipeline = CompilerPipeline()
result = pipeline.analyze(open("samples/syntax_errors.c").read())
print(result.summary())

for d in result.diagnostics:
    print(d.error_id, d.line, d.column, d.message, "->", d.suggestion)

fixer = CodeFixer(result.source)
best = next(c for d in result.diagnostics for c in d.corrections if c.edits)
fixer.apply(best, confirmed=True)      # fixer.undo() reverts
print(pipeline.analyze(fixer.current).summary())
```

---

## ✅ Testing

The project has **226 automated tests** covering the lexer, parser, semantic analyzer, correction engine, code fixer, UI session logic and the full pipeline. All pass with real PLY 3.11 and pytest on Python 3.12.

```bash
pytest -v
```

More details in [docs/testing.md](docs/testing.md).

---

## 🛠️ Troubleshooting

| Problem | Fix |
|---|---|
| `ModuleNotFoundError: ply` | Activate the virtual environment, then `pip install -r requirements.txt` |
| `ModuleNotFoundError: compiler` / `ui` / `utils` | Run commands from the project root folder |
| Editor shows a plain text box | `pip install streamlit-ace` (optional component) |
| Port 8501 is busy | `streamlit run app.py --server.port 8502` |
| Stale parser tables | Delete `parsetab.py`, `parser.out` and `__pycache__` |

---

## 📚 Documentation

- [Architecture](docs/architecture.md)
- [Grammar](docs/grammar.md)
- [Algorithms](docs/algorithms.md)
- [User guide](docs/user_guide.md)
- [Testing](docs/testing.md)
- [Project report](docs/project_report.md)

---

## 🔮 Future Work

- Pointers, `struct`, `switch` and other C features
- Multi-dimensional arrays and preprocessor macros
- Support for more languages
- ML-based suggestion ranking
- Online deployment (Streamlit Community Cloud)

---

## 👤 Author

**Jayanta Dey**
B.Tech CSE, Brainware University

---

## 📄 License

This project is licensed under the **MIT License**. See [LICENSE](LICENSE) for details.

