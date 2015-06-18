---
title: Fun(ction) with OCaml
layout: post
date: Wed Jun 17 19:14:02 EDT 2015
summary: Unit testing and automating an OCaml project. Python helps!
---

I've loved functional programming ever since an elephant taught me Haskell[^1], but working with OCaml gave me heartburn.

[^1]: [Learn You a Haskell for Great Good!](http://learnyouahaskell.com/)

## Two's Company

As the second project for my Spring 2015 programming languages class, I was tasked with implementing a collection of OCaml functions, mostly operating on lists of lists of integers (the `int list list` type). My first obstacle was deciphering my professor's surprisingly cryptic specifications -- that is, I couldn't figure out immediately what each function's *function* was.

Beyond the specification hid a huge testing script[^2], unhelpful because it only dumped sample input results without explicitly verifying anything, and would only run with a fully-implemented library. Output diffing was my [last project]({% post_url 2015-04-20-flexing-the-bison-chops %})'s salvation, but here... it wasn't as approachable a solution. I set to figuring out how to test that my functions *while* I was developing them, instead of all at once after I was finished.

[^2]: Copying code out of PDFs is already less fun than it should be. It gets worse when you find lines that have flown off the page and into the infinite abyss. I'm glad my viewer was brave enough to bring them back when I ripped text from the entire page.

As in the semester's first project, my professor ordered us to provide a log file of the OCaml interpreter, proving our functions valid for at least two inputs apiece. In other words: *way* more effort sunk into copy-and-paste than any good Python user should ever have to muster.

There must be a way to minimize my efforts here.

## OCaml by the Pound

I decided that writing [OUnit](http://ounit.forge.ocamlcore.org/) tests before composing the functions themselves would help me to both understand the project spec and test my functions after I finished them. Lengthy but straightforward, I ended up after a few days[^5] with a file full of tests:

[^5]: I slowly parsed the whole thing over a number of reading sessions, because I started early and wasn't pressed for time. My roommate read through the whole assignment in an hour and understood it almost immediately. Bah.


```ocaml
open OUnit2;;

let test_diffImRow test_ctxt =
  assert_equal
    (Motion.diffImRow([0;0;0;0;5],[0;0;0;0;0]))
    [0;0;0;0;-5];
  assert_equal
    (Motion.diffImRow([2; 5; 4; 3; 2; 2; 2; 2; 2], [2; 2; 2; 2; 2; 2; 2; 2; 2]))
    [0; -3; -2; -1; 0; 0; 0; 0; 0];
;;

(* ... *)

let test_tupledifffloat test_tcxt =
  assert_equal
    (Motion.tupledifffloat((5.0,4.0),(2.0, 5.0)))
    (-3.0, 1.0);
  assert_equal
    (Motion.tupledifffloat((7.0,4.0),(9.0, 3.0)))
    (2.0, -1.0)
;;

(* etc. Test execution not shown. *)
```

My unit tests actually met the requirement of proving two or more cases per function, but OUnit's paltry report...

```
................
Ran: 16 tests in: 0.12 seconds.
OK
```

...was a little different from the interpreter's format:

```
# Motion.diffImRow([0;0;0;0;5],[0;0;0;0;0]);;
- : int list = [0; 0; 0; 0; -5]

# Motion.diffImRow([2; 5; 4; 3; 2; 2; 2; 2; 2], [2; 2; 2; 2; 2; 2; 2; 2; 2]);;
- : int list = [0; -3; -2; -1; 0; 0; 0; 0; 0]

# Motion.diffIm([[0;0;1];[0;0;1];[0;0;0]],[[0;0;0];[0;0;1];[0;0;1]]);;
- : int list list = [[0; 0; -1]; [0; 0; 0]; [0; 0; 1]]

# Motion.diffIm([[2;2;4;5;6;2;2;2];[2;2;4;5;6;2;2;2];[2;2;2;2;2;2;2;2]],[[2;2;2;2;2;2;2;2];[2;2;4;5;6;2;2;2];[2;2;4;5;6;2;2;2]]);;
- : int list list =
[[0; 0; -2; -3; -4; 0; 0; 0]; [0; 0; 0; 0; 0; 0; 0; 0];
 [0; 0; 2; 3; 4; 0; 0; 0]]

# Motion.noDiff([[0;0];[0;0]]);;
- : bool = true

# Motion.noDiff([[0;0;0];[0;0;0];[0;0;0]]);;
- : bool = true

# Motion.noDiff([[0;1;0];[0;0;0];[0;0;0]]);;
- : bool = false

# ...you get the idea.
```

## Python Strikes!

Python to the rescue! I bet that Python could make it easy to extract the unit tests' function calls, get the interpreter to run them one-by-one, and build a log out of their output -- and Python certainly could! Here's the code (what it does should be pretty intuitive):

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

That's... not intuitive. Maybe it'll taste sweeter annotated:

```python
import re
import subprocess

with open('test.ml', 'r') as tests:

    # Search my unit tests for function calls from the Motion module,
    # for example: `Motion.diffImRow(...)`. The messy regex
    # attempts to grab one full parenthesis scope (i.e. function
    # arguments), but because it's not that smart...
    full = ("".join(tests.read().splitlines()))
    for match in re.finditer(r'Motion[^\s]+\(.*?(?=\))\)\)??', full):
        if match:
            line = match.group(0).strip()

            # ...I have to handle a couple of exceptions manually.
            # There was one function that took tuple arguments, so
            # the regex couldn't handle it normally. Something
            # more lexical would have been better.
            line = line + ")" if line.count('(') == 3 else line
            line = line + ";;"

            # `line` has become a full-formed OCaml function call
            # that matches whatever I was unit testing, sans output
            # checking. Run that call against the Motion module.
            output = subprocess.check_output(
                    "echo '{}' | ocaml motion.cmo -noprompt"
                        .format(line),
                    shell=True)
                .decode("utf-8")
                .splitlines()

            # Cut the first two lines of output,
            # they're just diagnostics.
            output = "\n".join(output[2:])
            # Echo the query and output to look like it's
            # interpreter output.
            print("#", line)
            print(output)
```

That's better.

## More Demonstrations, Please!

Essentially, my Dr. Frankenscript readies his tools and cuts up this test:

```
let test_tupledifffloat test_tcxt =
  assert_equal
    (Motion.tupledifffloat((5.0,4.0),(2.0, 5.0)))
    (-3.0, 1.0);
  assert_equal
    (Motion.tupledifffloat((7.0,4.0),(9.0, 3.0)))
    (2.0, -1.0)
;;
```

Cackling, he constructs a matching pair of monsters:

```
echo 'Motion.tupledifffloat((5.0,4.0),(2.0, 5.0))' | ocaml motion.cmo -noprompt
echo 'Motion.tupledifffloat((7.0,4.0),(9.0, 3.0))' | ocaml motion.cmo -noprompt
```

Lightning strikes! OCaml springs to life:

```
- : float * float = (-3., 1.)
- : float * float = (2., -1.)
```

The good doctor stitches his creations together for the last time. They are... beautiful!

```
# Motion.tupledifffloat((5.0,4.0),(2.0, 5.0))
- : float * float = (-3., 1.)

# Motion.tupledifffloat((7.0,4.0),(9.0, 3.0))
- : float * float = (2., -1.)
```

## In Closing

Under the guidance of a simple Makefile that runs OUnit and the logging script, whatever functions I've verified are correct are automatically built into a log file. Perfect! For my sanity's sake, I also check the difference between the big testing script's real and correct outputs. I can relax while `make` zips up my project, `README` and logs, so what's to be learned from this whole ordeal?

After bashing my head in over my last project's shell scripting issues, using Python felt **spectacular**. I probably could have wrangled the same results with some tricky piping, but I bet it would have *hurt*. This project hardened my resolve to use Python scripts more often.

With that said: look at my un-annotated script again. Ugh! *Blegh!* **Gross!** It's ugly, hacky and specific; not scalable at all! My original outline for this post wished for code coverage, seeing as my regex almost skipped a couple test cases -- but that would just be a workaround for my poor approach, probably improved with lexical parsing[^3]. I'm not planning to re-visit this project or this script in particular, but writing a blog post about it just a few months after its birth took a bit of thinking. That's not a level of quality I'm comfortable sinking to for any reason.

[^3]: I thought it would be easy to find a module to parse parentheses, but [Shlex](https://docs.python.org/3.5/library/shlex.html) and [Parse](https://pypi.python.org/pypi/parse) both fizzled out with this task.

I mentioned heartburn all the way at the top of this page: I just didn't (and still don't) like OCaml. Coming from Haskell, writing OCaml felt poorly structured, at times unbearably rigid[^4], and just plain inelegant. I was happy to see it go.

[^4]: I had some trouble with floating point math. I felt a fool after my roommate informed me not only that algebraic operations on floating point numbers warrant a dot after the operator (`2.0 +. 2.0`), but also that my professor *explicitly* told us to be careful about it in class. I'm dumb.


I earned another perfect score on this one. Great!
