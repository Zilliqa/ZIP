| ZIP | Title                                      | Status | Type            | Author                         | Created (yyyy-mm-dd) | Updated (yyyy-mm-dd) |
| --- | ------------------------------------------ | ------ | --------------- | ------------------------------ | -------------------- | -------------------- |
| -   | URL Scheme Format for Transaction Requests | -      | Standards Track | Tibi Krisboi <tibi@moonlet.io> | 2019-05-06           | 2019-05-06           |

## Abstract

URLs embedded in QR-codes, hyperlinks in web-pages, buttons in native applications, emails or chat messages provide for robust cross-application signaling between very loosely coupled applications. A standardized URL format for transactions requests allows for instant invocation of the user’s preferred wallet application (even if it is a webapp or native app), with the correct parameterization of the payment transaction only to be confirmed by the (authenticated) user.

## Motivation

The convenience of representing payment requests by standard URLs has been a major factor in the wide adoption of Bitcoin. Bringing a similarly convenient mechanism to Zilliqa would speed up its acceptance as a payment platform among end-users. In particular, URLs embedded in broadcast Intents are the preferred way of launching applications on the Android operating system and work across practically all applications, iOS has a simmilar feature. Desktop web browsers have a standardized way of defining protocol handlers for URLs with specific protocol specifications. Other desktop applications typically launch the web browser upon encountering a URL. Thus, payment request URLs could be delivered through a very broad, ever growing selection of channels.

## Specification

### Syntax

Transaction Requests URLs contain zilliqa in their schema (protocol) part and are constructed as follows:

```
URL              = <schema> :// <to_address> [ "@" <chain_id> ][ "/" <function_name> ] [ "?" <parameters> ]
schema           = "zilliqa"
to_address       = bech32 address (ZIP-1 standard) | ZNS address
chain_id         = NUMBER (1 | 333)
function_name    = STRING
parameters       = <parameter> ("&" <parameter>)*
parameter        = <key>=<value>
key              = "amount" | "gasLimit" | "gasPrice" | "to" | TYPE "-" <parameter_name>
value            = STRING | NUMBER
```

Where:

- `NUMBER` is a valid number. Note that it can be expressed in scientific notation, with a multiplier of a power of 10. The use of this notation is strongly encouraged when expressing monetary value in ZILs or ZRC-2 tokens in atomic units
- `STRING` is a URL-encoded unicode string of arbitrary length, where delimiters and the percentage symbol (%) are mandatorily hex-encoded with a % prefix.
- `TYPE` a standard scilla type

### Semantics

- `to_address` (required), denotes either the beneficiary of native token payment (see below) or the contract address with which the user is asked to interact. ZNS name could also be used. Bech32 addresses always take precedence over ZNS names, e.g. if `to_address` is a valid bech32 zilliqa address then ZNS resolution should be skipped.
- `chain_id` (optional), contains the decimal chain ID, such that transactions on various test- and private networks can be requested. If no chain_id is present, the client’s current network setting remains effective
- `function_name` (optional), if missing the URL is requesting payment in the native token of the blockchain, which is ZIL in our case. The amount is specified in `amount` parameter, in the atomic unit (i.e. QA)
- parameters:
  - `amount` (optional) the indicated amount is only a suggestion which the user is free to change. With no indicated amount, the user should be prompted to enter the amount to be paid. For ZRC-2 token transfers `amount` parameter should be used to indicate an amount to transfer, since ZRC-2 tokens are smart contracts it's possible to indicate the amount in `Uint128-amount` parameter. If `amount` and `Uint128-amount` are both present and have different values, the URL should be considered invalid.
  - `to` - this is just an alias for `ByStr20-to`, for making ZRC-2 transfer request url more readable
  - `gasLimit` and `gasPrice` (optional) are suggested user-editable values for gas limit and gas price, respectively, for the requested transaction

### Examples

