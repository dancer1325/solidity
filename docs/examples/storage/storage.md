* `// SPDX-License-Identifier: GPL-3.0`
  * Machine-readable license specifiers
      * enable
          * AUTOMATICALLY, check the license
      * make clear code's legal status

* `pragma solidity >=0.4.16 <0.9.0;`
    * specify Solidity compiler version -- to -- run the smart contract
        * Reason: 🧠inestable Solidity versioning `README.md`_ 🧠
    * `pragma`
        * COMMON compilers' instructions
         * `pragma once <https://en.wikipedia.org/wiki/Pragma_once>`_

* `uint storedData;`
    * == 1! slot | database
        * if AFTERWARD, invoke `.set` & change the value ->
            * overwrite the number
            * ORIGINAL value stored | blokchain history
    * if you want to access a contract's member ->
        * ❌NOT use `this.` prefix❌
        * -- via -- its name
            * _Example:_ SimpleStorage.storedData

* ⚠️if you use Unicode text, ALTHOUGH SIMILAR looking characters -> they can be encoded -- as -- DIFFERENT byte array ⚠️
    * Reason: 🧠characters can have -> DIFFERENT code points 🧠
    * -> ALL identifiers (contract names, function names & variable names) are restricted -- to the -- ASCII character set

* UTF-8 data can be encoded data | string variables

# how to run it?
* TODO:
