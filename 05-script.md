---
layout: page
title: The Unix Shell
subtitle: Shell Scripts
minutes: 15
---
> ## Learning Objectives {.objectives}
>
> *   Write a shell script that runs a command or series of commands for a fixed set of files.
> *   Run a shell script from the command line.
> *   Write a shell script that operates on a set of files defined by the user on the command line.
> *   Create pipelines that include user-written shell scripts.

We are finally ready to see what makes the shell such a powerful programming environment.
We are going to take the commands we repeat frequently and save them in files
so that we can re-run all those operations again later by typing a single command.
For historical reasons,
a bunch of commands saved in a file is usually called a **shell script**,
but make no mistake:
these are actually small programs.

Let's start by going back to `molecules/` and putting the following line in the file `middle.sh`:

~~~ {.bash}
$ nano add.py
~~~
~~~ {.python}
import sys

accumulator = 0

for line in sys.stdin:
    accumulator = int(line) + accumulator
    
print accumulator
~~~

This is a simple python script that reads a list of numbers and adds them together, printing the result .

~~~
run script
~~~

Once we have saved the file, we can execute it. .py is a convention that means that it is a python script.
We can do the same with a file for the bash shell. This is similar to the lines that we were using in the
previous sections, with one addition, notice the $1 variable. That is exactly like the arguments that Devendra
taught this morning.

~~~ {.bash}
$ nano births.sh
~~~
~~~
cut -f7 TX.txt | tail -n +2 | python add.py
~~~

~~~ {.bash}
bash birth.sh
~~~

~~~ {.output}
1353198
~~~

Sure enough,
our script's output is exactly what we would get if we ran that pipeline directly.

> ## Text vs. Whatever {.callout}
>
> We usually call programs like Microsoft Word or LibreOffice Writer "text
> editors", but we need to be a bit more careful when it comes to
> programming. By default, Microsoft Word uses `.docx` files to store not
> only text, but also formatting information about fonts, headings, and so
> on. This extra information isn't stored as characters, and doesn't mean
> anything to tools like `head`: they expect input files to contain
> nothing but the letters, digits, and punctuation on a standard computer
> keyboard. When editing programs, therefore, you must either use a plain
> text editor, or be careful to save files as plain text.

What if we want to select lines from an arbitrary file?
We could edit `middle.sh` each time to change the filename,
but that would probably take longer than just retyping the command.
Instead,
let's edit `births.sh` and replace `TX` with a special variable called `$1`:

~~~ {.bash}
$ cat middle.sh
~~~
~~~ {.output}
cut -f7 $1.txt | tail -n +2 | python add.py
~~~

Inside a shell script,
`$1` means "the first filename (or other parameter) on the command line".
We can now run our script like this:

~~~ {.bash}
$ bash births.sh TX
~~~
~~~ {.output}
1353198
~~~

or on a different file like this:

~~~ {.bash}
$ bash birth.sh FL
~~~
~~~ {.output}
227011
~~~


> ## Double-Quotes Around Arguments {.callout}
>
> We put the `$1` inside of double-quotes in case the filename happens to contain any spaces.
> The shell uses whitespace to separate arguments,
> so we have to be careful when using arguments that might have whitespace in them.
> If we left out these quotes, and `$1` expanded to a filename like
> `methyl butane.pdb`,
> the command in the script would effectively be:
>
>     head -15 methyl butane.pdb | tail -5
>
> This would call `head` on two separate files, `methyl` and `butane.pdb`,
> which is probably not what we intended.

We would still need to edit the script if we wanted to add, say, the deaths from a state.

~~~ {.bash}
$ cp births.sh demo.sh
$ nano demo.sh
$ cat demo.sh
~~~
~~~ {.output}
cut -f$2 $1.txt | tail -n +2 | python add.py
~~~
~~~ {.bash}
$ bash demo.sh TX 8
~~~
~~~ {.output}
648033
~~~

This works,
but it may take the next person who reads `middle.sh` a moment to figure out what it does.
We can improve our script by adding some **comments** at the top:

~~~ {.bash}
$ cat demo.sh
~~~
~~~ {.output}
# Add columns of demographic data
# Usage: middle.sh state col_num
cut -f$2 $1.txt | tail -n +2 | python add.py
~~~

A comment starts with a `#` character and runs to the end of the line.
The computer ignores comments,
but they're invaluable for helping people understand and use scripts.

