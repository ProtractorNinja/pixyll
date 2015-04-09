---
title: Flexing the Bison Chops
layout: post
date: Mon Apr  6 23:22:52 EDT 2015
summary: How I automated testing, building and logging a Flex and Bison project.
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

What happens when you mix a script script with a compiler compiler?

## A Challenger Approaches

One of my class assignments at Clemson this semester (Spring 2015) was to build a scanner and parser for simple 2D paths using Flex for the scanning and Bison for the parsing. The paths, formatted as string input, would generally look like this:

```
unudlrnnnud // up, up, no movement, down, left, right...
u33         // up 33 times
udlrx       // up, down, left, right, illegal character: syntax error!
udlr        // a closed path that deserves recognition
```

My program, run with a single input string on `stdin`, was supposed to print a header, parse the path, and print a result that indicated whether the path was a plain old path or a special closed path. Illegal input would eschew the result and cry `syntax error`.

The most tedious part of the assignment? My final submission archive had to contain `motion.log`, a file that showed at least five demonstrations of program correctness for each type of path (valid, valid & closed, and invalid).

My thoughts at the beginning of the project went something like this:

- I want to automate building the log *without* needing to do a ton of typing if I want to add, remove or change any test cases.
- My program's output formatting should be totally accurate, just in case my instructor's testing script was designed to be particularly draconian (it wasn't).
- My program should always be correct, for all manner of ridiculously formatted inputs. Any path containing invalid characters or more than one `newline` should result in a `syntax error`.
- I want to be able to create a ready-to-submit zip archive (containing my sources, a readme, and the log) whenever I want to, very quickly.
- I should track the whole thing under version control, just in case something terrible happens.

Now, without sharing any core project code (which would walking a little _too_ close to the academic dishonesty line), here's how I did all that.

## The Solution

### Preliminaries

Throughout my code, I used the keywords `valid`, `closed`, and `invalid` to refer to correctly presented (but not closed) paths, correctly presented _and_ closed paths, and shadily-composed paths that should cause a syntax error.

My solution to the problem of easily adding test cases to both the log and my unit tests came in the form of three text files: `valid.in`, `closed.in`, and `invalid.in`. Each contained multiple lines with one test case each. `invalid.in`, for example, ended up looking like this:

```
_hotnsebu_
  ddddd
rrr uuu
lurd  
lurdg
rrrddlurbllluu
66u6
 uu6
rlrl-1
uu\nu
\nrl
rl\n
```

A `\n` should, hopefully, get expanded to a newline somewhere down the road. All of these tests have something terribly wrong with them (the non-obvious ones are probably trailing whitespace; that's a no-no), so they would all (in theory) send off a syntax error if run through the program. 

I also made a `*.out` file for each keyword that represented the ideal output text when testing a path of that type. For instance, `closed.out` looked something like this:

```

 Motion Trajectory Checker (Scanner/Parser)
 CPSC/ECE 3520 Spring 2015

***** congratulations *****
***** valid motion path AND CLOSED PATH *****
```

Naturally, all of that is useless just sitting in a file; which brings me to...

### Going Batty, or How I Handled Unit Testing

Rather than write my own testing scripts, I turned to Google and found [Bats][bats], a lovely little testing framework for Bash. With Bats' help, I effectively wrote a shell script that would generate a shell script to verify that every test case in my `*.in` files produced proper output when run with my program.

The enabler to this terrifying process would be a horrendously ugly shell script called `build-bats.sh`:

{% highlight bash %}

#!/usr/bin/env bash
NAME=motion.bats # Honestly, I'm not sure why I added this variable but never used it.

echo "#!/usr/bin/env bats" > motion.bats

echo '
function check {
  diff <(echo "$1" | ./motion) "$2.out"
} ' >> motion.bats

testings=( valid closed invalid )

for i in "${testings[@]}"; do
  while IFS='' read -r v; do
    echo "
@test \"$v\" {
  echo '\"$v\"' should be \"$i\".
  run check \"\$BATS_TEST_DESCRIPTION\" $i
  [ \$status -eq 0 ]
}" >> motion.bats
  done < $i.in
done

{% endhighlight %}

Running `build-bats.sh` would produce `motion.bats`, a long file that looked something like this:

{% highlight bash %}

#!/usr/bin/env bats

function check {
  diff <(echo "$1" | ./motion) "$2.out"
} 

@test "lllll" {
  echo '"lllll"' should be "valid".
  run check "$BATS_TEST_DESCRIPTION" valid
  [ $status -eq 0 ]
}

@test "n56rl" {
  echo '"n56rl"' should be "closed".
  run check "$BATS_TEST_DESCRIPTION" closed
  [ $status -eq 0 ]
}

@test "_hotnsebu_" {
  echo '"_hotnsebu_"' should be "invalid".
  run check "$BATS_TEST_DESCRIPTION" invalid
  [ $status -eq 0 ]
}

# ...etc.

{% endhighlight %}

And finally, executing `bats motion.bats` yielded a deliciously comprehensive fruit of validity:

```
1..24
ok 1 lllll
ok 2 nuunllrn
ok 3 rrdlurdlu
ok 4 rrrdldrull
ok 5 nnnnn
ok 6 n6d6u6r85555
ok 7 r0
ok 8 ludr
ok 9 u3l4d5r4u2
ok 10 rrrddlurdllluu
ok 11 u6d6nnnn
ok 12 n56rl
ok 13 _hotnsebu_
ok 14   ddddd
ok 15 rrr uuu
ok 16 lurd  
ok 17 lurdg
ok 18 rrrddlurbllluu
ok 19 66u6
ok 20  uu6
ok 21 rlrl-1
ok 22 uu\nu
ok 23 \nrl
ok 24 rl\n
```

### Logging

Log generation wasn't quite so meta: I wrote a script to run all of my test cases, format them as if the text was an interactive shell session, and shove the text was in `motion.log`.

`generate-log.sh` handled the heavy lifting:

{% highlight bash %}

#!/usr/bin/env bash
testings=( valid closed invalid )

for i in "${testings[@]}"; do
  while IFS='' read -r v; do
    echo "$ echo \"$v\" | ./motion"
    echo "$v" | ./motion
    echo ""
  done < $i.in
done

exit 0
{% endhighlight %}

...and boy, did it do good work:

```
$ echo "lllll" | ./motion

 Motion Trajectory Checker (Scanner/Parser)
 CPSC/ECE 3520 Spring 2015

***** congratulations *****
***** scan/parse for valid motion path successful *****


$ echo "ludr" | ./motion

 Motion Trajectory Checker (Scanner/Parser)
 CPSC/ECE 3520 Spring 2015

***** congratulations *****
***** valid motion path AND CLOSED PATH *****


$ echo " uu6" | ./motion

 Motion Trajectory Checker (Scanner/Parser)
 CPSC/ECE 3520 Spring 2015
syntax error

```

## Reflection

The scripts, combined in a lovingly usable Makefile, performed admirably... *eventually*. You may have noticed the abundance of awkward script quoting that I had to do in my shell scripts; a painful side effect of writing bash to generate *more bash*.

One memorable annoyance was figuring out how to keep `echo` from stripping intentional whitespace around my strings. Another was properly including newline characters in the strings to test. Yet another, perhaps my favorite, was resolving the differences between single and double quotes to actually print the strings that I wanted to see. All those and more were the reasons I decided to try out Python for my next class project.

In the end, I got a perfect score. Great!

[bats]: https://github.com/sstephenson/bats
