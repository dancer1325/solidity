* goal
    * implements the simplest form of a cryptocurrency /
        * ONLY its creator can create NEW coins
        * ANYONE can send coins -- to -- EACH OTHER
            * , WITHOUT registering with a username & pass,
            * -- via an -- Ethereum keypair

# Notes
* ``address``
    * == type /
        * 160-bit value
        * ❌NOT allow arithmetic operations❌
    * allows
        * storing
            * contracts' addresses
            * accounts' keypair' hash

* ``public``
    * AUTOMATICALLY generates a function /
        * 👀enables you to access the CURRENT value of the state variable -- from -- outside of the contract👀
        * if you want to create MANUALLY the function -> function's name (MUST BE) == state variable name
          ```
          function minter() external view returns (address) { return minter; }
          ```
    * ❌otherwise, OTHER contracts can NO access the variable ❌

* ``mapping(address => uint) public balances;``
  * 👀ALSO creates a public state variable👀 / MORE complex datatype
  * == `hash tables <https://en.wikipedia.org/wiki/Hash_table>`_ /
      * VIRTUALLY initialized == | START,
          * EVERY possible key EXISTS
          * values' byte-representation == ALL zeros
      * NOT possible to obtain
          * ALL keys
          * ALL values
  * recommendations
      * keep values | list
      * use MORE suitable data type
  * see [mapping-types](types/mapping-types.md)

* [getter functions](../../contracts/visibility-and-getters.md)
    * created -- by the -- ``public`` keyword
    * uses
        * query 1 account's balance

.. code-block:: solidity

    function balances(address account) external view returns (uint) {
        return balances[account];
    }

.. index:: event

* ``event Sent(address from, address to, uint amount);``
    * uses
        * web applications / listen for these events
    * declares an :ref:`"event" <events>`

* Event
    * 's arguments
        * ``from``,
        * ``to``
        * ``amount``

* _Example:_ listen for event -- via -- `web3.js <https://github.com/web3/web3.js/>`_

.. code-block:: javascript

    Coin.Sent().watch({}, '', function(error, result) {
        if (!error) {
            console.log("Coin transfer: " + result.args.amount +
                " coins were sent from " + result.args.from +
                " to " + result.args.to + ".");
            console.log("Balances now:\n" +
                "Sender: " + Coin.balances.call(result.args.from) +
                "Receiver: " + Coin.balances.call(result.args.to));
        }
    })

.. index:: coin

* TODO:
The :ref:`constructor<constructor>` is a special function that is executed during the creation of the contract and
cannot be called afterwards. In this case, it permanently stores the address of the person creating the
contract. The ``msg`` variable (together with ``tx`` and ``block``) is a
:ref:`special global variable <special-variables-functions>` that
contains properties which allow access to the blockchain. ``msg.sender`` is
always the address where the current (external) function call came from.

The functions that make up the contract, and that users and contracts can call are ``mint`` and ``send``.

The ``mint`` function sends an amount of newly created coins to another address. The :ref:`require
<assert-and-require>` function call defines conditions that reverts all changes if not met. In this
example, ``require(msg.sender == minter);`` ensures that only the creator of the contract can call
``mint``. In general, the creator can mint as many tokens as they like, but at some point, this will
lead to a phenomenon called "overflow". Note that because of the default :ref:`Checked arithmetic
<unchecked>`, the transaction would revert if the expression ``balances[receiver] += amount;``
overflows, i.e., when ``balances[receiver] + amount`` in arbitrary precision arithmetic is larger
than the maximum value of ``uint`` (``2**256 - 1``). This is also true for the statement
``balances[receiver] += amount;`` in the function ``send``.

:ref:`Errors <errors>` allow you to provide more information to the caller about
why a condition or operation failed. Errors are used together with the
:ref:`revert statement <revert-statement>`. The ``revert`` statement unconditionally
aborts and reverts all changes, much like the :ref:`require function <assert-and-require-statements>`.
Both approaches allow you to provide the name of an error and additional data which will be supplied to the caller
(and eventually to the front-end application or block explorer) so that
a failure can more easily be debugged or reacted upon.

The ``send`` function can be used by anyone (who already
has some of these coins) to send coins to anyone else. If the sender does not have
enough coins to send, the ``if`` condition evaluates to true. As a result, the ``revert`` will cause the operation to fail
while providing the sender with error details using the ``InsufficientBalance`` error.

.. note::
    If you use
    this contract to send coins to an address, you will not see anything when you
    look at that address on a blockchain explorer, because the record that you sent
    coins and the changed balances are only stored in the data storage of this
    particular coin contract. By using events, you can create
    a "blockchain explorer" that tracks transactions and balances of your new coin,
    but you have to inspect the coin contract address and not the addresses of the
    coin owners.

# how to run it?
* TODO:
