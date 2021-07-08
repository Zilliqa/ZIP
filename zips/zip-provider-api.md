| ZIP | Title                                      | Status | Type            | Author                         | Discussions-to | Created (yyyy-mm-dd) | Updated (yyyy-mm-dd) |
| :-: | ------------------------------------------ | :----: | --------------- | ------------------------------ | -------------- | -------------------- | -------------------- |
|  X  | URL Scheme Format for Transaction Requests | Draft  | Standards Track | Tibi Krisboi <tibi@moonlet.io> | -              | 2020-12-11           | 2020-12-11           |

## Summary

A JavaScript Zilliqa Provider API for consistency across clients and applications.

## Abstract

On Zilliqa ecosystem there should be a common convention for key management solutions (wallets) to expose thier API via a JavaScript object. This object is called "the Provider".

This ZIP formalizes an Zilliqa Provider API to promote wallet interoperability. The API is designed to be minimal, event-driven, and agnostic of transport and RPC protocols.

## Definitions

- Provider
  - A JavaScript Object made available to an application, that provides access to Zilliqa by means of a node
- Node
  - An endpoint that receives Remote Procedures Call (RPC) requests from the provider, and return their results.
- Wallet
  - An end-user application that manages private keys, performs signing operations, and acts as a middleware between the Provider and the Node

## Connectivity

The Provider is said to be **connected** when it can service RPC requests to ar least one network.

## API

### Properties

- **_currentNetwork_**: number
  - the chain ID of the current network the wallet is connected to
- **_defaultAccount_**: string
  - the address of the currently selected account in wallet (bech32 format)

### Methods

- **_send(method: string, params: unknown[] | object): Promise<unknown>_**
  - The request method is intended as a transport and protocol-agnostic wrapper function for Remote Procedure Calls (RPCs).
  - This method should return an error if the Provider is not **connected**

### Events

- **_connected_**
  - If the Provider becomes connected, the Provider **MUST** emit the event named connect.
  - The event must be emmited with an object of the following form: {chainId: number, account: string}, specifying the integer ID of the current network and the address in bech32 format of the current account.
- **_currentNetworkChanged_**
  - If the network the Provider is connected to changes, the Provider MUST emit the event named _currentNetworkChanged_ with an object {chainId: number}, specifying the integer ID of the new network.
- **_defaultAccountChanged_**
  - If the account the Provider is connected to changes, the Provider MUST emit the event named _defaultAccountChanged_ with an object {account: string}, specifying the address in bech32 format of the newly selected account.

## Supported RPC Methods

Providers MAY support whatever RPC methods required to fulfill their purpose, standardized or otherwise.

If an RPC method defined in Zilliqa Core or a finalized ZIP is not supported, it SHOULD be rejected with a -32601 error core per the Provider Errors section below, or an appropriate error per the RPC method's specification.

### Provider RPC methods

Since Zilliqa RPC methods are not defined in a ZIP, this ZIP will define a small number of RPC methods in order to formalize the communication between an web application (dApp) and a Wallet using the Provider.

- **GetAccount**
  - Params: [forceConsentWindow]
    - forceConsentWindow: boolean (optional) - if present and true, even if the user already gave access to an account, the consent window/screen will be forced.
  - this method will return the currently selected address in wallet in bech32 format.
  - Errors:
    - code: -1 message: CANCELED_BY_USER: Operation cancelled by user
- **CreateTransaction**
  - This method is already defined in [Zilliqa Core RPC docs](https://dev.zilliqa.com/docs/apis/api-transaction-create-tx)
  - when sending a transaction through the provider the following fields should not be present on TX object:
    - version (this is computed based on the current network, the wallet should do it)
    - nonce (the wallet will add this field as it could have its internal nonce logic)
    - pubKey (this will be filled with current account pub key)
    - signature (the signature will be computed by the wallet and addedd)
  - Errors: (beside the error return by the network)
    - code: -1 message: ACCESS_UNAUTHORIZED: User did not authorize zil11235... address for dapp.com.
    - code: -1 message: CANCELED_BY_USER: Operation cancelled by user
- **SignMessage**
  - Params: [message: string]
    - message: string (required) - this method will be executed on wallet side, it will just singn the message and return an object of this form {signature: string, publicKey: string, message: string}
  - Errors:
    - code: -1 message: CANCELED_BY_USER: Operation cancelled by user

### Errors

Error responses **must** be in this format: {error: {code: number, message: string}}, where _code_ is the code of the error and _message_ a human readable error message.

## Implementations

At the time of writing, the following projects have working implementations:

- [Moonlet Wallet Provider](https://github.com/Moonlet/wallet-providers)

## Copyright

Copyright and related rights waived via CC0.
