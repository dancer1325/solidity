.. index:: ! visibility, external, public, private, internal

.. |visibility-caveat| replace::

* ``private`` OR ``internal``
  * prevents OTHER contracts -- from -- reading OR modifying the information
  * 👀STILL visible | whole world outside of the blockchain👀

.. _visibility-and-getters:

**********************
Visibility and Getters
**********************

State Variable Visibility
=========================

* ``public``
  * vs `internal`
    * 👀compiler AUTOMATICALLY generates [`getter functions`](#getter-functions)👀
      * ❌NOT generate `setter` functions❌
        * -> OTHER contracts can NOT -- DIRECTLY modify -- their values
  * uses
    * | SAME contract,
      * external access (e.g. ``this.x``) -- invokes the -- `getter`
      * internal access (e.g. ``x``) -- gets it, from the -- storage

* ``internal``
  * accessibility
    * |
      * SAME contract
      * derived contracts
      * ❌NOT EXTERNAL contracts ❌
  * 👀 default visibility level 👀

* ``private``
  * vs `internal`
    * accessibility
      * | ONLY SAME contract

.. warning::
    |visibility-caveat|

Function Visibility
===================

Furthermore, internal functions can be made inaccessible to derived contracts.

* ``external``
  * == part of the contract interface
    * == can be called
      * -- from --
        * OTHER contracts
        * via transactions
      * NOT internally
        * _Example:_ external function ``f``
          * ``f()`` does NOT work
          * ``this.f()`` works
  * create an actual EVM message call

* ``public``
  * == part of the contract interface
      and can be either called internally or via message calls.

* ``internal``
  * accessibility
    * |
      * SAME contract
      * derived contracts
      * ❌NOT EXTERNAL contracts ❌
  * can take parameters of internal types
    * _Example:_ mappings or storage references
    * Reason:🧠they are NOT exposed -- , through the contract's ABI, to the -- outside🧠
  * NOT create an actual EVM message call

* ``private``
  * vs `internal`
    * accessibility
      * | ONLY SAME contract

* location | declaration
  * AFTER state variables type
  * BETWEEN parameter list -- & -- return parameter list
  * _Example:_
    ```solidity
    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.16 <0.9.0;

    contract C {
        function f(uint a) private pure returns (uint b) { return a + 1; }
        function setData(uint a) internal { data = a; }
        uint public data;
    }
    ```

* _Example:_
  * Contract ``D``
    * -- can call -- ``c.getData()``
    * -- can NOT call -- ``c.f``
  * Contract ``E``
    * is derived -- from -- ``C``
      * -> can call ``compute``
    ```solidity
    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.16 <0.9.0;

    contract C {
        uint private data;

        function f(uint a) private pure returns(uint b) { return a + 1; }
        function setData(uint a) public { data = a; }
        function getData() public view returns(uint) { return data; }
        function compute(uint a, uint b) internal pure returns (uint) { return a + b; }
    }

    // This will not compile
    contract D {
        function readData() public {
            C c = new C();
            uint local = c.f(7); // error: member `f` is not visible
            c.setData(3);
            local = c.getData();
            local = c.compute(3, 5); // error: member `compute` is not visible
        }
    }

    contract E is C {
        function g() public {
            C c = new C();
            uint val = compute(3, 5); // access to internal member (from derived to parent contract)
        }
    }
    ```

.. index:: ! getter;function, ! function;getter
.. _getter-functions:

Getter Functions
================

* allows
  * OTHER contracts can read their values
* | `public` state variables,
  * compiler AUTOMATICALLY creates `getter` functions -- for -- them
    ```solidity
    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.16 <0.9.0;

    contract C {
        uint public data = 42;
    }

    contract Caller {
        C c = new C();
        function f() public view returns (uint) {
            return c.data();    // created AUTOMATICALLY -- by the -- compiler
        }
    }
    ```
* 👀external visibility👀
  * if the symbol is accessed
    * internally (== WITHOUT ``this.``) -> evaluates -- to a -- state variable
    * externally (== WITH ``this.``) -> evaluates -- to a -- function
    ```solidity
    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.0 <0.9.0;

    contract C {
        uint public data;
        function x() public returns (uint) {
            data = 3; // internal access
            return this.data(); // external access
        }
    }
    ```

* 👀if you have a ``public`` state variable of array type -> ONLY retrieve 1! array's elements👀
  * Reason: 🧠avoid high gas costs🧠
  * if you want to specify the element -> pass the arguments
    * _Example:_ ``myArray(0)``
  * if you want to return the ENTIRE array -> write a function
    * _Examples:_
      * _Example1:_
          ```solidity
          // SPDX-License-Identifier: GPL-3.0
          pragma solidity >=0.4.16 <0.9.0;

          contract arrayExample {
              // public state variable
              uint[] public myArray;

              // Getter function generated by the compiler
              /*
              function myArray(uint i) public view returns (uint) {
                  return myArray[i];
              }
              */

              // function that returns the ENTIRE array
              function getArray() public view returns (uint[] memory) {
                  return myArray;
              }
          }
          ```
      * _Example2:_
        ```solidity
        // SPDX-License-Identifier: GPL-3.0
        pragma solidity >=0.4.0 <0.9.0;

        contract Complex {
            struct Data {
                uint a;
                bytes3 b;
                mapping(uint => uint) map;
                uint[3] c;
                uint[] d;
                bytes e;
            }
            mapping(uint => mapping(bool => Data[])) public data;
        }
        ```
        * mapping & arrays (EXCEPT to byte arrays) are omitted
          * Reason: 🧠there is NO good way to
            * select individual struct members or
            * provide a key for the mapping🧠
        ```solidity
        function data(uint arg1, bool arg2, uint arg3)
            public
            returns (uint a, bytes3 b, bytes memory e)
        {
            a = data[arg1][arg2][arg3].a;
            b = data[arg1][arg2][arg3].b;
            e = data[arg1][arg2][arg3].e;
        }
        ```
