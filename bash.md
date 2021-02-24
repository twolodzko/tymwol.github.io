---
layout: post
title:  "Bash pocket guide"
date:   2021-02-23
---

Bash is like regular expressions: everyone uses it, nobody knows it well. Every time I need to write Bash, I find
myself googling a lot. The main purpose for writing this guide is to gather in a single
place the things I google most often.

People usually don't like Bash. I agree, it is not pretty and has many strange quirks. But Bash is useful if you
are running a Unix machine (Docker anyone?). While in most cases you could replace Bash with your favorite programming
language, using few lines of Bash could lead to a much simpler code. For me, Bash is useful for
automating repeated tasks like "download this, do something to those files, call this script, ...". Bash would be almost
always available on the machine, so it is portable (no problems with dependencies!). It is also simple enough that
others would easily understand what your code is doing. Bash is a tool for those fast-and-dirty tasks we often need to do on day to day basis.

If you are looking for more references on Bash, I recommend the [Bash Pocket Reference](https://www.oreilly.com/library/view/bash-pocket-reference/9781449388669/)
book by Arnold Robbins and the [*Bite Size Bash*](https://wizardzines.com/zines/bite-size-bash/) cheatsheet from the
[⭐ wizard zines ⭐](https://wizardzines.com/comics/) series by Julia Evans ([@b0rk](https://twitter.com/b0rk)). If
you need more advanced features than described below, then maybe you need some other tool than Bash for solving your
problem?

## What is `/bin/sh`?

People often wonder if `/bin/sh` and `/bin/bash` is the same and [the answer is *no*](https://stackoverflow.com/questions/5725296/difference-between-sh-and-bash). `/bin/sh` is just a symbolic link to `Bash`, or `Dash`, etc. To check what is your
default shell use `echo "$SHELL"`. To change your default shell, use the [`chsh` utility](https://www.tecmint.com/change-a-users-default-shell-in-linux/).
Because the shells may differ in details of the implementation, make sure to start bash scripts with the [shebang](https://github.com/koalaman/shellcheck/wiki/SC2148):

```bash
#!/bin/bash
```

Bash [should, but does not need to](https://stackoverflow.com/questions/4814006/can-i-assume-bash-is-installed), be
installed on all machines, so this is a safe choice. If unsure, use `#!/bin/sh`, but in such case, you need to
remember that the shell you'll be using would not guarantee to provide all the functionalities of Bash, only the
basic ones defined by [POSIX](https://pubs.opengroup.org/onlinepubs/009695399/utilities/xcu_chap02.html) standard.

## Hello World!

To print, you can use either `echo`, or `printf` (formatted).

```shell
$ echo "Hello World!"
Hello World!
$ printf "%.2f\n" 1,12345
1,12
$ printf "%s went to the %s and bought a %s\n" Jack shop lollypop
Jack went to the shop and bought a lollypop
```

In Bash, you [don't really](https://unix.stackexchange.com/questions/68694/when-is-double-quoting-necessary) need to
quote the printed strings, but it is generally considered a good practice. Quotes improve readability, make
the code more foolproof, and might be needed if the script will be evaluated using shells other than Bash. If you would
use [shellcheck](#Debugging-and-testing) for validating the script, it will always complain about variables that are not quoted,
since it [may lead to problems](https://github.com/koalaman/shellcheck/wiki/SC2086). When quoting strings, double quotes
`"` will evaluate the variables, while single quotes, will take the string as-is.

```shell
$ echo "$PWD"
/my/current/path
$ echo '$PWD'
$PWD
```

## Variables

To assign a local variable, use `=` without any spaces before or after it. The variables can be accessed by prefixing their name with `$`.

```shell
$ x = 2
x: command not found
$ x=2
$ x
x: command not found
$ echo "$x"
2
```

Alternatively, you can use `${}` to access the variable, it may be [useful when creating a string using a variable](https://stackoverflow.com/questions/8748831/when-do-we-need-curly-braces-around-shell-variables)

```shell
$ foobar="hello!"
$ foo="Whiskey "
$ echo "$foobar"
hello!
$ echo "$foo"'bar'
Whiskey bar
$ echo "${foo}bar"
Whiskey bar
```

The variable can be freed by using `unset`

```shell
$ x=1
$ echo "x=$x"
x=1
$ unset x
$ echo "x=$x"
x=
```

You can also define constants, that cannot be deleted, or altered

```shell
$ readonly PI=3.14
$ echo "$PI"
3.14
$ PI=3
sh: 5: PI: is read only
$ unset PI
sh: 6: unset: PI: is read only
```

Additionally, you can use `export` to [make the variable available also to the child processes](https://superuser.com/questions/153371/what-does-export-do-in-bash). There is a nice [guide on Bash variables](https://www.cyberciti.biz/faq/set-environment-variable-linux/)
that goes into more details.

## Operations on the variables

Bash does not check if the variable exists when asking for its value, so `echo $xsSXSaa` would print an empty string,
even if you never defined the `xsSXSaa` variable. Instead, it has a [very advanced syntax](https://www.cyberciti.biz/tips/bash-shell-parameter-substitution-2.html) for interacting with variables. If the variable does not have an assigned
value, you can use `${variable:-default}` to return the `default` value instead, or `${variable:=default}` to *assign*
and return the value. Other expressions are summarized in the table below taken from [this StackOverflow answer](https://stackoverflow.com/a/16753536/3986320).

```
+--------------------+----------------------+-----------------+-----------------+
|   Expression       |  FOO="world"         |     FOO=""      |    unset FOO    |
|   in script:       |  (Set and Not Null)  |  (Set But Null) |     (Unset)     |
+--------------------+----------------------+-----------------+-----------------+
| ${FOO:-hello}      | world                | hello           | hello           |
| ${FOO-hello}       | world                | ""              | hello           |
| ${FOO:=hello}      | world                | FOO=hello       | FOO=hello       |
| ${FOO=hello}       | world                | ""              | FOO=hello       |
| ${FOO:?hello}      | world                | error, exit     | error, exit     |
| ${FOO?hello}       | world                | ""              | error, exit     |
| ${FOO:+hello}      | hello                | ""              | ""              |
| ${FOO+hello}       | hello                | hello           | ""              |
+--------------------+----------------------+-----------------+-----------------+
```

Additionally, Bash offers syntax for operating strings stored in the variables (everything is a string for Bash):

 * to remove pattern from the beginning of the string: `${var#pattern}`, `${var##pattern}`,
 * to remove pattern from the back of the string: `${var%pattern}`, `${var%%pattern}`,
 * to substitute a pattern: `${var/pattern/replacement/}`, or all it's occurences `${var//pattern/replacement/}`,
 * to access substring `${var:offset}`, `${var:offset:length}`
 * convert first `${var^}`, or all `${var^^}` characters to uppercase,
 * convert first `${var,}`, or all `${var,,}` characters to lowercase.

## Arrays

Arrays can be created using round brackets. They are zero-indexed and the elements can be accessed using `${}`.

```shell
$ arr=(1 2 3)
$ echo "${arr[0]}"
1
$ echo "${arr[@]}"
1 2 3
$ arr+=(4 5)
$ echo "${arr[@]}"
1 2 3 4 5
```

You can also iterate over the elements using a `for` loop.

```bash
arr=(1 2 3)
for x in "${arr[@]}"; do
    echo "$x"
done
```

Using curly brackets, you can create sequences `${start..end..step}`.

```shell
$ echo {1..5}
1 2 3 4 5
$ echo {5..1..-2}
5 3 1
$ echo {a..z..3}
a d g j m p s v y
```

When using multiple curly brackets to create a string, it will create *all the combinations* of the possible strings. This can be used together with other commands, for example to create or remove multiple files.

```shell
$ touch file_{1..3}{a..c}.{txt,md}
$ ls file*
file_1a.md   file_1b.md   file_1c.md   file_2a.md   file_2b.md   file_2c.md   file_3a.md   file_3b.md   file_3c.md
file_1a.txt  file_1b.txt  file_1c.txt  file_2a.txt  file_2b.txt  file_2c.txt  file_3a.txt  file_3b.txt  file_3c.txt
```

## Conditional statements

In Bash, you can use two different kinds of methods for evaluating logical expressions `[` and `[[`. This can be very confusing at first since they can behave differently.  [This StackOverflow answer](https://stackoverflow.com/a/47576482/3986320)
compares those operators, and in [this thread](https://unix.stackexchange.com/questions/306111/what-is-the-difference-between-the-bash-operators-vs-vs-vs) that discusses additionally the use of `(` and `((`. More
details can be found on the man page of the [test](https://linux.die.net/man/1/test). TL;DR you can safely use single
`[`, unless you need some specific functionalities of the extended operator `[[`. 

In Bash `&` and `|` are binary AND and OR operators, for logical operators, use instead `&&`, `||`, and `!` for negation.
Additionally, `&&` is commonly used to chan operations, e.g. `sudo apt update && sudo apt upgrade`, in such case
the chain of operations fails at first failure because of using `&&` that is [lazily evaluated](https://en.wikipedia.org/wiki/Lazy_evaluation).

It is useful to know some basic checks: `-z` empty string, `-n` non-empty
string, `-d` directory exists, `-f` file exists, `-s` file is non-empty,
`-x` executable file exists. Strings can be compared using the `=`, `!=`, `<`, `>`, operators, but beware of using `==` that [behaves differently](https://kapeli.com/cheat_sheets/Bash_Test_Operators.docset/Contents/Resources/Documents/index)
when used in `[` and `[[`. For comparing numeric values use instead `-eq` equal, `-ne` not equal, `-lt` lower than,
`-le` less or equal, `-gt` greater than, `-ge` greater or equal. Alternatively, the `==`, `!=`, `<`, `<=`, `>`, `>=`
operators can be used in double round brackets to compare numeric values e.g. `(( 2 < 3 ))` is equivalent to `[ 2 -lt 3 ]`.

## Control flow

The control flow commands use their names inverted for closing the blocks, so there is `if ... fi` and `case ... esac`. 

```bash
if cond1 ; then
    ...
elif cond2 ; then
    ...
fi
```

```shell
$ if [ $(( 2 + 2 )) -eq 4 ]; then
>   echo "wow! math works!"
> fi
wow! math works!
```

For checking multiple conditions, you can use the `case ... in` syntax.

```bash
case "$variable" in
    pattern)
        commands
        ;;
    pattern)
        commands
        ;;
    *)
        commands
        ;;
esac
```

Where the patterns [can be either](https://www.thegeekstuff.com/2010/07/bash-case-statement/) exact values that are
matched, or wildcards and patterns. Moreover, different patterns can be combined using `|`. Additionally, there
is [a cool trick](https://unix.stackexchange.com/a/75356/91505), that you can use `;&` as a delimiter to call all the 
cases following the matched pattern, or `;;&` to be able to match multiple patterns.


## `for` and `while` loops

The `for` loop can either be used to iterate over explicitly listed elements

```shell
$ for name in "one" "two" "three"; do
>   echo "$name"
> done
one
two
three
```

or outputs of commands and arrays (see [Arrays](#Arrays))

```bash
for f in "$(ls)"; do
    echo "$f"
done
```

For iterating until brake condition is met, use `while` loop. The popular use case is [iterating over lines of a file](https://stackoverflow.com/a/10929511/3986320).

```bash
while IFS= read -r line; do
    echo "Text read from file: $line"
done < my_filename.txt
```

## Evaluating expressions

To evaluate an expression you can use ``` `...` ``` or `$(...)`, but using `$(...)` [is recommended](https://mywiki.wooledge.org/BashFAQ/082).
While the quotes in the example below might look awkward, this is a [valid approach in Bash](https://unix.stackexchange.com/questions/118433/quoting-within-command-substitution-in-bash), since variables need to be quoted and the whole
expression also should.

```shell
cmd='date'
echo "$("$cmd")"
Tue 23 Feb 14:56:12 CET 2021 
```

To evaluate math expressions, use double round brackets.

```shell
$ echo "$( 2 + 2 )"
2: command not found
$ echo "$(( 2 + 2 ))"
4
```

## Functions

Functions in Bash are quite different from what you may know from another programming (scripting?) languages. They don't
include inputs in their definitions, instead, but use positional arguments accessed by `$1`, `$2`,
... , `${10}`, ... etc. `$0` is reserved for the [name of the shell, or the shell script that contains the code](https://bash.cyberciti.biz/guide/$0).

```shell
$ hello() {
>   echo "Hello $1!"
> }
$ hello "Tim"
Hello Tim!
```

To access all elements, you can use `$@`.

```shell
$ first() { echo "$1"; }
$ first 1 2 3
1
$ all() { echo "$@"; }
$ all 1 2 3
1 2 3
$ tail() { shift; echo "$@"; }
$ tail 1 2 3
2 3
```

and `$#` holds the number of the arguments that were passed to the function.

```shell
$ count () { echo "$#"; }
$ count a b c
3
```

Remember to use semicolon `;` when writing multiple commands in single line, this also applies to `if ...; then`, `for ...; do`, and if closing the curly braces in the same line `...; }`.

Functions can also use `read` command to access files, or [collect input from the user](https://stackoverflow.com/questions/18544359/how-to-read-user-input-into-a-variable-in-bash).

Functions in Bash do not return anything but the [exit
status](https://en.wikipedia.org/wiki/Exit_status). To provide an exit code use `exit 0` for success,
or any non-zero status, like `exit 1` for error. The exit status of the most recently executed command is available
through the `$?` variable. To communicate with the outside world, they use side effects like printing to [stdout](https://www.howtogeek.com/435903/what-are-stdin-stdout-and-stderr-on-linux/), or saving files.

## Scripts

Bash code often comes not as functions, but as scripts. The scripts behave like functions, so if you create the
`hello.sh` script, you can call it by invoking its name `./hello.sh`, you can also provide positional arguments like
`./hello.sh -h`. When the function is saved in a directory that was [added to the `$PATH`](https://linuxize.com/post/how-to-add-directory-to-path-in-linux/),
for example, `/usr/bin/`, you can call it by just invoking its filename. Example of a trivial script is given below.

```bash
#!/bin/bash

if [[ "$1" == "-h" || "$1" == "--help" ]]; then
    echo "Usage: $(basename "$0") [-h|--help|name]"
    echo
    echo "Print 'Hello World!' or 'Hello [name]!' if name is provided."
    echo "options:"
    echo "-h or --help    Print this Help."
    exit 0
fi

if [ -n "$1" ]; then
    name="$1"
    echo "Hello $name!"
else
    echo "Hello World!"
fi
```

As you can see, since Bash only has positional arguments, flags like `-h` are just strings passed as arguments. For
simple scripts a bunch of `if'`s would be enough, but otherwise you might need to use `case ... in`, combined with
`shift` as described in the answers in [this thread](https://stackoverflow.com/questions/192249/how-do-i-parse-command-line-arguments-in-bash).
But, if it gets that complicated, I usually drop Bash and switch to using a language that has more advanced ways of
parsing the arguments.

The help gets printed when using the `-h` or `--help` flag and then the scripts exits with status `0` (success). There is
no standard format for the documentation, though there is a [popular convention](https://stackoverflow.com/questions/9725675/is-there-a-standard-format-for-command-line-shell-help-text) that optional arguments are described in square brackets and alternatives are separated with `|`.

If you save the script to the `hello.sh` file, next, you can [validate it with `shellcheck`](#Debugging-and-testing), make it
executable, and run it.

```shell
$ shellcheck hello.sh
$ chmod +x hello.sh
$ ./hello.sh -h
Usage: hello.sh [name]

Print 'Hello World!' or 'Hello [name]!' if name is provided.
$ ./hello.sh
Hello World!
$ ./hello.sh Tim
Hello Tim!
```

## Background & parallel processes

To start two simultaneous processes, just combine them with `&`.

```bash
command1 &
command2
```

You can [initialize multiple processes from a `for` loop](https://stackoverflow.com/a/5238146/3986320).


```bash
echo "Spawning 100 processes"
for i in {1..100}; do
    ( ./my_script & )
; done
```

To display a list of active jobs use the `jobs` command.  `fg [job_spec]` moves job to foreground, Ctrl+Z or
`bg [job_spec]` to background, `disown [job_spec]` terminates it. To prevent the processes from dying with the shell
being closed, you can use the "no hangup" `nohup` command.


Those commands are build-in and do not have `man` pages, so use `fg --help` or `help fg` for details. To list all
the build-in commands use `help`.

## Debugging and testing

To run a Bash script in [debug mode](https://tldp.org/LDP/Bash-Beginners-Guide/html/sect_02_03.html) use
`bash -x script.sh`. The debug mode can also be activated for chosen lines in a script by encapsulating them in
`set -x` and `set +x`

Since Bash does not fail by default but continues running (to read more on the `EOF` trick, check
[this thread](https://stackoverflow.com/questions/2500436/how-does-cat-eof-work-in-bash)). 

```shell
$ cat << EOF > test.sh
> #!/bin/bash
> foo
> bar
> echo "Done!"
> EOF
$ bash test.sh
./test.sh: line 1: foo: command not found
./test.sh: line 2: bar: command not found
Done!
```

To turn this behavior off, you [can add](https://twitter.com/b0rk/status/1314345978963648524) the following line in
the beginning of your script:

```bash
set -euo pipefail
```

notice that using it [has some pitfalls](https://mywiki.wooledge.org/BashPitfalls#set_-euo_pipefail), so don't
use it blindly.

To discover common bugs and code smells in Bash scripts, you can use the [open-source `shellcheck` tool](https://github.com/koalaman/shellcheck).
It conducts static analysis of the script and provides many helpful hints for solving the issues.

If you want to add unit tests to your code, there is a [useful `assert.sh` script](https://torokmark.github.io/assert.sh/).
