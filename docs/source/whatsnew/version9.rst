============
 9.x Series
============

.. _version 9.4:

IPython 9.4
===========

Featuring ``%autoreload``, ``%whos``, ``%%script``, ``%%time`` magic improvements, along with a fix for use of list comprehensions and generators in the interactive debugger (and ipdb).

- :ghpull:`14922` Improved reloading of decorated functions when using ``%autoreload``
- :ghpull:`14872` Do not always import all variables with ``%autoreload 3``
- :ghpull:`14906` Changed behaviour of ``%time`` magic to always interrupt execution on exception and always show execution time
- :ghpull:`14926` Support data frames, series, and objects with ``__len__`` in the ``%whos`` magic
- :ghpull:`14933` List comprehensions and generators now work reliably in debugger on all supported Python versions
- :ghpull:`14931` Fix streaming multi-byte Unicode characters in the ``%script`` magic and its derivatives

The ``%time`` magic no longer swallows exceptions raised by the measured code, and always prints the time of execution. If you wish the execution to continue after measuring time to execute code that is meant to raise an exception, pass the new ``--no-raise-error`` flag.
The ``--no-raise-error`` flag does not affect ``KeyboardInterrupt`` as this exception is used to signal intended interruption of execution flow.

Previously the debugger (ipdb) evaluation of list comprehensions and generators could fail with ``NameError`` due to generator implementation detail in CPython. This was recently fixed in Python 3.13. Because IPython is often used for interactive debugging, this release includes a backport of that fix, providing users who cannot yet update from Python 3.11 or 3.12 with a smoother debugging experience.

The ``%autoreload`` magic is now more reliable. The behaviour around decorators has been improved and `%autoreload 3` no longer imports all symbols when reloading the module, however, the heuristic used to determine which symbols to reload can sometimes lead to addition of imports from non-evaluated code branches, see `issue #14934 <https://github.com/ipython/ipython/issues/14934>`__.

.. _version 9.3:

IPython 9.3
===========

This release includes improvements to the tab and LLM completer, along with typing improvements:

- :ghpull:`14911` Implement auto-import and evaluation policy overrides
- :ghpull:`14910` Eliminate startup delay when LLM completion provider is configured
- :ghpull:`14898` Fix attribute completion for expressions with comparison operators
- :ghpull:`14908` Fix typing of `error_before_exec`, enhance ``mypy`` coverage

Notably, the native completer can now suggest attribute completion on not-yet-imported modules.
This is particularly useful when writing code which includes an import and the use of the imported
module in the same line or in the same cell; the default implementation does not insert
the imported module into the user namespace, for which an actual execution is required.

The auto-import of modules by completer is turned off and requires opting-in using
a new :std:configtrait:`Completer.policy_overrides` traitlet.
To enable auto-import on completion specify:

.. code-block::

    ipython --Completer.policy_overrides='{"allow_auto_import": True}' --Completer.use_jedi=False

This change aligns the capability of both jedi-powered and the native completer.
The function used for auto-import can be configured using :std:configtrait:`Completer.auto_import_method` traitlet.

.. _version 9.2:

IPython 9.2
===========

This is a small release with minor changes in the context passed to the LLM completion
provider along few other bug fixes and documentation improvements:

- :ghpull:`14890` Fixed interruption of ``%%time`` and ``%%debug`` magics
- :ghpull:`14877` Removed spurious empty lines from ``prefix`` passed to LLM, and separated part after cursor into the ``suffix``
- :ghpull:`14876` Fixed syntax warning in Python 3.14 (remove return from finally block)
- :ghpull:`14887` Documented the recommendation to use ``ipykernel.embed.embed_kernel()`` over ``ipython.embed``.

.. _version 9.1:

IPython 9.1
===========

This is a small release that introduces enhancements to ``%notebook`` and ``%%timeit`` magics,
and a number of bug fixes related to colors/formatting, performance, and completion.

``%notebook`` saves outputs
---------------------------

The ``%notebook`` magic can be used to create a Jupyter notebook from the
commands executed in the current IPython session (since the interpreter startup).

Prior to IPython 9.1, the resulting notebook did not include the outputs,
streams, or exceptions. IPython 9.1 completes the implementation of this
magic allowing for an easier transition from an interactive IPython session
to a Jupyter notebook.

To capture streams (stdio/stderr), IPython temporarily swaps the `write`
method of the active stream class during code execution. This ensures
compatibility with ipykernel which swaps the entire stream implementation
and requires it to remain an instance of ``IOStream`` subclass.
If this leads to undesired behaviour in any downstream applications,
your feedback and suggestions would be greatly appreciated.


``%%timeit -v`` argument
------------------------

New ``-v`` argument allows users to save the timing result
directly to a specified variable, e.g.

.. code::

   %%timeit -v timing_result
   2**32


Completer improvements
----------------------

The LLM-based completer will now receive the request number for each subsequent
execution.

The tab completer used when jedi is turned off now correctly completes
variables in lines where it previously was incorrectly attempting to complete
attributes due to simplistic context detection based on the presence of a dot.

Thanks
------

A big thank you to everyone who contributed towards the 9.1 release,
including new contributors: @Darshan808, @kwinkunks, @carschandler,
returning contributors (shout out to @wjandrea!), and of course
@Carreau whom I would like to thank for the guidance in the preparation
of this release and stewardship of IPython over the years - Mike.

As usual, you can find the full list of PRs on GitHub under `the 9.1
<https://github.com/ipython/ipython/milestone/142?closed=1>`__ milestone.


.. _version90:

IPython 9.0
===========

Welcome to IPython 9.0. As with any version of IPython before this release, it
should not be majorly different from the previous version, at least on the surface. 
We still hope you can upgrade as soon as possible and look forward to your feedback.

I take the opportunity of this new release to remind you that IPython is
governed by the `Jupyter code of conduct
<https://jupyter.org/governance/conduct/code_of_conduct.html>`_. And that even
beyond so we strive to be an inclusive, accepting and progressive community,
Here is a relevant extract from the COC.

    We strive to be a community that welcomes and supports people of all backgrounds
    and identities. This includes, but is not limited to, members of any race,
    ethnicity, culture, national origin, color, immigration status, social and
    economic class, educational level, sex, sexual orientation, gender identity and
    expression, age, physical appearance, family status, technological or
    professional choices, academic discipline, religion, mental ability, and
    physical ability.


As a short overview of the changes in 9.0, we have over 100 PRs merged since 8.x,
many of which are refactors, cleanups and simplifications.

 - (optional) LLM integration in the CLI. 
 - Complete rewrite of color and theme handling, which now supports more colors and symbols. 
 - Move tests out of tree in the wheel with a massive reduction in file size. 
 - Tips at startup
 - Removal of (almost) all deprecated functionalities and options.
 - Stricter and more stable codebase.


Removal and deprecation
-----------------------

I am not going to list the removals and deprecations, but anything deprecated since before IPython 8.16 is gone, 
including many shim modules and indirect imports that would just re-expose IPykernel, qtconsole, etc. 

A number of new deprecations have been added (run your test suites with `-Werror`), as those will be removed in the future. 


Color and theme rewrite
-----------------------

IPython's color handling had grown many options through the years, and it was
quite entrenched in the codebase, directly emitting ansi escape sequences deep
in traceback printing and other places. 

This made developing new color schemes difficult, and limited us to the 16 colors
of the original ansi standard defined by your terminal. 

Syntax highlighting was also inconsistent, and not all syntax elements were
always using the same theme.

Using (style, token) pairs 
~~~~~~~~~~~~~~~~~~~~~~~~~~

Starting with 9.0, the color and theme handling has been rewritten, and
internally all the printing is done by yielding pairs of Style and token objects
(compatible with pygments and prompt_toolkit), then as much as possible, IPython
formats these objects at the last moment, using the current theme.

256-bit colors and unicode symbols
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This means that new themes can now use all of pygments's color names and
functionalities, and you can define for each token style, the foreground,
background, underline, bold, italic and likely a few other options. 

In addition, themes now provide a number of `symbols`, that can be used when
rendering traceback or debugger prompts. This let you customize the appearance a
bit more. For example, instead of using dash and greater-than sign, The arrow
pointing the current frame can actually use horizontal line and right arrow
unicode symbol, for a more refined experience.


New themes using colors and symbols
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

All the existing themes (Linux, LightBG, Neutral and NoColor) should not see any
changes, but I added two new *pride themes*, that show the use of 256bits colors
and unicode symbols. I'm not a designer, so feel free to suggest updates and new
themes to add. 

Themes  currently still require writing a bit of Python, but I hope to get
contributions for IPython to be able to load them from text files, for easier
redistribution.

Tips at startup
---------------

IPython now displays a few tips at startup (1 line), to help you discover new features.
All those are in the codebase, and can be displayed randomly or based on date. 
You can disable it via a configuration option or the ``--no-tips`` flag. 

Please contribute more tips by sending pull requests!

Out-of-tree tests
-----------------

And more generally I have changed the folder structure and what is packaged in
the wheel to reduce the file size. The wheel is down from 825kb to 590kb
(-235kb) which is about a 28% reduction. This should help when you run IPython
via Pyodide – when your browser needs to download it.

According to https://pypistats.org/packages/ipython, IPython is downloaded about
13 million times per week, so this should reduce PyPI bandwidth by about 2Tb each
week, which is small compared to the total download, but still, trying to reduce
resource usage is a worthy goal.

Integration with Jupyter-AI LLM
-------------------------------

This feature allow IPython CLI to make use of Jupyter-AI provider to use LLM for
suggestion, and completing the current text. Unlike many features
of IPython this is disabled by default, and need several configuration options to
be set to work:

 - Choose a provider in ``jupyter-ai`` and set it as default one:
   ``c.TerminalInteractiveShell.llm_provider_class = <fully qualified path>``
   You likely need to setup your provider with API key or other things.
 - Choose and available shortcut (I'll take ``Ctrl-Q`` as an example) and bind
   to trigger ``llm_autosuggestion`` only while typing.

.. code::
   
   c.TerminalInteractiveShell.shortcuts = [
        {
            "new_keys": ["c-q"],
            "command": "IPython:auto_suggest.llm_autosuggestion",
            "new_filter": "navigable_suggestions & default_buffer_focused",
            "create": True,
        },
    ]

See :ref:`llm_suggestions` for more.

Thanks as well to the `D. E. Shaw group <https://deshaw.com/>`_ for sponsoring
this work.


For something completely different
----------------------------------

Ruth Bader Ginsburg 1933-2020 was an American lawyer and jurist who served on
the Supreme Court of the United States. Ginsburg spent much of her legal career
as an advocate for gender equality, women's rights, abortion rights, and religious
freedom.

Thanks
------

Thanks to everyone who helped with the 9.0 release and working toward 9.0.

As usual you can find the full list of PRs on GitHub under `the 9.0
<https://github.com/ipython/ipython/milestone/138?closed=1>`__ milestone.