- Native token transfer

  - zilliqa://zil102n74869xnvdwq3yh8p0k9jjgtejruft268tg8 - this is just an address share, the user would need to specify an amount and fee options (gasPrice and gasLimit)
  - zilliqa://zil102n74869xnvdwq3yh8p0k9jjgtejruft268tg8?amount=1000000000000 - transfer 1 ZIL to `zil102n74869xnvdwq3yh8p0k9jjgtejruft268tg8`
  - zilliqa://zil102n74869xnvdwq3yh8p0k9jjgtejruft268tg8?amount=1e12 - transfer 1 ZIL to `zil102n74869xnvdwq3yh8p0k9jjgtejruft268tg8`
  - zilliqa://zil102n74869xnvdwq3yh8p0k9jjgtejruft268tg8?amount=1E12 - transfer 1 ZIL to `zil102n74869xnvdwq3yh8p0k9jjgtejruft268tg8`
  - zilliqa://zil102n74869xnvdwq3yh8p0k9jjgtejruft268tg8?amount=1e12&gasPrice=1000000000&gasLimit=1 - transfer 1 ZIL to `zil102n74869xnvdwq3yh8p0k9jjgtejruft268tg8` with suggestion of gasPrice (1000 Li) and gasLimit (1)
  - zilliqa://zil102n74869xnvdwq3yh8p0k9jjgtejruft268tg8@333?amount=1000000000000 - transfer 1 ZIL to `zil102n74869xnvdwq3yh8p0k9jjgtejruft268tg8` on testnet

- ZNS address

  - zilliqa://cool.zil?amount=1000000000000 - transfer 1 ZIL to cool.zil address

- ZRC-2 transfer (request for 1000 atomic units of token with address `zil148fy8yjxn6jf5w36kqc7x73qd3ufuu24a4u8t9` to `zil102n74869xnvdwq3yh8p0k9jjgtejruft268tg8`)

  - zilliqa://zil148fy8yjxn6jf5w36kqc7x73qd3ufuu24a4u8t9/Transfer?Uint128-amount=1000&ByStr20-to=zil102n74869xnvdwq3yh8p0k9jjgtejruft268tg8
  - zilliqa://zil148fy8yjxn6jf5w36kqc7x73qd3ufuu24a4u8t9/Transfer?Uint128-amount=1e3&ByStr20-to=zil102n74869xnvdwq3yh8p0k9jjgtejruft268tg8
  - zilliqa://zil148fy8yjxn6jf5w36kqc7x73qd3ufuu24a4u8t9/Transfer?amount=1e3&ByStr20-to=zil102n74869xnvdwq3yh8p0k9jjgtejruft268tg8
  - zilliqa://zil148fy8yjxn6jf5w36kqc7x73qd3ufuu24a4u8t9/Transfer?amount=1e3&to=zil102n74869xnvdwq3yh8p0k9jjgtejruft268tg8
  - zilliqa://zil148fy8yjxn6jf5w36kqc7x73qd3ufuu24a4u8t9/Transfer?amount=1e12&Uint128-amount=1e3&ByStr20-to=zil102n74869xnvdwq3yh8p0k9jjgtejruft268tg8 - **_INVALID_**

- Smart contract call
  - zilliqa://zil148fy8yjxn6jf5w36kqc7x73qd3ufuu24a4u8t9/DoMagic?String-parameter1=some%20url%20encoded%20text&Uint128-parameter2=123 - call `DoMagic` transition of smart contract with address `zil148fy8yjxn6jf5w36kqc7x73qd3ufuu24a4u8t9` with:
    - `parameter1` of type `String` and value `some url encoded text`
    - `parameter2` of type `Uint128` and value `123`

## Rationale

The proposed format is chosen to resemble bitcoin: URLs as closely as possible, as both users and application programmers are already familiar with that format. Handling different orders of magnitude is delegated to the application, just like in the case of bitcoin:, but lacking access to the block chain, the application can take a hint from the exponent in the URL. Additional parameters may be added, if popular use cases requiring them emerge in practice.

## Backward Compatibility

This ZIP is backward compatible as the required changes are only at the wallet and the SDKs levels. There is no change required at the core protocol layer.

## Implementations

[TODO] - will be done after we agree on the format

## Copyright Waiver

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
