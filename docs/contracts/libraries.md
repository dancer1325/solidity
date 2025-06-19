* Libraries
  * == contracts,
  * 👀's goal👀
      * deploy 1! | specific address
      * 's code is -- , via EVM's feature `DELEGATECALL` -- reused
  * == 👀isolated piece of source code👀
      * -> ONLY can access -- to -- calling contract's state variables
  * 👀if you call a library's function -> their code is executed | calling contract's context 👀
      * == `this` -- points to the -- calling contract
      * -> you can access calling contract's storage
  * if they do NOT modify the state (== `view` OR `pure` functions) -> can ONLY be called DIRECTLY
      * == WITHOUT using `DELEGATECALL`
      * Reason: 🧠libraries are stateless🧠
      * == NOT possible to be destroyed
          * FROM Solidity v0.4.20
          * Reason: 🧠thanks to [call-protection](#call-protection----for----libraries)
  * uses
    * base -- for -- contracts
  * ❌| inheritance hierarchy, NOT visible ❌
    * Reason: 🧠calls to library functions == calls to contract's functions🧠
  * calls to library's internal functions -> follow the internal calling convention
    * ==
      * ALL internal types -- can be -- passed
      * types / stored | memory [data-location](../types/reference-types.md) -- will be passed by -- reference
        * ALL included | compile time
        * use `JUMP` call -- instead of -- `DELEGATECALL`

* TODO:
.. note::
    The inheritance analogy breaks down when it comes to public functions.
    Calling a public library function with ``L.f()`` results in an external call (``DELEGATECALL``
    to be precise).
    In contrast, ``A.f()`` is an internal call when ``A`` is a base contract of the current contract.

.. index:: using for, set

The following example illustrates how to use libraries (but using a manual method,
be sure to check out :ref:`using for <using-for>` for a
more advanced example to implement a set).

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.6.0 <0.9.0;


    // We define a new struct datatype that will be used to
    // hold its data in the calling contract.
    struct Data {
        mapping(uint => bool) flags;
    }

    library Set {
        // Note that the first parameter is of type "storage
        // reference" and thus only its storage address and not
        // its contents is passed as part of the call.  This is a
        // special feature of library functions.  It is idiomatic
        // to call the first parameter `self`, if the function can
        // be seen as a method of that object.
        function insert(Data storage self, uint value)
            public
            returns (bool)
        {
            if (self.flags[value])
                return false; // already there
            self.flags[value] = true;
            return true;
        }

        function remove(Data storage self, uint value)
            public
            returns (bool)
        {
            if (!self.flags[value])
                return false; // not there
            self.flags[value] = false;
            return true;
        }

        function contains(Data storage self, uint value)
            public
            view
            returns (bool)
        {
            return self.flags[value];
        }
    }


    contract C {
        Data knownValues;

        function register(uint value) public {
            // The library functions can be called without a
            // specific instance of the library, since the
            // "instance" will be the current contract.
            require(Set.insert(knownValues, value));
        }
        // In this contract, we can also directly access knownValues.flags, if we want.
    }

Of course, you do not have to follow this way to use
libraries: they can also be used without defining struct
data types. Functions also work without any storage
reference parameters, and they can have multiple storage reference
parameters and in any position.

The calls to ``Set.contains``, ``Set.insert`` and ``Set.remove``
are all compiled as calls (``DELEGATECALL``) to an external
contract/library. If you use libraries, be aware that an
actual external function call is performed.
``msg.sender``, ``msg.value`` and ``this`` will retain their values
in this call, though (prior to Homestead, because of the use of ``CALLCODE``, ``msg.sender`` and
``msg.value`` changed, though).

The following example shows how to use :ref:`types stored in memory <data-location>` and
internal functions in libraries in order to implement
custom types without the overhead of external function calls:

.. code-block:: solidity
    :force:

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity ^0.8.0;

    struct bigint {
        uint[] limbs;
    }

    library BigInt {
        function fromUint(uint x) internal pure returns (bigint memory r) {
            r.limbs = new uint[](1);
            r.limbs[0] = x;
        }

        function add(bigint memory a, bigint memory b) internal pure returns (bigint memory r) {
            r.limbs = new uint[](max(a.limbs.length, b.limbs.length));
            uint carry = 0;
            for (uint i = 0; i < r.limbs.length; ++i) {
                uint limbA = limb(a, i);
                uint limbB = limb(b, i);
                unchecked {
                    r.limbs[i] = limbA + limbB + carry;

                    if (limbA + limbB < limbA || (limbA + limbB == type(uint).max && carry > 0))
                        carry = 1;
                    else
                        carry = 0;
                }
            }
            if (carry > 0) {
                // too bad, we have to add a limb
                uint[] memory newLimbs = new uint[](r.limbs.length + 1);
                uint i;
                for (i = 0; i < r.limbs.length; ++i)
                    newLimbs[i] = r.limbs[i];
                newLimbs[i] = carry;
                r.limbs = newLimbs;
            }
        }

        function limb(bigint memory a, uint index) internal pure returns (uint) {
            return index < a.limbs.length ? a.limbs[index] : 0;
        }

        function max(uint a, uint b) private pure returns (uint) {
            return a > b ? a : b;
        }
    }

    contract C {
        using BigInt for bigint;

        function f() public pure {
            bigint memory x = BigInt.fromUint(7);
            bigint memory y = BigInt.fromUint(type(uint).max);
            bigint memory z = x.add(y);
            assert(z.limb(1) > 0);
        }
    }

It is possible to obtain the address of a library by converting
the library type to the ``address`` type, i.e. using ``address(LibraryName)``.

As the compiler does not know the address where the library will be deployed, the compiled hex code
will contain placeholders of the form ``__$30bbc0abd4d6364515865950d3e0d10953$__`` `(format was different <v0.5.0) <https://docs.soliditylang.org/en/v0.4.26/contracts.html#libraries>`_. The placeholder
is a 34 character prefix of the hex encoding of the keccak256 hash of the fully qualified library
name, which would be for example ``libraries/bigint.sol:BigInt`` if the library was stored in a file
called ``bigint.sol`` in a ``libraries/`` directory. Such bytecode is incomplete and should not be
deployed. Placeholders need to be replaced with actual addresses. You can do that by either passing
them to the compiler when the library is being compiled or by using the linker to update an already
compiled binary. See :ref:`library-linking` for information on how to use the commandline compiler
for linking.

In comparison to contracts, libraries are restricted in the following ways:

- they cannot have state variables
- they cannot inherit nor be inherited
- they cannot receive Ether
- they cannot be destroyed

# | Libraries, function Signatures & Selectors

While external calls to public or external library functions are possible, the calling convention for such calls
is considered to be internal to Solidity and not the same as specified for the regular :ref:`contract ABI<ABI>`.
External library functions support more argument types than external contract functions, for example recursive structs
and storage pointers. For that reason, the function signatures used to compute the 4-byte selector are computed
following an internal naming schema and arguments of types not supported in the contract ABI use an internal encoding.

The following identifiers are used for the types in the signatures:

- Value types, non-storage ``string`` and non-storage ``bytes`` use the same identifiers as in the contract ABI.
- Non-storage array types follow the same convention as in the contract ABI, i.e. ``<type>[]`` for dynamic arrays and
  ``<type>[M]`` for fixed-size arrays of ``M`` elements.
- Non-storage structs are referred to by their fully qualified name, i.e. ``C.S`` for ``contract C { struct S { ... } }``.
- Storage pointer mappings use ``mapping(<keyType> => <valueType>) storage`` where ``<keyType>`` and ``<valueType>`` are
  the identifiers for the key and value types of the mapping, respectively.
- Other storage pointer types use the type identifier of their corresponding non-storage type, but append a single space
  followed by ``storage`` to it.

The argument encoding is the same as for the regular contract ABI, except for storage pointers, which are encoded as a
``uint256`` value referring to the storage slot to which they point.

Similarly to the contract ABI, the selector consists of the first four bytes of the Keccak256-hash of the signature.
Its value can be obtained from Solidity using the ``.selector`` member as follows:

.. code-block:: solidity

    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.5.14 <0.9.0;

    library L {
        function f(uint256) external {}
    }

    contract C {
        function g() public pure returns (bytes4) {
            return L.f.selector;
        }
    }

# Call Protection -- For -- Libraries

* if a library's code is executed via `CALL` -> it will revert
  * EXCEPTION | functions
    * `view` or
    * `pure`

* contract
  * ways to check WHERE the code is running
    * ❌NOT provided DIRECTLY -- by -- EVM❌
      * == (if it was called -- via -- ``CALL``)
    * contract use the ``ADDRESS`` opcode /
      * output is compared -- vs -- address / used | construction time

* library's
  * runtime code
    * ALWAYS starts with a push instruction
    * | compilation time,
      * compiler add a ⚠️constant / 0's of 20 bytes⚠️
  * deploy code runs
    * constant is replaced -- , in memory, by the -- CURRENT address
    * modified code is stored | contract
  * | runtime,
    * deploy time address == first constant / pushed | stack
    * dispatcher code compares the CURRENT address vs this constant / ANY non-view / non-pure function
      * -> actual code / stored on chain for a library != code / reported by the compiler -- as -- ``deployedBytecode``
