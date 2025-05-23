.. index:: ! functions, ! function;free

.. _functions:

*********
Functions
*********

* Functions
  * places | define them
    * within contracts
    * 💡outside of contracts == "free functions"💡

* "free functions"
  * 👀BUT ALWAYS executed | context of a contract👀
  * allows
    * call OTHER contracts
    * send Ether -- to -- contracts
    * destroy the contract / called them
  * := functions / outside of a contract
  * 👀ALWAYS have implicit ``internal``👀
  * 's code is included | ALL contracts / call them
    * == internal library functions
    ```solidity
    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.1 <0.9.0;

    function sum(uint[] memory arr) pure returns (uint s) {
        for (uint i = 0; i < arr.length; i++)
            s += arr[i];
    }

    contract ArrayExample {
        bool found;
        function f(uint[] memory arr) public {
            // This calls the free function internally.
            // The compiler -- will add -- its code | contract
            uint s = sum(arr);
            require(s >= 10);
            found = true;
        }
    }
    ```
  * vs within contracts
    * ❌free functions do NOT have DIRECT access -- to the --
      * variable ``this``
      * storage variables
      * functions / NOT | their scope ❌

.. _function-parameters-return-variables:

Function Parameters and Return Variables
========================================

* Functions parameters == function's inputs
  * typed parameters

* return variables
  * return an ARBITRARY number of values

Function Parameters == function's inputs
-------------------

* declaration
  * == variables declaration
  * if you are NOT going to use them -> omit them

* _Example:_ contract / accept 1 kind of external (TODO: Where ❓) call & 2 integers
    ```solidity
    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.16 <0.9.0;

    contract Simple {
        uint sum;
        function taker(uint a, uint b) public {
            sum = a + b;
        }
    }
    ```

* uses
  * == ANY OTHER local variable

.. index:: return array, return string, array, string, array of strings, dynamic array, variably sized array, return struct, struct

Return Variables
----------------

* ``returns (variableToReturn1 returnType1,variableToReturn2 returnType2, ...)``
  * `variableToReturni`
    * can be omitted
    * 👀's value
      * valid UNTIL they are (re-)assigned👀
    * ways to specify
      * explicitly
      * implicitly
        ```solidity
        // SPDX-License-Identifier: GPL-3.0
        pragma solidity >=0.4.16 <0.9.0;

        contract Simple {
            function arithmetic(uint a, uint b)
                public
                pure
                returns (uint sum, uint product)
            {
                return (a + b, a * b);          // implicit
            }
        }
        ```

* _Example:_ return the sum & product of 2 integers / passed -- as -- function parameters
    ```solidity
    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.16 <0.9.0;

    contract Simple {
        function arithmetic(uint a, uint b)
            public
            pure
            returns (uint sum, uint product)
        {
            sum = a + b;
            product = a * b;
        }
    }
    ```

* uses
  * == ANY OTHER local variable
  * 👀initialized -- via -- their [default value](../control-structures.md)👀

* |
  * non-internal functions,
    * types / can NOT be returned
      - mappings,
      - internal function types,
      - reference types / location == ``storage``
      - multi-dimensional arrays
        - applies ONLY | :ref:`ABI coder v1 <abi_coder>`
      - structs
        - applies ONLY | :ref:`ABI coder v1 <abi_coder>`
      - ANY mix of these types
  * library functions,
    * ❌NO restriction ❌
      * Reason: 🧠DIFFERENT :ref:`internal ABI <library-selectors>`🧠

.. _multi-return:

Returning Multiple Values
-------------------------

* TODO: When a function has multiple return types, the statement ``return (v0, v1, ..., vn)`` can be used to return multiple values.
The number of components must be the same as the number of return variables
and their types have to match, potentially after an :ref:`implicit conversion <types-conversion-elementary-types>`.

.. _state-mutability:

State Mutability
================

.. index:: ! view function, function;view

.. _view-functions:

View Functions
--------------

* way to declare
  * -- via -- ``view``
  * -- via -- TODO: are there more ways ❓

* == functions /
  * ❌NOT modify the state❌

