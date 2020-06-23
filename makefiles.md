## Makefiles for for not-only programmers

Make is commonly used in software development for managing compilation of the source code. Use cases of make however
go far beyond compiling C code, as it can be used as a tool for writing any kind of command pipelines. While this may
be considered as a heresy by some, Makefiles can be quite useful as a place to store *collection of commands to
execute*, I agree on this with Peter Baumgartner:

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">After using Makefiles on a few DS projects, my advice is: forget everything you&#39;ve read about them, put `SHELL := /bin/bash` at the top, and just use it as a place to name and document project-specific shell commands.</p>&mdash; Peter Baumgartner (@pmbaumgartner) <a href="https://twitter.com/pmbaumgartner/status/1275203940829839363?ref_src=twsrc%5Etfw">June 22, 2020</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

For learning make, there is a great, freely available, book [*Managing Projects with GNU Make*][book] by Robert
Mecklenburg and [extensive online documentation][docs].

### :checkered_flag: Basics and syntax

To use make, you need to define the instructions in the Makefile. The [syntax for the instructions][tutorial] is

```
target: prerequisites # comment
<TAB> recipe
```

where usually the *target* is a compiled, binary file, *recipe* is the set of instructions needed for generating the
file, and *prerequisites* are the target names of the other instruction that needs to run before running the current one.
The recipe is just a set of shell instructions.

Using tabs in Makefiles is important, for example if you used spaces instead of tab, you could see an errors like:

```console
Makefile:3: *** missing separator.  Stop.
```

### :wave: Hello World!

Let's start with "Hello World!" example. To run it, you need to install make, and create a file named *Makefile*,
having the following content

```makefile
hello:
	echo "Hello World!"
```

To run it, got to the directory containing the Makefile and run `make hello` command from the command line.

### :page_facing_up: Multi-line instructions

The instructions are not limited to single-line ones, they can consist of any number of tab-prefixed lines.

```makefile
hello:
	touch hello
	echo "Hello World!" > hello
	cat hello
```

The example is overtly complicated for such simple task, but it shows how you can define multiple steps to be run
sequentially (create empty file with `touch`, write "Hello World!" to it using `echo`, and print it with `cat`).

### :open_file_folder: Make keeps files up-to-date

Make always checks if the *target* file is available, and if this is the case, it doesn't run the instruction to build
it. Moreover, if any of the prerequisites are newer than the target, it re-runs the sequence of instructions. This
is helpful when compiling source code stored in multiple files, since it keeps the compiled binaries up-to-date with
each other.

