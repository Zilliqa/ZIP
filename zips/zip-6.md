|  ZIP | Title | Status| Type | Author | Created (yyyy-mm-dd) | Updated (yyyy-mm-dd)
|--|--|--|--| -- | -- | -- |
| 6  | Zilliqa Isolated Server | Implemented | Standards Track  | Kaustubh Shamshery <kaustubh@zilliqa.com>| 2020-05-06 | 2020-06-08

## Abstract

This ZIP details the use of a standalone binary used to mimic the behaviour of a Zilliqa node, its objective being to test transactions and smart contracts. This is hereby referred to as the **Isolated Server**.

## Motivation

Before this utility, developers testing a transaction had to send it to either the developer testnet or mainnet, where they had to wait a non-trivial amount of time to check if the transaction is correct. The issue becomes manifold if a complicated smart contract transaction is
being deployed. For every bug encountered, the developer would have had to wait some time, modify the contract and try to deploy the transaction again. A standalone binary which mimics the behaviour of a Zilliqa node would make this process faster. It would be able to instantaneously tell if the transaction is correct or not.


## Specification

The transaction is sent to the Isolated Server in the same way as it would have been to a Zilliqa network, i.e., through a JSON-RPC API call [CreateTransaction](https://apidocs.zilliqa.com/#createtransaction). The returned message from the JSON-RPC API call is also similar. Interacting with the created transaction can then be done through the transaction-related APIs.

The APIs available in the Isolated Server are:

* `CreateTransaction` 
* `GetSmartContractSubState`
* `GetSmartContractCode`
* `GetMinimumGasPrice`
* `GetBalance`
* `GetSmartContracts` 
* `GetNetworkID` 
* `GetSmartContractInit` 
* `GetTransaction` 

Refer to [API docs](https://apidocs.zilliqa.com) for more information for these APIs.

APIs specific to the Isolated Server:

* `IncreaseBlocknum` : Increase the block number by a given input (For event-triggered system).
* `GetBlocknum` : Get the block number of the blockchain.
* `SetMinimumGasPrice`: Set the minimum gas price of the blockchain.

> For a detailed guide to deploying and using the Isolated Server, see [Isolated Server Instructions](https://github.com/Zilliqa/Zilliqa/blob/master/ISOLATED_SERVER_setup.md).

The Isolated Server has two variants, i.e., `time-triggered` and `event-triggered`.
In a time-triggered system, the transaction block number increases at fixed time intervals. This interval can be user-defined.
In an event-triggered system, the transaction block number is increased by using a JSON-RPC API call `IncreaseBlocknum`. 

The variant can be specified with parameter `-t` when starting the system. See [Isolated Server Boostrap Options](https://github.com/Zilliqa/Zilliqa/blob/master/ISOLATED_SERVER_setup.md#bootstrap-options) for more details.

## Rationale

When the Isolated Server receives a transaction, it applies the same set of checks to the transaction as a Zilliqa node would apply. The transaction is then confirmed or rejected and the corresponding state is saved. This saves on time for the consensus between nodes, which is a major bottleneck in terms of testing.

The server tries to enable all the APIs which are applicable to testing a transaction. However, it does not form an actual blockchain; it just simulates it.

It is a conscious decision here to not enable block related APIs as no blocks are being generated nor being stored, since it is a single node environment. Hence, APIs like `GetTxBlock` or `GetNumTxnsTxEpoch` do not make sense. Since block number is an important parameter in testing, APIs to change and query it have been provided.

To test a transaction, however, two approaches emerged.

* Event Trigger: Say a developer wants to test his/her smart contract at block number `N`. He/she deploys the transaction at block number `M`, and then can use the `IncreaseBlocknum` API to increment the block number by
`N-M` to figure out the transaction's behaviour at block number `N`. This gave rise to the event-triggered mechanism. The user will not have to wait to reach block number `N`.

* Time Trigger: The event-triggered mechanism can fail when the Isolated Server is being used by multiple users; one or more of those users might change the block number, causing confusion for the rest. In this situation, the time-triggered variant can be enabled instead, so that the block number automatically updates for everyone at regular time intervals. When the time-triggered variant is used, the `IncreaseBlocknum` API is disabled.

The Isolated Server is just a layer above the original Zilliqa code. Hence, most of the updates in the Zilliqa code base are automatically applied to the Isolated Server.


## Test Cases

- Run an instance of the Isolated Server. Refer to [Isolated Server Instructions](https://github.com/Zilliqa/Zilliqa/blob/master/ISOLATED_SERVER_setup.md)
. Example bootstrap command:

```bash 
./build/bin/isolatedServer -f isolated-server-accounts.json -t 100
```


- Issue a `CreateTransaction` to the Isolated Server to test the transaction. Example to deploy a smart contract (Fungible Token):

 ```json
curl -d '{
    "id": "1",
    "jsonrpc": "2.0",
    "method": "CreateTransaction",
    "params": [{"version":21823489,"nonce":3980,"toAddr":"0x0000000000000000000000000000000000000000","amount":"0","pubKey":"0246e7178dc8253201101e18fd6f6eb9972451d121fc57aa2a06dd5c111e58dc6a","gasPrice":"1000000000","gasLimit":"10000","code":"scilla_version 0\n\n(* This contract implements a fungible token interface a la ERC20.*)\n\n(***************************************************)\n(*               Associated library                *)\n(***************************************************)\nlibrary FungibleToken\n\nlet min_int =\n  fun (a : Uint128) =\u003e fun (b : Uint128) =\u003e\n  let alt = builtin lt a b in\n  match alt with\n  | True =\u003e\n    a\n  | False =\u003e\n    b\n  end\n\nlet le_int =\n  fun (a : Uint128) =\u003e fun (b : Uint128) =\u003e\n    let x = builtin lt a b in\n    match x with\n    | True =\u003e True\n    | False =\u003e\n      let y = builtin eq a b in\n      match y with\n      | True =\u003e True\n      | False =\u003e False\n      end\n    end\n\n\n(***************************************************)\n(*             The contract definition             *)\n(***************************************************)\n\ncontract FungibleToken\n(owner : ByStr20,\n total_tokens : Uint128,\n decimals : Uint32,\n name : String,\n symbol : String)\n\n(* Initial balance is not stated explicitly: it's initialized when creating the contract. *)\n\nfield balances : Map ByStr20 Uint128 =\n  let m = Emp ByStr20 Uint128 in\n    builtin put m owner total_tokens\nfield allowed : Map ByStr20 (Map ByStr20 Uint128) = Emp ByStr20 (Map ByStr20 Uint128)\n\ntransition BalanceOf (tokenOwner : ByStr20)\n  bal \u003c- balances[tokenOwner];\n  match bal with\n  | Some v =\u003e\n\te = {_eventname : \"BalanceOf\"; address : tokenOwner; balance : v};\n\tevent e\n  | None =\u003e\n\te = {_eventname : \"BalanceOf\"; address : tokenOwner; balance : Uint128 0};\n    event e\n  end\nend\n\ntransition TotalSupply ()\n  e = {_eventname : \"TotalSupply\"; caller : _sender; balance : total_tokens};\n  event e\nend\n\ntransition Transfer (to : ByStr20, tokens : Uint128)\n  bal \u003c- balances[_sender];\n  match bal with\n  | Some b =\u003e\n    can_do = le_int tokens b;\n    match can_do with\n    | True =\u003e\n      (* subtract tokens from _sender and add it to \"to\" *)\n      new_sender_bal = builtin sub b tokens;\n      balances[_sender] := new_sender_bal;\n\n      (* Adds tokens to \"to\" address *)\n      to_bal \u003c- balances[to];\n      new_to_bal = match to_bal with\n      | Some x =\u003e builtin add x tokens\n      | None =\u003e tokens\n      end;\n\n  \t  balances[to] := new_to_bal;\n      e = {_eventname : \"TransferSuccess\"; sender : _sender; recipient : to; amount : tokens};\n      event e\n    | False =\u003e\n      (* balance not sufficient. *)\n      e = {_eventname : \"TransferFailure\"; sender : _sender; recipient : to; amount : Uint128 0};\n      event e\n    end\n  | None =\u003e\n    (* no balance record, can't transfer *)\n  \te = {_eventname : \"TransferFailure\"; sender : _sender; recipient : to; amount : Uint128 0};\n    event e\n  end\nend\n\ntransition TransferFrom (from : ByStr20, to : ByStr20, tokens : Uint128)\n  bal \u003c- balances[from];\n  (* Check if _sender has been authorized by \"from\" *)\n  sender_allowed_from \u003c- allowed[from][_sender];\n  match bal with\n  | Some a =\u003e\n    match sender_allowed_from with\n    | Some b =\u003e\n        (* We can only transfer the minimum of available or authorized tokens *)\n        t = min_int a b;\n        can_do = le_int tokens t;\n        match can_do with\n        | True =\u003e\n            (* tokens is what we should subtract from \"from\" and add to \"to\" *)\n            new_from_bal = builtin sub a tokens;\n            balances[from] := new_from_bal;\n            to_bal \u003c- balances[to];\n            match to_bal with\n            | Some tb =\u003e\n                new_to_bal = builtin add tb tokens;\n                balances[to] := new_to_bal\n            | None =\u003e\n                (* \"to\" has no balance. So just set it to tokens *)\n                balances[to] := tokens\n            end;\n            (* reduce \"allowed\" by \"tokens\" *)\n            new_allowed = builtin sub b tokens;\n            allowed[from][_sender] := new_allowed;\n            e = {_eventname : \"TransferFromSuccess\"; sender : from; recipient : to; amount : tokens};\n            event e\n        | False =\u003e\n            e = {_eventname : \"TransferFromFailure\"; sender : from; recipient : to; amount : Uint128 0};\n            event e\n        end\n    | None =\u003e\n        e = {_eventname : \"TransferFromFailure\"; sender : from; recipient : to; amount : Uint128 0};\n        event e\n    end\n  | None =\u003e\n\te = {_eventname : \"TransferFromFailure\"; sender : from; recipient : to; amount : Uint128 0};\n\tevent e\n  end\nend\n\ntransition Approve (spender : ByStr20, tokens : Uint128)\n  allowed[_sender][spender] := tokens;\n  e = {_eventname : \"ApproveSuccess\"; approver : _sender; spender : spender; amount : tokens};\n  event e\nend\n\ntransition Allowance (tokenOwner : ByStr20, spender : ByStr20)\n  spender_allowance \u003c- allowed[tokenOwner][spender];\n  match spender_allowance with\n  | Some n =\u003e\n      e = {_eventname : \"Allowance\"; owner : tokenOwner; spender : spender; amount : n};\n      event e\n  | None =\u003e\n      e = {_eventname : \"Allowance\"; owner : tokenOwner; spender : spender; amount : Uint128 0};\n      event e\n  end\nend","data":"[{\"vname\":\"_scilla_version\",\"type\":\"Uint32\",\"value\":\"0\"},{\"vname\":\"owner\",\"type\":\"ByStr20\",\"value\":\"0x9bfec715a6bd658fcb62b0f8cc9bfa2ade71434a\"},{\"vname\":\"total_tokens\",\"type\":\"Uint128\",\"value\":\"1000000000\"},{\"vname\":\"decimals\",\"type\":\"Uint32\",\"value\":\"0\"},{\"vname\":\"name\",\"type\":\"String\",\"value\":\"BobCoin\"},{\"vname\":\"symbol\",\"type\":\"String\",\"value\":\"BOB\"}]","signature":"dea51a6af3300ec320a1c8152ddbdf90a71d88b769dafedeb738d5843016c6ce39cb9f10e563e658f520f5747b23d60af6ac15baad7655f7de0441fa338be501","priority":false}]
}' -H "Content-Type: application/json" -X POST "https://localhost:5555"

```

Where `https://localhost:5555` is the location of the Isolated Server.

- Use `GetTransaction` to check the receipt and check if the transaction has succeeded.
  Example of a successful deployment:
```json
  {
  "id": "1",
  "jsonrpc": "2.0",
  "result": {
    "ID": "655107c300e86ee6e819af1cbfce097db1510e8cd971d99f32ce2772dcad42f2",
    "amount": "0",
    "gasLimit": "10000",
    "gasPrice": "1000000000",
    "nonce": "20",
    "receipt": {
      "cumulative_gas": "1662",
      "epoch_num": "134",
      "success": true
    },
    "senderPubKey": "0x03EBCBAEEDD5F98F428BCEBC83E609E5F98136470708A57D61B71BF0B332200EEA",
    "signature": "0x6DEA9FE535AB3557963CA47323B150979CB7C3990515389AF18AFFDD1049ECF3C5AEB5107A64636A946E75219B9482AFE9C7E1D8E5C59D55A1A28A24C0B877B6",
    "toAddr": "0000000000000000000000000000000000000000",
    "version": "21823489"
  }
}

```

- The transaction would be confirmed (or rejected) almost instantaneously.

- The API calls can also be made using [Zilliqa SDKs](https://dev.zilliqa.com/docs/en/api-sdk).


## Implementation

This ZIP is implemented in the following pull requests in the Zilliqa core repository:
- [PR 2065](https://github.com/Zilliqa/Zilliqa/pull/2065)
- [PR 1879](https://github.com/Zilliqa/Zilliqa/pull/1987)

## References

- [Zilliqa API documentation](https://apidocs.zilliqa.com/)
- [Isolated Server Instructions](https://github.com/Zilliqa/Zilliqa/blob/master/ISOLATED_SERVER_setup.md)
- [Zilliqa SDKs](https://dev.zilliqa.com/docs/en/api-sdk)

## Copyright Waiver

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
