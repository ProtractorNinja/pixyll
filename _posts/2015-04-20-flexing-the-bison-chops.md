---
title: Flexing the Bison Chops
layout: post
date: Mon Apr 20 14:15:02 EDT 2015
summary: How I automated testing, building and logging a Flex and Bison project.
---

This is what happens when you mix a script script with a compiler compiler.

My scripts are largely unchanged from how I ended up using them, except for a few fresh comments here and there. I figure that if I start publishing them in all their ugliness, I'll end up scaring myself into writing prettier scripts.

## A Challenge Presented

One of my class assignments at Clemson during the Spring 2015 semester was to build a scanner and parser for simple 2D movement paths. Flex would would handle the scanning, and Bison the parsing. The paths, formatted as string input, would generally look like this:

```
unudlrnnnud // up, up, no movement, down, left, right...
u33         // up 33 times
udlrx       // up, down, left, right, illegal character: syntax error!
udlr        // a closed path, deserving of recognition
```

My program, run with a single input string on `stdin`, was supposed to echo a generic header, parse the path, and print a result that indicated whether the path was a plain old path or a special closed path. Upon detection of illegal input, the program would eschew the result and cry `syntax error`.

The most tedious part of the assignment? My ultimate submission archive had to contain `motion.log`, a file containing at least five demonstrations of program correctness for each type of path (valid, closed, and invalid).

My thoughts at the beginning of the project went something like this:

- I want to automate building the log, so that if I want to add, remove or change any test cases I can regenerate it *without* doing a ton of typing.
- My program's output formatting should be totally accurate, just in case my instructor's testing script was designed to be particularly draconian (it wasn't).
- My program should always be correct, for all manner of ridiculously formatted inputs. Any path containing invalid characters or more than one `newline` should result in a `syntax error`.
- I want to be able to create a ready-to-submit zip archive (containing my sources, a readme, and the log) whenever I want to, very quickly.
- I should track the whole thing under version control, just in case something terrible happens.

Now, without sharing any core project code (which would walking a little _too_ close to the academic dishonesty line), here's how I did all that.

## A Solution Detailed

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

Note that escaped newlines (`\n`) are all expanded into genuine newlines by `echo`, a little further down the page from here. 

All of those tests have something terribly wrong with them (the non-obvious ones are probably illegal trailing whitespace; one of the more obtuse error cases), so they should all send off a syntax error if run through the main program.

I also made a `.out` file for each keyword that represented the ideal output text when testing a path of that type. For instance, `closed.out` looked something like this:

```

 Motion Trajectory Checker (Scanner/Parser)
 CPSC/ECE 3520 Spring 2015

***** congratulations *****
***** valid motion path AND CLOSED PATH *****
```

Naturally, everything in this section is useless just sitting in their text files; which brings me to...

### Going Batty Over Unit Testing

Rather than write my own testing scripts, I turned to Google and found [Bats][bats], a lovely little testing framework for Bash. With Bats' help, I effectively wrote a shell script that would generate a shell script to verify that every test case in my `.in` files produced proper output when run through my main program.

The enabler to this terrifying process was a horrendously ugly shell script called `build-bats.sh`:

{% highlight bash %}

#!/usr/bin/env bash
# Honestly, I'm not sure why I added this variable
# but never actually used it.
NAME=motion.bats

echo "#!/usr/bin/env bats" > motion.bats

echo '
function check {
  # Example useage: check udlr closed -> diffs output from former 
  # command with latter correct output
  diff <(echo "$1" | ./motion) "$2.out"
} ' >> motion.bats

testings=( valid closed invalid )

# For every line in each .in file, check the line 
# against the prefix's .out file.
for i in "${testings[@]}"; do
  while IFS='' read -r v; d
    echo "
@test \"$v\" {
  echo '\"$v\"' should be \"$i\".
  run check \"\$BATS_TEST_DESCRIPTION\" $i
  [ \$status -eq 0 ]
}" >> motion.bats
  done < $i.in
done

{% endhighlight %}

Running `build-bats.sh` generated `motion.bats`, a file that looked something like this (except ten times longer):

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

And finally, executing `bats motion.bats` verified the correctness of every one of my tests:

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

Neat, huh?

### Logging

Log generation wasn't quite so meta. I wrote a script to run all of my test cases, format them as if the text was an interactive shell session, and shove the text into `motion.log`.

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

The scripts, combined in a lovingly usable Makefile, performed admirably... *eventually*. You may have noticed all the awkward script quoting that I had to do to make my scripts work: a painful side effect of writing bash to generate *more bash*.

One memorable annoyance was figuring out how to keep `echo` from stripping intentional whitespace around my strings. Another was properly including newline characters in the strings to test. Yet another, perhaps my favorite, was resolving the differences between single and double quotes to finally print the strings that I wanted to see.

You can probably see why I decided to use Python to fulfill my next project's scripting needs.

In the end, I got a perfect score. Great!

[bats]: https://github.com/sstephenson/bats
