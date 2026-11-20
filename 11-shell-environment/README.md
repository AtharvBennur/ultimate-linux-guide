# Shell Environment and Command Line

The shell is where users run commands, combine programs, and configure their
interactive environment. This chapter introduces Bash environment variables,
input and output streams, job control, and terminal sessions.

## Environment Variables

Environment variables are named values available to a process. Bash variables
are not automatically passed to child processes; use `export` to make a value
part of the environment:

```bash
APP_ENV=development
export APP_ENV
export APP_ENV=development
printenv APP_ENV
printf '%s\n' "$HOME"
```

`printenv` displays environment variables. Variable names are conventionally
uppercase, and use `$NAME` or `${NAME}` to expand a value.

## PATH

`PATH` is a colon-separated list of directories the shell searches when a
command is entered without its full path.

```bash
printf '%s\n' "$PATH"
command -v bash
export PATH="$HOME/bin:$PATH"
```

The last command prepends `~/bin` for the current shell and its child processes.
Use an absolute executable path when a script must not depend on command lookup.

## Bash Startup Files

For Bash, startup behavior depends on how the shell starts:

- A login shell reads `/etc/profile`, then the first available file among
  `~/.bash_profile`, `~/.bash_login`, and `~/.profile`.
- An interactive non-login shell reads `~/.bashrc`.
- A non-interactive Bash script normally does not read `~/.bashrc`.

Many systems configure `~/.bash_profile` to read `~/.bashrc`, but this is not
automatic in every setup. Other shells have different startup rules.

Put interactive settings such as aliases in `~/.bashrc`. Login-session
environment settings are commonly placed in `~/.profile` or
`~/.bash_profile`. After editing the Bash configuration for the current
interactive shell, reload it with:

```bash
source ~/.bashrc
```

## Aliases

An alias is a shortcut for a command in an interactive shell:

```bash
alias ll='ls -alF'
alias
unalias ll
```

To keep an alias for future interactive Bash sessions, add it to `~/.bashrc`.
Scripts should use commands or shell functions rather than relying on
interactive aliases.

## Standard Input and Output

Commands commonly use three standard streams:

| Stream | File descriptor | Purpose |
|---|---:|---|
| Standard input | 0 | Input read by a command |
| Standard output | 1 | Normal command output |
| Standard error | 2 | Diagnostic and error messages |

## Redirection

Redirection changes where a command reads input or sends output:

```bash
command < input.txt
command > output.txt
command >> output.txt
command 2> errors.txt
command > command.log 2>&1
```

- `<` reads standard input from a file.
- `>` writes standard output to a file, replacing its previous contents.
- `>>` appends standard output to a file.
- `2>` sends standard error to a file.
- `2>&1` sends standard error to the current standard-output destination. In
  `command > command.log 2>&1`, both streams go to the log. Redirection order
  matters because the streams are connected from left to right.

## Pipes and `tee`

A pipe (`|`) sends one command's standard output to the next command's standard
input:

```bash
printf '%s\n' gamma alpha beta | sort
```

Standard error is not included in a pipe unless it is redirected explicitly.
`tee` writes input to both standard output and a file:

```bash
ip addr | tee interface-info.txt
ip route | tee -a network-info.txt
```

Use `tee -a` to append rather than replace the file.

## Background Jobs

Add `&` to start a command as a background job in the current shell:

```bash
sleep 300 &
jobs
fg %1
```

`jobs` lists jobs belonging to the current shell, and `fg %1` brings job 1 to
the foreground. Press `Ctrl+Z` to suspend a foreground job, then use
`bg %1` to resume it in the background. Job numbers are shown by `jobs`.

Background jobs still belong to the shell and may be affected when that shell
closes. `nohup` makes a command ignore the hangup signal; combine it with `&`
to run it in the background and redirect its output:

```bash
nohup ./long-task.sh > long-task.log 2>&1 &
```

## `tmux` Basics

`tmux` keeps terminal sessions running independently of a terminal window.
Install it if needed:

```bash
# Debian/Ubuntu
sudo apt install tmux

# RHEL-family
sudo dnf install tmux
```

Start a named session, detach from it with `Ctrl+B` followed by `D`, list
sessions, and reattach:

```bash
tmux new -s maintenance
tmux ls
tmux attach-session -t maintenance
```

The shell job-control commands above manage jobs in one shell; `tmux` provides
a persistent terminal session that can contain one or more shells.

## Practical Scenario: Why Does My Script Work in the Terminal but Fail in Cron?

An interactive terminal can have a customized `PATH`, aliases, exported
variables, and a particular working directory. Cron usually starts with a
smaller environment and does not normally read interactive Bash startup files.
As a result, a command found in the terminal may not be found by cron, and a
relative file path may refer to a different directory.

Make the script set what it needs and use explicit paths:

```bash
#!/usr/bin/env bash
set -eu

PATH=/usr/local/bin:/usr/bin:/bin
cd /srv/app
/usr/bin/python3 /srv/app/report.py
```

Use the real paths and required environment values for the system. Then capture
cron output for diagnosis:

```cron
*/5 * * * * /home/alex/bin/report.sh >> /home/alex/report.log 2>&1
```

Replace `/home/alex` with the account's actual home directory. Check the log,
script permissions, command paths, working directory, and any variables the
script expects. See [Cron Jobs](../14-cron-jobs/) for cron schedules and
troubleshooting.

## Common Mistakes

- Assuming a shell variable is available to child processes without `export`.
- Assuming cron reads `~/.bashrc` or inherits an interactive terminal's `PATH`.
- Using relative paths in scripts that run from a different working directory.
- Expecting aliases to be available in non-interactive scripts.
- Reversing redirection order when combining standard output and standard error.
- Assuming a job started with `&` survives after its shell exits.

## Interview Questions

**Q: What does `export` do?**  
It marks a shell variable to be included in the environment of child processes.

**Q: What are file descriptors 0, 1, and 2?**  
They are standard input, standard output, and standard error.

**Q: Why can a command work in a terminal but fail in cron?**  
Cron may have a different `PATH`, working directory, and environment, and does
not normally load interactive shell configuration.

## Exercises

1. Print `HOME` and `PATH`, then locate the `bash` executable with `command -v`.
2. Export a variable and verify it with `printenv`.
3. Redirect a command's output and errors to separate files, then combine both
   streams into one log.
4. Pipe a list of words to `sort` and save a copy with `tee`.
5. Start `sleep 300` in the background, inspect it with `jobs`, bring it to the
   foreground, suspend it with `Ctrl+Z`, and resume it with `bg`.
6. Start a named `tmux` session, detach, list sessions, and reattach.
7. Update a script that relies on a relative path so it sets its working
   directory and uses explicit command paths.

**Related:** [Cron Jobs](../14-cron-jobs/) | [Text Processing](../18-text-processing/) | [Process Management](../07-process-management/)
