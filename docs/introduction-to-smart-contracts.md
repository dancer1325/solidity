# Examples

## Storage Example

* [here](examples/storage)

## Subcurrency Example

* [here](examples/storage)

# Blockchain Basics

* Blockchains
  * concepts
    * mining,
    * [hashing](https://en.wikipedia.org/wiki/Cryptographic_hash_function)
    * [elliptic-curve cryptography](https://en.wikipedia.org/wiki/Elliptic_curve_cryptography)
    * [peer-to-peer networks](https://en.wikipedia.org/wiki/Peer-to-peer)

## Transactions

* blockchain
  * == database /
    * GLOBALLY shared,
      * == ANYONE can read entries
    * transactional

* transaction
  * allows you to
    * | database, change SOMETHING / ⚠️ALL others MUST accept ⚠️
  * ALLOWED values
    * NOT done or
    * COMPLETELY applied
  * 👀ALWAYS cryptographically signed -- by the -- creator👀
  * _Example:_ FROM an account -- transfer amount to -- ANOTHER account
    * if transfer
      * succeeds -> | ORIGIN account, amount subtracted & | TARGET account, added
        * Reason: 🧠transactionality🧠
      * fails -> | ORIGIN account, amount NOT subtracted & | TARGET account, NOT added
        * Reason: 🧠transactionality🧠

## Blocks

* "double-spend attack"
  * == major obstacle -- to -- overcome
  * use case
    * 👀2 transactions EXIST / want to empty SAME account👀
  * Problem
    * 1! transactions can be valid
      * TYPICALLY, the one / accepted FIRST
  * Solution
    * 💡transactions are bundled | "block" 💡
      * == GLOBALLY ACCEPTED order of the transactions / selected for you
      * / | ALL participating nodes,
        * executed
        * distributed
      * 👀if 2 transactions contradict each other -> the second one
        * rejected
        * NOT part of the block 👀

* "blockchain"
  * == blocks | linear sequence in time

* Blocks
  * | "regular" intervals, added | chain
    * "regular" == can change

* [Etherscan](https://etherscan.io/chart/blocktime)
  * allows
    * monitoring the network

* ["attestation"](https://ethereum.org/en/developers/docs/consensus-mechanisms/pos/attestations/)
  * == "order selection mechanism"
  * | "tip" of the chain (== MOST CURRENT block),
    * ⚠️blocks can be reverted ⚠️
      * 👀the more blocks are added | top of a particular block -> the less likely to revert this block 👀

* Transactions
  * ❌NOT guaranteed -- to be -- included | next OR future blocks ❌
    * Reason: 🧠-- depends on the -- miners🧠

* ways to schedule FUTURE calls of your contract
  * use a smart contract automation tool
  * use oracle service

# Ethereum Virtual Machine (EVM)

## Overview

* EVM
    * == 💡runtime environment -- for -- smart contracts | Ethereum 💡 /
        * COMPLETELY isolated
            * == ⚠️code running | EVM has NO access -- to -- network, filesystem or other processes ⚠️
            * ⚠️Smart contracts have LIMITED access -- to -- other smart contracts⚠️

## Accounts

* 👀accounts / share the SAME address space👀
  * are
    * **External accounts**
        * are controlled -- by -- public-private key pairs (== humans)
        * 's address is determined -- from the -- public key while the address of a contract is
    * **contract accounts**
        * are controlled -- by -- code / stored with the account
        * 's address is determined | create contract
            * == creator address + "nonce"
  * EQUALLY treated -- by the -- EVM

* "nonce"
    * := number of transactions / sent -- from -- that address

* **storage**
    * == persistent key-value store mapping 256-bit words -- to -- 256-bit
    * EXIST / EVERY account

* **balance**
    * EXIST / EVERY account
        * SPECIFICALLY, in Wei (1 ether == 10**18 wei)
    * if you want to modify -> send transactions / include Ether

## Transactions

* transaction
  * := message /
    * FROM 1 account -- is sent to -- ANOTHER account
      * 👀ALLOWED target accounts👀
        * SAME as original account
        * empty ==
          * | transaction, NOT set
          * set `null`
    * == binary data + Ether
      * binary data == "payload"
  * 👀if the target account contains code ->
    * transaction's payload -- is used as -- code's input data
    * code is executed👀/
      * 💡execution's output -- stored as -- contract's code 💡
  * if the target account is NOT set -> transaction creates a NEW contract

* contract's address
  * != 0 address
  * == address -- derived from the -- sender & "nonce"
    * "nonce" == 's number of transactions sent

* | create a contract,
  * contract's code STILL empty
  * ⭐️| end upt the initialization, returns the contract's code ⭐️
  * recommendations
    * | constructor, NOT call back TILL constructor has finished executing

### Gas

* := amount /
  * set -- by the -- transaction originator
    * if AFTER execution some gas is left -> refunded -- to the -- transaction originator
  * paid -- by the -- transaction originator
  * charged / EACH transaction
  * ⚠️EXIST MAXIMUM / block⚠️
    * Reason: 🧠 limits the amount of work -- needed to validate a -- block🧠
    * == decrease -- according to -- specific rules
      * if the gas is used up | ANY point & <0 -> trigger an out-of-gas exception ->
        * ends execution
        * reverts ALL modifications
        * ❌gas NOT refunded to transaction originator ❌

* goal
  * incentivizes economical use -- of -- EVM execution time
  * compensates EVM executors (== miners / stakers)

## Storage, Transient Storage, Memory and the Stack

* goal
  * EVM's areas | store data

* storage
  * == 👀EACH account's data area👀 /
    * PERSISTENT BETWEEN
      * function calls
      * transactions
  * == key-value store /
    * 256-bit words -- are mapped to -- 256-bit words
  * | contract,
    * ❌NOT possible to enumerate storage❌
    * if you want | storage
      * to
        * read -> cost gas
        * initialise or modify storage -> cost gas /
          * \>> read's cost
      * ONLY ALLOWED contract's OWN storage
        * == ❌NOT ALLOWED | OTHER contract's OWN storage❌
  * recommendations
    * | contract,
      * minimize use of store
    * | outside of contract,
      * store data
        * _Example:_ calculations, caching, & aggregates

* transient storage
  * == data area
  * vs storage
    * | END of EACH transaction, reset
      * == values UNAVAILABLE -- to -- calls | subsequent transactions
    * 's cost << storage's cost
  * uses
    * nested function calls

* memory
  * == data area /
    * linear
    * reads < 256 bits
    * writes [8 bits, 256 bits]
  * allows
    * obtains a freshly cleared instance / EACH message call
  * uses
    * by contract,
  *
  * Memory is expanded by a word (256-bit), when accessing (either reading or writing) a previously untouched memory word (i.e. any offset
  within a word)
  * At the time of expansion, the cost in gas must be paid. Memory is more
  costly the larger it grows (it scales quadratically).

* stack
  * == data area /
    * ALL computations are performed | it
    * \<= 1024 elements
    * == words of 256 bits
  * ⚠️DIRECTLY, ONLY you can access -- to -- topmost 16 elements⚠️
    * if you want to access OTHER elements -> swap 16 topmost elements -- by -- NEXT below 16
  * if you want deeper access to the stack -> move stack elements -- to -- storage or memory

* EVM
  * == stack machine
    * != register machine

## Calldata, Returndata and Code

There are also other data areas which are not as apparent as those discussed previously.
However, they are routinely used during the execution of smart contract transactions.

The calldata region is the data sent to a transaction as part of a smart contract transaction.
For example, when creating a contract, calldata would be the constructor code of the new contract.
The parameters of external functions are always initially stored in calldata in an ABI-encoded form
and only then decoded into the location specified in their declaration.
If declared as ``memory``, the compiler will eagerly decode them into memory at the beginning of the function,
while marking them as ``calldata`` means that this will be done lazily, only when accessed.
Value types and ``storage`` pointers are decoded directly onto the stack.

The returndata is the way a smart contract can return a value after a call.
In general, external Solidity functions use the ``return`` keyword to ABI-encode values into the returndata area.

The code is the region where the EVM instructions of a smart contract are stored.
Code is the bytes read, interpreted, and executed by the EVM during smart contract execution.
Instruction data stored in the code is persistent as part of a contract account state field.
Immutable and constant variables are stored in the code region.
All references to immutables are replaced with the values assigned to them.
A similar process is performed for constants which have their expressions inlined
in the places where they are referenced in the smart contract code.

.. index:: ! instruction

## Instruction Set

The instruction set of the EVM is kept minimal in order to avoid
incorrect or inconsistent implementations which could cause consensus problems.
All instructions operate on the basic data type, 256-bit words or on slices of memory
(or other byte arrays).
The usual arithmetic, bit, logical and comparison operations are present.
Conditional and unconditional jumps are possible. Furthermore,
contracts can access relevant properties of the current block
like its number and timestamp.

For a complete list, please see the :ref:`list of opcodes <opcodes>` as part of the inline
assembly documentation.

.. index:: ! message call, function;call

## Message Calls

Contracts can call other contracts or send Ether to non-contract
accounts by the means of message calls. Message calls are similar
to transactions, in that they have a source, a target, data payload,
Ether, gas and return data. In fact, every transaction consists of
a top-level message call which in turn can create further message calls.

A contract can decide how much of its remaining **gas** should be sent
with the inner message call and how much it wants to retain.
If an out-of-gas exception happens in the inner call (or any
other exception), this will be signaled by an error value put onto the stack.
In this case, only the gas sent together with the call is used up.
In Solidity, the calling contract causes a manual exception by default in
such situations, so that exceptions "bubble up" the call stack.

As already said, the called contract (which can be the same as the caller)
will receive a freshly cleared instance of memory and has access to the
call payload - which will be provided in a separate area called the **calldata**.
After it has finished execution, it can return data which will be stored at
a location in the caller's memory preallocated by the caller.
All such calls are fully synchronous.

Calls are **limited** to a depth of 1024, which means that for more complex
operations, loops should be preferred over recursive calls. Furthermore,
only 63/64th of the gas can be forwarded in a message call, which causes a
depth limit of a little less than 1000 in practice.

.. _delegatecall:
.. index:: delegatecall, library

## Delegatecall and Libraries

There exists a special variant of a message call, named **delegatecall**
which is identical to a message call apart from the fact that
the code at the target address is executed in the context (i.e. at the address) of the calling
contract and ``msg.sender`` and ``msg.value`` do not change their values.

This means that a contract can dynamically load code from a different
address at runtime. Storage, current address and balance still
refer to the calling contract, only the code is taken from the called address.

This makes it possible to implement the "library" feature in Solidity:
Reusable library code that can be applied to a contract's storage, e.g. in
order to implement a complex data structure.

.. index:: log

## Logs

It is possible to store data in a specially indexed data structure
that maps all the way up to the block level. This feature called **logs**
is used by Solidity in order to implement :ref:`events <events>`.
Contracts cannot access log data after it has been created, but they
can be efficiently accessed from outside the blockchain.
Since some part of the log data is stored in `bloom filters <https://en.wikipedia.org/wiki/Bloom_filter>`_, it is
possible to search for this data in an efficient and cryptographically
secure way, so network peers that do not download the whole blockchain
(so-called "light clients") can still find these logs.

.. index:: contract creation

## Create

Contracts can even create other contracts using a special opcode (i.e.
they do not simply call the zero address as a transaction would). The only difference between
these **create calls** and normal message calls is that the payload data is
executed and the result stored as code and the caller / creator
receives the address of the new contract on the stack.

.. index:: ! selfdestruct, deactivate

## Deactivate and Self-destruct

The only way to remove code from the blockchain is when a contract at that
address performs the ``selfdestruct`` operation. The remaining Ether stored
at that address is sent to a designated target and then the storage and code
is removed from the state. Removing the contract in theory sounds like a good
idea, but it is potentially dangerous, as if someone sends Ether to removed
contracts, the Ether is forever lost.

.. warning::
    From ``EVM >= Cancun`` onwards, ``selfdestruct`` will **only** send all Ether in the account to the given recipient and not destroy the contract.
    However, when ``selfdestruct`` is called in the same transaction that creates the contract calling it,
    the behaviour of ``selfdestruct`` before Cancun hardfork (i.e., ``EVM <= Shanghai``) is preserved and will destroy the current contract,
    deleting any data, including storage keys, code and the account itself.
    See `EIP-6780 <https://eips.ethereum.org/EIPS/eip-6780>`_ for more details.

    The new behaviour is the result of a network-wide change that affects all contracts present on
    the Ethereum mainnet and testnets.
    It is important to note that this change is dependent on the EVM version of the chain on which
    the contract is deployed.
    The ``--evm-version`` setting used when compiling the contract has no bearing on it.

    Also, note that the ``selfdestruct`` opcode has been deprecated in Solidity version 0.8.18,
    as recommended by `EIP-6049 <https://eips.ethereum.org/EIPS/eip-6049>`_.
    The deprecation is still in effect and the compiler will still emit warnings on its use.
    Any use in newly deployed contracts is strongly discouraged even if the new behavior is taken into account.
    Future changes to the EVM might further reduce the functionality of the opcode.

.. warning::
    Even if a contract is removed by ``selfdestruct``, it is still part of the
    history of the blockchain and probably retained by most Ethereum nodes.
    So using ``selfdestruct`` is not the same as deleting data from a hard disk.

.. note::
    Even if a contract's code does not contain a call to ``selfdestruct``,
    it can still perform that operation using ``delegatecall`` or ``callcode``.

If you want to deactivate your contracts, you should instead **disable** them
by changing some internal state which causes all functions to revert. This
makes it impossible to use the contract, as it returns Ether immediately.


.. index:: ! precompiled contracts, ! precompiles, ! contract;precompiled

.. _precompiledContracts:

## Precompiled Contracts

There is a small set of contract addresses that are special:
The address range between ``1`` and (including) ``0x0a`` contains
"precompiled contracts" that can be called as any other contract
but their behavior (and their gas consumption) is not defined
by EVM code stored at that address (they do not contain code)
but instead is implemented in the EVM execution environment itself.

Different EVM-compatible chains might use a different set of
precompiled contracts. It might also be possible that new
precompiled contracts are added to the Ethereum main chain in the future,
but you can reasonably expect them to always be in the range between
``1`` and ``0xffff`` (inclusive).
