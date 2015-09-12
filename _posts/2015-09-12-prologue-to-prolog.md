---
title: The Prologue to Prolog
layout: post
summary: How I used Python to automate another project during my junior year at Clemson University.
---

The third and final project that I was assigned during my Spring 2015 Programming Languages class at Clemson University was not quite as exciting as its predecessors: my class was told to re-create a [certain previous project][ocaml] in Prolog. Prolog is a declarative, faintly magical language; nothing to be afraid of until you ask anyone what a cut is[^2].

[ocaml]:{% post_url 2015-06-19-ocaml-and-python %}
[^2]: [Something like advanced horticulture.](https://en.wikibooks.org/wiki/Prolog/Cuts_and_Negation)

Essentially, my assignment was to design a collection of Prolog rules that dealt with lists of lists of numbers. Along with some example data, the assignment contained a specification-complete log of resolution results[^1] involving those examples. As with all previous class projects, I had to turn in a log file, much like the one provided, proving that either my program worked or I was bored enough to fake it.

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

Thanks to my previous work on the class's OCaml project, I already had a solution to the logging problem close at hand -- that is, I already knew how to use Python to feed queries one-at-a-time into an interpreter and combine the results into my own log. Thanks to my instructor, I also had a near-complete test suite in the "correct" log, so I figured that if I could extract the queries from his sample log, then I could use them to build my own log. Then I could diff the two logs to check my program for correctness.

Why not take the time [like before][ocaml] to unit test my rules? Easy answer: writing unit tests took *forever*[^3] when I did them last, and I really only wanted the tests to help me understand the assignment. They'd do me little good here, if I could figure out another way to test overall correctness.

[^3]: A couple hours, but hey.

Let's jump right in and take a look at the Python script that I wrote. I haven't cleaned up my code at all since I wrote it, but I added some comments for clarity.

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

Running something like `logger.py < sample.log | tee real.log` effectively duplicated the sample log (as long as I did a decent job coding up the Prolog rules) while I would skim for immediately obvious errors. Then, `colordiff -ZbwBbd sample.log real.log` [checked for whitespace-independent differences](http://linux.die.net/man/1/diff) (I ended up trimming empty the lines out from `sample.log` for some reason, so the real log had empty lines and the sample one didn't).

I slammed all that into a Makefile that handled everything short of project submission[^4]. All I had to do to check to see if I was done coding was type `make`, and that'd fail if the log it generated was incorrect. Otherwise, I'd get a tidy little archive ready to send in to my instructor. Automating this project was a lot easier than previous ones!

I earned full marks on this assignment, rounding out my semester at three perfect project scores. Great!

[^4]: Our course projects were submitted through BlackBoard, an automation-unfriendly web-based class management system. I *bet* I could submit automatically with Selenium and PhantomJS, but I haven't tried that yet. I may run into trouble with those projects that don't allow multiple submissions...

