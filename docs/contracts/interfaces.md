.. index:: ! contract;interface, ! interface contract

.. _interfaces:

**********
Interfaces
**********

* Interfaces
  * vs abstract contracts
    * ❌| functions, NOT implementation❌
  * restrictions
    - can inherit
      - NOT -- from -- other contracts
      - -- from -- other interfaces
        - == NORMAL inheritance rules

    ```solidity
    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.6.2 <0.9.0;

    interface ParentA {
        function test() external returns (uint256);
    }

    interface ParentB {
        function test() external returns (uint256);
    }

    interface SubInterface is ParentA, ParentB {
        // Must redefine test in order to assert that the parent
        // meanings are compatible.
        function test() external override(ParentA, ParentB) returns (uint256);
    }
    ```

    - ALL declared functions MUST be EXTERNAL
      - ALTHOUGH, they are public
    - can NOT declare
      - a constructor
      - state variables
      - modifiers
    - what the Contract ABI can represent
  * `interface interfaceName {}`

    ```solidity
    // SPDX-License-Identifier: GPL-3.0
    pragma solidity >=0.6.2 <0.9.0;

    interface Token {
        enum TokenType { Fungible, NonFungible }
        struct Coin { string obverse; string reverse; }
        function transfer(address recipient, uint amount) external;
    }
    ```
  * if you convert ABI -- & -- interface -> NO information loss

* Contracts
  * can inherit interfaces
    * Reason: 🧠they would inherit other contracts🧠

* functions /
  * declared | interfaces
    * IMPLICITLY, are ``virtual``
  * override interfaces
    * ❌NOT need the ``override`` keyword❌
  * overrides & you want to override again -> mark the function as ``virtual``

* types /
  * defined | interfaces OR other contract-like structures,
    * can be accessed -- from -- other contracts
      ```
      Token.TokenType

      Token.Coin
      ```

* support
  * | [Solidity v0.5.0](../050-breaking-changes.md)
    * ``enum`` types
      * -> specify `pragma solidity >=0.5.0`
