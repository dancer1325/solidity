.. index:: ! contract;creation, constructor

******************
Creating Contracts
******************

* ways to create contracts
  * "from outside" -- via -- Ethereum transactions
    * `web3.eth.Contract`
      * [web3.js](https://github.com/web3/web3.js)
  * from | Solidity contracts
    * 💡cyclic creation dependencies are IMPOSSIBLE💡
      * Reason: 🧠creator needs to know the created contract🧠

* [Remix](https://remix.ethereum.org/)
  * enables create contracts -- via -- UI elements

* `constructor`
  * uses
    * | create contract,  1! executed
      * -> 👀ALL contract's final code is stored | blockchain 👀
  * optional
  * ALLOWED
    * ⚠️1!⚠️
      * == overloading is NOT supported
  * 's arguments
    * ⚠️MUST be passed [ABI encoded](../abi-spec.md#argument-encoding)⚠️
      * 👀if you use a library (_Example:_ `web3.js`) -> you do NOT have to care about it👀

* contract's final code
  * ==💡public & external functions + functions / reachable -- , through function calls, from -- there💡
    * == ❌NOT include
      * constructor code
      * internal functions / ONLY called -- from the -- constructor❌

* _Example:_ [here](../examples/creatingcontracts)
