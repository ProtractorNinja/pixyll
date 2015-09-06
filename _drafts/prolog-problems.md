---
title: Prolog Problems
layout: post
date: Sat Sep  5 16:37:11 EDT 2015
summary: A third interesting project from my junior year. Contains automation and Python.
---

Prolog is *weird*.

# The Prologue to Prolog

As the third and final project of those I was assigned during my Spring 2015 Programming Languages class at Clemson University, my class was told to re-create a [certain previous project]({% post_url 2015-06-19-ocaml-and-python %}) in Prolog. Prolog is a declarative, faintly magical language: it's nothing to be afraid of until you ask anyone what a cut is[^2].

[^2]: [Something like advanced horticulture.](https://en.wikibooks.org/wiki/Prolog/Cuts_and_Negation)

Essentially, my assignment was to design a collection of Prolog rules and facts that dealt with lists of lists of numbers. Along with some example data, the project specification happily included a complete log of resolution results[^1] involving those examples. As with all previous class projects, I had to turn in a log file, much like the one provided, proving that my program worked.

[^1]: That was *horrendously obnoxious* to extract from the project PDF.

Here's a snippet of the log file from the project specification. Queries to the SWI-Prolog interpreter start with `?- `, and Prolog's answers are suffixed by a period.

```
?- ['sde3_example_data.pro'].
true.

?- startup.
true.

?- image_tgr11(A),printImage(A).
0 0 0 20 0 
0 10 10 0 0 
0 10 10 0 0 
0 0 0 0 0 
0 0 0 0 0 
A = [[0, 0, 0, 20, 0], [0, 10, 10, 0, 0], [0, 10, 10, 0, 0], [0, 0, 0, 0, 0], [0, 0, 0, 0|...]].

?- image_tgr11(A),image_tgr12(B),diffIm(A,B,C).
A = [[0, 0, 0, 20, 0], [0, 10, 10, 0, 0], [0, 10, 10, 0, 0], [0, 0, 0, 0, 0], [0, 0, 0, 0|...]],
B = [[0, 0, 0, 0, 0], [0, 10, 10, 0, 0], [0, 10, 10, 0, 0], [0, 20, 0, 0, 0], [0, 0, 0, 0|...]],
C = [[0, 0, 0, -20, 0], [0, 0, 0, 0, 0], [0, 0, 0, 0, 0], [0, 20, 0, 0, 0], [0, 0, 0, 0|...]].
```

Time to get to work!

# Python Shows its Power

Thanks to my previous work on the class's OCaml project, I already had a solution to the logging problem close at hand -- that is, I already knew how to feed queries one-at-a-time into an interpreter and combine the results into my own log. Thanks to my instructor, I also had a near-complete test suite in the "correct" log, so I figured that if I could extract the queries from his sample log, then I could use them to build my own log. Then I could diff the two logs to check my program for correctness.

Why not do unit testing like last time? I already knew what to do, and writing unit tests for *every* function took *forever*.

Let's jump right in and take a look at the Python script that I wrote. I didn't clean up my code at all, but I added some comments for clarity.

```python
#!/usr/bin/env python

import subprocess
import sys

# Extract the queries from the correct log, cutting off
# the prompt's query indicator ( ?- )
qs = sys.stdin.readlines()
in_lines = [ x[3:] for x in filter(lambda l: '?-' in l, qs) ]

# Dump the queries into a quiet version of the prolog interpreter.
pipe = subprocess.Popen(["swipl", "-q"], 
        universal_newlines=True,
        stderr=subprocess.STDOUT,
        stdin=subprocess.PIPE,
        stdout=subprocess.PIPE)
(o, _) = pipe.communicate("".join(in_lines))

# Save all the output lines into a list, without removing newline
# characters from each line. An isolated newline character ends
# up as a delimiter between the output for a query and the next
# query; we'll be using that later to insert the missing queries
# into our log, so we'll prepend our list of output lines with
# another newline.
real_lines = o.splitlines(keepends=True)
real_lines.insert(0, '\n')

# Replace each isolated newline with one of the input queries.
for i, l in enumerate(real_lines):
    if l == "\n" and len(in_lines) > 0:
        real_lines[i] = '\n?- ' + in_lines.pop(0)

# Print the now-complete log!
print("".join(real_lines).strip())
```

Neat, huh?

Now that I have my own log, I just have to compare it to the real log with `colordiff -ZbwBbd`, which ignores all differences in whitespace. Coupled with a Makefile that automates log generation and testing (everything short of project submission[^3]), automating this project was a lot easier than previous ones!

I earned full marks on this assignment, rounding out my semester at three perfect project scores. Great!


[^3]: Our course projects were submitted through BlackBoard, an automation-unfriendly web-based class management system. I could *probably* submit automatically with Selenium and PhantomJS if it were that important.

