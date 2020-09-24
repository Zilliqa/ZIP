
|  ZIP | Title | Status| Type | Author | Created (yyyy-mm-dd) | Updated (yyyy-mm-dd)
|--|--|--|--| -- | -- | -- |
| 12  | Initiated TX Mempool | Draft | Standards Track  | Alex Cyon <alex.cyon@gmail.com> | 2020-09-24 | -


## Abstract

This ZIP how the UX of making transactions on the Zilliqa ledger can be improved by faster response time back to user about status of a new transaction, by adding a new mempool for initiated transactions.

## Motivation

Zilliqa being a blockchain forces the user to wait for an initiated transaction to be included in the next block, during this time the user has no positive feedback about the transaction. During this limbo the user might be afraid she entered the wrong recipient address and the length of this limbo is pretty long, around 1-2 minutes as it currently stands.

It would be a much improved user experience if she could see the state of this initiated transaction just a second after she made it (after the wallet broadcasts the transaction to the Zilliqa API).

## Specification

The Zilliqa ledger could maintain a new kind of mempool for initiated/pending transactions that the `getTxDetails` call of the Zilliqa API could use as a data source for transactions (by hash id). The user should then be able to view the pending transaction in the explorer just moments after it has been broadcasted.

## Rationale

Crypto is a dangerous landscape where it's easy to make mistakes, some of which might result in permanent loss of funds. Why we strive to make it as intuitive and easy as possible to use wallets and hard as possible to make these mistakes (e.g. validating checksumming addresses, use of BIP39 mnemonics instead of private keys directly etc..). One important aspect of making the user feel safe is **feedback back to the user**, informing her of e.g. result of validation by text dialogs, colors, green checkmarks etc. Possibly the most important flow where the user needs to feel safe is when senindg money, where e.g. inputting the incorrect recipient address might result in loss of funds.

Since the Zilliqa send tokens transaction flow does not allow for any feedback to the user at all within 1-2 minutes, these hundreds of seconds can be quite scary. This can be remedied by supporting initiated/pending status updates of the transaction.

## Implementations

I lack insight in the Zilliqa core to know how it would be implemented, but I think possibly by a separate distributed mempool of pending transactions?

## References
When making a transaction on the Ethereum blockchain, the explorers pick the transaction up instantanly and displays it with status `Pending`, as seen in e.g. [etherscan.io](etherscan.io). I would like to see a similar solution to this.
