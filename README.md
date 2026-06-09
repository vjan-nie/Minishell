# Minishell 42 - serjimen & vjan-nie

# Minishell

A Bash-like command interpreter written in C, built with a partner at 42 Madrid.

**Tech:** C · Unix processes · pipes · file descriptors · signals

## What it does
Reads and executes shell command lines interactively: external commands with arguments, pipelines, input/output redirections (`<`, `>`, `>>`, `<<`), environment-variable expansion, single/double quoting, the core built-ins (`cd`, `echo`, `env`, `export`, `unset`, `pwd`, `exit`), and interactive signal handling (`Ctrl-C`, `Ctrl-D`, `Ctrl-\`).

## What I worked on
The work was split between the two of us: my partner owned the **parser** (tokenizing, quote handling, variable expansion) and I owned the **executor** — turning the parsed command structure into running processes. That meant:

- Creating processes with `fork()` / `execve()` and resolving binaries against `PATH`.
- Building pipelines so each command's output feeds the next, using `pipe()` and `dup2()`, and reaping children with `waitpid()` to propagate the correct exit status.
- Applying redirections by rewiring file descriptors before `execve()`, including here-documents.
- Running built-ins in the correct context — some must run in the parent process because they change the shell's own state (e.g. `cd`, `export`).
- Handling interactive signals with `sigaction`, so `Ctrl-C` redraws a fresh prompt, `Ctrl-\` is ignored, and `Ctrl-D` sends EOF.

We reviewed and refactored each other's code regularly, so both of us understood the whole shell rather than just our half.

## Design decisions
- **Built-ins in the parent vs. a child process.** Built-ins that change the shell's own state (`cd`, `export`, `unset`, `exit`) run in the parent; the rest run in a forked child like any external command. That split is what makes `export VAR=value` actually persist across commands.
- **Closing unused pipe file descriptors.** In a pipeline, every process has to close the pipe ends it isn't using — otherwise a reader never sees EOF and the shell hangs. Disciplined `close()` after each `dup2()` was the difference between a pipeline that terminates and one that blocks forever.
- **Exit-status propagation.** Each command's status is captured with `waitpid()` and exposed as `$?`, taking the last command's status for a pipeline, matching Bash's behaviour.

## Limitations / what I'd improve
Scope was the mandatory specification: no logical operators (`&&`, `||`), subshells, or wildcard expansion. Those are the natural next features to build on top of the current executor.

## Build & run
```sh
make
./minishell
```

## Authors

- **serjimen** - Sergio Jiménez
- **vjan-nie** - Vadim Jan

## Notes

This project implements a functional but simplified shell. It does not include all Bash features, focusing on the fundamental logic of a command interpreter.
