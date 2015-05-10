---
title: Breaking in OCaml
layout: post
date: Sat May  9 22:25:57 EDT 2015
summary: Unit testing and log building for an OCaml project.
---

# The Hassle

- Assignment: build a collection of ocaml functions (simple operations on int list list, mostly)
- Annoyance: only provided test is a big script that dumps all values, only checkable with diff of complete program output (hard to test during dev)
- Must make log of 2+ function output (in ocaml interpreter)
- I wanted to do unit tests, and didn't want to have to retype everything to log it
- Spec was initially confusing / convoluted

# Solution 

- Unit tests with OUnit
  - Helped to understand specs
  - Validated functions worked correctly
- Makefile to build + archive everything
  - Copy .ml to .caml because .caml isn't correct type
- Python script to regex search for all functions in unit tests, feed into ocaml interpreter, generate log from results
- Make also runs all tests
- Make sure to show code for python & example call

# Looking Back

- Python is great
- Solution was not scalable, general, or pretty, but worked well
    - Ugly! Look at houw ugly it is! Gross!
    - Messy check for regex could have been replaced with a different mechanism
- Code coverage would have been nice (almost missed some test cases in regex), but would be weird to implement

* * *

I've loved functional programming ever since an elephant taught me Haskell[^1], but working with OCaml gave me heartburn.

[^1]: [Learn You a Haskell for Great Good!](http://learnyouahaskell.com/)

All of my scripts are the same ones that I ended up submitting. Their ugliness shall be immortalized.

## The Assignment

As the second semester project for my programming languages class, I was tasked with implementing a collection of poorly-explained OCaml functions, most of which operated on lists of lists of integers (the `int list list` type). Deciphering my professor's cryptic specification was my first obstacle. The class was provided with a huge testing script that would run -- and subsequently dump -- the numbers: useful for a few final tests, I guess. That left me with the task of figuring out how to test that my functions all did what they were supposed to do, *while* I was developing them.

The functions' functions turned out to be fairly simple, so I decided to try out OCaml-based unit testing. Diffing may have worked with my last project, but not this time.

As in the last project, my professor ordered us to provide a log file, demonstrating in the OCaml interpreter that our functions were valid for at least two inputs. In other words: *way* more effort sunk into copy-and-paste than anyone with knowledge of Python should ever have to muster.

Surely, there must be a way to minimize my efforts here.

## The Solution

I decided to try [OUnit](http://ounit.forge.ocamlcore.org/) to 

Here's some Python. This was before I started to write my scripts as if I'd ever need to revisit them.

```python
import re
import subprocess

with open('test.ml', 'r') as tests:
    full = ("".join(tests.read().splitlines()))
    for match in re.finditer(r'Motion[^\s]+\(.*?(?=\))\)\)??', full): # What?
        if match:
            line = match.group(0).strip()
            line = line + ")" if line.count('(') == 3 else line # What?!
            line = line + ";;"
            output = subprocess.check_output("echo '{}' | ocaml motion.cmo -noprompt".format(line), shell=True).decode("utf-8").splitlines()
            output = "\n".join(output[2:]) # WHAT?!?
            print("#", line)
            print(output)
```

Let me try that again.

```python
import re
import subprocess

with open('test.ml', 'r') as tests:

    # Search my unit tests for function calls from the Motion module, for example:
    # Motion.diffImRow(...)
    # That messy regex attempts to grab one full parenthesis scope (i.e. function
    # arguments), but because it's not that smart...
    full = ("".join(tests.read().splitlines()))
    for match in re.finditer(r'Motion[^\s]+\(.*?(?=\))\)\)??', full):

        if match:
            line = match.group(0).strip()
            # ...I have to handle a couple of exceptions manually.
            line = line + ")" if line.count('(') == 3 else line # What?!
            line = line + ";;"

            # 
            output = subprocess.check_output("echo '{}' | ocaml motion.cmo -noprompt".format(line), shell=True).decode("utf-8").splitlines()
            output = "\n".join(output[2:]) # WHAT?!?
            print("#", line)
            print(output)
```
