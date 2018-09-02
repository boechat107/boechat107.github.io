---
layout: single
title: "6 Unix Commands Every Data Scientist Should Know"
description: "6 Unix Commands Every Data Scientist Should Know"
category: Programming
tags: [Shell, Unix]
---

In some situations, unprepared data scientists can spend much more time than
necessary on secondary tasks.
Although their main concern should stay at analysing data, checking hypothesis,
engineering features, etc., they often need to get their hands dirty and code
auxiliary scripts and parsers to get the information they need.

This post presents 6 basic Unix commands (maybe seven) that, once incorporated
in the day-by-day toolset, can potentially improve the productivity of any data
scientist:

* [grep](#grep)
* [cat](#cat)
* [find](#find)
* [wc](#wc)
* [head/tail](#headtail)
* [awk](#awk)

In addition, it is shown how two auxiliary commands (`xargs` and `man`) can
improve even further the usability of the six commands above.

## grep

`grep` searches for patterns in files or in the standard input. It is really
useful to find stuff in files without opening them in an editor.


### Examples

Recursively search in the directory **r_project** for the sequence of characters
**ggplot**.
* option `-r` stands for recursion
* option `-n` tells `grep` to show the line number in the files

```
$ grep -rn ggplot r_project/
r_project/script.R:4:library(ggplot2)
r_project/script.R:247:t<-ggplot(base.test.c, aes(x=score, colour=gap_dic, group=gap_dic))
r_project/script.R:265:t<-ggplot(base.test.c[base.test.c$gap_total<40000,], aes(x=gap_total, colour=corte2, group=corte2))
```

Show all occurrences of the sequence **TRAIN -** in the log file. Note that
`grep` is case sensitive by default.

```
$ grep 'TRAIN -' my_app.log
```

Search the pattern **2018_05** in the list of files/directories returned
by `ls`.
Here `grep` is searching for a pattern in the standard input.

```
$ ls models_by_date/ | grep 2018_05
```

This is a way to find installed version of a Python package.
`pip freeze` returns a list of installed packages to the standard output and
`grep` searches for **gpyopt** (`-i` makes it case insensitive) in the standard
input.

```
$ pip freeze | grep -i gpyopt
GPyOpt==1.2.5
```

This example uses Perl regular expression to search for packages and versions in
a Python setup file.

```
$ grep -oP "'[\w]+ == [\d.]+'" python_library/setup.py
'numpy == 1.15.0'
'fire == 0.1.3'
'gpyopt == 1.2.5'
'recsys_commons == 0.1.0'
```

## cat

Simply prints on the screen (standard output) the contents of files or of the
standard input. It is very simple, but it also saves us the time of opening a
file in an editor for a quick look.

```
$ cat script.sh
#!/bin/bash

set -eo pipefail

echo "$BLEU"
```

## find

Like its name suggests, it finds files by specifying many different (and
optionally) kinds of parameters. It is also able to execute some simple actions
or an entire command line using the resultant files.

### Examples

Recursively find all files with **json** extension in the current directory.

```
$ find . -name '*.json'
./third-party/wiwinwlh/src/26-data-formats/example.json
./third-party/wiwinwlh/src/26-data-formats/crew.json
```

Find files with **pyc** extension and delete them.

```
$ find my_library/modules -name '*.pyc' -delete
```

Here we tell it to search only for files with the exact name **setup.py**,
ignoring directories (`-type f`), and executing `grep` (`-exec`) on each one of
them.
Note that `{}` is used to pass a file path as an argument to `grep` and
`\;` to mark the end of the command. `grep` parameter `-H` makes it print the
filename; play with these options to understand them.

```
$ find . -name setup.py -type f -exec grep -Hn boto3 {} \;
```

## wc

`wc` is very useful to count lines, words or even characters in files or the
standard input.

### Examples

Count the number of lines in a single file.

```
$ wc -l data.csv
1024 data.csv
```

Count the number of lines in each **csv** file and show the total number of
lines.

```
$ wc -l data_dir/*.csv
102224 data_dir/part-00000-02aa95cd-3907-44c8-87ee-97ff44677349-c000.csv
102513 data_dir/part-00001-02aa95cd-3907-44c8-87ee-97ff44677349-c000.csv
204737 total
```

## head/tail

Like working with *DataFrames*, `head` and `tail` print the first and the last
lines of files or the standard input.

### Examples

Take only the first line of a file. This could be used to get of the header of
a CSV file.

```
$ head -n 1 data.csv
```

Print the last 20 lines of a log file.

```
$ tail -n 20 app.log
```

## awk

`awk` uses a programming language (AWK) for text processing. It is powerful and
might seem complicated to learn and use. However, there are a few commands that
can be used very frequently.

### Examples

Print the number of columns by analysing only the first line of a CSV file.
First, head sends the first line to the standard output, which is consumed (as
the standard input) by `awk` and broken up into a sequence of fields delimited
by `,`. `NF` holds the number of fields in a line.

```
$ head -n 1 data.csv | awk -F ',' '{print NF}'
91
```

Print the first 10 lines (default for head) of the file's third column only.

```
$ head data.csv | awk -F ',' '{print $3}'
var_x
3.0
4.0
3.0
3.0
3.0
3.0
3.0
2.0
3.0
```

## xargs

`xargs` is a kind of an assistant program, since its role is convert the
standard input into an argument of another program. This is really useful to
make a chain of processing programs, passing the output of one as an input of
another.

### Examples

Install Python libraries, one by one, from a **setup.py** file.  We first take
the libraries and their versions in patterns like `'gpyopt == 1.2.5'` using
`grep`, then we pass each one of them as an argument to `pip`. Note that `{}` is
used as a placeholder.

```
$ grep -oP "'[\w]+ == [\d.]+'" setup.py | xargs -i pip install {}
```

Recursively find **__pycache__** directories (files are ignored because of
`-type d`) and remove them, including their contents (`rm` is applied
recursively with the option `-r`).

```
$ find my_app -name '__pycache__' -type d | xargs -i rm -r {}
```

Safely remove Git branches. First we list all Git branches of a repository,
then we pass each branch name to `git branch -d`, which delete only fully merged
branches.

```
$ git branch | xargs -i git branch -d {}
```

## man

`man` is also another assistant program. It provides an interface to reference
manuals of almost all UNIX commands. The image below shows the result of
checking the manual of `man` itself (`man man`).


{% include figure
    image_path="https://upload.wikimedia.org/wikipedia/commons/d/db/Unix_manual.png"
    alt="man img"
    caption="By Kamran Mackey - Arch Linux using the Cinnamon display manager., GPL, https://commons.wikimedia.org/w/index.php?curid=524936"
%}
