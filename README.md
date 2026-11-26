# Compiler Construction: Lexical & Syntax Analysis

**Name:** Nitesh Naudiyal | **Roll No.:** 2472031

A Python implementation of core compiler front-end components, covering lexical analysis, `FIRST`/`FOLLOW` set generation, top-down parsing (`LL(1)`), operator precedence parsing, and bottom-up shift-reduce parsing (`LR(0)`, `SLR(1)`, `LALR(1)`, and `CLR(1)`).

## Project Structure

| File | Description |
| :--- | :--- |
| `lexer.py` | Tokenizes source code into keywords, identifiers, numbers, strings, operators, and delimiters; builds a symbol table and reports lexical errors. |
| `first_follow.py` | Computes `FIRST` and `FOLLOW` sets for context-free grammars and demonstrates left-recursion elimination. |
| `ll1_parser.py` | Builds an `LL(1)` predictive parsing table, detects conflicts on left-recursive grammars, and traces stack-based parsing. |
| `operator_precedence.py` | Computes `LEADING`/`TRAILING` sets and builds operator precedence relation tables ($\lessdot, \doteq, \gtrdot$) for layered and ambiguous grammars. |
| `lr0_parser.py` | Constructs canonical `LR(0)` item sets and `ACTION`/`GOTO` tables, demonstrating shift/reduce conflicts on expression grammars. |
| `slr1_parser.py` | Resolves `LR(0)` conflicts using `FOLLOW` sets and illustrates grammars that require stronger lookahead (`S -> L = R`). |
| `lalr1_parser.py` | Merges `LR(1)` item sets with identical cores into `LALR(1)` states and demonstrates reduce/reduce conflicts on non-LALR grammars. |
| `clr1_parser.py` | Builds full canonical `LR(1)` item sets and parsing tables, successfully parsing grammars where `SLR(1)` and `LALR(1)` fail. |

## Usage

Run any module independently using Python 3:

```bash
python3 lexer.py
python3 first_follow.py
python3 ll1_parser.py
python3 operator_precedence.py
python3 lr0_parser.py
python3 slr1_parser.py
python3 lalr1_parser.py
python3 clr1_parser.py
```

## Example Outputs

### 1. Lexical Analysis (`lexer.py`)

Given a sample C-like snippet containing comments, numbers (`3.14e2`), operators (`>=`, `&&`, `+=`), strings, and an invalid character (`@`):

- **Tokens Extracted:** `40` tokens across `KEYWORD`, `ID`, `NUMBER`, `STRING`, `OP`, and `DELIM`.
- **Error Handling:** Reports `ERROR line 9, col 12: illegal character '@'` without halting tokenization.
- **Symbol Table:** Records first-seen `(line:col)` positions for identifiers (`main`, `x1`, `count`, `total`, `s`, `y`).

### 2. Grammar Transformations & `LL(1)` (`first_follow.py`, `ll1_parser.py`)

Left recursion in standard arithmetic grammars causes `LL(1)` table conflicts:

$$
\begin{aligned}
E &\to E + T \mid T \\
T &\to T * F \mid F \\
F &\to ( E ) \mid \text{id}
\end{aligned}
$$

After eliminating left recursion, the grammar becomes `LL(1)` conflict-free:

$$
\begin{aligned}
E &\to T E' \\
E' &\to + T E' \mid \varepsilon \\
T &\to F T' \\
T' &\to * F T' \mid \varepsilon \\
F &\to ( E ) \mid \text{id}
\end{aligned}
$$

- `id + id * id` $\implies$ **ACCEPTED**
- `id + * id` $\implies$ **REJECTED** (no table entry $M[T, \texttt{*}]$)

### 3. Bottom-Up Parser Comparison (`lr0`, `slr1`, `lalr1`, `clr1`)

Each bottom-up parser tests progressively harder grammars to highlight their relative parsing power:

| Grammar | `LR(0)` | `SLR(1)` | `LALR(1)` | `CLR(1)` | Notes |
| :--- | :---: | :---: | :---: | :---: | :--- |
| Balanced Parentheses (`S -> (S) \| x`) | Pass (6 states) | Pass | Pass | Pass | Pure `LR(0)` grammar |
| Expression Grammar (`+`, `*`, `id`) | Fail (2 S/R) | Pass (12 states) | Pass (12 states) | Pass (22 states) | `FOLLOW` sets resolve shift/reduce conflicts |
| Pointer Assignment (`S -> L=R \| R`) | Fail | Fail (1 S/R) | Pass (10 states) | Pass (14 states) | Context-sensitive lookahead prevents false reduction |
| Core-Merge Conflict (`aAd \| bBd \| aBe \| bAe`) | Fail | Fail | Fail (2 R/R) | Pass (14 states) | Merging states in `LALR(1)` conflates lookaheads `d` and `e` |
