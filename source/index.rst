======================================
Testing BoF
======================================

GNU Tools Cauldron 2016

David Malcolm <dmalcolm@redhat.com>

Topics
------

Dave:

  * selftest framework
  * RTL frontend (vs RTL selftests)
  * gimple frontend (are Prasad or Richi here?)

Jeremy:

  * making test suite more independent of GCC

Ed and Graham:

  * parallelism ideas


Automated testing improvements
------------------------------

* We have good end-to-end testing:

  * ~300k DejaGnu tests [*]

* We've not been so great at unit-testing.

  * Some testing of data structures via compiler plugins

The earlier a bug is detected, the better.

[*] (expressed as the number of PASS results)


"-fself-test"
-------------

(this is in trunk for GCC 7)

* A new framework for writing unit tests for GCC.

* Run by default at each of the 3 stages of a bootstrap.

  * ...but compiled out (by preprocessor) in a release build

* Exercises the host code; aborts and fails the build if anything
  goes wrong.

* Trivial to hack out the test suite if a test is failing
  on a particular host.

* Good for our modularization efforts

  * Modular tests make it easier to have modular code.

.. nextslide::
   :increment:

* About 30000 assertions [*], so far, covering:

  * core data structures (vec, bitmap, etc)

  * gengtype and the garbage-collector

  * CFG-handling

  * location-handling

  * diagnostics subsystem

  * etc

[*] in terms of the count of checks that are run at runtime

.. nextslide::
   :increment:

The location & diagnostics tests are actually each run 24 times,
exercising the various combinations of

* without/without the range-packing optimization

* at/near various interesting boundary values of the 32-bit
  location_t type

Very hard to achieve this level of coverage with our
traditional DejaGnu end-to-end approach

.. nextslide::
   :increment:

* Unit testing and end-to-end testing are both worthwhile

* Consider "belt and braces" approach when fixing a bug

  * i.e. add *both* kinds of tests

.. nextslide::
   :increment:

Questions about -fself-test:

* what about post stage 1, when we transition to
  ``--enable-checking=release`` by default


RTL frontend -> selftests
-------------------------

* Ability to write unit tests for individual RTL passes.

  * Parse RTL dumps going into one pass, run just that one pass.

  * Likely to be very target-specific

* Under development, hopefully ready by close of GCC 7 stage 1.

* Rewritten to work as selftests, rather than "just" a frontend

  * "[PATCH 0/9] RFC: selftests based on RTL dumps"

    * https://gcc.gnu.org/ml/gcc-patches/2016-09/msg00483.html

gimple frontend
---------------

(are Prasad or Richi here?)
