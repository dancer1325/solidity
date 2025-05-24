* goal
  * Solidity Source File's layout

* Solidity Source files
  * can contain ARBITRARY number of
    * `contract`
    * `import`
    * `pragma`
    * `using for`
    * `struct`
    * `enum`
    * `function`
    * `error`
    * `constant variable`

# SPDX License Identifier -- `SPDX-License-Identifier` --

* [website](https://spdx.org)
* machine-readable
* recommendations
  * 👀place as comment | EACH top file👀
    * Reason: 🧠| touch source code, avoid copyright legal problems 🧠
    * ALSO recognized the comment | ANYWHERE | file
  * if you do NOT want to specify it OR NOT open-source -> use `UNLICENSED`
    * ❌NOT [ALLOWED by SPDX](https://spdx.org/licenses/)❌
    * != `UNLICENSE` == grant ALL RIGHTS | EVERYONE

* _Example:_
    ```solidity
    // SPDX-License-Identifier: MIT
    ```

* Solidity compiler
  * ❌NOT validate that license is [ALLOWED by SPDX](https://spdx.org/licenses/)❌
  * 👀if it's specified -> included | [`bytecode metadata`](metadata.md)👀
  * follows [npm recommendation](https://docs.npmjs.com/cli/v7/configuring-npm/package-json#license)

* ❌ADDITIONAL licensing requirements / NOT supplied by `SPDX-License-Identifier`❌
  * specific license header
  * original copyright holder
  * _Example:_
    ```
    /*
    * Copyright (c) 2023 Original Copyright Holder Name
    *
    * Licensed under [License Name] (e.g., MIT, Apache 2.0, etc.)
    * [Optional: Brief description or terms]
    * [Optional: Link to full license text]
    */
    ```

# Pragmas

* `pragma`
  * keyword /
    * 👀INDIRECTLY enable CERTAIN
      * compiler features or
      * compiler checks👀
  * ⚠️local | source file⚠️
    * if you want to enable it | WHOLE project -> add | ALL source files
    * if you `import` ANOTHER file -> pragma NOT AUTOMATICALLY applied | imported file

## Version Pragma -- `pragma solidity` --

* ❌NOT
  * change the version of the compiler❌
  * enable or disable compiler's features❌
* read by the compiler -- to check -- if they match
  * == 👀version pragma -- matches with -- compiler version 👀
  * if they do NOT match -> compiler issues an error

* recommendations
  * specify it

* ``pragma solidity versionToSpecify;``
  * `versionToSpecify` follows [semver](https://docs.npmjs.com/cli/v6/using-npm/semver)
  * _Example:_
    ```solidity
    ...
    pragma solidity ^0.5.2;
    ...
    ```
    * source file
      * does NOT work | compiler
        * earlier than version 0.5.2,
        * from version 0.6.0
      * works | compiler `v0.5.z`

## ABI Coder Pragma -- `pragma abicoder` --

* == ABI encoder + ABI decoder

* ALLOWED ones
  * ``pragma abicoder v1``
  * ``pragma abicoder v2``

* `pragma abicoder v2`
  * == NEW ABI coder
  * allows you
    * encode & decode ARBITRARILY nested arrays & structs
  * vs `v1`
    * 's supported types == `v1`'s strict superset
      * MORE types
    * MORE extensive validation & safety checks
      * -> higher gas costs
  * |
    * Solidity 0.6.0,
      * non-experimental
    * | Solidity 0.7.4-,
      * if you set `pragma experimental ABIEncoderV2` -> ❌NOT possible select explicitly `v1`❌
    * Reason: 🧠v1 was the default🧠
    * Solidity 0.8.0,
      * enabled by default
        * ⚠️ALTHOUGH, you can specify v1 ⚠️

* Contracts /
  * use `pragma abicoder` -> can interact -- with -- ones / NOT use
    * WITHOUT limitations
  * NOT use `pragma abicoder` -> can interact -- with -- ones / use
    * requirements
      * ⚠️NON-``abicoder v2`` contract does NOT try -- to make -- calls / would require decoding types / ONLY supported -- by the -- NEW encoder⚠️
        * OTHERWISE, compiler detect it & issue an error
        * FAST SOLUTION: use ``pragma abicoder v2``

* apply |
  * ALL file's code
    * == if contract's  source file specifies ABI coder `v1` -> can contain code / uses v2
      * by inheriting -- from -- ANOTHER contract
      * requirements
        * NEW types are used
          * ONLY internally
          * NOT | external function signatures

## Experimental Pragma == v2

* uses
  * enable
    * compiler's features / NOT yet enabled by default,
    * language's features / NOT yet enabled by default

* CURRENTLY supported
  * [ABIEncoderV2](#ABIEncoderV2)
  * [SMTChecker](#SMTChecker)

### ABIEncoderV2

* TODO: Because the ABI coder v2 is not considered experimental anymore,
it can be selected via ``pragma abicoder v2`` (please see above)
since Solidity 0.7.4.

.. index:: ! pragma; SMTChecker
.. _smt_checker:

### SMTChecker

* This component has to be enabled when the Solidity compiler is built
and therefore it is not available in all Solidity binaries.
* The :ref:`build instructions<smt_solvers_build>` explain how to activate this option
* It is activated for the Ubuntu PPA releases in most versions,
but not for the Docker images, Windows binaries or the
statically-built Linux binaries
* It can be activated for solc-js via the
`smtCallback <https://github.com/ethereum/solc-js#example-usage-with-smtsolver-callback>`_ if you have an SMT solver
installed locally and run solc-js via node (not via the browser).

* If you use ``pragma experimental SMTChecker;``, then you get additional
:ref:`safety warnings<formal_verification>` which are obtained by querying an
SMT solver
* The component does not yet support all features of the Solidity language and
likely outputs many warnings
* In case it reports unsupported features, the
analysis may not be fully sound.

## Importing other Source Files

### Syntax and Semantics

Solidity supports import statements to help modularise your code that
are similar to those available in JavaScript
(from ES6 on). However, Solidity does not support the concept of
a `default export <https://developer.mozilla.org/en-US/docs/web/javascript/reference/statements/export#description>`_.

At a global level, you can use import statements of the following form:

.. code-block:: solidity

    import "filename";

The ``filename`` part is called an *import path*.
This statement imports all global symbols from "filename" (and symbols imported there) into the
current global scope (different than in ES6 but backwards-compatible for Solidity).
This form is not recommended for use, because it unpredictably pollutes the namespace.
If you add new top-level items inside "filename", they automatically
appear in all files that import like this from "filename". It is better to import specific
symbols explicitly.

The following example creates a new global symbol ``symbolName`` whose members are all
the global symbols from ``"filename"``:

.. code-block:: solidity

    import * as symbolName from "filename";

which results in all global symbols being available in the format ``symbolName.symbol``.

A variant of this syntax that is not part of ES6, but possibly useful is:

.. code-block:: solidity

  import "filename" as symbolName;

which is equivalent to ``import * as symbolName from "filename";``.

If there is a naming collision, you can rename symbols while importing. For example,
the code below creates new global symbols ``alias`` and ``symbol2`` which reference
``symbol1`` and ``symbol2`` from inside ``"filename"``, respectively.

.. code-block:: solidity

    import {symbol1 as alias, symbol2} from "filename";

### Import Paths

In order to be able to support reproducible builds on all platforms, the Solidity compiler has to
abstract away the details of the filesystem where source files are stored.
For this reason import paths do not refer directly to files in the host filesystem.
Instead the compiler maintains an internal database (*virtual filesystem* or *VFS* for short) where
each source unit is assigned a unique *source unit name* which is an opaque and unstructured identifier.
The import path specified in an import statement is translated into a source unit name and used to
find the corresponding source unit in this database.

Using the :ref:`Standard JSON <compiler-api>` API it is possible to directly provide the names and
content of all the source files as a part of the compiler input.
In this case source unit names are truly arbitrary.
If, however, you want the compiler to automatically find and load source code into the VFS, your
source unit names need to be structured in a way that makes it possible for an :ref:`import callback
<import-callback>` to locate them.
When using the command-line compiler the default import callback supports only loading source code
from the host filesystem, which means that your source unit names must be paths.
Some environments provide custom callbacks that are more versatile.
For example the `Remix IDE <https://remix.ethereum.org/>`_ provides one that
lets you `import files from HTTP, IPFS and Swarm URLs or refer directly to packages in NPM registry
<https://remix-ide.readthedocs.io/en/latest/import.html>`_.

For a complete description of the virtual filesystem and the path resolution logic used by the
compiler see :ref:`Path Resolution <path-resolution>`.

.. index:: ! comment, natspec

# Comments

Single-line comments (``//``) and multi-line comments (``/*...*/``) are possible.

.. code-block:: solidity

    // This is a single-line comment.

    /*
    This is a
    multi-line comment.
    */

.. note::
  A single-line comment is terminated by any unicode line terminator
  (LF, VF, FF, CR, NEL, LS or PS) in UTF-8 encoding. The terminator is still part of
  the source code after the comment, so if it is not an ASCII symbol
  (these are NEL, LS and PS), it will lead to a parser error.

Additionally, there is another type of comment called a NatSpec comment,
which is detailed in the :ref:`style guide<style_guide_natspec>`. They are written with a
triple slash (``///``) or a double asterisk block (``/** ... */``) and
they should be used directly above function declarations or statements.
