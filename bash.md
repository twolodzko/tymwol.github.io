# Bash

`/bin/bash` and `/bin/sh` [are *not*](https://stackoverflow.com/questions/5725296/difference-between-sh-and-bash) the same.

```sh
~: sh
$ echo "Default shell is: '$SHELL', but currently I am using '$0'"
Default shell is: '/bin/bash', but currently I am using 'sh'
$
~: dash
$ echo "Default shell is: '$SHELL', but currently I am using '$0'"
Default shell is: '/bin/bash', but currently I am using 'dash'
$
~: fish
Welcome to fish, the friendly interactive shell
Type `help` for instructions on how to use fish
$ echo "Default shell is: '$SHELL', but currently I am using '$0'"
Default shell is: '/bin/bash', but currently I am using ''
```

```sh
#!/bin/bash
```

## Hello World!

```sh
$ echo "Hello World!"
Hello World!
$ printf "Hello %s\n" "World!"
Hello World!
```

```sh
$ printf "%.2f\n" 1,12345
1,12
$ printf "%s went to the %s and bought a %s\n" Jack shop lollypop
Jack went to the shop and bought a lollypop
```

## Functions

```shell
$ hello() {
>   echo "Hello $1!"
> }
$ hello "Tim"
Hello Tim!
```

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

`$0` is the name of the command that was called, `$1` is the first argument, `$2` is the second, etc. There is also `$@` that holds all the arguments.

## Variables

```shell
$ x = 2
x: command not found
$ x=2
$ x
x: command not found
$ echo "$x"
2
```

```shell
$ x=1
$ echo "x=$x"
x=1
$ unset x
$ echo "x=$x"
x=
```

You can also define constants, that cannot be deleted, or altered:

```shell
$ readonly PI=3.14
$ echo $PI
3.14
$ PI=3
sh: 5: PI: is read only
$ unset PI
sh: 6: unset: PI: is read only
```

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



```shell
$ files=$(ls)
```

[guide of variables](https://www.cyberciti.biz/faq/set-environment-variable-linux/)

You can also use ``` `...` ``` instead of `$(...)` but [it's *not* recommended](https://mywiki.wooledge.org/BashFAQ/082)
and Shellcheck would complain.


`export` is used [to set enviroment variables](https://superuser.com/questions/153371/what-does-export-do-in-bash), that
are available to other processes

```shell
export FOO
FOO=1
export BAR=2
```


```shell
$ if [ $((2+2)) -eq 4 ]; then
>   echo "wow! math works!"
> fi
wow! math works!
```


```shell
$ for name in "one" "two" "three"; do
>   echo "$name"
> done
one
two
three
```

```sh
while ...; do
  ...
done
```

```shell
$ die() {
>   exit 1
> }
$ die
$ echo $?
1
```


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

```shell
$ cat << EOF > test.sh
> #!/bin/bash
> set -e
> foo
> bar
> echo "Done!"
> EOF
$ bash test.sh
test.sh: line 2: foo: command not found
```

[the `<<EOF` trick](https://stackoverflow.com/questions/2500436/how-does-cat-eof-work-in-bash)

```shell
$ cat << gummybear
> hi
> this is a
> gummybear
hi
this is a
```


[*Bite Size Bash*](https://wizardzines.com/zines/bite-size-bash/) from the [⭐ wizard zines ⭐](https://wizardzines.com/comics/)
series by Julia Evans ([@b0rk](https://twitter.com/b0rk))

[](https://twitter.com/b0rk/status/1314345978963648524)

```bash
set -euo pipefail
```

[has pitfails](https://mywiki.wooledge.org/BashPitfalls#set_-euo_pipefail)

https://github.com/koalaman/shellcheck






[great `[` vs `[[` comparison](https://stackoverflow.com/a/47576482/3986320), and [this thread](https://unix.stackexchange.com/questions/306111/what-is-the-difference-between-the-bash-operators-vs-vs-vs)


The control flow commands use their names inverted for closing the blocks, so there is `if ... fi` and `case ... esac`. 

```bash
if cond1 ; then
    ...
elif cond2 ; then
    ...
fi
```



```bash
case $variable in
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
ia [a cool trick](https://unix.stackexchange.com/a/75356/91505), that you can use `;&` as a delimiter to call all the 
cases following the matched pattern, or `;;&` to be able to match multiple patterns.



```bash
for value in 1 2 3
do
    echo $value
done
```