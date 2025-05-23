.. index:: ! contract;abstract, ! abstract contract

.. _abstract-contract:

******************
Abstract Contracts
******************

* 👀use cases / mark -- as -- abstract contract👀
  * NECESSARY
    * \>=1 their functions are NOT implemented
    * their functions do NOT provide arguments / ALL their base contract constructors
    * contract inherits -- from an -- abstract contract & NO implement ALL NON-implemented functions -- by -- overriding
  * POSSIBLE
    * NOT DIRECTLY create the contract

* vs :ref:`interfaces`
  * SIMILAR
  * `interfaces`
    * declarations MORE limited

* ``abstract``

* can NOT
  * DIRECTLY be instantiated
  * override an implemented virtual function -- with an -- unimplemented one

* _Example:_
  * _Example1:_
      ```solidity
      // SPDX-License-Identifier: GPL-3.0
      pragma solidity >=0.6.0 <0.9.0;

      abstract contract Feline {
          // NEXT function is declared, BUT NOT implemented -> REQUIRED to declare as abstract
          function utterance() public virtual returns (bytes32);
      }
      ```
  * _Example2:_ abstract contract -- used as a -- base class
    ```solidity
    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.6.0 <0.9.0;

    abstract contract Feline {
        function utterance() public pure virtual returns (bytes32);
    }

    contract Cat is Feline {
        function utterance() public pure override returns (bytes32) { return "miaow"; }
    }
    ```

* function WITHOUT implementation
  * != :ref:`Function Type <function_types>`
  * _Example:_
    ```solidity
    # function declaration  ==   function WITHOUT implementation
    function foo(address) external returns (address);

    # variable declaration / 's type == function type
    function(address) external returns (address) foo;
    ```

* uses
  * define methods
    * == interface's uses

* allows
  * decoupling contract's definition -- from -- contract's implementation
    * ->
      * provide BETTER extensibility & self-documentation
      * facilitate patterns
        * _Example:_ `Template method <https://en.wikipedia.org/wiki/Template_method_pattern>`_
      * remove code duplication
