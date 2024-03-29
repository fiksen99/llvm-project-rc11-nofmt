========================================
Clang 11.0.0 (In-Progress) Release Notes
========================================

.. contents::
   :local:
   :depth: 2

Written by the `LLVM Team <https://llvm.org/>`_

.. warning::

   These are in-progress notes for the upcoming Clang 11 release.
   Release notes for previous releases can be found on
   `the Download Page <https://releases.llvm.org/download.html>`_.

Introduction
============

This document contains the release notes for the Clang C/C++/Objective-C
frontend, part of the LLVM Compiler Infrastructure, release 11.0.0. Here we
describe the status of Clang in some detail, including major
improvements from the previous release and new feature work. For the
general LLVM release notes, see `the LLVM
documentation <https://llvm.org/docs/ReleaseNotes.html>`_. All LLVM
releases may be downloaded from the `LLVM releases web
site <https://llvm.org/releases/>`_.

For more information about Clang or LLVM, including information about the
latest release, please see the `Clang Web Site <https://clang.llvm.org>`_ or the
`LLVM Web Site <https://llvm.org>`_.

Note that if you are reading this file from a Git checkout or the
main Clang web page, this document applies to the *next* release, not
the current one. To see the release notes for a specific release, please
see the `releases page <https://llvm.org/releases/>`_.

What's New in Clang 11.0.0?
===========================

Some of the major new features and improvements to Clang are listed
here. Generic improvements to Clang as a whole or to its underlying
infrastructure are described first, followed by language-specific
sections with improvements to Clang's support for those languages.

Major New Features
------------------

- ...

Improvements to Clang's diagnostics
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

- -Wpointer-to-int-cast is a new warning group. This group warns about C-style
  casts of pointers to a integer type too small to hold all possible values.

- -Wuninitialized-const-reference is a new warning controlled by 
  -Wuninitialized. It warns on cases where uninitialized variables are passed
  as const reference arguments to a function.

Non-comprehensive list of changes in this release
-------------------------------------------------

- For the ARM target, C-language intrinsics are now provided for the full Arm
  v8.1-M MVE instruction set. ``<arm_mve.h>`` supports the complete API defined
  in the Arm C Language Extensions.

- For the ARM target, C-language intrinsics ``<arm_cde.h>`` for the CDE
  instruction set are now provided.

- clang adds support for a set of  extended integer types (``_ExtInt(N)``) that
  permit non-power of 2 integers, exposing the LLVM integer types. Since a major
  motivating use case for these types is to limit 'bit' usage, these types don't
  automatically promote to 'int' when operations are done between two
  ``ExtInt(N)`` types, instead math occurs at the size of the largest
  ``ExtInt(N)`` type.

- Users of UBSan, PGO, and coverage on Windows will now need to add clang's
  library resource directory to their library search path. These features all
  use runtime libraries, and Clang provides these libraries in its resource
  directory. For example, if LLVM is installed in ``C:\Program Files\LLVM``,
  then the profile runtime library will appear at
  ``C:\Program Files\LLVM\lib\clang\11.0.0\lib\windows\clang_rt.profile-x86_64.lib``.
  To ensure that the linker can find the appropriate library, users should pass
  ``/LIBPATH:C:\Program Files\LLVM\lib\clang\11.0.0\lib\windows`` to the
  linker. If the user links the program with the ``clang`` or ``clang-cl``
  drivers, the driver will pass this flag for them.

- Clang's profile files generated through ``-fprofile-instr-generate`` are using
  a fixed hashing algorithm that prevents some collision when loading
  out-of-date profile informations. Clang can still read old profile files.

New Compiler Flags
------------------

- -fstack-clash-protection will provide a protection against the stack clash
  attack for x86, s390x and ppc64 architectures through automatic probing of
  each page of allocated stack.

- -ffp-exception-behavior={ignore,maytrap,strict} allows the user to specify
  the floating-point exception behavior. The default setting is ``ignore``.

