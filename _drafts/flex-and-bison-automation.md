# A Problem Arises

- Testing a program that prints 1 of 3 outputs on command line (err, valid, semantically a "loop")
- Validate output & format was correct (maybe overkill)
- Had to generate log file to prove to prof that it worked
- Project has already been submitted, won't share details, just scripts

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
