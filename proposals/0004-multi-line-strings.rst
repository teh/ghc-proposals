.. proposal-number:: Leave blank. This will be filled in when the proposal is
                     accepted.

.. trac-ticket:: Leave blank. This will eventually be filled with the Trac
                 ticket number which will track the progress of the
                 implementation of the feature.

.. implemented:: Leave blank. This will be filled in with the first GHC version which
                 implements the described feature.

.. highlight:: haskell

This proposal is `discussed at this pull requst <https://github.com/ghc-proposals/ghc-proposals/pull/0>`_. **After creating the pull request, edit this file again, update the number in the link, and delete this bold sentence.**

.. contents::

Multi-line strings
==================

This proposal would add new syntax to allow writing multi-line strings more conveniently.


Motivation
----------

I often need to add small sections of text, e.g. an email template, a script or a SQL query to my files. My current options are:

* Intercalate a list of strings ``["line 1", "line 2"]`` with newlines.
* Use ``\..\`` syntax.
* Using e.g. ``<>`` to join strings wrapped over multiple lines.
* Using a ``QuasiQuoter`` like ``[str|...|]``.

Scala and Python have triple quotes ``"""``, Go and ES6 have ````` and in Rust all strings can be multiline.

The following example:

.. code-block:: haskell

    let foo =
      print fooText
        where fooText =
          """
            A parser for things
            Is a function from strings
            To lists of pairs
            Of things and strings
          """

would print without indentation, and without the empty first line:

.. code-block::

    A parser for things
    Is a function from strings
    To lists of pairs
    Of things and strings


Proposed Change Specification
-----------------------------

We introduce a new delimiter ``"""`` to delineate a multi-line string. It behaves as follows:

* The delimiter can be represented in the string by escaping one or all of the markers with ``\``.
* If the first line is only ``\n`` we remove it.
* We remove the same amount of whitespace indentation from each line, aligning the left-most line to have no indentation. The goal is to allow writing multi-line strings inline in indented code, and still have them start at position 0 in the output.
* The literal would be parsed the same as a normal ``"``-delimited string, and ``IsString`` applies.

This behaviour is strongly inspired by ``''`` in the Nix language.


Effect and Interactions
-----------------------

We cannot use `''` like Nix because it's already used by template Haskell. We can use ``"""`` and parse it as a new token, with e.g. ``lex_multiline_string_tok``. I don't think this will conflict with anything, and we might not even need a new extension.


Costs and Drawbacks
-------------------

* It's introducing new syntax without adding expressive power.
* It'll make editor integrations harder - yet another feature to highlight.
* It might encourage stringly-typed programming because it makes creating strings easier.


Alternatives
------------

Apart from from the alternatives mentioned in the "Motivation" section we could also:

* Use the pre-processor to rewrite multi-line strings.
* Not do this


Unresolved questions
--------------------

* Does this feature carry its weight?
* Should we assume the same encoding as for other string literals, i.e. UTF-8?


Implementation Plan
-------------------

I'd give this a shot (https://github.com/teh).