- -ffp-model={precise,strict,fast} provides the user an umbrella option to
  simplify access to the many single purpose floating point options. The default
  setting is ``precise``.

- The default module cache has moved from /tmp to a per-user cache directory.
  By default, this is ~/.cache but on some platforms or installations, this
  might be elsewhere. The -fmodules-cache-path=... flag continues to work.

- -fpch-instantiate-templates tries to instantiate templates already while
  generating a precompiled header. Such templates do not need to be
  instantiated every time the precompiled header is used, which saves compile
  time. This may result in an error during the precompiled header generation
  if the source header file is not self-contained. This option is enabled
  by default for clang-cl.

- -fpch-codegen and -fpch-debuginfo generate shared code and/or debuginfo
  for contents of a precompiled header in a separate object file. This object
  file needs to be linked in, but its contents do not need to be generated
  for other objects using the precompiled header. This should usually save
  compile time. If not using clang-cl, the separate object file needs to
  be created explicitly from the precompiled header.
  Example of use:

  .. code-block:: console

    $ clang++ -x c++-header header.h -o header.pch -fpch-codegen -fpch-debuginfo
    $ clang++ -c header.pch -o shared.o
    $ clang++ -c source.cpp -o source.o -include-pch header.pch
    $ clang++ -o binary source.o shared.o

  - Using -fpch-instantiate-templates when generating the precompiled header
    usually increases the amount of code/debuginfo that can be shared.
  - In some cases, especially when building with optimizations enabled, using
    -fpch-codegen may generate so much code in the shared object that compiling
    it may be a net loss in build time.
  - Since headers may bring in private symbols of other libraries, it may be
    sometimes necessary to discard unused symbols (such as by adding
    -Wl,--gc-sections on ELF platforms to the linking command, and possibly
    adding -fdata-sections -ffunction-sections to the command generating
    the shared object).

Deprecated Compiler Flags
-------------------------

The following options are deprecated and ignored. They will be removed in
future versions of Clang.

- ...

Modified Compiler Flags
-----------------------

- -fno-common has been enabled as the default for all targets.  Therefore, C
  code that uses tentative definitions as definitions of a variable in multiple
  translation units will trigger multiple-definition linker errors. Generally,
  this occurs when the use of the ``extern`` keyword is neglected in the
  declaration of a variable in a header file. In some cases, no specific
  translation unit provides a definition of the variable. The previous
  behavior can be restored by specifying ``-fcommon``.
- -Wasm-ignored-qualifier (ex. `asm const ("")`) has been removed and replaced
  with an error (this matches a recent change in GCC-9).