What if we want to process many files in a single pipeline?
For example, if we want to sort our `.txt` files by length, we would type:

~~~ {.bash}
$ wc -l ??.txt | sort -n
~~~

because `wc -l` lists the number of lines in the files
(recall that wc stands for 'word count', adding the -l flag means 'count lines' instead)
and `sort -n` sorts things numerically.
We could put this in a file,
but then it would only ever sort a list of `.txt` files in the current directory.
If we want to be able to get a sorted list of other kinds of files,
we need a way to get all those names into the script.
We can't use `$1`, `$2`, and so on
because we don't know how many files there are.
Instead, we use the special variable `$@`,
which means,
"All of the command-line parameters to the shell script."
We also should put `$@` inside double-quotes
to handle the case of parameters containing spaces
(`"$@"` is equivalent to `"$1"` `"$2"` ...)
Here's an example:

~~~ {.bash}
$ cat sorted.sh
~~~
~~~ {.output}
wc -l "$@" | sort -n
~~~
~~~ {.bash}
$ bash sorted.sh *.txt ./spaces/*.txt
~~~
~~~ {.output}

~~~

> ## Why Isn't It Doing Anything? {.callout}
>
> What happens if a script is supposed to process a bunch of files, but we
> don't give it any filenames? For example, what if we type:
>
>     $ bash sorted.sh
>
> but don't say `*.txt` (or anything else)? In this case, `$@` expands to
> nothing at all, so the pipeline inside the script is effectively:
>
>     wc -l | sort -n
>
> Since it doesn't have any filenames, `wc` assumes it is supposed to
> process standard input, so it just sits there and waits for us to give
> it some data interactively. From the outside, though, all we see is it
> sitting there: the script doesn't appear to do anything.

We have two more things to do before we're finished with our simple shell scripts.
If you look at a script like:

~~~
wc -l "$@" | sort -n
~~~

you can probably puzzle out what it does.
On the other hand,
if you look at this script:

~~~
# List files sorted by number of lines.
wc -l "$@" | sort -n
~~~

you don't have to puzzle it out --- the comment at the top tells you what it does.
A line or two of documentation like this make it much easier for other people
(including your future self)
to re-use your work.
The only caveat is that each time you modify the script,
you should check that the comment is still accurate:
an explanation that sends the reader in the wrong direction is worse than none at all.

Second,
suppose we have just run a series of commands that did something useful --- for example,
that created a graph we'd like to use in a paper.
We'd like to be able to re-create the graph later if we need to,
so we want to save the commands in a file.
Instead of typing them in again
(and potentially getting them wrong)
we can do this:

~~~ {.bash}
$ history | tail -4 > redo-figure-3.sh
~~~

The file `redo-figure-3.sh` now contains:

~~~
297 bash goostats -r NENE01729B.txt stats-NENE01729B.txt
298 bash goodiff stats-NENE01729B.txt /data/validated/01729.txt > 01729-differences.txt
299 cut -d ',' -f 2-3 01729-differences.txt > 01729-time-series.txt
300 ygraph --format scatter --color bw --borders none 01729-time-series.txt figure-3.png
~~~

After a moment's work in an editor to remove the serial numbers on the commands,
we have a completely accurate record of how we created that figure.

In practice, most people develop shell scripts by running commands at the shell prompt a few times
to make sure they're doing the right thing,
then saving them in a file for re-use.
This style of work allows people to recycle
what they discover about their data and their workflow with one call to `history`
and a bit of editing to clean up the output
and save it as a shell script.

> ## Why record commands in the history before running them? {.challenge}
> 
> If you run the command:
> 
> ~~~
> history | tail -5 > recent.sh
> ~~~
> 
> the last command in the file is the `history` command itself, i.e.,
> the shell has added `history` to the command log before actually
> running it. In fact, the shell *always* adds commands to the log
> before running them. Why do you think it does this?

> ## Script reading comprehension {.challenge}
> 
> Joel's `data` directory contains three files: `fructose.dat`,
> `glucose.dat`, and `sucrose.dat`. Explain what a script called
> `example.sh` would do when run as `bash example.sh *.dat` if it
> contained the following lines:
> 
> ~~~
> # Script 1
> echo *.*
> ~~~
> 
> ~~~
> # Script 2
> for filename in $1 $2 $3
> do
>     cat $filename
> done
> ~~~
> 
> ~~~
> # Script 3
> echo $@.dat
> ~~~
