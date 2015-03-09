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
