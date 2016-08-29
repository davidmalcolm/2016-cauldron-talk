======================================
Stuff Dave's been working on for GCC 7
======================================

GNU Tools Cauldron 2016

David Malcolm <dmalcolm@redhat.com>

dmalcolm's work on GCC 7
------------------------

Two broad areas:

* Usability improvements

* Automated testing improvements

along with a few code cleanups


GCC Usability Improvements
--------------------------

Usability (done)
----------------

The following improvements are already in trunk, for GCC 7.

(See https://gcc.gnu.org/gcc-7/changes.html )

.. nextslide::
   :increment:

GCC 6's C and C++ frontends were able to offer suggestions for misspelled
field names::

  spellcheck-fields.cc:52:13: error: 'struct s' has no member named
  'colour'; did you mean 'color'?
     return ptr->colour;
                 ^~~~~~

GCC 7 greatly expands the scope of these suggestions. Firstly, it adds
fix-it hints to such suggestions::

  spellcheck-fields.cc:52:13: error: 'struct s' has no member named
  'colour'; did you mean 'color'?
     return ptr->colour;
                 ^~~~~~
                 color

.. nextslide::
   :increment:

The suggestions now cover many other things, such as misspelled function names::

  spellcheck-identifiers.c:11:3: warning: implicit declaration of function
  'gtk_widget_showall'; did you mean 'gtk_widget_show_all'?
  [-Wimplicit-function-declaration]
     gtk_widget_showall (w);
     ^~~~~~~~~~~~~~~~~~
     gtk_widget_show_all

.. nextslide::
   :increment:

misspelled macro names and enum values::

  spellcheck-identifiers.cc:85:11: error: 'MAX_ITEM' undeclared here
  (not in a function); did you mean 'MAX_ITEMS'?
   int array[MAX_ITEM];
             ^~~~~~~~
             MAX_ITEMS

.. nextslide::
   :increment:

misspelled type names::

  spellcheck-typenames.c:7:14: error: unknown type name 'singed';
  did you mean 'signed'?
   void test (singed char e);
              ^~~~~~
              signed

.. nextslide::
   :increment:

and, in the C frontend, named initializers::

  test.c:7:20: error: 'struct s' has no member named 'colour';
  did you mean 'color'?
   struct s test = { .colour = 3 };
                      ^~~~~~
                      color

.. nextslide::
   :increment:

The preprocessor can now offer suggestions for misspelled directives, e.g.::


  test.c:5:2: error:invalid preprocessing directive #endfi;
  did you mean #endif?
   #endfi
    ^~~~~
    endif

.. nextslide::
   :increment:

Warnings about format strings now underline the pertinent part of the string,
and can offer suggested fixes. In some cases, the pertinent argument is
underlined::

  test.c:51:29: warning: format '%s' expects argument of type 'char *',
  but argument 3 has type 'int' [-Wformat=]
     printf ("foo: %d  bar: %s baz: %d", 100, i + j, 102);
                            ~^                ~~~~~
                            %d


The C++ frontend will now provide fix-it hints for some missing
semicolons::

  test.cc:4:11: error: expected ';' after class definition
   class a {}
             ^
             ;

The idea is that IDEs and the like can hopefully auto-apply these
fix-it hints.

.. nextslide::
   :increment:


-fdiagnostics-parseable-fixits
------------------------------

``-fdiagnostics-parseable-fixits`` allows for fix-it hints to be emitted
in a machine-readable form, suitable for consumption by IDEs.

.. nextslide::
   :increment:

Given e.g.::

  spellcheck-fields.cc:52:13: error: 'struct s' has no member named
  'colour'; did you mean 'color'?
     return ptr->colour;
                 ^~~~~~
                 color

it emits::

  fix-it:"spellcheck-fields.cc":{52:13-52:19}:"color"

(designed to be compatible with the clang option of the same name).

Support in Emacs would be most welcome!  (any Emacs hackers here?)


Usability (in progress)
-----------------------

The following is still in development (i.e. not yet in trunk).

* dmalcolm is working on ``-fdiagnostics-generate-patch`` e.g.::

    --- ../../src/gcc/testsuite/c-c++-common/Wlogical-not-parentheses-2.c
    +++ ../../src/gcc/testsuite/c-c++-common/Wlogical-not-parentheses-2.c
    @@ -8,7 +8,7 @@
     {
       int r = 0;
    -  r += !aaa == bbb;
    +  r += (!aaa) == bbb;

* (``-fdiagnostics-apply-fixits`` was rejected: touching the user's code is
  too risky)

.. nextslide::
   :increment:

* I have a few other fix-it hints under construction.

* Adding a fix-it hint to a diagnostic is a relatively easy hack,
  maybe a good way to get involved in GCC development.

    * Email dmalcolm@redhat.com if interested.


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

* Trivial to hack out the test suite if a test if failing
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


RTL frontend
------------

* Ability to write unit tests for individual RTL passes.

  * Parse RTL dumps going into one pass, run just that one pass.

  * Likely to be very target-specific

* Under development, hopefully ready by close of GCC 7 stage 1.
