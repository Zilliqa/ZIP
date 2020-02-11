|  ZIP | Title | Status| Type | Author | Created (yyyy-mm-dd) | Updated (yyyy-mm-dd)
|--|--|--|--| -- | -- | -- |
| 1  | Zilliqa Internal Transactions | Draft | Standards Track  | Haichuan Liu <haichuan@zilliqa.com> <br> Xiaohuo Ren <lulu@zilliqa.com>| 2020-01-15 | 2020-02-05

## Abstract

This ZIP details the standard to expose the internal transactions (commonly referred to as message calls) to external parties such as users, explorers. 

## Motivation

As more and more contracts get deployed on the Zilliqa blockchain, it becomes important to be able to track inter-contract message calls and balance transfers in some cases (e.g., the rewardees and their corresponding reward amounts in the case of a bounty contract). Since a transaction sent to the blockchain only indicates the first contract and the first transition to be invoked, there is no apparent way to tell which other contracts were invoked as a part of this transaction execution and the parameters that were passed along. This causes inconveniences to contract developers and end users.

This ZIP proposes a way for users to track all the relevant "internal" information by recording all internal message calls invoked while processing a transaction in the transaction's receipt.

## Specification

Please refer to [`GetTransaction`](https://apidocs.zilliqa.com/#gettransaction) for more details of the transaction receipt specifications.

The `GetTransaction` API call returns a JSON message that includes a `receipt` field. Under this `receipt`, the internal transaction records are added as individual entries under a new field named `transitions`. Each entry in `transitions` further consists of the following fields:

|      Field    |                          Description                            |
| ------------- | --------------------------------------------------------------- |
|     `addr`    | Address of the contract that emitted this transition            |
|     `depth`   | Depth of current transition if a tree call is invoked           |
|     `msg`     | Message emitted by the Scilla interpreter for this transition   |

The `msg` field is fetched from the `message` field of the Scilla interpreter output, for more detail, please refer to the [Scilla specification](https://scilla.readthedocs.io/en/latest/interface.html#interpreter-output).

## Rationale

There are at least two possible solutions to the problem of tracking internal transactions. The first one is the one put forth in this proposal, i.e., the recording of those transactions in the contract transaction's receipt.

The alternative solution is to require seed nodes to re-execute confirmed transactions in each epoch, and based on the states from the last epoch. The output emitted by the seed node's Scilla interpreter are then aggregated and made accessible through an API call.

The following table summarizes the mechanism and pros and cons of each approach.

|   Solution    | Mechanism                                                   | Pros | Cons |
| ------------- | ----------------------------------------------------------- | ---- | ---- |
| Receipt       | Records all the internal transitions in transaction receipt | - Simple <br> - No extra computation | - Old transactions are not supported |
| Seed node     | Seed node replays all transactions                          | - Consensus-independent, saving bandwidth <br> - Backward compatible | - Difficult to implement <br> - Computationally heavy and may exceed epoch time since it must be performed over all the shards' transactions |

Considering all the aspects above, the receipt approach is chosen for the current implementation.

## Backward Compatibility

Due to transaction receipt hashing, all the receipts that have been generated before this ZIP is implemented are immutable. Thus, only receipts generated after implementation can benefit from this feature.

## Test Cases

This ZIP uses the following curl request as a test and example:
```
curl -d '{
    "id": "1",
    "jsonrpc": "2.0",
    "method": "GetTransaction",
    "params": ["cd7f35b26710e3cf80fcdf2eceb169c8be0d008ad6838a458f83710953bef2bc"]
}' -H "Content-Type: application/json" -X POST "https://api.zilliqa.com/"
```
<b>Expected Response: </b>
<pre><code>
{
   "id":"1",
   "jsonrpc":"2.0",
   "result":{
      "ID":"cd7f35b26710e3cf80fcdf2eceb169c8be0d008ad6838a458f83710953bef2bc",
      "amount":"0",
      "data":"{\"_tag\":\"bestow\",\"params\":[{\"vname\":\"label\",\"value\":\"sindulgents\",\"type\":\"String\"},{\"vname\":\"owner\",\"value\":\"0x4816d2f109d7d8e5857e4bbe52188a2fe9a49383\",\"type\":\"ByStr20\"},{\"vname\":\"resolver\",\"value\":\"0x0000000000000000000000000000000000000000\",\"type\":\"ByStr20\"}]}",
      "gasLimit":"5000",
      "gasPrice":"1000000000",
      "nonce":"2337",
      "receipt":{
         "cumulative_gas":"2671",
         "epoch_num":"424061",
         "event_logs":[
            {
               "_eventname":"Configured",
               "address":"0x9611c53be6d1b32058b2747bdececed7e1216793",
               "params":[
                  {
                     "type":"ByStr32",
                     "value":"0xfbc5638b8e2e12034252da020297058737d3e15f53f165008b844a777b6151c6",
                     "vname":"node"
                  },
                  {
                     "type":"ByStr20",
                     "value":"0x4816d2f109d7d8e5857e4bbe52188a2fe9a49383",
                     "vname":"owner"
                  },
                  {
                     "type":"ByStr20",
                     "value":"0x0000000000000000000000000000000000000000",
                     "vname":"resolver"
                  }
               ]
            },
            {
               "_eventname":"NewDomain",
               "address":"0x9611c53be6d1b32058b2747bdececed7e1216793",
               "params":[
                  {
                     "type":"ByStr32",
                     "value":"0x9915d0456b878862e822e2361da37232f626a2e47505c8795134a95d36138ed3",
                     "vname":"parent"
                  },
                  {
                     "type":"String",
                     "value":"sindulgents",
                     "vname":"label"
                  }
               ]
            }
         ],
         "success":true,
         <b>"transitions":[
            {
               "addr":"0xa11de7664f55f5bdf8544a9ac711691d01378b4c",
               "depth":0,
               "msg":{
                  "_amount":"0",
                  "_recipient":"0x9611c53be6d1b32058b2747bdececed7e1216793",
                  "_tag":"bestow",
                  "params":[
                     {
                        "type":"String",
                        "value":"sindulgents",
                        "vname":"label"
                     },
                     {
                        "type":"ByStr20",
                        "value":"0x4816d2f109d7d8e5857e4bbe52188a2fe9a49383",
                        "vname":"owner"
                     },
                     {
                        "type":"ByStr20",
                        "value":"0x0000000000000000000000000000000000000000",
                        "vname":"resolver"
                     }
                  ]
               }
            }
         ] </b>
      },
      "senderPubKey":"0x036CA3314525C8A796F9224198E67F6AF95EDBBEE1FDA0C370E44C1B1C71E1C99E",
      "signature":"0xFBB1DF2D9E73552576978A2425457ED770C8E98A5BEBC1A03E074440F4578937A0EF976A0E6F2904E7668B4E488C6295585BC0A041CB78897C176D49819A922C",
      "toAddr":"a11de7664f55f5bdf8544a9ac711691d01378b4c",
      "version":"65537"
   }
}
</code></pre>

Notice that a new field `transitions` is created in the `receipt` entry. 

## Implementation

This ZIP is implemented in the following pull requests in the Zilliqa core repository:
- [PR 1982](https://github.com/Zilliqa/Zilliqa/pull/1982)
- [PR 1987](https://github.com/Zilliqa/Zilliqa/pull/1987)


## References
- [Zilliqa API documentation for GetTransaction](https://apidocs.zilliqa.com/#gettransaction)
- [Scilla documentation for interpreter output](https://scilla.readthedocs.io/en/latest/interface.html#interpreter-output)

## Copyright Waiver

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