* if the compiler's EVM target == Byzantium or newer (default) ->
  * | call ``view`` functions
    * | Solidity 0.5.0+=,
      * use opcode ``STATICCALL``
        * == | EVM execution,❌state is NOT modified❌
    * | Solidity 0.5.0-,
      * & you wanted state NOT modified -> use invalid explicit type conversions
  * | library ``view`` functions, use ``DELEGATECALL``
    * Reason: 🧠 there is NO combination of ``DELEGATECALL`` + ``STATICCALL`` 🧠
    * == library ``view`` functions do NOT have run-time checks / prevent state modifications
      * should NOT impact security negatively
        * Reason: 🧠 library code is USUALLY known | compile-time & static checker -- performs -- compile-time checks🧠

* statements / modify the state
  * writing | state variables (storage and transient storage)
  * [Emitting events](events.md)
  * [Creating other contracts](creating-contracts.md)
  * Using ``selfdestruct``
  * Sending Ether -- via -- calls
  * Calling ANY function / NOT marked ``view`` or ``pure``
  * Using low-level calls
  * Using inline assembly / contains certain opcodes

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.5.0 <0.9.0;

contract C {
    function f(uint a, uint b) public view returns (uint) {
        return a * (b + 42) + block.timestamp;
    }
}
```

* ``constant`` | functions
  * | Solidity 0.5.0.-
    * == alias to ``view``
  * | Solidity 0.5.0.+
    * ⚠️ dropped ⚠️

* Getter methods
  * AUTOMATICALLY marked -- by -- ``view``

.. index:: ! pure function, function;pure

.. _pure-functions:

Pure Functions
--------------

* ways to declare
  * -- via -- ``pure``
    * == ❌NOT read from OR NOT modify the state ❌
  * -- via -- TODO: are there more ways ❓

* ALLOWED
  * | compile-time,
    * passing function's inputs & ``msg.data`` -> evaluate a ``pure`` function
      * ⚠️WITHOUT knowing CURRENT blockchain state ⚠️
        * -> read from ``immutable`` variables -- can be a -- NON-pure operation

* if the compiler's EVM target == Byzantium or newer (default) -> use opcode ``STATICCALL``
  * ❌NOT guarantee that the state is NOT read❌
  * guarantee that the state is NOT modified

* statements / read from the state
  * [statements / modify the state](#view-functions)
  * Reading -- from -- state variables (storage and transient storage).
  * Accessing ``address(this).balance`` OR ``<address>.balance``
  * Accessing ``block``, ``tx``, ``msg``'s ANY members
    * EXCEPTION of ``msg.sig`` & ``msg.data``
  * Calling any function / NOT marked ``pure``
  * Using inline assembly / contains certain opcodes

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.5.0 <0.9.0;

contract C {
    function f(uint a, uint b) public pure returns (uint) {
        return a * (b + 42);
    }
}
```


* TODO:
Pure functions are able to use the ``revert()`` and ``require()`` functions to revert
potential state changes when an :ref:`error occurs <assert-and-require>`.

Reverting a state change is not considered a "state modification", as only changes to the
state made previously in code that did not have the ``view`` or ``pure`` restriction
are reverted and that code has the option to catch the ``revert`` and not pass it on.

This behavior is also in line with the ``STATICCALL`` opcode.

.. warning::
  It is not possible to prevent functions from reading the state at the level
  of the EVM, it is only possible to prevent them from writing to the state
  (i.e. only ``view`` can be enforced at the EVM level, ``pure`` can not).

.. note::
  Prior to version 0.5.0, the compiler did not use the ``STATICCALL`` opcode
  for ``pure`` functions.
  This enabled state modifications in ``pure`` functions through the use of
  invalid explicit type conversions.
  By using  ``STATICCALL`` for ``pure`` functions, modifications to the
  state are prevented on the level of the EVM.

.. note::
  Prior to version 0.4.17 the compiler did not enforce that ``pure`` is not reading the state.
  It is a compile-time type check, which can be circumvented by doing invalid explicit conversions
  between contract types, because the compiler can verify that the type of the contract does
  not do state-changing operations, but it cannot check that the contract that will be called
  at runtime is actually of that type.

.. _special-functions:

Special Functions
=================

.. index:: ! receive ether function, function;receive, ! receive

.. _receive-ether-function:

Receive Ether Function -- `receive()` --
----------------------

