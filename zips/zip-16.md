| ZIP | Title | Status| Type | Author | Created (yyyy-mm-dd) | Updated (yyyy-mm-dd)
|--|--|--|--| -- | -- | -- |
| 16  | Merkle Proof | Implemented | Standards Track  | Liu Haichuan <haichuan@zilliqa.com> <br> Ren Lulu <lulu@zilliqa.com> | 2021-01-06 | 2022-01-31

## Abstract

With Merkle Patricia Tree we are allowed to verify all the account state data in Zilliqa. But in order to verify accounts offline, we need a feature to provide proof which is important for the security of Layer 2 services and cross-chain bridges.

This ZIP introduces Merkle Proof which allows us to verify each data entry stored in Merkle Patricia Tree, including account status, contract states and so on. But currently, the standard RPC API does not provide access to these proofs. This ZIP suggests an additional RPC method at the seed node to provide Merkle Proof for account and contract states.

## Motivation

Verification is important for trustless decentralised protocol. Verifying the full state may not always be possible since the storage cost could be huge for some light weight usage such as light client. With Merkle Proof, a user 
can request with a state root (which can be easily fetched from a specific TxBlock header), and an account address, to get the data they are interested with to be verified offline.

## Specification

As part of the Zilliqa protocol module, an additional RPC method called `GetStateProof` should be defined as follows:

### GetStateProof

Returns the account status or contract state values together with the Merkle Proof.

#### Parameters

1. `DATA`, 20 Bytes - address of the account.
2. `ARRAY` - an array of objects which indicates the key of entries to be proofed and included.
3. `QUANTITY|TAG` - Integer block number, or the string `"latest"`.

#### Returns

1. `accountProof`: `ARRAY` - Array of rlp-serialized Merkle Tree nodes, starting with the state root hash as from the TxBlock header, following the path of the address as key.

And the following provided only if address belongs to a contract

2. `stateProof`: `ARRAY` - Array of rlp-serialized Merkle Tree nodes, starting with the state root hash of the account, following the path of all the keys as user requested.

### Other information

Storing proofs of every TxBlock can take up a significant amount of storage space. As such, the node will only store the latest 50 TxBlock proof. This value can be adjusted in future upgrades. 
#### Example

Assuming a contract at block 981000 has following states:
```
{
	"balance": "12000",
	"nonce": "0",
	"codeHash": "b2a31a414d193aaecfa06113341e137e12f111232f3f3b7842ec3c2ec5201200",
	"rootHash": "56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
	"state": {
		"entry_1": "100",
		"entry_2": {
			"entry_A": {
				"entry_A1": "a1",
				"entry_A2": "a2",
				"entry_A3": "a3",
			},
			"entry_B": {
				"entry_B1": "b1",
				"entry_B2": "b2",
				"entry_B3": "b3",
				"entry_B4": "b4"
			}
		},
		"entry_3": "hello",
		"entry_4": "world"
	}
}
```

The `GetProof` request will be like this:

<pre><code><b>
{
  "id": "1",
  "jsonrpc": "2.0",
  "method": "GetProof",
  "params": [
    "986556789012345678901234567890123456abcd",
    [  
      "entry_1", 
      {
        "entry_2":[
          "entry_A",
    	  {
    	    "entry_B":[
    		  "entry_B1",
    		  "entry_B3"
    		]
    	  }
    	]
      },
      "entry_3"
    ],
    "981000"
  ]
}
</b></code></pre>

The result will look like this:
<pre><code><b>
{
  "id": "1",
  "jsonrpc": "2.0",
  "result": {
  	"accountProof": [
  	  "F851808...0808080",
	  "F8D180A...0808080",
	  "F851808...0808080"
  	],
  	"stateProof": [
  	  "F851808...0808080",
	  "F871A08...0808080",
	  "EBA5206...9393939"
  	],
  }
}
</b></code></pre>

From the proof above the user can get the following data by proper deserialization:
```
{
	"balance": "12000",
	"nonce": "0",
  	"codeHash": "b2a31a414d193aaecfa06113341e137e12f111232f3f3b7842ec3c2ec5201200",
  	"rootHash": "56e81f171bcc55a6ff8345e692c0f86e5b48e01b996cadc001622fb5e363b421",
  	"state": {
  		"entry_1":"100",
  		"entry_2":{
  			"entry_A":{
  				"entry_A1":"a1",
  				"entry_A2":"a2",
  				"entry_A3":"a3"
  			},
  			"entry_B":{
  				"entry_B1":"b1",
  				"entry_B3":"b3"
  			}
		},
		"entry_3":"hello"
  	}
  }
}
```

As above, only interested state entries are provided in result after parsing. If the user want to dump the proof for all the state entries of a contract, an empty array should be provided in the request.

## Implementation

This ZIP is implemented and deployed on the Zilliqa mainnet as of [v8.1.0](https://github.com/Zilliqa/Zilliqa/releases/tag/v8.1.0).

## Backward Compatibility

As this introduces a new data structure, contract state migration will be required.

## References

- [GetStateProof API Documentation](https://dev.zilliqa.com/docs/apis/api-contract-get-state-proof)

## Copyright Waiver

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
