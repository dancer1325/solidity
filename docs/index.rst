Solidity
========

* Solidity
    * `language-influences.rst`_
    * uses
        * voting,
        * crowdfunding,
        * blind auctions,
        * MULTI-signature wallets

Getting Started
---------------

**1. Understand the Smart Contract Basics**

* `introduction-to-smart-contracts.md`_

**2. Get to Know Solidity**

* `solidity-by-example.rst`_

**3. Install the Solidity Compiler**

* `installing-solidity.md`_
* ALTERNATIVES
    * `Remix IDE <https://remix.ethereum.org>`_
        * == 👀web browser-based IDE / allows you, about smart contracts, 👀
            * write,
            * deploy
            * administer Solidity

**4. Learn More**

* `Ethereum Developer Resources <https://ethereum.org/en/developers/>`_
* `Ethereum StackExchange <https://ethereum.stackexchange.com/>`_
* `Gitter channel <https://gitter.im/ethereum/solidity>`_

Contents
========

:ref:`Keyword Index <genindex>`, :ref:`Search Page <search>`

.. toctree::
   :maxdepth: 2
   :caption: Basics

   introduction-to-smart-contracts.rst
   solidity-by-example.rst
   installing-solidity.rst

.. toctree::
   :maxdepth: 2
   :caption: Language Description

   layout-of-source-files.rst
   structure-of-a-contract.rst
   types.rst
   units-and-global-variables.rst
   control-structures.rst
   contracts.rst
   assembly.rst
   cheatsheet.rst
   grammar.rst

.. toctree::
   :maxdepth: 2
   :caption: Compiler

   using-the-compiler.rst
   analysing-compilation-output.rst
   ir-breaking-changes.rst

.. toctree::
   :maxdepth: 2
   :caption: Internals

   internals/layout_in_storage.rst
   internals/layout_in_memory.rst
   internals/layout_in_calldata.rst
   internals/variable_cleanup.rst
   internals/source_mappings.rst
   internals/optimizer.rst
   metadata.rst
   abi-spec.rst

.. toctree::
   :maxdepth: 2
   :caption: Advisory content

   security-considerations.rst
   bugs.rst
   050-breaking-changes.rst
   060-breaking-changes.rst
   070-breaking-changes.rst
   080-breaking-changes.rst

.. toctree::
   :maxdepth: 2
   :caption: Additional Material

   natspec-format.rst
   smtchecker.rst
   yul.rst
   path-resolution.rst

.. toctree::
   :maxdepth: 2
   :caption: Resources

   style-guide.rst
   common-patterns.rst
   resources.rst
   contributing.rst
   language-influences.rst
   brand-guide.rst
