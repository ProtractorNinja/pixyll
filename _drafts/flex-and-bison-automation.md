---
title: Flexing the Bison Chops
layout: post
date: Wed Mar 11 12:42:25 EDT 2015
summary: How I automated testing and logging for a Flex and Bison project.
---

# One Solution Prevails

- Common stems: error, closed, valid
- 3 files w/ one test case per line
- each file represents output type
- 3 files w/ correct output for type
- Bash script -> generate bats test script that feeds each line from each file into program & checks output
- bash script -> run each line of every file in program + append to log output
- combine w/ magic makefile to test on lab machines
- Result: edit three files, autobuild log (in case I need to change program) & build & run unit tests for verification

# Takeaway

- Bash scripts didn't work well for printing, lots of problems with properly escaping things -> try Python for next setup
    - Don't use bash to generate bash scripts again, ugh
- Bats is cool for bash scripts
- Maybe could have done python unit testing?
- Got 100% correct

* * *

## A Challenger Approaches

One of my class assignments at Clemson this semester (Spring 2015) was to build a scanner and parser for simple 2D paths using Flex (for scanning) and Bison (for parsing). The paths, formatted as string input, would generally look like this:

```
unudlrnnnud // up, up, no movement, down, left, right...
u33         // up 33 times
udlrx       // up, down, left, right, illegal character!
udlr        // a closed path that deserves recognition
```

My program, run with a single input string on `stdin`, was supposed to print a header, parse the path, and print a result that indicated whether the path was a plain old path or a special closed path. Illegal input would fail with a `syntax error`.

The most tedious part of the assignment was the instruction to provide in the final submission archive a log file containing at least five demonstrations of program correctness for each type of path.

Here's an outline of my thoughts at the beginning of the project:

- I want to automate building the log *without* needing to do a ton of typing if I want to add, remove or change any test cases.
- My program's output formatting should be totally accurate, just in case my instructor's testing script was designed to be particularly draconian (it wasn't).
- My program should always be correct, for all manner of ridiculously formatted inputs. Any path containing invalid characters or more than one `newline` should result in a `syntax error`.
- I want to be able to create a ready-to-submit zip archive (containing my sources, a readme, and the log) whenever I want to, very quickly.
- I should track the whole thing under version control, just in case something terrible happens.

Now, here's how I did all that. I'm not sharing any core project code, which would walking a little _too_ close to the line of academic dishonesty.

## The Solution
