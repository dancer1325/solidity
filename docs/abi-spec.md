* goal
  * Contract ABI Specification

Basic Design
============

* Contract Application Binary Interface (ABI)
  * == 👀| Ethereum ecosystem, standard way to interact -- with -- contracts👀
    * ALLOWED |
      * contract -- & -- outside the blockchain
      * contract-to-contract
    * EXCEPTION
      * contracts / 's interface is
        * dynamic OR
        * known ONLY | run-time
  * uses
    * encoding data -- according to -- data's type /
      * requires a decode schema
  * assumptions
    * contract's interface functions /
      * typed
        * STRONGLY
        * known | compilation time
        * static
    * | compile-time,
      * ALL contracts have the contracts' interface definitions / they call
  * -- for -- libraries
    * [here](contracts/libraries.md)

Function Selector
=================

* == call data's FIRST 4th bytes
  * == signature of the function's Keccak-256 hash's FIRST (from left) 4 bytes
* 👀specify the function to be called 👀

* signature of the function
  * := canonical expression of the basic prototype /
    * == function name + parenthesised list of parameter types /
      * parameter types -- are split by -- 1! `,`
        * ⚠️WITHOUT spaces ⚠️
    * WITHOUT
      * data location
      * return type
        * see [overload function](contracts/functions.md#function-overloading)
        * [ABI Json](#json) takes in account
      * Reason of PREVIOUS: 🧠call resolution context-independent 🧠

Argument Encoding
=================

* encoding -- using -- FROM 5th byte, |
  * return values
  * event arguments
* encoding -- using -- FIRST 4th byte, |
  * functions

Types
=====

* elementary types
  - `uint<M>`
    - unsigned integer type of `M` bits /
      - `0 < M <= 256`
      - `M % 8 == 0`
    - _Example:_ `uint32`, `uint8`, `uint256`
  - `int<M>`
    - 2's complement signed integer type of `M` bits /
      - `0 < M <= 256`
      - `M % 8 == 0`
  - `address`
    - == `uint160` / EXCEPT for
      - assumed interpretation
      - language typing
    - uses
      - compute the function selector
  - `uint`, `int`
    - == ``uint256``, ``int256`` respectively
    - uses
      - compute the function selector
  - `bool`
    - == `uint8` / ALLOWED values [0, 1]
    - uses
      - compute the function selector
  - `fixed<M>x<N>`
    - signed fixed-point decimal number of `M` bits /
      - ``8 <= M <= 256``
      - ``M % 8 == 0``
      - ``0 < N <= 80``
    - ``v`` -- denoted as -- ``v / (10 ** N)``
  - ``ufixed<M>x<N>``
    - unsigned variant of ``fixed<M>x<N>``
  - ``fixed``, ``ufixed``
    - == ``fixed128x18``, ``ufixed128x18`` respectively
    - uses
      - compute the function selector, ``fixed128x18`` and ``ufixed128x18`` have to be used.
  - ``bytes<M>``
    - binary type of ``M`` bytes /
      - ``0 < M <= 32``
  - ``function``
    - == address (20 bytes) + function selector (4 bytes) /
      - BOTH encoded -- to -- ``bytes24``

* (fixed-size) array type
  - ``<type>[M]``
    - fixed-length array of ``M`` elements / ``M >= 0``
      - ❌ALLOWED `M = 0`, BUT NOT supported -- by the -- compiler❌

* non-fixed-size types
  - ``bytes``
    - dynamic sized byte sequence
  - ``string``
    - dynamic sized unicode string / UTF-8 encoded
  - ``<type>[]``
    - variable-length array

* combine types
  - ``(T1,T2,...,Tn)``
    - tuple == types ``T1``, ..., ``Tn``, ``n >= 0``
  - ``((T1,T2,...,Tn),(T1,T2,...,Tn))``
    - tuples of tuples

* library ABIs' types != ABIs' types
  * _Example:_ non-storage structs
  * see [library selectors](contracts/libraries.md)

Mapping Solidity -- to -- ABI types
-----------------------------

| Solidity                 | ABI                                 |
|--------------------------|-------------------------------------|
| address payable          | address                             |
| contract                 | address                             |
| enum                     | uint8 <br/> \|Solidity <0.8.0, int8 |
| user defined value types | its underlying value type           |
| struct                   | tuple                               |

Design Criteria for the Encoding
================================

* 💡encoding's design💡
  * useful
    * if SOME arguments == nested arrays
  1. 👀NUMBER of reads / necessary to access a value < argument array structure's value's depth👀
     * _Example:_ if you want to retrieve ``a_i[k][l][r]`` -> you need 4 reads
     * | PREVIOUS ABI versions,
       * NUMBER of reads == (worst case) `TOTALNumberOfDynamicParameters*`
         * == scaled linearly
  2. variable's data OR array's elements
     * NOT interleaved
     * it is relocatable
       * == ONLY uses relative "addresses"

Formal Specification of the Encoding
====================================

* EXISTING types
  * static
    * encoded in-place
  * dynamic
    * encoded |
      * SEPARATED allocated location
      * AFTER CURRENT block

* types / called
  * "dynamic"
    * ``bytes``
    * ``string``
    * ``T[]`` /
      * ANY ``T``
    * ``T[k]`` /
      * ANY dynamic ``T``
      * ANY ``k >= 0``
    * ``(T1,...,Tk)``
      * if ``Ti`` is dynamic / ``1 <= i <= k``
  * "static"
    * REST of types

* ``len(a)``
  * := number of bytes /
    * ``a`` == binary string
  * 's type == ``uint256``

* ``enc``
  * := actual encoding
  * == map ABI types' values -- to -- binary strings

* ``len(enc(X))``
  * depends on ``X``'s value -- if and only if -- ``X``'s type is dynamic

* | ANY ABI value ``X``,
  * recursively define ``enc(X)`` -- depending on -- ``X``'s type /
    - ``(T1,...,Tk)`` / ``k >= 0`` & ANY ``T1``, ..., ``Tk``
      ```
      enc(X) = head(X(1)) ... head(X(k)) tail(X(1)) ... tail(X(k))
      ```
      - if ``Ti`` is static
        ```
        head(X(i)) = enc(X(i))
        tail(X(i)) = ""
        ```
      - if ``Ti`` is dynamic:
        ```
        head(X(i)) = enc(len( head(X(1)) ... head(X(k)) tail(X(1)) ... tail(X(i-1)) ))
        # well-defined -- Reason:🧠head parts' lengths ONLY -- depend on the -- types (NOT the values)

        tail(X(i)) = enc(X(i))
        ```
      - ``head(X(i))``'s value == beginning of ``tail(X(i))``'s offset / -- relative to the -- start of ``enc(X)``
    - ``T[k]`` / ANY ``T`` & ``k``
      ```
      enc(X) = enc((X[0], ..., X[k-1]))
      ```
      - == encoded -- as a tuple with -- ``k`` elements / SAME type
    - ``T[]`` / ``X`` has ``k`` elements (``k``'s type == ``uint256``)
      ```
      enc(X) = enc(k) enc((X[0], ..., X[k-1]))
      ```
      - encoded -- as a tuple with -- ``k`` elements /
        - SAME type
        - prefixed -- with the -- NUMBER of elements
    - ``bytes`` / length ``k`` ('s type == ``uint256``)
      ```
      enc(X) = enc(k) pad_right(X)
      ```
      - number of bytes -- is encoded as a -- ``uint256`` + actual value of ``X`` (| byte sequence) + minimum number of zero-bytes /
        - ``len(enc(X))`` == 32x (== MULTIPLE of 32)
    - ``string``
      ```
      enc(X) = enc(enc_utf8(X))
      ```
      - ``X`` -- is -- UTF-8 encoded
      - value -- is interpreted as -- ``bytes`` type / encoded FURTHER
        - FURTHER encoding's length == UTF-8 encoded string's NUMBER of bytes
          - != number of characters
    - ``uint<M>``: ``enc(X)``
      - big-endian encoding of ``X``
      - padded | higher-order (left) side /
        - zero-bytes
        - length == 32 bytes
    - ``address``
      - ``uint160`` case
    - ``int<M>``: ``enc(X)``
      - big-endian 2's complement encoding of ``X``
      - padded | higher-order (left) side /
        - ``0xff`` bytes -- for -- negative ``X``
        - zero-bytes -- for -- non-negative ``X``
        - length == 32 bytes
    - ``bool`` : ``uint8``
      - ``1`` == ``true``
      - ``0`` == ``false``
    - ``fixed<M>x<N>``: ``enc(X)``
      - ``enc(X * 10**N)``
        - ``X * 10**N`` == ``int256``
    - ``fixed``
      - == ``fixed128x18`` case
    - ``ufixed<M>x<N>``: ``enc(X)``
      - ``enc(X * 10**N)``
        - ``X * 10**N`` == ``int256``
    - ``ufixed``
      - == ``ufixed128x18`` case
    - ``bytes<M>``: ``enc(X)``
      - == sequence of bytes in ``X`` /
        - padded with trailing zero-bytes / length == 32 bytes
    - ANY ``X`` / ``len(enc(X))`` == 32x (multiple of 32)

Function Selector and Argument Encoding
=======================================

* call to the function ``f`` -- with -- parameters ``a_1, ..., a_n`` / is encoded as
  ```
  function_selector(f) enc((a_1, ..., a_n))
  ```
  * return values ``v_1, ..., v_k`` of ``f`` -- are
    * encoded as -- `enc((v_1, ..., v_k))`
    * combined into -- a tuple

Examples
========

* let's have

    ```solidity
    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.4.16 <0.9.0;

    contract Foo {
        function bar(bytes3[2] memory) public pure {}
        function baz(uint32 x, bool y) public pure returns (bool r) { r = x > 32 || y; }
        function sam(bytes memory, bool, uint[] memory) public pure {}
    }
    ```
  * if we want to call ``bar`` -- with the -- argument ``["abc", "def"]`` -> we would pass 68 bytes (`0xfce353f661626300000000000000000000000000000000000000000000000000000000006465660000000000000000000000000000000000000000000000000000000000`)
    - ``0xfce353f6``:
      - FIRST 4 bytes
      - == Method ID
        - derived -- from the -- signature ``bar(bytes3[2])``
    - ``0x6162630000000000000000000000000000000000000000000000000000000000``
      - FIRST parameter's FIRST part
      - ``bytes3`` value
      - ``"abc"` == 616263 | hexadecimal
        - "a" == "61"
        - "b" == "62"
        - "c" == "63"
        - LEFT-aligned
    - ``0x6465660000000000000000000000000000000000000000000000000000000000``
      - FIRST parameter's SECOND part
      - ``bytes3`` value
      - ``"def"``== 646566 | hexadecimal
        - "d" == "64"
        - "e" == "65"
        - "f" == "66"
        - LEFT-aligned
  * if we want to call ``baz`` -- with the -- parameters ``69`` & ``true`` -> we would pass 68 bytes (`0xcdcd77c000000000000000000000000000000000000000000000000000000000000000450000000000000000000000000000000000000000000000000000000000000001`)
    * ``0xcdcd77c0``
      * == Method ID
      * derived -- as -- steps
        * ``baz(uint32,bool)`` | ASCII form
        * Keccak hash of PREVIOUS output
        * PREVIOUS output's FIRST 4 bytes
    - ``0x0000000000000000000000000000000000000000000000000000000000000045``
      - FIRST parameter
      - uint32 value ``69`` -- padded to -- 32 bytes
    - ``0x0000000000000000000000000000000000000000000000000000000000000001``
      - SECOND parameter
      - ``true`` -- padded to -- 32 bytes
    - return 1! ``bool``
      - if ``false`` -> output = ``0x0000000000000000000000000000000000000000000000000000000000000000``
  - if we want to call ``sam`` -- with the -- arguments ``"dave"``, ``true`` and ``[1,2,3]`` -> we would pass 292 bytes (`0xa5643bf20000000000000000000000000000000000000000000000000000000000000060000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000000a0000000000000000000000000000000000000000000000000000000000000000464617665000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000003000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000000000000000000000000003`)
    - ``0xa5643bf2``
      - == Method ID
      - derived -- from the -- signature ``sam(bytes,bool,uint256[])``
        - 👀``uint`` is replaced -- with its -- canonical representation ``uint256``👀
    - ``0x0000000000000000000000000000000000000000000000000000000000000060``
      - FIRST parameter's location of the data /
        - measured in bytes
        - from the arguments block's start
      - ``0x60``
    - ``0x0000000000000000000000000000000000000000000000000000000000000001``
      - SECOND parameter == boolean
        - true
    - ``0x00000000000000000000000000000000000000000000000000000000000000a0``
      - THIRD parameter's location of the data/
        - measured in bytes
      - ``0xa0``
    - ``0x0000000000000000000000000000000000000000000000000000000000000004``
      - FIRST argument's data part
      - starts -- with -- NUMBER of byte array's elements
    - ``0x6461766500000000000000000000000000000000000000000000000000000000``
      - FIRST argument's contents
      - steps
        - UTF-8 encode ``"dave"``
          - == ASCII-encoded
        - padded | right -- to -- 32 bytes
    - ``0x0000000000000000000000000000000000000000000000000000000000000003``
      - THIRD argument's data part
        - starts -- with -- NUMBER of array's elements
    - ``0x0000000000000000000000000000000000000000000000000000000000000001``
      - THIRD parameter's FIRST entry
    - ``0x0000000000000000000000000000000000000000000000000000000000000002``
      - THIRD parameter's SECOND entry
    - ``0x0000000000000000000000000000000000000000000000000000000000000003``
      - THIRD parameter's THIRD entry

Use of Dynamic Types
====================

* call to a function / signature ``f(uint256,uint32[],bytes10,bytes)`` -- with -- values``(0x123, [0x456, 0x789], "1234567890", "Hello, world!")``
  * is encoded -- via --
    * take the FIRST 4 bytes of ``keccak("f(uint256,uint32[],bytes10,bytes)")``
      * _Example:_ ``0x8be65246``
    * encode ALL 4 arguments' head parts
      * | static types, ``uint256`` & ``bytes10``,
        * encoded values == 4 arguments' head parts
      * | dynamic types, ``uint32[]`` & ``bytes``,
        * encoded values == offset in bytes | start of their data area
          * == NOT count the first 4 bytes

* _Example:_
  - ``0x0000000000000000000000000000000000000000000000000000000000000123``
    - == ``0x123`` padded -- to -- 32 bytes
  - ``0x0000000000000000000000000000000000000000000000000000000000000080``
    - == SECOND parameter's start of data part's offset
    - 4*32 bytes
      - == head part's size
  - ``0x3132333435363738393000000000000000000000000000000000000000000000``
    - == ``"1234567890"`` padded -- to -- 32 bytes | right
  - ``0x00000000000000000000000000000000000000000000000000000000000000e0``
    - == FOURTH parameter's start of data part's offset = FIRST dynamic parameter's start of data part's offset + FIRST dynamic parameter's data part's size = 4\*32 + 3\*32

- FIRST dynamic argument's data part -- ``[0x456, 0x789]`` --
  - ``0x0000000000000000000000000000000000000000000000000000000000000002``
    - NUMBER of array's elements
      - 2
  - ``0x0000000000000000000000000000000000000000000000000000000000000456``
    - FIRST element
  - ``0x0000000000000000000000000000000000000000000000000000000000000789``
    - SECOND element

- SECOND dynamic argument -- ``"Hello, world!"`` --
  - ``0x000000000000000000000000000000000000000000000000000000000000000d``
    - NUMBER of elements | bytes
      -
  - ``0x48656c6c6f2c20776f726c642100000000000000000000000000000000000000``
    - ``"Hello, world!"`` -- padded to -- 32 bytes | right

* ALL together encoding

  ```
  0x8be65246
      0000000000000000000000000000000000000000000000000000000000000123
      0000000000000000000000000000000000000000000000000000000000000080
      3132333435363738393000000000000000000000000000000000000000000000
      00000000000000000000000000000000000000000000000000000000000000e0
      0000000000000000000000000000000000000000000000000000000000000002
      0000000000000000000000000000000000000000000000000000000000000456
      0000000000000000000000000000000000000000000000000000000000000789
      000000000000000000000000000000000000000000000000000000000000000d
      48656c6c6f2c20776f726c642100000000000000000000000000000000000000
  ```

* TODO: Let us apply the same principle to encode the data for a function with a signature ``g(uint256[][],string[])``
with values ``([[1, 2], [3]], ["one", "two", "three"])`` but start from the most atomic parts of the encoding:

First we encode the length and data of the first embedded dynamic array ``[1, 2]`` of the first root array ``[[1, 2], [3]]``:

- ``0x0000000000000000000000000000000000000000000000000000000000000002`` (number of elements in the first array, 2; the elements themselves are ``1`` and ``2``)
- ``0x0000000000000000000000000000000000000000000000000000000000000001`` (first element)
- ``0x0000000000000000000000000000000000000000000000000000000000000002`` (second element)

Then we encode the length and data of the second embedded dynamic array ``[3]`` of the first root array ``[[1, 2], [3]]``:

- ``0x0000000000000000000000000000000000000000000000000000000000000001`` (number of elements in the second array, 1; the element is ``3``)
- ``0x0000000000000000000000000000000000000000000000000000000000000003`` (first element)

Then we need to find the offsets ``a`` and ``b`` for their respective dynamic arrays ``[1, 2]`` and ``[3]``.
To calculate the offsets we can take a look at the encoded data of the first root array ``[[1, 2], [3]]``
enumerating each line in the encoding:

.. code-block:: none

    0 - a                                                                - offset of [1, 2]
    1 - b                                                                - offset of [3]
    2 - 0000000000000000000000000000000000000000000000000000000000000002 - count for [1, 2]
    3 - 0000000000000000000000000000000000000000000000000000000000000001 - encoding of 1
    4 - 0000000000000000000000000000000000000000000000000000000000000002 - encoding of 2
    5 - 0000000000000000000000000000000000000000000000000000000000000001 - count for [3]
    6 - 0000000000000000000000000000000000000000000000000000000000000003 - encoding of 3

Offset ``a`` points to the start of the content of the array ``[1, 2]`` which is line
2 (64 bytes); thus ``a = 0x0000000000000000000000000000000000000000000000000000000000000040``.

Offset ``b`` points to the start of the content of the array ``[3]`` which is line 5 (160 bytes);
thus ``b = 0x00000000000000000000000000000000000000000000000000000000000000a0``.


Then we encode the embedded strings of the second root array:

- ``0x0000000000000000000000000000000000000000000000000000000000000003`` (number of characters in word ``"one"``)
- ``0x6f6e650000000000000000000000000000000000000000000000000000000000`` (utf8 representation of word ``"one"``)
- ``0x0000000000000000000000000000000000000000000000000000000000000003`` (number of characters in word ``"two"``)
- ``0x74776f0000000000000000000000000000000000000000000000000000000000`` (utf8 representation of word ``"two"``)
- ``0x0000000000000000000000000000000000000000000000000000000000000005`` (number of characters in word ``"three"``)
- ``0x7468726565000000000000000000000000000000000000000000000000000000`` (utf8 representation of word ``"three"``)

In parallel to the first root array, since strings are dynamic elements we need to find their offsets ``c``, ``d`` and ``e``:

.. code-block:: none

    0 - c                                                                - offset for "one"
    1 - d                                                                - offset for "two"
    2 - e                                                                - offset for "three"
    3 - 0000000000000000000000000000000000000000000000000000000000000003 - count for "one"
    4 - 6f6e650000000000000000000000000000000000000000000000000000000000 - encoding of "one"
    5 - 0000000000000000000000000000000000000000000000000000000000000003 - count for "two"
    6 - 74776f0000000000000000000000000000000000000000000000000000000000 - encoding of "two"
    7 - 0000000000000000000000000000000000000000000000000000000000000005 - count for "three"
    8 - 7468726565000000000000000000000000000000000000000000000000000000 - encoding of "three"

Offset ``c`` points to the start of the content of the string ``"one"`` which is line 3 (96 bytes);
thus ``c = 0x0000000000000000000000000000000000000000000000000000000000000060``.

Offset ``d`` points to the start of the content of the string ``"two"`` which is line 5 (160 bytes);
thus ``d = 0x00000000000000000000000000000000000000000000000000000000000000a0``.

Offset ``e`` points to the start of the content of the string ``"three"`` which is line 7 (224 bytes);
thus ``e = 0x00000000000000000000000000000000000000000000000000000000000000e0``.


Note that the encodings of the embedded elements of the root arrays are not dependent on each other
and have the same encodings for a function with a signature ``g(string[],uint256[][])``.

Then we encode the length of the first root array:

- ``0x0000000000000000000000000000000000000000000000000000000000000002`` (number of elements in the first root array, 2; the elements themselves are ``[1, 2]``  and ``[3]``)

Then we encode the length of the second root array:

- ``0x0000000000000000000000000000000000000000000000000000000000000003`` (number of strings in the second root array, 3; the strings themselves are ``"one"``, ``"two"`` and ``"three"``)

Finally we find the offsets ``f`` and ``g`` for their respective root dynamic arrays ``[[1, 2], [3]]`` and
``["one", "two", "three"]``, and assemble parts in the correct order:

.. code-block:: none

    0x2289b18c                                                            - function signature
     0 - f                                                                - offset of [[1, 2], [3]]
     1 - g                                                                - offset of ["one", "two", "three"]
     2 - 0000000000000000000000000000000000000000000000000000000000000002 - count for [[1, 2], [3]]
     3 - 0000000000000000000000000000000000000000000000000000000000000040 - offset of [1, 2]
     4 - 00000000000000000000000000000000000000000000000000000000000000a0 - offset of [3]
     5 - 0000000000000000000000000000000000000000000000000000000000000002 - count for [1, 2]
     6 - 0000000000000000000000000000000000000000000000000000000000000001 - encoding of 1
     7 - 0000000000000000000000000000000000000000000000000000000000000002 - encoding of 2
     8 - 0000000000000000000000000000000000000000000000000000000000000001 - count for [3]
     9 - 0000000000000000000000000000000000000000000000000000000000000003 - encoding of 3
    10 - 0000000000000000000000000000000000000000000000000000000000000003 - count for ["one", "two", "three"]
    11 - 0000000000000000000000000000000000000000000000000000000000000060 - offset for "one"
    12 - 00000000000000000000000000000000000000000000000000000000000000a0 - offset for "two"
    13 - 00000000000000000000000000000000000000000000000000000000000000e0 - offset for "three"
    14 - 0000000000000000000000000000000000000000000000000000000000000003 - count for "one"
    15 - 6f6e650000000000000000000000000000000000000000000000000000000000 - encoding of "one"
    16 - 0000000000000000000000000000000000000000000000000000000000000003 - count for "two"
    17 - 74776f0000000000000000000000000000000000000000000000000000000000 - encoding of "two"
    18 - 0000000000000000000000000000000000000000000000000000000000000005 - count for "three"
    19 - 7468726565000000000000000000000000000000000000000000000000000000 - encoding of "three"

Offset ``f`` points to the start of the content of the array ``[[1, 2], [3]]`` which is line 2 (64 bytes);
thus ``f = 0x0000000000000000000000000000000000000000000000000000000000000040``.

Offset ``g`` points to the start of the content of the array ``["one", "two", "three"]`` which is line 10 (320 bytes);
thus ``g = 0x0000000000000000000000000000000000000000000000000000000000000140``.

.. _abi_events:

Events
======

* Events
  * == abstraction -- of -- Ethereum logging/event-watching protocol
  * \+ function ABI
    * events -- are interpreted as -- typed structure

* Log entries
  * -- provide --
    * contract's address
    * \<= 4 topics
    * arbitrary length binary data
  * ==
    - ``address``
      - address of the contract
      - provided -- by -- Ethereum
    - ``topics[0]``
      - == ``keccak(EVENT_NAME+"("+EVENT_ARGS.map(canonical_type_of).join(",")+")")``
        - ``canonical_type_of``
          - function /
            - you pass an argument -> returns the canonical type
              - _Example:_ ``canonical_type_of(uint indexed foo)`` == ``uint256``
      - requirements
        - ❌event NOT declared as ``anonymous``❌
    - ``topics[n]``
      - if event NOT declared as ``anonymous`` -> == ``abi_encode(EVENT_INDEXED_ARGS[n - 1])``
      - if event declared as ``anonymous`` -> == ``abi_encode(EVENT_INDEXED_ARGS[n])``
        - ``EVENT_INDEXED_ARGS`` == serie of ``EVENT_ARGS`` / are indexed
    - ``data``
      - == ``abi_encode(EVENT_NON_INDEXED_ARGS)``
        - ``EVENT_NON_INDEXED_ARGS`` == series of ``EVENT_ARGS`` / NOT indexed

* `abi_encode(function)`
  * return a series of typed values

* event name & series of event parameters -- are split into -- 2 sub-series
  * sub-serie / is indexed
    * may number up -- to --
      * 3 == NON-anonymous events
      * 4 == anonymous ones
  * \+ event signature's Keccak hash -- form -- log entry's topics
  * sub-serie / is NOT indexed
    * -- form -- event's byte array

* `EVENT_INDEXED_ARGS`
  * == array /
    * | types /
      * 👀length < 32 bytes -> ``EVENT_INDEXED_ARGS`` array's value padded or sign-extended (for signed integers) -- to -- 32 bytes👀
        * == regular ABI encoding
      * "complex" OR dynamic length -> ``EVENT_INDEXED_ARGS`` array's value == special encoded value's *Keccak hash*
        * _Example:_ arrays, ``string``, ``bytes`` and structs
        * -> applications
          * able to query -- , via topic == hash of the encoded value, for -- dynamic-length types's values
            * trade-off
              * BETWEEN
                * fast search / predetermined values (if the argument is indexed) &
                * legibility of arbitrary values (requirements: arguments NOT indexed)
              * way to achieve BOTH
                * define events -- with -- 2 arguments / hold SAME value
                  * indexed argument &
                  * NOT-indexed argument
          * unable to decode indexed values / have NOT queried for
        * see [indexed_event_encoding](#encoding-of-indexed-event-parameters)

.. _abi_errors:
.. index:: error, selector; of an error

Errors
======

* context
  * failure | contract

* 👀ways / contract specify a failure👀
  * use special opcode -- to --
    * abort execution
    * revert ALL state changes
  * return , to the caller, descriptive data
    * == encoding of an error + its arguments
    * 👀NO trust on it👀
      * Reason: 🧠
        * error data, by default, bubbles up -- through the -- chain of external calls
          * == error can come -- from -- any of the contracts / DIRECTLY calls
        * contracts can fake -- , via returning data / == error signature, -- any error 🧠

* _Example:_
    ```solidity

        // SPDX-License-Identifier: GPL-3.0
        pragma solidity ^0.8.4;

        contract TestToken {
            error InsufficientBalance(uint256 available, uint256 required);
            function transfer(address /*to*/, uint amount) public pure {
                revert InsufficientBalance(0, amount);      # ALWAYS return a custom error
            }
        }
    ```
  `InsufficientBalance(0, amount)` / `InsufficientBalance(uint256,uint256)` ->
  * ``0xcf479181``
    * == [function selector](#function-selector)
  * ``uint256(0)``,
  * ``uint256(amount)``.

* ``0x00000000`` & ``0xffffffff``
  * error selectors /
    * 👀reserved for future use👀

.. _abi_json:

JSON
====

* contract's interface / JSON format
  * == array of function + event + error descriptions
    * function's fields
      - ``type``
        - ``"function"``
        - ``"constructor"``
        - ``"receive"``
        - ``"fallback"``
      - ``name``
        - name -- of the -- function
        - syntaxes / NEVER have it
          - Constructor,
          - receive,
          - fallback
      - ``inputs``
        - array of objects / EACH contains
          * ``name``
            * name of the parameter
          * ``type``
            * canonical type of the parameter
          * ``components``
            * uses
              * tuple types
        - syntaxes / NEVER have it
          - Constructor,
          - receive,
          - fallback
      - ``outputs``
        - array of objects == ``inputs``
      - ``stateMutability``
        - string
        - ALLOWED values
          - ``pure``
          - ``view``
          - ``nonpayable``
          - ``payable``

* if you send non-zero Ether | non-payable function -> will revert the transaction

* ``nonpayable``
  * == state mutability /
    * reflected -- by -- NOT specifying a state mutability modifier AT ALL

* event description
  * JSON object with FAIRLY SIMILAR fields
    - ``type``
      - ALWAYS ``"event"``
    - ``name``
      - name of the event
    - ``inputs``
      - array of objects / contains
        * ``name``
          * name of the parameter
        * ``type``
          * canonical type of the parameter
        * ``components``
          * used for tuple types
        * ``indexed``
          * if the field is part of the log's topics -> ``true``
          * if it is one of the log's data segments -> ``false``

- ``anonymous``
  - if the event was declared as ``anonymous`` -> ``true``

* Errors look
  - ``type``
    - ALWAYS ``"error"``
  - ``name``
    - name of the error
  - ``inputs``
    - array of objects / contains
      * ``name``
        * name of the parameter.
      * ``type``
        * canonical type of the parameter
      * ``components``
        * used for tuple types

* MULTIPLE errors with the same name and even with identical signature in the JSON array
  * _Example:_
    * if the errors
      * originate -- from -- DIFFERENT files | smart contract
      * are referenced -- from -- ANOTHER smart contract
  * | ABI,
    * ONLY name of the error is relevant

* _Example:_
    ```solidity
    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.4;


    contract Test {
        constructor() { b = hex"12345678901234567890123456789012"; }
        event Event(uint indexed a, bytes32 b);
        event Event2(uint indexed a, bytes32 b);
        error InsufficientBalance(uint256 available, uint256 required);
        function foo(uint a) public { emit Event(a, b); }
        bytes32 b;
    }
    ```
    would result in
    ```json
        [{
        "type":"error",
        "inputs": [{"name":"available","type":"uint256"},{"name":"required","type":"uint256"}],
        "name":"InsufficientBalance"
        }, {
        "type":"event",
        "inputs": [{"name":"a","type":"uint256","indexed":true},{"name":"b","type":"bytes32","indexed":false}],
        "name":"Event"
        }, {
        "type":"event",
        "inputs": [{"name":"a","type":"uint256","indexed":true},{"name":"b","type":"bytes32","indexed":false}],
        "name":"Event2"
        }, {
        "type":"function",
        "inputs": [{"name":"a","type":"uint256"}],
        "name":"foo",
        "outputs": []
        }]
    ```

Handling tuple types
--------------------

Despite the fact that names are intentionally not part of the ABI encoding, they do make a lot of sense to be included
in the JSON to enable displaying it to the end user
The structure is nested in the following way:

An object with members ``name``, ``type`` and potentially ``components`` describes a typed variable.
The canonical type is determined until a tuple type is reached and the string description up
to that point is stored in ``type`` prefix with the word ``tuple``, i.e. it will be ``tuple`` followed by
a sequence of ``[]`` and ``[k]`` with
integers ``k``
The components of the tuple are then stored in the member ``components``,
which is of an array type and has the same structure as the top-level object except that
``indexed`` is not allowed there.

As an example, the code

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.7.5 <0.9.0;
    pragma abicoder v2;

    contract Test {
        struct S { uint a; uint[] b; T[] c; }
        struct T { uint x; uint y; }
        function f(S memory, T memory, uint) public pure {}
        function g() public pure returns (S memory, T memory, uint) {}
    }

would result in the JSON:

.. code-block:: json

    [
      {
        "name": "f",
        "type": "function",
        "inputs": [
          {
            "name": "s",
            "type": "tuple",
            "components": [
              {
                "name": "a",
                "type": "uint256"
              },
              {
                "name": "b",
                "type": "uint256[]"
              },
              {
                "name": "c",
                "type": "tuple[]",
                "components": [
                  {
                    "name": "x",
                    "type": "uint256"
                  },
                  {
                    "name": "y",
                    "type": "uint256"
                  }
                ]
              }
            ]
          },
          {
            "name": "t",
            "type": "tuple",
            "components": [
              {
                "name": "x",
                "type": "uint256"
              },
              {
                "name": "y",
                "type": "uint256"
              }
            ]
          },
          {
            "name": "a",
            "type": "uint256"
          }
        ],
        "outputs": []
      }
    ]

.. _abi_packed_mode:

Strict Encoding Mode
====================

Strict encoding mode is the mode that leads to exactly the same encoding as defined in the formal specification above.
This means that offsets have to be as small as possible while still not creating overlaps in the data areas, and thus no gaps are
allowed.

Usually, ABI decoders are written in a straightforward way by just following offset pointers, but some decoders
might enforce strict mode. The Solidity ABI decoder currently does not enforce strict mode, but the encoder
always creates data in strict mode.

Non-standard Packed Mode
========================

Through ``abi.encodePacked()``, Solidity supports a non-standard packed mode where:

- types shorter than 32 bytes are concatenated directly, without padding or sign extension
- dynamic types are encoded in-place and without the length.
- array elements are padded, but still encoded in-place

Furthermore, structs as well as nested arrays are not supported.

As an example, the encoding of ``int16(-1), bytes1(0x42), uint16(0x03), string("Hello, world!")`` results in:

.. code-block:: none

    0xffff42000348656c6c6f2c20776f726c6421
      ^^^^                                 int16(-1)
          ^^                               bytes1(0x42)
            ^^^^                           uint16(0x03)
                ^^^^^^^^^^^^^^^^^^^^^^^^^^ string("Hello, world!") without a length field

More specifically:

- | encoding,
  - everything is encoded in-place
    - ==
      - ❌NO distinction BETWEEN head -- & -- tail❌
        - ⚠️!= ABI encoding⚠️
      - array's length NOT encoded
- The direct arguments of ``abi.encodePacked`` are encoded without padding,
  as long as they are not arrays (or ``string`` or ``bytes``).
- The encoding of an array is the concatenation of the
  encoding of its elements **with** padding.
- Dynamically-sized types like ``string``, ``bytes`` or ``uint[]`` are encoded
  without their length field.
- The encoding of ``string`` or ``bytes`` does not apply padding at the end,
  unless it is part of an array or struct (then it is padded to a multiple of
  32 bytes).

In general, the encoding is ambiguous as soon as there are two dynamically-sized elements,
because of the missing length field.

If padding is needed, explicit type conversions can be used: ``abi.encodePacked(uint16(0x12)) == hex"0012"``.

Since packed encoding is not used when calling functions, there is no special support
for prepending a function selector. Since the encoding is ambiguous, there is no decoding function.

.. warning::

    If you use ``keccak256(abi.encodePacked(a, b))`` and both ``a`` and ``b`` are dynamic types,
    it is easy to craft collisions in the hash value by moving parts of ``a`` into ``b`` and
    vice-versa. More specifically, ``abi.encodePacked("a", "bc") == abi.encodePacked("ab", "c")``.
    If you use ``abi.encodePacked`` for signatures, authentication or data integrity, make
    sure to always use the same types and check that at most one of them is dynamic.
    Unless there is a compelling reason, ``abi.encode`` should be preferred.


.. _indexed_event_encoding:

Encoding of Indexed Event Parameters
====================================

* indexed event parameters
  * are NOT value types
    * == arrays & structs are stored
      * NOT DIRECTLY
      * 👀by previous Keccak-256 hash encoding👀

* Keccak-256 hash encoding
  * encode ``bytes`` & ``string`` value == string contents /
    * NO padding
    * NO length prefix
  - encode a struct
    - == encode member1 + encode member2 + ... /
      - ALWAYS padded -- to a -- 32x bytes (== MULTIPLE of 32)
  - encode an array (dynamically- & statically-sized)
    - == encode member1 + encode member2 + ... /
      - ALWAYS padded -- to a -- 32x bytes (== MULTIPLE of 32)
      - NO length prefix

* negative number
  * padded
    * -- by -- sign extension
    * -- NOT by -- zero

* ``bytesNN`` types
  * are padded | right
* ``uintNN`` / ``intNN``
  * are padded | left

* ⚠️if a struct has >1 dynamically-sized array -> encoding of the struct is ambiguous ⚠️
  * recommendations
    * ALWAYS re-check the event data
    * NOT rely on the search result / -- based ONLY on -- indexed parameters
