# minishell

A small but capable Unix shell written in C, built as part of the [42 school](https://42.fr) curriculum. `minishell` reads commands from an interactive prompt and executes them much like `bash`, supporting pipes, redirections, logical operators, subshells, environment expansion, wildcards and a set of built-in commands.

```console
minishell ~ echo "hello $USER" | tr a-z A-Z > greeting.txt
minishell ~ cat greeting.txt && echo done
HELLO PIERRE
done
```

## Features

- **Interactive prompt** with command history, powered by GNU Readline.
- **Built-in commands**: `cd`, `echo` (with `-n`), `env`, `export`, `pwd`, `unset` and `exit`.
- **Pipelines** (`|`) chaining any number of commands.
- **Logical operators** `&&` and `||`, with **parentheses** `( )` for grouping and subshells.
- **Redirections**: input `<`, output `>`, append `>>` and here-documents `<<`.
- **Environment expansion**: `$VAR`, `$?` (last exit status), with single/double quote handling.
- **Wildcards**: `*` globbing against the current directory.
- **Signal handling** that mirrors bash for `Ctrl-C`, `Ctrl-\` and `Ctrl-D`.
- **Memory-safe execution** via an internal garbage collector tracking open file descriptors and allocations.

## How it works

A command line travels through four stages before producing output:

1. **Lexing & tokenizing** — the raw input is split into a doubly linked list of typed tokens (words, operators, redirections).
2. **Syntax & grammar checks** — quotes, operators and parentheses are validated before anything runs.
3. **Tree building** — tokens are assembled into a binary tree (`t_btree`) ordered by operator priority, so `&&`, `||`, `|` and subshells nest correctly.
4. **Execution** — the tree is walked recursively, expanding variables, resolving wildcards, wiring up pipes and redirections, then forking and `execve`-ing each command.

## Getting started

### Prerequisites

- A C compiler (`cc` / `gcc`)
- `make`
- GNU Readline development headers

On Debian/Ubuntu:

```bash
sudo apt-get install build-essential libreadline-dev
```

On macOS (with Homebrew):

```bash
brew install readline
```

### Build

```bash
make
```

This compiles the bundled `libft` and links the final `minishell` executable.

Useful targets:

| Target | Description |
| ------ | ----------- |
| `make` | Build the `minishell` binary. |
| `make clean` | Remove object files. |
| `make fclean` | Remove object files and the binary. |
| `make re` | Rebuild everything from scratch. |

### Run

```bash
./minishell
```

You'll get an interactive prompt. Type commands as you would in bash, and `exit` (or `Ctrl-D`) to quit.

## Usage examples

```bash
# Pipelines and redirection
minishell ~ ls -la | grep ".c" | wc -l > count.txt

# Logical operators and subshells
minishell ~ (make && ./minishell) || echo "build failed"

# Here-document
minishell ~ cat << EOF
> line one
> line two
> EOF

# Environment expansion
minishell ~ export NAME=world
minishell ~ echo "hello $NAME, exit was $?"

# Wildcards
minishell ~ echo *.c
```

> [!NOTE]
> `minishell` takes no arguments — it is meant to be launched and used interactively. Running it with arguments exits immediately.

## Project layout

```
minishell/
├── main.c               # Entry point and main read-eval loop
├── include/             # Public headers (minishell.h, structs.h)
├── libft/               # Bundled C standard-library helpers + ft_printf + GNL
└── src/
    ├── init/            # Data and environment initialization
    ├── parsing/         # Tokenizer, lexer, syntax & grammar checks, quotes
    ├── tree/            # Binary-tree construction from the token list
    ├── exec/            # Execution: pipes, redirections, heredoc, expansion, wildcards
    ├── builtins/        # Built-in command implementations
    ├── signals/         # Parent- and child-process signal handlers
    ├── errors/          # Error reporting
    ├── gb_collector/    # File-descriptor and allocation tracking
    └── utils/           # Shared helpers (env list, list ops, exit codes, …)
```

## Authors

Built by [pbeyloun](https://github.com/) and pajimene as a 42 project.