- -Wasm-file-asm-volatile (ex. `asm volatile ("")` at global scope) has been
  removed and replaced with an error (this matches GCC's behavior).
- Duplicate qualifiers on asm statements (ex. `asm volatile volatile ("")`) no
  longer produces a warning via -Wduplicate-decl-specifier, but now an error
  (this matches GCC's behavior).
- The deprecated argument ``-f[no-]sanitize-recover`` has changed to mean
  ``-f[no-]sanitize-recover=all`` instead of
  ``-f[no-]sanitize-recover=undefined,integer`` and is no longer deprecated.
- The argument to ``-f[no-]sanitize-trap=...`` is now optional and defaults to
  ``all``.
- ``-fno-char8_t`` now disables the ``char8_t`` keyword, not just the use of
  ``char8_t`` as the character type of ``u8`` literals. This restores the
  Clang 8 behavior that regressed in Clang 9 and 10.
- -print-targets has been added to print the registered targets.

New Pragmas in Clang
--------------------

- ...

Attribute Changes in Clang
--------------------------

- Attributes can now be specified by clang plugins. See the
  `Clang Plugins <ClangPlugins.html#defining-attributes>`_ documentation for
  details.

Windows Support
---------------

C Language Changes in Clang
---------------------------

- The default C language standard used when `-std=` is not specified has been
  upgraded from gnu11 to gnu17.

- Clang now supports the GNU C extension `asm inline`; it won't do anything
  *yet*, but it will be parsed.

- ...

C++ Language Changes in Clang
-----------------------------

- Clang now implements a restriction on giving non-C-compatible anonymous
  structs a typedef name for linkage purposes, as described in C++ committee
  paper `P1766R1 <http://wg21.link/p1766r1>`. This paper was adopted by the
  C++ committee as a Defect Report resolution, so it is applied retroactively
  to all C++ standard versions. This affects code such as:

  .. code-block:: c++

    typedef struct {
      int f() { return 0; }
    } S;

  Previous versions of Clang rejected some constructs of this form
  (specifically, where the linkage of the type happened to be computed
  before the parser reached the typedef name); those cases are still rejected
  in Clang 11. In addition, cases that previous versions of Clang did not
  reject now produce an extension warning. This warning can be disabled with
  the warning flag ``-Wno-non-c-typedef-for-linkage``.

  Affected code should be updated to provide a tag name for the anonymous
  struct:

  .. code-block:: c++

    struct S {
      int f() { return 0; }
    };

  If the code is shared with a C compilation (for example, if the parts that
  are not C-compatible are guarded with ``#ifdef __cplusplus``), the typedef
  declaration should be retained, but a tag name should still be provided:

  .. code-block:: c++

    typedef struct S {
      int f() { return 0; }
    } S;

C++1z Feature Support
^^^^^^^^^^^^^^^^^^^^^

...

Objective-C Language Changes in Clang
-------------------------------------

OpenCL C Language Changes in Clang
----------------------------------

...

ABI Changes in Clang
--------------------

OpenMP Support in Clang
-----------------------

- ...

CUDA Support in Clang
---------------------

- ...

Internal API Changes
--------------------

These are major API changes that have happened since the 10.0.0 release of
Clang. If upgrading an external codebase that uses Clang as a library,
this section should help get you past the largest hurdles of upgrading.

- ``RecursiveASTVisitor`` no longer calls separate methods to visit specific
  operator kinds. Previously, ``RecursiveASTVisitor`` treated unary, binary,
  and compound assignment operators as if they were subclasses of the
  corresponding AST node. For example, the binary operator plus was treated as
  if it was a ``BinAdd`` subclass of the ``BinaryOperator`` class: during AST
  traversal of a ``BinaryOperator`` AST node that had a ``BO_Add`` opcode,
  ``RecursiveASTVisitor`` was calling the ``TraverseBinAdd`` method instead of
  ``TraverseBinaryOperator``. This feature was contributing a non-trivial
  amount of complexity to the implementation of ``RecursiveASTVisitor``, it was
  used only in a minor way in Clang, was not tested, and as a result it was
  buggy. Furthermore, this feature was creating a non-uniformity in the API.
  Since this feature was not documented, it was quite difficult to figure out
  how to use ``RecursiveASTVisitor`` to visit operators.

  To update your code to the new uniform API, move the code from separate
  visitation methods into methods that correspond to the actual AST node and
  perform case analysis based on the operator opcode as needed:

  * ``TraverseUnary*() => TraverseUnaryOperator()``
  * ``WalkUpFromUnary*() => WalkUpFromUnaryOperator()``
  * ``VisitUnary*() => VisiUnaryOperator()``
  * ``TraverseBin*() => TraverseBinaryOperator()``
  * ``WalkUpFromBin*() => WalkUpFromBinaryOperator()``
  * ``VisitBin*() => VisiBinaryOperator()``
  * ``TraverseBin*Assign() => TraverseCompoundAssignOperator()``
  * ``WalkUpFromBin*Assign() => WalkUpFromCompoundAssignOperator()``
  * ``VisitBin*Assign() => VisiCompoundAssignOperator()``

Build System Changes
--------------------

These are major changes to the build system that have happened since the 10.0.0
release of Clang. Users of the build system should adjust accordingly.

- clang-tidy and clang-include-fixer are no longer compiled into libclang by
  default. You can set ``LIBCLANG_INCLUDE_CLANG_TOOLS_EXTRA=ON`` to undo that,
  but it's expected that that setting will go away eventually. If this is
  something you need, please reach out to the mailing list to discuss possible
  ways forward.

AST Matchers
------------

- ...

clang-format
------------

- Option ``IndentExternBlock`` has been added to optionally apply indenting inside ``extern "C"`` and ``extern "C++"`` blocks.

- ``IndentExternBlock`` option accepts ``AfterExternBlock`` to use the old behavior, as well as Indent and NoIndent options, which map to true and false, respectively.

  .. code-block:: c++

    Indent:                       NoIndent:
     #ifdef __cplusplus          #ifdef __cplusplus
     extern "C" {                extern "C++" {
     #endif                      #endif

          void f(void);          void f(void);

     #ifdef __cplusplus          #ifdef __cplusplus
     }                           }
     #endif                      #endif

- Option ``IndentCaseBlocks`` has been added to support treating the block
  following a switch case label as a scope block which gets indented itself.
  It helps avoid having the closing bracket align with the switch statement's
  closing bracket (when ``IndentCaseLabels`` is ``false``).

  .. code-block:: c++

    switch (fool) {                vs.     switch (fool) {
    case 1:                                case 1: {
      {                                      bar();
         bar();                            } break;
      }                                    default: {
      break;                                 plop();
    default:                               }
      {                                    }
        plop();
      }
    }

- Option ``ObjCBreakBeforeNestedBlockParam`` has been added to optionally apply
  linebreaks for function arguments declarations before nested blocks.

- Option ``InsertTrailingCommas`` can be set to ``TCS_Wrapped`` to insert
  trailing commas in container literals (arrays and objects) that wrap across
  multiple lines. It is currently only available for JavaScript and disabled by
  default (``TCS_None``).

- Option ``BraceWrapping.BeforeLambdaBody`` has been added to manage lambda
  line break inside function parameter call in Allman style.

  .. code-block:: c++

      true:
      connect(
        []()
        {
          foo();
          bar();
        });

      false:
      connect([]() {
          foo();
          bar();
        });

- Option ``AlignConsecutiveBitFields`` has been added to align bit field
  declarations across multiple adjacent lines

  .. code-block:: c++

      true:
        bool aaa  : 1;
        bool a    : 1;
        bool bb   : 1;

      false:
        bool aaa : 1;
        bool a : 1;
        bool bb : 1;

- Option ``BraceWrapping.BeforeWhile`` has been added to allow wrapping
  before the ```while`` in a do..while loop. By default the value is (``false``)

  In previous releases ``IndentBraces`` implied ``BraceWrapping.BeforeWhile``.
  If using a Custom BraceWrapping style you may need to now set
  ``BraceWrapping.BeforeWhile`` to (``true``) to be explicit.

  .. code-block:: c++

      true:
      do {
        foo();
      }
      while(1);

      false:
      do {
        foo();
      } while(1);

libclang
--------

- ...

Static Analyzer
---------------

- ...

.. _release-notes-ubsan:

Undefined Behavior Sanitizer (UBSan)
------------------------------------

Core Analysis Improvements
==========================

- ...

New Issues Found
================

- ...

Python Binding Changes
----------------------

The following methods have been added:

-  ...

Significant Known Problems
==========================

Additional Information
======================

A wide variety of additional information is available on the `Clang web
page <https://clang.llvm.org/>`_. The web page contains versions of the
API documentation which are up-to-date with the Git version of
the source code. You can access versions of these documents specific to
this release by going into the "``clang/docs/``" directory in the Clang
tree.

If you have any questions or comments about Clang, please feel free to
contact us via the `mailing
list <https://lists.llvm.org/mailman/listinfo/cfe-dev>`_.
