* goal
  * Structure of a Contract

* special kinds of contracts
    * [libraries](contracts/libraries.md)
    * [interfaces](contracts/interfaces.md)

State Variables
===============

* == variables /
  * 's values
    * PERMANENTLY stored | contract's storage OR
    * TEMPORALLY stored | transient storage
      * | EACH transaction's end, cleaned

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.4.0 <0.9.0;

contract SimpleStorage {
    uint storedData; // State variable
    uint public data = 42;      // ANOTHER state variable / | declare, intialized
    // ...
}
```

* see
  * [types](types)
  * [visibility & getters](contracts/visibility-and-getters.md)

Functions -- `function` --
=========

* == executable units of code
* uses
  * NORMALLY, | contract

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.7.1 <0.9.0;

contract SimpleAuction {
    function bid() public payable { // Function
        // ...
    }
}

// Helper function defined outside of a contract
function helper(uint x) pure returns (uint) {
    return x * 2;
}
```

* see
  * [functions](contracts/functions.md)
  * [function's modifiers](contracts/function-modifiers.md)
  * [| contract, visibility & getters](contracts/visibility-and-getters.md)
  * [function-calls](control-structures.md#function-calls)

Function Modifiers -- `modifier` --
==================

* allows
  * amend the functions' semantics

* overloading
  * := SAME modifier name / DIFFERENT parameters
  * ❌NOT possible❌

* 👀can be override👀
  * [here](contracts/inheritance.md)

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.4.22 <0.9.0;

contract Purchase {
    address public seller;

    modifier onlySeller() { // Modifier
        require(
            msg.sender == seller,
            "Only seller can call this."
        );
        _;
    }

    function abort() public view onlySeller { // Modifier usage
        // ...
    }
}
```

* see
  * [modifiers](contracts/function-modifiers.md)

Events -- `event` --
======

* := 👀convenience interfaces / have EVM logging facilities 👀

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.22;

event HighestBidIncreased(address bidder, uint amount); // Event

contract SimpleAuction {
    function bid() public payable {
        // ...
        emit HighestBidIncreased(msg.sender, msg.value); // Triggering event
    }
}
```

* see
  * [events](contracts/events.md)

Errors -- `error` --
======

* allow you to
  * | failure situations, define descriptive names & data

* uses
  * [`revert`](control-structures.md#revert)

* vs string descriptions
  * cheaper
  * allow you to, encode ADDITIONAL data

* recommendations
  * | describe the error, use [NatSpec](natspec-format.md)

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity ^0.8.4;

/// Not enough funds for transfer. Requested `requested`,
/// but only `available` available.
error NotEnoughFunds(uint requested, uint available);

contract Token {
    mapping(address => uint) balances;
    function transfer(address to, uint amount) public {
        uint balance = balances[msg.sender];
        if (balance < amount)
            revert NotEnoughFunds(amount, balance);
        balances[msg.sender] -= amount;
        balances[to] += amount;
        // ...
    }
}
```

* see
  * [errors](contracts/errors.md)

Struct Types -- `struct` --
=============

* == CUSTOM defined types /
  * group SEVERAL variables (see
  :ref:`structs` in types section).

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.4.0 <0.9.0;

contract Ballot {
    struct Voter { // Struct
        uint weight;
        bool voted;
        address delegate;
        uint vote;
    }
}
```

* see
  * [structs](types/reference-types.md#structs)

Enum Types -- `enum` --
==========

* allows
  * create CUSTOM types / FINITE set of 'constant values'

```solidity
// SPDX-License-Identifier: GPL-3.0
pragma solidity >=0.4.0 <0.9.0;

contract Purchase {
    enum State { Created, Locked, Inactive } // Enum
}
```

* see
  * [enums](types/value-types.md)
