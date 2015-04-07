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

One of my class assignments at Clemson this semester (Spring 2015) was to build a scanner and parser for simple 2D paths using Flex (the scanning) and Bison (the parsing). The paths, formatted as string input, would generally look like this:

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

Rather than write my own testing scripts, I turned to Google and found [Bats][bats], a lovely little testing framework for Bash. To automatically generate bats test cases from my `*.in` files, I wrote a horrendously ugly shell script called `build-bats.sh`:

{% highlight bash %}

#!/usr/bin/env bash
NAME=motion.bats

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

Running `build-bats.sh` would subsequently build `motion.bats`, with a unique test case for every test from my input files. Thanks to my standard prefixes, it was pretty easy to automatically indicate which output file to check against:

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

@test "nuunllrn" {
  echo '"nuunllrn"' should be "valid".
  run check "$BATS_TEST_DESCRIPTION" valid
  [ $status -eq 0 ]
}

...

{% endhighlight %}

Then, running `bats motion.bats` generates output that looks a lot like this:

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

TODO: note escapes to make sure every whitespace bit is accounted for

That takes care of the verification part. What about generating the log?

### Logging

I wrote a logging script a lot like the bats script:

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

It outputs exactly what I want it to:

```
$ echo "lllll" | ./motion

 Motion Trajectory Checker (Scanner/Parser)
 CPSC/ECE 3520 Spring 2015

***** congratulations *****
***** scan/parse for valid motion path successful *****

...

$ echo "ludr" | ./motion

 Motion Trajectory Checker (Scanner/Parser)
 CPSC/ECE 3520 Spring 2015

***** congratulations *****
***** valid motion path AND CLOSED PATH *****

...

$ echo " uu6" | ./motion

 Motion Trajectory Checker (Scanner/Parser)
 CPSC/ECE 3520 Spring 2015
syntax error

```

### Putting it All Together

I have a makefile with both of these:

```
# Run with "make test". Best test everything!
test: all
	./build-bats.sh
	bats motion.bats
```

```
verify: ama2-SDE1.zip
	rm -rf $@
	unzip -d $@ $<
	cp generate-log.sh yyerror.c buildit invalid.* closed.* valid.* $@
	cd $@ && ./buildit motion && \
		diff <(./generate-log.sh) ../motion.log && echo -e "\nYou are good!"
```

## Reflection

Nothing yet!

[bats]: https://github.com/sstephenson/bats
