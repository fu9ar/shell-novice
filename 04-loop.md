---
layout: page
title: The Unix Shell
subtitle: Loops
minutes: 15
---
> ## Learning Objectives {.objectives}
>
> *   Write a loop that applies one or more commands separately to each file in a set of files.
> *   Trace the values taken on by a loop variable during execution of the loop.
> *   Explain the difference between a variable's name and its value.
> *   Explain why spaces and some punctuation characters shouldn't be used in file names.
> *   Demonstrate how to see what commands have recently been executed.
> *   Re-run recently executed commands without retyping them.

**Loops** are key to productivity improvements through automation as they allow us to execute 
commands repetitively. Similar to wildcards and tab completion, using loops also reduces the 
amount of typing (and typing mistakes).
We have these .txt files, but we need to copy a subset of them to another directory as .tsv files.
TSV is a file extension for tab seperated values, basically a text file that is read as a spreadsheet.
We can't use:

~~~ {.bash}
$ cp *.txt data/*.tsv
~~~

because that would expand to:

~~~ {.bash}
$ cp [all .txt files in the dir] *.tsv
~~~

This wouldn't back up our files, instead we get an error:

~~~ {.error}
cp: target `data/*.tsv' is not a directory`
~~~

This a problem arises when `cp` receives more than two inputs. When this happens, it
expects the last input to be a directory where it can copy all the files it was passed.
Since there is no directory named `data/*.tsv` in the directory we get an
error.

Instead, we can use a **loop**
to do some operation once for each thing in a list.
Here's a simple example that displays the first three lines of each file in turn:

~~~ {.bash}
$ for filename in TX.txt NM.txt OK.txt AL.txt LA.txt
> do
>    head -3 $filename
> done
~~~
~~~ {.output}
output
~~~

When the shell sees the keyword `for`,
it knows it is supposed to repeat a command (or group of commands) once for each thing in a list.
In this case, the list the filenames.
Each time through the loop,
the name of the thing currently being operated on is assigned to
the **variable** called `filename`.
Inside the loop,
we get the variable's value by putting `$` in front of it:
`$filename` is `TX.txt` the first time through the loop,
`NM.txt` the second,
and so on.

By using the dollar sign we are telling the shell interpreter to treat
`filename` as a variable name and substitute its value on its place,
but not as some text or external command. When using variables it is also 
possible to put the names into curly braces to clearly delimit the variable
name: `$filename` is equivalent to `${filename}`, but is different from
`${file}name`. You may find this notation in other people's programs.

Finally,
the command that's actually being run is our old friend `head`,
so this loop prints out the first three lines of each data file in turn.

> ## Follow the Prompt {.callout}
>
> The shell prompt changes from `$` to `>` and back again as we were
> typing in our loop. The second prompt, `>`, is different to remind
> us that we haven't finished typing a complete command yet. A semicolon, `;`, 
> can be used to separate two commands written on a single line.

We have called the variable in this loop `filename`
in order to make its purpose clearer to human readers.
The shell itself doesn't care what the variable is called;
if we wrote this loop as:

~~~ {.bash}
for x in TX.txt NM.txt OK.txt AL.txt LA.txt
do
    head -3 $x
done
~~~

or:

~~~ {.bash}
for temperature in TX.txt NM.txt OK.txt AL.txt LA.txt
do
    head -3 $temperature
done
~~~

it would work exactly the same way.
*Don't do this.*
Programs are only useful if people can understand them,
so meaningless names (like `x`) or misleading names (like `temperature`)
increase the odds that the program won't do what its readers think it does.

Here's a slightly more complicated loop:

~~~ {.bash}
for filename in *.txt
do
    echo $filename
    cut -f3 $filename
done
~~~

The shell starts by expanding `*.dat` to create the list of files it will process.
The **loop body**
then executes two commands for each of those files.
The first, `echo`, just prints its command-line parameters to standard output.
For example:

~~~ {.bash}
$ echo hello there
~~~

prints:

~~~ {.output}
hello there
~~~

In this case,
since the shell expands `$filename` to be the name of a file,
`echo $filename` just prints the name of the file.
Note that we can't write this as:

~~~ {.bash}
for filename in *.txt
do
    $filename
    cut -f3 $filename | tail -n +2
done
~~~

because then the first time through the loop,
when `$filename` expanded to `AK.txt`, the shell would try to run `AK.txt` as a program.
Finally, cut and tail prints the third column starting from line 2

> ## Spaces in Names {.callout}
> 
> Filename expansion in loops is another reason you should not use spaces in filenames.
> Suppose our data files are named:
> 
> ~~~
> Texas.txt
> New Mexico.txt
> Oklahoma.txt
> ~~~
> 
> If we try to process them using:
> 
> ~~~
> for filename in *.txt
> do> ## Variables in shell scripts {.challenge}
335
>
336
> In the molecules directory, you have a shell script called `script.sh` containing the 
337
> following commands:
338
>
339
> ~~~
340
> head $2 $1
341
> tail $3 $1
342
> ~~~
343
> 
344
> While you are in the molecules directory, you type the following command:
345
>
346
> ~~~
347
> bash script.sh '*.pdb' -1 -1
348
> ~~~
349
> 
350
> Which of the following outputs would you expect to see?
351
>
352
> 1. All of the lines between the first and the last lines of each file ending in `*.pdb`
353
>    in the molecules directory 
354
> 2. The first and the last line of each file ending in `*.pdb` in the molecules directory
355
> 3. The first and the last line of each file in the molecules directory
356
> 4. An error because of the quotes around `*.pdb`
357
​
358
> ## List unique species {.challenge}
359
> 
360
> Leah has several hundred data files, each of which is formatted like this:
361
> 
362
> ~~~
363
> 2013-11-05,deer,5
364
> 2013-11-05,rabbit,22
365
> 2013-11-05,raccoon,7
366
> 2013-11-06,rabbit,19
367
> 2013-11-06,deer,2
368
> 2013-11-06,fox,1
369
> 2013-11-07,rabbit,18
370
> 2013-11-07,bear,1
371
> ~~~
372
> 
373
> Write a shell script called `species.sh` that takes any number of
374
> filenames as command-line parameters, and uses `cut`, `sort`, and
375
> `uniq` to print a list of the unique species appearing in each of
376
> those files separately.
377
​
378
> ## Find the longest file with a given extension {.challenge}
379
> 
380
> Write a shell script called `longest.sh` that takes the name of a
381
> directory and a filename extension as its parameters, and prints
382
> out the name of the file with the most lines in that directory
383
> with that extension. For example:
384
> 
385
> ~~~
386
> $ bash longest.sh /tmp/data pdb
387
> ~~~
388
> 
389
> would print the name of the `.pdb` file in `/tmp/data` that has
390
> the most lines.
>     cut -f3 $filename | tail -n +2
> done
> ~~~
> 
> then the shell will expand `*.dat` to create:
> 
> ~~~
> Texas.txt New Mexico.txt Oklahoma.txt
> ~~~
> 
> With older versions of Bash,
> or most other shells,
> `filename` will then be assigned the following values in turn:
> 
> ~~~
> output
> ~~~
>
> That's a problem: `head` can't read files called `New` and `Mexico.txt`
> because they don't exist,
> and won't be asked to read the file `New Mexico.txt`.
> 
> We can make our script a little bit more robust
> by **quoting** our use of the variable:
> 
> ~~~
> for filename in *.txt
> do
>     cut -f3 $filename | tail -n +2
> done
> ~~~
>
> but it's simpler just to avoid using spaces (or other special characters) in filenames.

Going back to our original file copying problem,
we can solve it using this loop:

~~~ {.bash}
for filename in *.txt
do
    cp $filename data/"$filename".tsv
done
~~~

This loop runs the `cp` command once for each filename.
The first time,
when `$filename` expands to `New Mexico.txt`,
the shell executes:

~~~ {.bash}
output
~~~

The second time, the command is:

~~~ {.bash}
cp Texas.txt data/Texas.txt.tsv
~~~

> ## Measure Twice, Run Once {.callout}
> 
> A loop is a way to do many things at once --- or to make many mistakes at
> once if it does the wrong thing. One way to check what a loop *would* do
> is to echo the commands it would run instead of actually running them.
> For example, we could write our file copying loop like this:
> 
> ~~~
> for filename in *.txt
> do
>     echo cp $filename data/$filenamoriginale
> done
> ~~~
> 
> Instead of running `cp`, this loop runs `echo`, which prints out:
> 
> ~~~
> cp basilisk.dat original-basilisk.dat
> cp unicorn.dat original-unicorn.dat
> ~~~
> 
> *without* actually running those commands. We can then use up-arrow to
> redisplay the loop, back-arrow to get to the word `echo`, delete it, and
> then press "enter" to run the loop with the actual `cp` commands. This
> isn't foolproof, but it's a handy way to see what's going to happen when
> you're still learning how loops work.



> ## Beginning and End {.callout}
>
> We can move to the beginning of a line in the shell by typing `^A`
> (which means Control-A)
> and to the end using `^E`.

When she runs her program now,
it produces one line of output every five seconds or so:

~~~ {.output}
NENE01729A.txt
NENE01729B.txt
NENE01736A.txt
...
~~~

1518 times 5 seconds,
divided by 60,
tells her that her script will take about two hours to run.
As a final check,
she opens another terminal window,
goes into `north-pacific-gyre/2012-07-03`,
and uses `cat stats-NENE01729B.txt`
to examine one of the output files.
It looks good,
so she decides to get some coffee and catch up on her reading.

> ## Those Who Know History Can Choose to Repeat It {.callout}
> 
> Another way to repeat previous work is to use the `history` command to
> get a list of the last few hundred commands that have been executed, and
> then to use `!123` (where "123" is replaced by the command number) to
> repeat one of those commands. For example, if Nelle types this:
> 
> ~~~
> $ history | tail -5
>   456  ls -l NENE0*.txt
>   457  rm stats-NENE01729B.txt.txt
>   458  bash goostats NENE01729B.txt stats-NENE01729B.txt
>   459  ls -l NENE0*.txt
>   460  history
> ~~~
> 
> then she can re-run `goostats` on `NENE01729B.txt` simply by typing
> `!458`.

> ## Variables in Loops {.challenge}
> 
> Suppose that `ls` initially displays:
> 
> ~~~
> fructose.dat    glucose.dat   sucrose.dat
> ~~~
> 
> What is the output of:
> 
> ~~~
> for datafile in *.dat
> do
>     ls *.dat
> done
> ~~~
>
> Now, what is the output of:
>
> ~~~
> for datafile in *.dat
> do
>	ls $datafile
> done
> ~~~
>
> Why do these two loops give you different outputs?

> ## Saving to a File in a Loop - Part One {.challenge}
>
> In the same directory, what is the effect of this loop?
> 
> ~~~
> for sugar in *.dat
> do
>     echo $sugar
>     cat $sugar > xylose.dat
> done
> ~~~
> 
> 1.  Prints `fructose.dat`, `glucose.dat`, and `sucrose.dat`, and the text from `sucrose.dat` will be saved to a file called `xylose.dat`.
> 2.  Prints `fructose.dat`, `glucose.dat`, and `sucrose.dat`, and the text from all three files would be 
>     concatenated and saved to a file called `xylose.dat`.
> 3.  Prints `fructose.dat`, `glucose.dat`, `sucrose.dat`, and
>     `xylose.dat`, and the text from `sucrose.dat` will be saved to a file called `xylose.dat`.
> 4.  None of the above.

> ## Saving to a File in a Loop - Part Two {.challenge}
>
> In another directory, where `ls` returns:
>
> ~~~
> fructose.dat    glucose.dat   sucrose.dat   maltose.txt
> ~~~
> 
> What would be the output of the following loop?
>
> ~~~
> for datafile in *.dat
> do
>     cat $datafile >> sugar.dat
> done
> ~~~
>
> 1.  All of the text from `fructose.dat`, `glucose.dat` and `sucrose.dat` would be 
>     concatenated and saved to a file called `sugar.dat`.
> 2.  The text from `sucrose.dat` will be saved to a file called `sugar.dat`.
> 3.  All of the text from `fructose.dat`, `glucose.dat`, `sucrose.dat` and `maltose.txt`
>     would be concatenated and saved to a file called `sugar.dat`.
> 4.  All of the text from `fructose.dat`, `glucose.dat` and `sucrose.dat` would be printed
>     to the screen and saved to a file called `sugar.dat`

> ## Doing a Dry Run {.challenge}
>
> Suppose we want to preview the commands the following loop will execute
> without actually running those commands:
>
> ~~~ {.bash}
> for file in *.dat
> do
>   analyze $file > analyzed-$file
> done
> ~~~
>
> What is the difference between the two loops below, and which one would we
> want to run?
>
> ~~~ {.bash}
> # Version 1
> for file in *.dat
> do
>   echo analyze $file > analyzed-$file
> done
> ~~~
>
> ~~~ {.bash}
> # Version 2
> for file in *.dat
> do
>   echo "analyze $file > analyzed-$file"
> done
> ~~~

> ## Nested Loops and Command-Line Expressions {.challenge}
>
> The `expr` does simple arithmetic using command-line parameters:
> 
> ~~~
> $ expr 3 + 5
> 8
> $ expr 30 / 5 - 2
> 4
> ~~~
> 
> Given this, what is the output of:
> 
> ~~~
> for left in 2 3
> do
>     for right in $left
>     do
>         expr $left + $right
>     done
> done
> ~~~