* contract
  * ⚠️has <=1 ``receive`` function -- declared, via -- ``receive() external payable { ... }`` ⚠️/
    * ❌does NOT contain ``function`` keyword❌
    * can
      * NOT
        * have arguments
        * return anything
      * be virtual
      * override & have modifiers
    * MUST have
      * ``external`` visibility
      * ``payable`` state mutability

* executed |
  * call to the contract / EMPTY calldata,
  * plain Ether transfers
    * _Example:_ -- via -- ``.send()`` or ``.transfer()``
    * if NO such function exists, BUT exists a [payable](#fallback-function) -> fallback function will be called | plain Ether transfer

* recommendations
  * ALWAYS define it

* TODO:
If neither a receive Ether nor a payable fallback function is present, the
    contract cannot receive Ether through a transaction that does not represent a payable function call and throws an
    exception.

In the worst case, the ``receive`` function can only rely on 2300 gas being
available (for example when ``send`` or ``transfer`` is used), leaving little
room to perform other operations except basic logging
The following operations will consume more gas than the 2300 gas stipend:

- Writing to storage
- Creating a contract
- Calling an external function which consumes a large amount of gas
- Sending Ether

.. warning::
    When Ether is sent directly to a contract (without a function call, i.e. sender uses ``send`` or ``transfer``)
    but the receiving contract does not define a receive Ether function or a payable fallback function,
    an exception will be thrown, sending back the Ether (this was different
    before Solidity v0.4.0). If you want your contract to receive Ether,
    you have to implement a receive Ether function (using payable fallback functions for receiving Ether is
    not recommended, since the fallback is invoked and would not fail for interface confusions
    on the part of the sender).


.. warning::
    A contract without a receive Ether function can receive Ether as a
    recipient of a *coinbase transaction* (aka *miner block reward*)
    or as a destination of a ``selfdestruct``.

    A contract cannot react to such Ether transfers and thus also
    cannot reject them. This is a design choice of the EVM and
    Solidity cannot work around it.

    It also means that ``address(this).balance`` can be higher
    than the sum of some manual accounting implemented in a
    contract (i.e. having a counter updated in the receive Ether function).

Below you can see an example of a Sink contract that uses function ``receive``.

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.6.0 <0.9.0;

    // This contract keeps all Ether sent to it with no way
    // to get it back.
    contract Sink {
        event Received(address, uint);
        receive() external payable {
            emit Received(msg.sender, msg.value);
        }
    }

.. index:: ! fallback function, function;fallback

.. _fallback-function:

Fallback Function -- `fallback()` --
-----------------

* contract
  * ⚠️has <=1 ``fallback`` function -- declared, via -- ``fallback () external [payable]`` OR ``fallback (bytes calldata input) external [payable] returns (bytes memory output)`` ⚠️/
    * ❌does NOT contain ``function`` keyword❌
    * MUST have
      * ``external`` visibility
    * can
      * be virtual
      * override & have modifiers
    * ALWAYS receives data
    * `[payable]` -> enable receiving Ether
      * [ONLY AVAILABLE 2300 gas](#receive-ether-function----retrieve---)
        * == ALLOWED operations to execute -- depend on -- AVAILABLE gas

* executed |
  * call to the contract &
    * NO other functions -- match the -- GIVEN function signature, or
    * NO data was supplied & there is NO [`retrieve`](#receive-ether-function----retrieve---)
  * plain Ether transfers
    * _Example:_ -- via -- ``.send()`` or ``.transfer()``
    * == NO such function exists
      * == if NO such function exists, BUT exists a [payable](#fallback-function) -> fallback function will be called | plain Ether transfer

* `fallback (bytes calldata input) external [payable] returns (bytes memory output)`
  * `input`
    * == FULL data / sent -- to the -- contract
      * == `msg.data`
    * if you want to decode it & NO proper functions defined -> use [function selector](../abi-spec.md#function-selector)'s FIRST 4 bytes + `abi.decode`
      `(c, d) = abi.decode(input[4:], (uint256, uint256));`
  * `output`
    * ❌NOT ABI-encoded ❌
      * Reason: 🧠returned WITHOUT modifications (NOT EVEN padding)🧠

```solidity
    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.6.2 <0.9.0;

    contract Test {
        uint x;
        // This function is called for all messages sent to
        // this contract (there is no other function).
        // Sending Ether to this contract will cause an exception,
        // because the fallback function does not have the `payable`
        // modifier.
        fallback() external { x = 1; }
    }

    contract TestPayable {
        uint x;
        uint y;
        // This function is called for all messages sent to
        // this contract, except plain Ether transfers
        // (there is no other function except the receive function).
        // Any call with non-empty calldata to this contract will execute
        // the fallback function (even if Ether is sent along with the call).
        fallback() external payable { x = 1; y = msg.value; }

        // This function is called for plain Ether transfers, i.e.
        // for every call with empty calldata.
        receive() external payable { x = 2; y = msg.value; }
    }

    contract Caller {
        function callTest(Test test) public returns (bool) {
            (bool success,) = address(test).call(abi.encodeWithSignature("nonExistingFunction()"));
            require(success);
            // results in test.x becoming == 1.

            // address(test) will not allow to call ``send`` directly, since ``test`` has no payable
            // fallback function.
            // It has to be converted to the ``address payable`` type to even allow calling ``send`` on it.
            address payable testPayable = payable(address(test));

            // If someone sends Ether to that contract,
            // the transfer will fail, i.e. this returns false here.
            return testPayable.send(2 ether);
        }

        function callTestPayable(TestPayable test) public returns (bool) {
            (bool success,) = address(test).call(abi.encodeWithSignature("nonExistingFunction()"));
            require(success);
            // results in test.x becoming == 1 and test.y becoming 0.
            (success,) = address(test).call{value: 1}(abi.encodeWithSignature("nonExistingFunction()"));
            require(success);
            // results in test.x becoming == 1 and test.y becoming 1.

            // If someone sends Ether to that contract, the receive function in TestPayable will be called.
            // Since that function writes to storage, it takes more gas than is available with a
            // simple ``send`` or ``transfer``. Because of that, we have to use a low-level call.
            (success,) = address(test).call{value: 2 ether}("");
            require(success);
            // results in test.x becoming == 2 and test.y becoming 2 ether.

            return true;
        }
    }
```

.. index:: ! overload

.. _overload-function:

Function Overloading
====================

* overloading
  * := process / functions & inherited 👀can have MULTIPLE functions / SAME name BUT DIFFERENT parameter types👀
    * EXIST |
      * contract's functions
      * external interface's functions

* _Example:_ overloading of contract `A`'s function ``f``
    ```solidity
    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.16 <0.9.0;

    contract A {
        function f(uint value) public pure returns (uint out) {
            out = value;
        }

        function f(uint value, bool really) public pure returns (uint out) {
            if (really)
                out = value;
        }
    }
    ```

* ⚠️if 2 EXTERNAL visible functions != Solidity types BUT == EXTERNAL types -> error ⚠️
  * _Example:_
    ```solidity
    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.16 <0.9.0;

    // This will not compile
    contract A {
        function f(B value) public pure returns (B out) {
            out = value;
        }

        function f(address value) public pure returns (address out) {
            out = value;
        }
    }

    contract B {
    }
    ```

| Overload resolution, how matches Argument
-----------------------------------------

* if there are Overloaded functions -> they are -- , 👀by matching function declarations | current scope vs arguments | function call 👀-- selected
  * if ⚠️ALL⚠️ arguments can be implicitly converted -- to the -- expected types -> functions are selected -- as -- overload candidates
  * ❌if there is NOT EXACTLY 1! candidate -> resolution fails❌
  * ❌FOR overload resolution, return parameters are NOT taken into account ❌
    * _Example:_
      ```solidity
      // SPDX-License-Identifier: GPL-3.0
      pragma solidity >=0.4.16 <0.9.0;

      contract A {
        function f(uint8 val) public pure returns (uint8 out) {
            out = val;
        }

        function f(uint256 val) public pure returns (uint256 out) {
            out = val;
        }
      }
      ```
      * if you calling ``f(50)`` -> type error
        * Reason: 🧠 ``50`` can be -- implicitly converted both to -- ``uint8`` & ``uint256`` types🧠
      * if ``f(256)`` -- would resolve to -- ``f(uint256)``
        * Reason: 🧠 ``256`` can NOT be -- implicitly converted to -- ``uint8``🧠
