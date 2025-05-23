* goal
  * show Solidity's features

* use case
  * electronic voting
    * Problems:
      * how to assign voting rights -- to the -- correct persons
      * how to prevent manipulation
    * partial solution:
      * delegated voting -> vote counting automatic & completely transparent

* TODO: The idea is to create one contract per ballot,
providing a short name for each option.
Then the creator of the contract who serves as
chairperson will give the right to vote to each
address individually.

The persons behind the addresses can then choose
to either vote themselves or to delegate their
vote to a person they trust.

At the end of the voting time, ``winningProposal()``
will return the proposal with the largest number
of votes.


Possible Improvements
=====================

* Currently, many transactions are needed to assign the rights to vote to all participants.
* Moreover, if two or more proposals have the same
number of votes, ``winningProposal()`` is not able
to register a tie. Can you think of a way to fix these issues?
