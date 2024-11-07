How to analyze Haskell performance
==================================

When a Haskell application is slow or uses too much memory,
Cabal and the
`Haskell GHC compiler <https://downloads.haskell.org/ghc/latest/docs/users_guide/>`__
can help you understand why.

The main steps are to
1. let GHC insert performance measuring code into your application,
2. run the application to collect a performance report and
3. visualize that report.

The process of inserting performance measuring code and collecting performance information
is called "profiling" and Cabal makes this is easy to do:

Profiling CPU performance
-------------------------

First, build your application, e.g. ``my-app``, with profiling enabled:

.. code-block:: console

    $ cabal build --enable-profiling --profiling-detail=late exe:my-app

Under the hood this will pass the corresponding
`profiling compiler options <https://downloads.haskell.org/ghc/latest/docs/users_guide/profiling.html#compiler-options-for-profiling>`__
to GHC. With the additional Cabal flag ``--profiling-detail=late`` GHC is instructed to use
`"late-cost-center" profiling <https://downloads.haskell.org/ghc/latest/docs/users_guide/profiling.html#ghc-flag--fprof-late>`__
and insert performance measuring code only after important optimisations
have been applied to minimize the impact of profiling on performance.
See the Cabal section on :ref:`profiling options <profiling-options>` for more details.

Second, run the application with the necessary
`runtime system (RTS) options <https://downloads.haskell.org/ghc/latest/docs/users_guide/runtime_control.html>`__
to produce a profile report.

.. code-block:: console

    $ cabal list-bin exe:my-app
    /path/to/my-app
    $ /path/to/my-app +RTS -pj -RTS
    <program runs and finishes>

The report is written to a ``<app-name>.prof`` file, i.e. ``my-app.prof``, in the current directory.
With the RTS option ``-pj`` the app produces a
`"profile JSON" (pj) file report <https://downloads.haskell.org/ghc/latest/docs/users_guide/profiling.html#rts-flag--pj>`__.
Other report format options can be found in the
`GHC format documentation. <https://downloads.haskell.org/ghc/latest/docs/users_guide/profiling.html#time-and-allocation-profiling>`__.

Finally, load the profiling report file into a visualizer to look for performance bottlenecks.
One popular open-source
`flame graph <https://www.brendangregg.com/flamegraphs.html>`__
visualizer is `Speedscope <https://speedscope.app>`, which runs in the browser and comes with
an example.

Profiling your dependencies too
-------------------------------

The setup so far only profiles your main application, which is usually what you want.
This happens by default, because Cabal command line options only apply to local packages
and dependencies are usually not local.
However, the bottlenecks may be in your dependencies so you would want to profile those too.

To enable ``late``-cost-center profiling`` all packages/dependencies in your project,
add the following to your project’s ``cabal.project``` file:

.. code-block:: console

    package *
        profiling: true
        profiling-detail: late

So, rebuild your application with ``cabal build``:

.. code-block:: console

    $ cabal build exe:my-app
    Resolving dependencies...
    Build profile: -w ghc-9.10.1 -O1
    In order, the following will be built (use -v for more details):
     - base64-bytestring-1.2.1.0 (lib)  --enable-profiling (requires build)
     - cryptohash-sha256-0.11.102.1 (lib)  --enable-profiling (requires build)
     <...>

There's no need to pass ``--enable-profiling`` to the build command manually,
because it's already enabled in the project file (and seen in the build log).

Then run the application with the ``-pj`` RTS option as before.

You should now find more information in the profiling report ``my-app.prof``.

Further information on how to apply Cabal options can be in the
:ref:`Cabal options sections <package-configuration-options>`.

Further in-depth information on profiling with GHC can be found in the
`GHC profiling guide <https://downloads.haskell.org/ghc/latest/docs/users_guide/profiling.html>`__.