You can try yourself running the Makefile below. What happens when any of the `one`, `two`, `three`, or `four` files
does not exist? What if they differ in time of creation? Notice that make will print all the instructions executed.
At any time, you can run `make clean` (don't bother with its syntax for now) to remove all of the files and start
from a scratch.

```makefile
.PHONY: clean
clean:
	@ rm -rf one two three four

one two: # this has two targets!
	touch one
	touch two

three:
	touch three

four: two three
	touch four
```

The `four` target depends on `three` and `two`, but not `one`. So `make four` checks if `four` file exists, than it
recursively checks its dependencies and their dependencies. Missing files, or discrepancies in file save dates invoke
commands for creating the file, and all the upstream commands.

### :sunglasses: Phony targets

I said that *target* is usually a filename, but this doesn't have to be the case. Let's again use the trivial Makefile:

```makefile
hello:
	echo "Hello World!"
```

If by chance you have the file named *hello* in the directory containing your Makefile, you would see the following
message:

```console
$ touch hello
$ make
make: 'hello' is up to date.
```

What make did, is it checked that `hello` file exists, so it doesn't need to build it. The above functionality is
not relevant when using [*phony* targets][phony], i.e. targets without related file. In such cases, make will display
the "strange" messages like above. To disable the check for target file, you can use the `.PHONY` variable to list
such targets:

```makefile
.PHONY: hello

hello:
	echo "Hello World!"
```

Using phony targets is not that uncommon, for example, you could add instructions like `help`, to print the help,
or `clean` to clean the working directory from unnecessary files, `test` to run unit tests, etc.

### :speak_no_evil: Silence showing the commands

When running the instructions, make by default will print the commands that were invoked, for example when running:

```makefile
hello:
	echo "Hello World!"
```

we'll see the following result:

```console
$ make hello
echo "Hello World!"
Hello World!
```

However, this can be silenced by adding `@` at the begging of line that we want to silence

```makefile
silent-hello:
	@ echo "Hello World!"
```

so will only print the result:

```console
$ make silent-hello
Hello World!
```

Usually you neither need to show all commands what are run, nor to silence them all.

### :sparkles: Using variables

Makefiles can [use variables][vars] that can be modified when calling `make` from command line. For example, with the
following Makefile:

```makefile
MESSAGE ?= "Hello World!"

hello:
	@ echo $(MESSAGE)
```

if `MESSAGE` is not provided, it prints the default:

```console
$ make hello
Hello World!
```

but [we can provide][cmd] it either from environment:

```console
$ MESSAGE="Hi!!!" make hello
Hi!!!
```

or as a parameter:

```console
$ make hello MESSAGE="Hi there!"
Hi there!
```

To set variables, [you can][set vars] use `=`, `:=` (simple expansion), `?=` (set if absent), `+=` (append). The
variables *are evaluated at the time of calling `make`*, so they do not persist:

```console
$ make hello MESSAGE="Hi"
Hi
$ make hello MESSAGE="Bye"
Bye
$ make hello
Hello World!
```

This may be even more obvious with another trivial Makefile:

```makefile
TIME != date

once:
	@ echo $(TIME)

twice:
	@ echo $(TIME)
	@ sleep 5s
	@ echo $(TIME)
```

Every time you call `make once`, it will print different time, but calling `make twice` will print same time twice,
since the `TIME` variable was evaluated only once per call. The above code uses `!=` to run the right hand side code
and assign it to the left hand side variable, alternatively you could use the `shell` command to execute the external:
command:

```makefile
TIME := $(shell date)
```

If you need to access environment variables, use the `${...}` syntax. For example, in Unix system `$USER` environment
variable holds username of the currently logged user, so we can access it with:

```makefile
who:
	@ echo ${USER}
```

### :collision: Macros

Besides variables, make supports [macros][vars & macros]. Macro can be a set of commands, for example:

```makefile
define commands
	@ echo "Hello!"
	@ echo "It's $(shell date)"
endef

default:
	$(commands)
	@ echo "Bye!"
```

Another usage may be to "paste" the parameters into a command:

```makefile
define tz
	--utc \
	'+%Y-%m-%d %H:%M:%s %Z'
endef

utctime:
	date $(tz)
```

This may be useful if we repeat some commands, or parameters in the code, and not want to repeat ourselves.

### :thinking: Conditional statements

Make supports [conditional][ifelse] statements: `ifeq`, `ifneq`, `ifdef`, and `ifndef`. The tricky part is that the
statements are not indented, so the formatting needs to be:

```makefile
COND ?= false

default:
ifeq "$(COND)" "true"
	@ echo "It's true"
else
	@ echo "It's false"
endif
```

### :question: Self-documenting the Makefile

It is useful to provide user with some kind of documentation of what are the functionalities of make. While there is no
build-in solution for that, it can be easily achieved with Makefile comments. Simple and useful solution was described
in a [blog post by François Zaninotto][help]:

```makefile
hello: ## Say hello
	echo "Hello World!"

help: ## Print help
	@ grep -E '^[a-zA-Z_-]+:.*?## .*$$' $(MAKEFILE_LIST) | sort | awk 'BEGIN {FS = ":.*?## "}; {printf "\033[36m%-30s\033[0m %s\n", $$1, $$2}'
```

As you can see, it assumes that the documentation is prefixed with the `##` signs, and prints those lines if yser calls
`make help`. In some cases it might be useful to set `help` as a default goal, by making it the first instruction, or
by setting `.DEFAULT_GOAL := help`.

### :bulb: Other uses of make

Makefiles are useful for [using together with docker][docker1], so instead of needing from the user to run the necessary
commands by hand, you can provide them [with ready recipes][docker2].

```makefile
USER ?= my-username
REPO ?= image-name
TAG ?= 0.1
IMAGE ?= $(USER)/$(REPO):$(TAG)

.PHONY: image push build

build: image push

image:
	docker build --rm -t $(IMAGE) -f Dockerfile .

push:
	docker push $(IMAGE)
```

Make can be used also for [many other tasks][other] like: setting up Python environments, running unit tests and linters
for the code, making API calls, [running Terraform][terraform] to setup cloud-based architecture, and other tasks that
need to be run repeatably, or by different users.

It can be used also for data science projects, where we are interested in building pipelines that download the data,
preprocess it, do feature engineering, train the model, validate the results, save them, etc., as described by
[Zachary M. Jones][jones], [Rob J. Hyndman][hyndman], [Mark Sellors][sellors], [Mike Bostock][bostock],
[Byron J. Smith][smith], [Jenny Bryan][bryan], and others.

### :art: Change the defaults

By default, instructions in Makefile assume using [`sh` as default shell][what shell], however [`/bin/sh` is just
a symbolic link, that in different systems can point to different shells][sh], so for consistency, it might be worth to
[change][change defaults] the `SHELL` [variable](#using-variables), e.g. to `SHELL=bash`.

Some [other useful defaults][change defaults] include using "strict" mode in bash `.SHELLFLAGS := -eu -o pipefail -c`,
or forcing make to check the Makefile for unused variables, and turning off the [automatic rules][rules] written for
parsing source code files:

```makefile
MAKEFLAGS += --warn-undefined-variables
MAKEFLAGS += --no-builtin-rules
```

[In newer versions of make][tabs] you can switch from using tabs for indenting the instructions by setting the
`.RECIPEPREFIX` [variable](#using-variables), though it is probably easier to stick to tabs when using make.

If you want to change the default make goal, so that `make` invokes other than the first recipe in the Makefile,
set `.DEFAULT_GOAL` variable to name of the desired instruction.

 [book]: https://www.oreilly.com/library/view/managing-projects-with/0596006101/
 [docs]: https://www.gnu.org/software/make/manual/make.html
 [tutorial]: https://opensource.com/article/18/8/what-how-makefile
 [what shell]: https://unix.stackexchange.com/questions/217243/which-shell-is-used-in-gnu-make-files
 [phony]: https://www.gnu.org/software/make/manual/make.html#Phony-Targets
 [vars]: https://www.gnu.org/software/make/manual/html_node/Using-Variables.html
 [vars & macros]: https://www.oreilly.com/library/view/managing-projects-with/0596006101/ch03.html
 [cmd]: https://stackoverflow.com/questions/2826029/passing-additional-variables-from-command-line-to-make
 [set vars]: https://stackoverflow.com/questions/448910/what-is-the-difference-between-the-gnu-makefile-variable-assignments-a
 [ifelse]: https://www.gnu.org/software/make/manual/html_node/Conditional-Syntax.html
 [help]: https://marmelab.com/blog/2016/02/29/auto-documented-makefile.html
 [docker1]: https://www.docker.com/blog/containerizing-test-tooling-creating-your-dockerfile-and-makefile/
 [docker2]: https://itnext.io/docker-makefile-x-ops-sharing-infra-as-code-parts-ea6fa0d22946
 [change defaults]: https://tech.davis-hansson.com/p/make/
 [sh]: https://askubuntu.com/questions/141928/what-is-the-difference-between-bin-sh-and-bin-bash
 [rules]: https://www.gnu.org/software/make/manual/html_node/Catalogue-of-Rules.html
 [other]: https://medium.com/@davidstevens_16424/make-my-day-ta-science-easier-e16bc50e719c
 [bryan]: https://stat545.com/automating-pipeline.html
 [smith]: https://byronjsmith.com/make-bml/
 [jones]: http://zmjones.com/make/
 [hyndman]: https://robjhyndman.com/hyndsight/makefiles/
 [bostock]: https://bost.ocks.org/mike/make/
 [sellors]: https://blog.sellorm.com/2018/06/02/first-steps-with-data-pipelines/
 [terraform]: https://medium.com/nubego/how-to-run-terraform-like-a-pro-in-a-devops-world-c21458ba402c
 [tabs]: https://stackoverflow.com/questions/2131213/can-you-make-valid-makefiles-without-tab-characters
